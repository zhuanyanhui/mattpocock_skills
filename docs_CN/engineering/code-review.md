## 功能说明

`code-review` 会审查 `HEAD` 与你指定的某个固定点——一个 commit、一个 branch、一个 tag、`main`、`HEAD~5`——之间的 diff，并沿两个轴进行审查。**Standards** 轴询问代码是否遵循了该 repo 的编码方式。**Spec** 轴询问代码是否完成了原始 issue 或[规范](https://www.aihero.dev/ai-coding-dictionary/spec)所要求的内容。每个轴在各自的[sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent)中运行，因此彼此不会看到对方的推理过程。

两个轴的结果永远不会合并，也永远不会重新排名。报告最后会给出*每个轴*的最严重问题，并拒绝命名一个跨轴的总体胜者，因为一个变更可以通过一个轴而失败于另一个轴：遵循所有约定但实现了错误东西的代码会通过 Standards 而失败于 Spec；精确完成了[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)要求但破坏了 repo 约定的代码则相反。混合的裁决结果会让通过的轴掩盖失败的轴。

## 何时使用

输入 `/code-review`，或者当你要求审查一个 branch、一个 PR、进行中的工作，或任何"自 X 以来"的内容时，agent 会自动使用它。

| 你的情况 | 使用 |
| --- | --- |
| 存在一个 diff，你想知道它是否构建正确*且*是否做了正确的事 | `code-review` |
| 你想在 diff 中猎杀 bug——空路径、竞态条件、差一错误 | Claude Code 自带的内置审查，而非这个（见下方的名称冲突） |
| 还没有任何代码，你想以测试优先的方式编写 | [tdd](https://aihero.dev/skills-tdd) |
| 需要构建整个规范，包含审查环节 | [implement](https://aihero.dev/skills-implement)，它会调用这个 skill 本身 |
| 整个代码库已经偏离轨道，不是单个 diff | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |
| 某些东西坏了而你不知道原因 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

你必须提供固定点。如果没有提供，skill 会向你询问而不是猜测；然后在生成任何 sub-agent 之前，它会检查 ref 是否能解析以及 diff 是否非空，因此拼写错误的 branch 名称会在你面前就失败，而不是在两个 sub-agent 内部才失败。

## 前置条件

Standards 轴不需要任何东西。它会读取 repo 文档化的内容（`CODING_STANDARDS.md`、`CONTRIBUTING.md` 等），如果 repo 没有文档化任何内容，则回退到内置基线。

Spec 轴需要规范存在且可找到。它按以下顺序查找：

1. commit 消息中的 issue 引用（`#123`、`Closes #45`、GitLab 的 `!67`），通过 `docs/agents/issue-tracker.md` 获取。
2. 你作为参数传入的路径。
3. `docs/`、`specs/` 或 `.scratch/` 下匹配 branch 或功能名称的 spec 文件。
4. 向你询问。

步骤 1 依赖于 `docs/agents/issue-tracker.md`，该文件由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 写入。没有它的话，只要你给它一个路径，该轴仍然可以工作。如果完全没有规范，Spec sub-agent 会被跳过，报告会显示"no spec available"而不是凭空编造需求。

## 两个轴

| | Standards | Spec |
| --- | --- | --- |
| 问题 | 它的构建方式正确吗？ | 它是正确的事情吗？ |
| 读取内容 | repo 已文档化的标准，加上气味基线 | 原始 issue 或规范 |
| 报告内容 | 已文档化的违规（可能是硬性违规），以及气味（永远是判断性意见） | 缺失或部分的需求、范围蔓延、需求被错误实现 |
| 每个发现的引用 | 标准文件及规则，或已命名的气味加上 hunk | 规范中的对应行 |

一个不了解你标准的通用审查 skill 正是这个设计试图避免的东西——它会标记你代码库中刻意为之的内容，却遗漏代码库实际依赖的不变量。因此 repo 自己的文档是 Standards 轴上的[主要来源](https://www.aihero.dev/ai-coding-dictionary/primary-source)，并且**repo 始终优先覆盖**。

**气味基线**是它之下的底线：来自《重构》第 3 章的十二个 Fowler 代码坏味——Mysterious Name、Duplicated Code（重复代码）、Feature Envy（依恋情结）、Data Clumps（数据泥团）、Primitive Obsession（基本类型偏执）、Repeated Switches（重复的 switch）、Shotgun Surgery（霰弹式修改）、Divergent Change（发散式变更）、Speculative Generality（夸夸其谈的未来性）、Message Chains（消息链）、Middle Man（中间人）、Refused Bequest（被拒绝的遗赠）。每个都是一个带标签的启发式信号（"可能是 Feature Envy"），而不是硬性违规，并且每个都以*是什么* → *如何修复*的形式陈述，因此一个发现附带了移动方案而不只是抱怨。你的 linter 已经强制执行的内容会被两个轴都跳过。

## 常见问题

**它与 Claude Code 自带的 `/code-review` 冲突了。我该怎么办？**

这是这个 skill 最常被报告的问题，而且尚未修复。Claude Code 自带了它自己的 `/code-review`，做的是不同的事情——它在 diff 中猎杀 bug，而这个 skill 检查规范合规性和 repo 标准。安装这个库意味着其中一个会胜出，而哪个胜出取决于你的安装方式。通过插件市场安装时，所有内容都会在 `mattpocock-skills:` 前缀下设置别名，内置的变得难以通过非限定名称访问；通过简单的 skills 安装时，本地文件胜出，这个 skill 会遮蔽内置版本。一个干净的解决方案是完全移除 Claude Code 的内置 skills：这是大量[上下文](https://www.aihero.dev/ai-coding-dictionary/context)的节省，碰撞也就不再重要。遮蔽本身可以说是一个 Claude Code[harness](https://www.aihero.dev/ai-coding-dictionary/harness) 的 bug——skill 作者应该可以自由命名 skill——所以另一个方案是重命名本地副本。编辑 frontmatter 或重命名目录会被 `npx skills update` 撤销；用户报告的持久方案是将 skill fork 到一个新名称并从受管集合中移除 `code-review`，同时记录你 fork 时的 commit，以便手动重新同步。

**它的 sub-agent 反复调用 `/code-review` 并生成更多 agent。**

已知的开放 bug，多人在多个 harness 中复现了这个问题。Standards 和 Spec 提示没有禁止委托，因此 sub-agent 可能重新发现这个 skill 并再次扇出——有报告达到了 50 多个 agent。人们在 fork 中应用的修复是在两个 sub-agent 的简报中都追加一行："Do not invoke `/code-review` or spawn additional agents — perform this review directly." 有些人更倾向于在 harness 层面处理，以便每个 skill 都继承这个防护。这两种方法都还没有进入已发布的 skill。如果你无人值守地运行它，请留意 agent 数量。

**我应该在与编写代码的同一个[session](https://www.aihero.dev/ai-coding-dictionary/session)中运行它吗？**

最好使用一个全新的 session。正如一位读者所说："相同的上下文审查自己不是审查，而是带有斜杠命令的确认偏误。" 编写会话中的审查 agent 持有塑造代码的每一个假设，而这正是一个独立审查者所不会拥有的上下文。这也是为什么人们要求 [implement](https://aihero.dev/skills-implement) 不带其内置审查步骤的原因——它在刚刚编写了 diff 的会话中运行审查。从一个干净的会话中自己调用 `/code-review` 才是诚实的版本。

**是每个 ticket 之后运行一次，还是在最后统一运行？**

两种方式都可以，skill 不会替你做决定。每个 ticket 运行一次能让每个 diff 保持足够小，使 Spec 轴有一个清晰的规范来对照检查，这是 `implement` 使用的模式。在 branch 末尾批量运行能捕获每个 ticket 单独审查时都会遗漏的 ticket 之间的交互。如果你不确定，每个 ticket 审查一次，然后针对 branch 点再运行最后一次审查。

**我可以信任这些发现吗？**

不经过检查就不行。Sub-agent 的输出是假设，而不是证据——有一个团队报告了十几个破坏性变更被基于文本的审查放行了。该 skill 会原样或轻度清理后汇总两个报告，而不是针对文件重新验证每个声明，因此一个发现可能引用错误的位置或夸大影响。在根据每个发现采取行动之前，请阅读其引用。每个发现都必须携带一个引用——一个标准规则、一个气味及其 hunk、或一个规范行——这正是使得检查成为可能的原因。

**为什么我每次运行它都会发现新问题？**

因为修复会创建新的表面，而且 Standards 轴中判断性意见的那部分在每次运行之间不是确定性的。一位读者直白地描述了这个循环："/code-review 和 /improve-code-architecture 每次都会发现新东西。我实施修复，重新运行这些 skill，然后一次又一次。" 没有收敛保证。把一次通过当作一份线索列表，对那些有引用规则支持的线索采取行动，然后停止——不要循环运行它直到它返回干净的结果，因为它永远不会。

**它会审查我未提交的工作吗？**

不会。它 diff 的是 `<fixed-point>...HEAD`，使用三点语法，这是从 merge-base 开始测量的，不包括 staged 和 working-tree 的变更。如果 `implement` 还没有进行中间的 commit，即将提交的工作对审查来说是不可见的。先 commit，然后审查，再修改或添加 fixup。

## 判断它是否正常工作的标准

- 它会在错误的 ref 或空的 diff 上拒绝启动，在任何 sub-agent 生成之前。
- 报告以 `## Standards` 和 `## Spec` 下两个独立的块到达，而不是一个合并的列表。
- 每个 Standards 发现都会命名你的 repo 文件中某条规则或十二种气味之一，并引用 hunk；每个 Spec 发现都会引用规范中的一行。
- 最后的总结给出每个轴的最严重问题，并拒绝挑选总体胜者。
- 当没有规范可用时，Spec 块会如此说明，而不是列出它从代码中推断出的需求。

## 它在流程中的位置

`code-review` 是构建链尾部的审查步骤——`grill-with-docs → to-spec → to-tickets → implement → code-review`——也可以独立用于你指向的任何 branch 或 PR。

- [implement](https://aihero.dev/skills-implement) 是最接近的邻居：它驱动构建并在 commit 之前调用这个 skill 作为自己的收尾审查。
- [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 生成 Spec 轴检查所对照的文档；模糊的规范会使该轴变得模糊。
- [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是整个代码库的对应物——这个 skill 永远只查看一个 diff。

当你不确定该情况需要哪个 skill 时，[ask-matt](https://aihero.dev/skills-ask-matt) 会在整个集合中进行路由。
