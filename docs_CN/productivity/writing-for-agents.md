## 作用

`writing-for-agents` 是你编写面向 agent 的文档时所参照的参考——无论是 skill、`AGENTS.md` / `CLAUDE.md`、[spec](https://www.aihero.dev/ai-coding-dictionary/spec)、运行时 prompt、README，还是任何 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 阅读的文档。包装形式各不相同，但写作原则不变：相同的杠杆使每一份文档都可预测，让 agent 每次运行都采取相同的*过程*，而不是产出相同的输出。

它的默认动作是删除，而非解释。让 agent 为另一个 agent 编写指令时，它会把大部分篇幅用来解释 [model](https://www.aihero.dev/ai-coding-dictionary/model) 已经知道的内容——这些每一行都是**无操作指令**，消耗 [context](https://www.aihero.dev/ai-coding-dictionary/context) 却不改变任何行为。这份参考就是发现它们的透镜，正因如此，它在已有文档上发挥价值的频率至少与在空白文件上一样高。

它在 v1.1 之前叫做 `writing-great-skills`。这次改名反映了它一贯的本质：其中几乎没有多少内容是 skill 专属的。仅限 skill 的机械部分——frontmatter、模型调用与用户调用的选择、路由器 skill——都被移到了链接的 `SKILL-MECHANICS.md`，只有当你面前的文档是 skill 时才需要阅读。

## 何时使用

输入 `/writing-for-agents`，或者当你在创建或编辑 skill，或修改 `AGENTS.md` 或 `CLAUDE.md` 时，agent 会自动调用它。

对于 agent 阅读的所有其他文档，手动调用它：你的 docs、specs 和 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)、系统 prompt 和 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) prompt。判断标准只有一个问题——agent 会阅读这个吗？——至于文档如何出现在它面前并不重要，无论是指针命名了它、人类粘贴了它，还是它就放在 repo 里。如果是要弄清楚代码库到底包含什么，使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)——这份参考管辖的是文档的可读性，而不是它知道什么。

## 两种负担

整份参考围绕的核心概念是一对预算，每份文档和指针都在花费它们：

- **上下文负担**——始终加载的材料对 agent 窗口的开销：一行 `AGENTS.md`、一个 skill 描述，或者任何在每个 [turn](https://www.aihero.dev/ai-coding-dictionary/turn) 都坐在上下文中的内容，无论它是否触发。
- **认知负担**——你身上的成本：有哪些文档存在，以及何时该调用哪一份。你是索引。这不是要最小化的成本——它是人类自主性的代价。

一旦你用这两种负担来思考，大多数编写决策——拆分还是不拆分、内联还是披露、指向还是推送——就变成了在不同位置做出的同一笔交易。

## 杠杆

- **[上下文指针](https://www.aihero.dev/ai-coding-dictionary/context-pointer)**——保持在上下文中的参考，它命名了上下文外的材料并编码了何时调用它。一个 skill 描述和一行命名了某个文档的 `AGENTS.md` 是同一个对象；指针的*措辞*（而非其目标）决定了 agent 多可靠地透过它访问。
- **信息层次结构**——从文件内步骤，到文件内参考，再到指针后面的披露式参考的阶梯。**[渐进式披露](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)** 是沿着这个阶梯下移的动作，以保持顶部可读。
- **完成标准**——每个步骤完成条件的清晰度和要求强度，以及这种要求所驱动的**深入查找**；这是对抗**过早完成**的防线。
- **引导词**——模型预训练中已有的紧凑概念（*tight*、*red*、*tracer bullet*），agent 在运行文档时用它来思考。它有两处锚点：正文中的执行，指针中的调用。
- **修剪**——单一事实来源、相关性，以及逐句应用的无操作指令测试，对抗**重复**、**沉积**和**蔓延**。

## 常见问题

**`/writing-great-skills` 去哪了？**
就是这个 skill，在 v1.1 中重命名了。实践者们早在名字跟上之前就已经把它指向 `AGENTS.md`、docs、specs、tickets 和运行时 prompt 了；结构、引导词和修剪终究是任何 agent 阅读的文本的技艺。没有别名——用新名称重新安装。

**"为 agent 写作"——所以是 agent 来写？**
恰恰相反。你是作者；agent 是读者。这就是这个体裁的全部难点：你在为一个已经读过一切的读者写作，所以解释是浪费，精准才是全部的工作。

**我不能直接让 agent 替我写吗？**
可以，而且它会产生一份冗长的东西。放任不管的话，模型会解释它已经知道的内容，它不会自己应用无操作指令测试或找到引导词。在草稿上使用这份参考——审查轮次才是它大部分价值落地的地方。

**我让 agent 裁剪文档，它把功能也砍掉了。**
被告知"精简"的 agent 会优化长度，因为长度是它们能看到的东西。无操作指令测试是行为性的，不是审美性的：删掉那行字，问自己 agent 的行为是否改变了。当一个句子通不过时，删掉整个句子而不是从中裁剪词语——而关于它的分歧通过运行文档来解决，而不是争论。

**我怎么知道什么时候算完成？**
当它奏效了，而且你再也找不到重复、沉积或无操作指令时。这里没有自动化的 eval；检查方式是手动运行加上失败模式词汇表作为诊断工具。当文档表现异常时，那份词汇表也是修复套件——先命名失败模式，然后修复它。

**这应该放在 `CLAUDE.md` 里还是别的地方？**
问自己想承担哪种负担。`CLAUDE.md` 无条件地加载到每个 [session](https://www.aihero.dev/ai-coding-dictionary/session) 中；指针后面的材料在触发前只消耗指针自身那一行的开销。任何只在十种情境之一下适用的内容，在其他九次都在支付上下文负担。

**我需要为每个新模型重写文档吗？**
大部分不需要，而且过度适配某个模型本身就是一个陷阱。为新模型更新通常只是再跑一次无操作指令测试，而不是重写。

**我的 skill 只在我构建它的那个确切任务上有效。**
常见路线——做一次工作，然后让 agent 把它写成 skill——会在那一次运行上过度拟合，范例也会变得过于具体。保留那次运行作为证据，然后刻意抽象化：剥离属于那个 repo 和那些文件的内容，为整类任务而写。

**英语不是我的母语。我会失去引导词的优势吗？**
不会——找到用最少 [tokens](https://www.aihero.dev/ai-coding-dictionary/token) 封装最多行为的词，正是这份参考为你做的工作。这就是它的用途之一。

## 正常工作的标志

- 文档随着质量提升而变短，你会惊讶于剩下的内容如此之少。
- 你能指着一个引导词，看到它在多处发挥作用。
- 没有任何内容以任何形式被重复陈述。重复是文档从未被测试过的最可靠标志。
- 只有一个分支需要的参考放在指针后面，而不是放在主文件中。

## 它在体系中的位置

这是一份随时可调用的独立参考。它在链条中没有邻居，因为它位于整个集合的底层，而不是与任何单一 skill 并列：这里的每份 skill 都是参照它写成的，而其他 skill 留下的文档——`CONTEXT.md` 及其 ADRs、一份 spec、一个 ticket——一旦 agent 需要阅读它们，就恰好是这份参考所管辖的文本。当你不确定哪个 skill 或流程适合某项任务时，[ask-matt](https://aihero.dev/skills-ask-matt) 会在整个集合上为你导航。
