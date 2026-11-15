Quickstart:

```bash
npx skills add mattpocock/skills --skill=grill-with-docs
```

```bash
npx skills update grill-with-docs
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs)

## 它做什么

`grill-with-docs` 就你的计划或设计对你进行访谈，直到你和 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 对它有了共同理解，并且在访谈过程中把词汇和困难的决策写入你的仓库。它和 [grill-me](https://aihero.dev/skills-grill-me) 是同一场访谈——一轮问题，然后等待，然后下一轮——指向代码库。

它是**[有状态的](https://www.aihero.dev/ai-coding-dictionary/stateful)**。所有其他 grilling 技能把[会话](https://www.aihero.dev/ai-coding-dictionary/session)留在你的脑子里；这个技能把文件留在磁盘上。一个术语被确定后就会立刻落入 `CONTEXT.md`，不是最后批量处理。一个决策通过三道门后就会作为 ADR 落入。这就是全部的区别，也是这个技能大多数麻烦的来源：产物是真实仓库中的真实文件，所以它们可能在你期望它们的时候缺席，也可能在多个人同时写它们时漂移。

## 何时使用

你通过输入 `/grill-with-docs` 来调用它——agent 不会主动调用它。

在变更开始时使用，在仓库中，当计划仍然模糊、事物的词汇还没有确定时。这是单次会话的工具。你想要哪个 grilling 技能取决于你面前有什么：

| 你有什么 | 使用 |
| --- | --- |
| 你根本没有在工作目录中工作 | [grill-me](https://aihero.dev/skills-grill-me) |
| 一个仓库，一个可以在一次会话中确定的变更 | `grill-with-docs` |
| 一个太大无法在一次会话中容纳的工作量——绿地构建、大型功能 | [wayfinder](https://aihero.dev/skills-wayfinder) |
| 一个完全没有领域文档的仓库，也没有特定的功能想法 | `grill-with-docs`，指向仓库而非特定变更 |
| 一个依赖于别人脑子里的知识的决策 | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

wayfinder 的分界在于会话数量：`/grill-with-docs` 用于单次会话规划，`/wayfinder` 用于多会话规划。

## 前置条件

这个技能会写入你的仓库，所以你需要在一个可以安全写入的地方。已确定的术语进入根目录的 `CONTEXT.md` 词汇表——或者，如果根目录的 `CONTEXT-MAP.md` 标记仓库为多上下文，则进入相关上下文的 `CONTEXT.md`。决策进入 `docs/adr/`。两者都是懒惰创建的；在第一个术语或决策结晶之前什么都不存在，所以前面没有需要搭建的东西。

它还需要另外两个技能在场，因为它自己的 `SKILL.md` 是一行委托给它们的代码：[grilling](https://aihero.dev/skills-grilling) 提供访谈，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供写入。只安装 `grill-with-docs` 会得到一个无法工作的技能。

## 文档记录

一次会话产生三样东西，而且它们不平等。

| 确定了什么 | 落入哪里 |
| --- | --- |
| 一个术语——项目自己对某事物的用词 | `CONTEXT.md`，内联，在确定的那一刻 |
| 一个难以撤销、没有上下文会令人惊讶、且是真正权衡结果的决策 | `docs/adr/` 下的 ADR |
| 你决定的其他一切 | 对话中，其他地方不存 |

第三行是最容易让人踩坑的。`CONTEXT.md` 是词汇表，而且被有意保持为词汇表——没有实现细节，没有 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，没有笔记。ADR 需要同时满足三个条件才能写入，所以大多数决策不符合条件，大多数会话不产生 ADR。一个产生了更清晰的词汇表和零个 ADR 的会话是正常工作的，但这意味着你同意的大部分内容只存在于你同意它的[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)中。把同样的对话交给 [to-spec](https://aihero.dev/skills-to-spec) 而不是[清理](https://www.aihero.dev/ai-coding-dictionary/clearing)它。

词汇表才是重点。领域语言才是这个技能真正在构建的东西——项目自己的词汇，约定一次，这样你、agent 和你的同事就不用再花钱重新推导它们。值得说的是，不是所有人都同意这能提升 agent 性能：最尖锐的公开反驳是，一个术语和它的纯英文展开从 [model](https://www.aihero.dev/ai-coding-dictionary/model) 得到的结果是一样的，而且词汇表真正压缩的是共享它的人类之间的沟通。这种理解仍然让词汇表有价值；只是把价值的重心移动了。

## 它假设只有一个写手

有状态的产物假设有一个人在管理它们。一个两人在一个仓库中干了四个月的小组报告说，在抽样合并的 PR 中约 20% 出现了状态漂移，ADR 引用和 README 声明是漂移最严重的表面——刻意的人工管理文档比 agent 记忆漂移得更厉害。修剪过时的文档也保持不住；同样的扫描几天后再次过时。有效的方法是直接删除影子状态，并在 CI 中加入确定性的引用和链接 lint 检查。

相关地：在一个仓库中跨不相关的变更反复运行这个技能往往会积累混合主题的文档，因为没有东西把一个会话的输出和另一个的分开。这两个问题在今天都还没有在技能中修复。

## 常见问题

**我应该用这个还是 `/wayfinder`？**

范围决定一切。对于任何可以在一次会话中确定的事情使用这个技能；当工作量太大无法在一次会话中容纳时使用 [wayfinder](https://aihero.dev/skills-wayfinder)，它会先将工作绘制为决策 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 的地图。Wayfinder 更慢、更密集，在一个范围明确的功能上使用它是常见错误。它不会替代这个技能——它可以落入 grilling 会话中处理地图中适合的部分。

**它运行了，但没有出现 `CONTEXT.md` 也没有 ADR。**

两个已知原因。普通的那个：没有符合条件的。ADR 需要三道门全部通过，而且一个没有新词汇的变更的会话确实没什么可写的。真正的 bug：当这个技能运行在另一个编排层内部时——spec 驱动开发包装、多 agent 框架、把它作为别人流水线一步的规则——文件写入的半边被报告为悄悄地不工作了，而访谈仍然在运行。这个问题已被归档且未修复。如果你在那个设置中，在信任会话的输出之前检查工作目录。

**它一次性问了所有问题，没有推荐答案，而且从没提到过 `CONTEXT.md`。**

那是这个技能没能加载它的两个依赖。因为 `SKILL.md` 是一行委托，一个没有加载 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 的 agent 会猜测 grilling 是什么意思，你就会得到一个没有分类的问题倾倒。部分加载是更令人困惑的情况——`grilling` 加载了，`domain-modeling` 没有，你就会得到一场好的访谈但没有文档记录。这与模型和[努力程度](https://www.aihero.dev/ai-coding-dictionary/effort)级别相关，也是这个技能报告最多的问题。如果你怀疑这一点，直接问 agent 它加载了哪些技能。

**我的其他决策去哪了？**

只存在于对话中。这是关于这个技能最有实质性的抱怨：词汇表不是 spec，大多数答案不配得 ADR，而且没有一个账本把每个已确定的答案连接到 spec、ticket 和测试上。精确的答案——顺序保证、否定性要求、数值默认值——在下游被弱化为更弱的文本，结果看起来完整但漏掉了你实际决定的东西。今天可用的缓解方法是保留会话并直接将其输入 [to-spec](https://aihero.dev/skills-to-spec)，以及根据你自己的答案重新阅读 spec 而不是假设它捕获了你的决定。

**我可以把它指向一个完全没有文档的已有仓库吗？**

可以。对于没有 ADR、没有领域语言、没有设计原则的代码库来说这是正确的技能——调用它并说"帮我记录我的仓库"。社区模式将它与 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 配对来构建或修复 `CONTEXT.md`。预计你需要引导它：它会阅读代码并问你关于它发现的问题，而你才是决定代码库中已有的哪些词是正确的那个人。

**会话结束时我应该做什么？**

这个技能的结束语往往是开放式的，这是一个已知的粗糙边缘。在主流程中答案是 [to-spec](https://aihero.dev/skills-to-spec)，在同一会话中。如果变更足够小可以立即构建，直接转到 [implement](https://aihero.dev/skills-implement)。

**为什么叫这个名字？**

没人对这个名字满意。有一个建议把它重命名为 `grill-domain-model`，这样更诚实地描述了行为。目前没有进展。如果重命名真的落地，文档页面会随着移动，URL 也会变。

## 它生效的标志

- `CONTEXT.md` 在会话*进行中*一个术语一个术语地变化，而不是在最后突然冒出一大块。
- 词汇表读起来是纯词汇——你项目的词汇带有精炼的定义——不包含实现细节或 spec 式的散文。
- 代码库能回答的问题通过阅读代码库来回答，而不是问你。
- 你得到很少或没有 ADR，而你得到的那些是你会对不得不重新辩论感到恼火的决策。
- 它挑战你用过的一个词，因为你现有的词汇表对它的定义不同。

## 在体系中的位置

`grill-with-docs` 是主构建链的头部：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它排在任何内容被写成 spec 之前——它产生共同理解和确定的词汇表，然后 [to-spec](https://aihero.dev/skills-to-spec) 将其合成而无需重新访谈你。它的近邻是 [grill-me](https://aihero.dev/skills-grill-me)，同一场访谈但没有仓库没有文件，和 [domain-modeling](https://aihero.dev/skills-domain-modeling)，它驱动的词汇表和 ADR 规训；两者都基于 [grilling](https://aihero.dev/skills-grilling) 原语。在上游，[wayfinder](https://aihero.dev/skills-wayfinder) 绘制太大无法一次完成的工作的地图，并且可以把地图的部分交回给它。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
