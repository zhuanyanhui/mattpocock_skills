## 作用

`implement` 用于构建已经决策好的工作。你只需指向一个[任务](https://www.aihero.dev/ai-coding-dictionary/ticket)、一份[规范](https://www.aihero.dev/ai-coding-dictionary/spec)，或你们刚刚在对话中达成共识的计划，它就会编写代码、在接缝处驱动 [tdd](https://aihero.dev/skills-tdd)、边写边做类型检查、在最后运行 [code-review](https://aihero.dev/skills-code-review)，然后 commit 到当前分支。

它不会重新打开计划。没有访谈环节，没有澄清轮次，也不会提出不同的方案。上游确定的任何内容就是输入，这个 skill 的全部工作就是把输入转化为一个 commit。这正是它与向一个全新的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 输入"构建这个"的区别——后者会欣然地在构建的同时重新设计工作。

## 何时使用

你通过输入 `/implement` 来调用——agent 不会自动触发它。它带有 `disable-model-invocation: true`，所以其他 skill 也无法调用它。无论 [ask-matt](https://aihero.dev/skills-ask-matt) 还是 [to-tickets](https://aihero.dev/skills-to-tickets) 说"然后按任务 `/implement`"，那都是对你的指令，而不是 agent 会自发去做的事情。

当前工作所在的位置决定了这是否是合适的 skill：

| 工作处于… | 使用 |
| --- | --- |
| 任务跟踪器上的一个任务 | `/implement #42`，每个[会话](https://www.aihero.dev/ai-coding-dictionary/session)处理一个任务，任务之间[清理](https://www.aihero.dev/ai-coding-dictionary/clearing)上下文 |
| 一份规范，尚未拆分，且构建跨多个会话 | 先用 [to-tickets](https://aihero.dev/skills-to-tickets)，然后按任务 `/implement` |
| 一份规范，且构建量很小 | 直接针对规范 `/implement` |
| 仅存在于你刚刚的对话中，且规模仍然很小 | 直接在同一个窗口中 `/implement` |
| 还没有写在任何地方 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，如果没有代码库则用 [grill-me](https://aihero.dev/skills-grill-me) |
| 一个你想要测试先行的具体行为，没有规范 | 直接使用 [tdd](https://aihero.dev/skills-tdd) |
| 已经构建完成，你想要检查 | 直接使用 [code-review](https://aihero.dev/skills-code-review) |

同会话的情况值得单独说明，因为 skill 自身的第一行描述并没有覆盖它。`SKILL.md` 说的是"规范或任务"，这会引导 [model](https://www.aihero.dev/ai-coding-dictionary/model) 去寻找一个不存在的文件。如果计划只存在于对话线程中，在调用时要说明这一点。

## 前置条件

`implement` 会 commit 到你当前所在的分支。它不会创建分支，也不会询问。在开始之前确认你处于想要承载工作的分支上。

如果任务来自 [to-tickets](https://aihero.dev/skills-to-tickets)，那么它们所在的任务跟踪器是由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 配置的。`code-review` 读取同样的配置来在收尾时找到原始规范。

## 一次运行做什么

一次运行包含五个节拍，按顺序执行：

1. 读取任务或规范，找出接缝。
2. 在预先商定好的接缝处驱动 [tdd](https://aihero.dev/skills-tdd)，一次一个红-绿切片。
3. 频繁进行类型检查，过程中运行单个测试文件。
4. 最后运行一次完整的测试套件。
5. 运行 [code-review](https://aihero.dev/skills-code-review)，然后 commit 到当前分支。

一次运行涵盖一个任务。[to-tickets](https://aihero.dev/skills-to-tickets) 生成的任务是示踪弹式的垂直切片，大小刚好适配一个全新的[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)，所以预期的节奏是：清理上下文，实现一个任务，commit，再次清理。每个任务都是自包含的，这正是上一个任务的上下文可以被丢弃的原因。

## 预先商定的接缝

这个 skill 运行的核心概念是**接缝**：你观察到行为的公共边界，无需深入内部。测试位于接缝处。在任何代码编写之前就商定好的接缝处工作，正是保持测试持久性的关键，因为底层实现可以被重写而测试不需要改动。

"预先商定"这个词在做实实在在的工作，它同时也是这个 skill 最薄弱的环节。`implement` 内部没有任何东西负责商定接缝。`tdd` 才是负责询问的 skill，并且它拒绝在未确认的接缝处编写测试。所以实际上，商定要么在上游的规范中完成，要么在运行的第一次交互中完成。如果任何地方都没有商定，前置条件就不会触发，运行就静默变成了"只管写代码"。在规范中命名接缝正是防止这种情况的手段。

## 常见问题

**它完成了，但我的任务仍然处于打开状态，验收标准也仍未勾选。**

正确，而且是预期行为。`implement` 没有完成步骤。它在 commit 处结束，从不动用工单，这一点在 GitHub Issues 和本地 markdown 跟踪器上都已确认，所以这不是跟踪器集成的问题。它也不会对 `code-review` 产生的发现采取行动，不会勾选原始任务上的 `- [ ]` 框。自己关闭任务并调和验收标准。这在依赖链上影响最大，因为 `to-tickets` 将前沿定义为所有阻塞项都已关闭的任务。如果没有任务被关闭，就没有任务会变得 visibly unblocked。

**我可以一次性指向所有任务，或者并行运行多个吗？**

不行。一次调用，一个任务。批量分派整个任务队列和 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) 扇出都被反复请求过，但都不存在。在一个 checkout 中并排运行多个 `/implement` 会话比"不支持"更糟糕：一份现场报告描述了一个会话中的 `git commit --amend` 落到了另一个会话的 commit 上，一个 stash 从 `refs/stash` 中消失，以及 commit 落到了错误的分支上，这一切都发生在一个下午、横跨三个任务。这些会话共享同一个工作目录、同一个 index 和同一个 HEAD。Git worktree 是社区推荐的绕过方案，注意 `refs/stash` 在 worktree 之间也是共享的，所以仅用 worktree 并不能解决 stash 的问题。如果你今天想要并行，你得自己组装。

**它能打开 pull request 而不是直接 commit 吗？**

没有内置功能。它直接 commit 到当前分支，有些人觉得这太急躁了：代码在他们有机会验证能正常工作之前就落地了。没有配置开关，也没有 PR 模式。人们通过在调用时覆盖（"commit 到一个分支并打开 PR"）或编辑他们本地的 skill 副本来实现。

**`code-review` 说它看不到我的变更。**

`code-review` 审查的是 `git diff <fixed-point>...HEAD`，这排除了 staged 和工作树的变更。`implement` 在 commit 之前运行它，所以除非已经存在一个中间的 commit，否则那个 diff 中没有任何可审查的内容。很多人报告过这个问题，两侧都尚未修复。先 commit，然后针对你分支出去的[固定点](https://www.aihero.dev/ai-coding-dictionary/fixed-point)进行审查。

另外，有些人故意不希望在运行内部进行审查，因为一个 agent 审查自己刚刚写的代码会偏向自己的解决方案。在一个全新的会话中针对一个固定点运行 [code-review](https://aihero.dev/skills-code-review) 是一个合法的替代方案，这也是那个 skill 在两个独立的子 agent 中运行它的双轴审查的原因。

**一个任务烧掉了 150k token。我用错了吗？**

很可能是任务太大了，而不是 skill 被误用了。一次运行包含代码库探索、每个接缝的红-绿循环、完整的测试套件和一次审查，所以一个非平凡的任务超过 100k [token](https://www.aihero.dev/ai-coding-dictionary/token) 是正常的，而不是出了问题的标志。杠杆在上游：在 [to-tickets](https://aihero.dev/skills-to-tickets) 中调整任务大小，使每个任务适配一个全新的窗口。如果单个任务反复膨胀，拆分它而不是提高 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 级别。

**在新会话中 `/implement #2` 处理了完全不相关的内容。**

`#2` 会被解析为 agent 能看到的任何编号列表，在新会话中这可能是一个 todo 文件、一个检查清单或其他工作列表，而不是配置好的跟踪器。这种解析是自信的而不是失败封闭的，所以错误在它开始之前并不明显。传递完整的引用、任务 URL 或 `owner/repo#2`，并在开始之前让它回确认标题。

## 正常工作的标志

- 会话通过读取任务或规范并重述它将构建什么来开始，而不是询问你要构建什么。
- 你可以在追踪中看到实际的 `/tdd` 调用，而不仅仅是 diff 中出现了测试。
- 类型检查和单个测试文件在运行过程中反复执行，完整的测试套件在接近尾声时运行一次。
- 运行在你无需提示继续的情况下在你的当前分支上达到了一个 commit。
- diff 是一个任务量的变更：一个贯穿每一层的垂直切片，而不是多个任务扫在一起。

## 它的位置

`implement` 是主链中的构建步骤，倒数第二：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它的邻居是 [to-tickets](https://aihero.dev/skills-to-tickets)，它生成消费的任务并声明决定顺序的阻塞边；[tdd](https://aihero.dev/skills-tdd)，它在每个接缝处内部驱动；以及 [code-review](https://aihero.dev/skills-code-review)，它在 commit 之前运行。它位于规划 skill 的下游并信任它们。它不会重新验证所接收内容的形状，所以一个结构不良的地图或一个水平分层的任务会按原样被构建。

这种信任就是为什么 [wayfinder](https://aihero.dev/skills-wayfinder) 在 [to-spec](https://aihero.dev/skills-to-spec) 处合并到链上，而不是将它的地图直接循环到 `implement` 中。只有当工作量确实很小时，才从地图直接转到 `implement`。

[ask-matt](https://aihero.dev/skills-ask-matt) 是整个集合的路由器，当你不确定自己处于哪个流程中时使用。
