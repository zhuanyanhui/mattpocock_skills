Quickstart:

```bash
npx skills add mattpocock/skills --skill=domain-modeling
```

```bash
npx skills update domain-modeling
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/domain-modeling)

## 它做什么

`domain-modeling` 在你设计时构建并打磨项目的**通用语言**——质疑与词汇表冲突的术语、在模糊的地方要求精确措辞、用具体场景对关系进行压力测试直到边界清晰无误。

这是**主动**的规训，而非被动的。阅读 `CONTEXT.md` 借用其词汇是任何技能都能做的一行习惯；这个技能是用于你*改变*模型时。这就是它会中断对话的原因。它会在术语被确定的当下就将其写入 `CONTEXT.md`，在对话进行中的时候，而不是在最后生成一份整齐的词汇表——因为批量处理的版本是[会话](https://www.aihero.dev/ai-coding-dictionary/session)的摘要，而内联版本才是会话的实际输出。

## 何时使用

输入 `/domain-modeling`，或者当任务合适时 agent 会自动调用它。实际上，自动调用是这个技能最弱的部分：当 `grill-with-docs` 或 `wayfinder` 说要加载它时，[模型](https://www.aihero.dev/ai-coding-dictionary/model)经常会加载 `grilling` 而跳过这个技能。如果一场 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话结束时 `CONTEXT.md` 没有被碰过，那就是发生了这种情况——要与其他技能一起按名称调用它。

当*词汇*是问题时使用它：

| 情况 | 行动 |
| --- | --- |
| 两个人对"cancellation"意味着不同的东西 | `domain-modeling` —— 选定规范术语，将其他词列入 `_Avoid_` |
| "Account" 在三个文件中做着三份工作 | `domain-modeling` —— 将它拆分为 Customer 和 User |
| 你刚刚做了一个难以撤销的架构选择 | `domain-modeling` —— 如果选择达标，它会提供 ADR |
| 模块的*形状*才是问题——seam 在哪里，接口有多深 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你希望在构建之前对整个计划进行审问 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它底层驱动这个技能 |
| 你想查找一个术语，而不是修改它 | 不用技能。读取 `CONTEXT.md`。它就是一个文件。 |

## 前置条件

没有前置条件。这个技能会写入两个位置，且都是懒惰创建的：

- **`CONTEXT.md`** 在仓库根目录，由第一个被确定的术语创建。在根目录有 `CONTEXT-MAP.md` 的仓库中，术语会写入映射指向的每个上下文的 `CONTEXT.md`。
- **`docs/adr/`**，由第一个达标的 ADR 创建。

开始之前不需要任何东西存在，也不会推测性地创建任何东西。

## 两个产物，两个标准

词汇表和 ADR 遵循不同的标准，混淆它们是这个技能中大多数麻烦的来源。

| | `CONTEXT.md` | `docs/adr/NNNN-slug.md` |
| --- | --- | --- |
| 承载 | 术语。一个事物**是什么**，一到两句话，被拒绝的同义词放在 `_Avoid_` 下 | 一个决策，一到三句话：背景、选择、理由 |
| 写入标准 | 一个模糊的术语变得规范 | **三个条件全部满足**：难以撤销、没有上下文会令人惊讶、真正权衡的结果 |
| 写入方式 | 内联，术语确定的那一刻 | 被提议，而非假定 |
| 绝不承载 | 实现细节、[spec](https://www.aihero.dev/ai-coding-dictionary/spec)、便签本、通用编程概念 | 本次会话中每个选择的日记 |

三个测试中缺失任何一个就不写 ADR。一个容易撤销的决策会被轻易推翻；一个不令人惊讶的决策不是任何人的问题；一个没有真正替代方案的决策只是记录了你做了显而易见的事。

`CONTEXT.md` 的规则才是真正要坚持的，因为它是在实际使用中最容易出问题的。**它是一个词汇表，仅此而已。** 如果不加约束，模型会把"写入 `CONTEXT.md`"理解为可以持久化你给出的每个答案，然后这个文件就变成了运行中的 spec——这是跨多个模型报告最多的问题。

## 交叉引用，以及在哪里停止

让这个技能发挥作用的关键动作：当你陈述某事物如何工作时，它会检查代码并揭示矛盾。*"你的代码取消了整个 Order，但你刚才说部分取消是可能的——哪个是对的？"* 语言和代码在两者被修改之前就公开地达成一致。

知道边界也有价值。它交叉引用的是**代码**和已提交的 `CONTEXT.md`/ADR，没有其他东西。它不会搜索你的 issue 跟踪器，所以几个月前在某个关闭的 issue 中被争论过并有意解决的问题，会被当作新问题重新提出来。[有一个开放的请求](https://github.com/mattpocock/skills/issues/717)来修复这个问题；在此之前，变通方法是在你自己的 `docs/agents/domain.md` 中放入指令，技能已经会读取这个文件。

## 常见问题

**我的 `CONTEXT.md` 有 500 行。1,000 行。3,000 行。怎么办？**

大小是症状，不是病因——这个文件吸收了实现细节和从来就不是词汇表材料的决策。修复方法是直接下达指令：`/grill-with-docs make my CONTEXT.md more concise and remove any implementation details from it`。在一个臃肿的文件上运行它，大部分内容都会被去掉。只有当文件真正精简之后仍然覆盖两个读者不想同时记住的领域时，才考虑用 `CONTEXT-MAP.md` 拆分；拆分一个臃肿的文件只会给你几个臃肿的文件。技能在这方面的指导还不够强，无法首先阻止增长，跟踪这个问题的 issue 仍然开放。

**为什么叫 `CONTEXT.md` 而不是 `GLOSSARY.md`？**

这是整个技能集中争论最多的命名问题，而且没有定论。反对当前名称的理由很充分：如果它"是词汇表，仅此而已"，那 `GLOSSARY.md` 就说明了这一点，而且——正如一位读者所说——"在 AI agent 中一切都是[上下文](https://www.aihero.dev/ai-coding-dictionary/context)"。支持它的理由在于映射：`CONTEXT-MAP.md` 指向多个 `CONTEXT.md` 文件读起来很自然，而 `GLOSSARY-MAP.md` 则不然，而且 `context` 是 DDD 中模型有界区域的标准术语。至少有一人在维护一个纯为了重命名这个文件的本地 fork。你也可以这样做，但技能集中的所有其他技能都在查找 `CONTEXT.md`，所以重命名意味着要修补所有这些技能。

**`/ubiquitous-language` 去哪了？**

它被移除了，而且不是被弃用的。它的工作移入了 `domain-modeling`，后者持续维护整个模型，而不是从一次对话中转储出词汇表。词汇表强制执行变得更关键了，而不是更不重要——它现在运行在 grilling、triage 和 mapping 的底层，而不是作为一个你记得要做的独立步骤。

**如何为一个没有词汇表的代码库获取词汇表？**

明确地要求它，而不是等着它慢慢积累。`/grill-with-docs help me scaffold my existing repo with a CONTEXT.md` 是文档化的路径；预计会有一场漫长的审问——有用户报告在词汇表成形之前被问了 50 多个问题。在一个存量仓库中偶然使用这个技能构建词汇表的速度太慢了。

**我可以保留领域模型并使用自己的 ADR 格式吗？**

目前还没有干净的做法。词汇表部分和 ADR 部分合并在一个技能中，所以一个有既定 ADR 约定的团队——不同的模板、不同的位置、不同的命名——会收到与其内部风格冲突的指令。目前的选择是在本地复制这个技能并编辑它，或者在你自己的 repo agent 文档中覆盖 ADR 约定。将两者拆分[是一个开放的请求](https://github.com/mattpocock/skills/issues/557)。

**词汇表真的有价值吗？它又是一个要审查的产物，可能会过时。**

有时候它没有价值，诚实地说在哪里。DDD 离实现越近就越没用——回报在上游，在命名和概念对齐上，而不是在聚合和层次仪式中。同义词控制在命名的边界处最重要：模块名、表名、状态枚举、issue 标题、CLI 命令。在普通正文中就没那么重要。还有一个现实的反驳：领域术语压缩了*已经共享这些术语的人类之间*的沟通，而且 agent 对纯英文描述的反应是一样的——按照这种理解，词汇表的价值是让你和你的审查者与 agent 的行为保持一致，而不是让 agent 变得更好。在一天就能完成的构建上，跳过它。而且一个未经审查的、agent 撰写的词汇表比没有更糟：它会变成听起来很有根据的传闻，被后来的会话当作事实。

**它能把我模糊的提示转换成领域语言吗？**

不能，而且目前没有计划做这样的技能。你自己都不理解的领域语言写下来就变成了毫无意义的废话。这个技能在你有了理解之后才强制执行精确性——它不会制造你没有的词汇。相关的陷阱是：使用领域词汇却没有做建模：正确的名词放在错误的概念结构上，产生的输出读起来正确但其实不是。

## 它生效的标志

- 它在你说到一半时停下来问你指的是两件事中的哪一件，而不是选一件继续。
- `CONTEXT.md` 在对话**进行中**变化，而不是在最后突然爆发。
- 它拒绝为明天就能撤销的事情写 ADR——并且说出三个测试中哪个没通过。
- 新条目用一到两句话定义一个事物*是什么*，并在 `_Avoid_` 下列出你放弃的词汇。
- 当你的代码和你的说法不一致时，它会把代码引述给你看。
- `CONTEXT.md` 变短的频率和变长的频率一样高。

## 在体系中的位置

`domain-modeling` 是一个**模型调用型参考技能**，它更常在其他技能*底层*运行，而不是单独运行。[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 通过 grilling 会话驱动它，[wayfinder](https://aihero.dev/skills-wayfinder) 在绘制地图时加载它，[triage](https://aihero.dev/skills-triage) 用它来保持[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)使用项目自己的语言，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在决策结晶时调用它。它最近的兄弟是 [codebase-design](https://aihero.dev/skills-codebase-design)：两者是其他一切东西底层的词汇层，这个用于*领域*，那个用于模块的*形状*。当你想要这个规训而不想承诺通常会引入它的技能的步骤时，也可以直接调用它。当你不确定哪个技能适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
