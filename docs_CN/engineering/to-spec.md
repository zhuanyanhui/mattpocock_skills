Quickstart:

```bash
npx skills add mattpocock/skills --skill=to-spec
```

```bash
npx skills update to-spec
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-spec)

## 它做什么

`to-spec` 将你刚刚进行的对话转化为一份 **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)**，并将其作为单个问题发布到你的问题跟踪器。

它不会访谈你。当你用到它时，决策工作已经完成，所以它综合已知内容——来自线程、来自代码库、来自你的 `CONTEXT.md` 和 ADR——而不是开启新一轮问题。规范是对已做出决策的记录，而非做出新决策的地方。

## 何时使用

你通过输入 `/to-spec` 来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会主动调用它。

当构建太大无法容纳单个 agent [会话](https://www.aihero.dev/ai-coding-dictionary/session) 且必须在多个会话中存活时使用。这就是整个触发条件：

| 你在哪里 | 运行什么 |
| --- | --- |
| 你还没有做任何决定 | 先用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 已决定，且工作适合一个[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) | [implement](https://aihero.dev/skills-implement)——跳过规范 |
| 已决定，且工作跨多个会话 | `/to-spec`，然后 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder) 地图已清晰 | `/to-spec #<map_issue>` |

## 前置条件

`to-spec` 将规范作为问题发布，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须已为此仓库配置了跟踪器和分类标签词汇。哪种都行：像 GitHub 这样的真实跟踪器，或者 `.scratch/` 下的本地 markdown 文件，开箱即用地支持。

## 规范是决策记录

规范存在是因为上下文窗口会结束。你在 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 时解决的所有东西——解决方案的形状、你争论过的选择、你刻意拒绝的东西——都在一个即将被清除的对话中。规范是在那之后存活下来的东西。

所以它不验证任何东西，也不决定任何东西。它用项目自己的词汇捕捉已决定的内容，这样一个新的会话可以接手工作而不需要你重新解释。规范断言的任何你从未真正说过的内容都是一个缺陷。

## 接缝先于正文

`to-spec` 在写一个字之前，先勾勒功能将在哪些**接缝**处测试，并与你确认。它优先使用已存在的接缝而非新接缝，并取它能取的最高接缝——一个变更的理想数量是一个。

这些商定的接缝接着会流转。[tdd](https://aihero.dev/skills-tdd) 只在预先商定的接缝上工作，而 [code-review](https://aihero.dev/skills-code-review) 根据规范审查 diff，所以一个没人商定过的接缝会作为审查发现出现出来。这种绑定是间接的——通过这份文档流转——而这正是为什么值得在这里认真对待接缝对话，而不是把它推迟到实现阶段。

## 常见问题

**`/to-prd` 去哪了？**

就是这个技能，在 v1.1 中重命名了。"Spec" 现在是贯穿始终的单一术语，旧的 `to-prd` 别名已死——用新名称重新安装。取代旧词汇的一对词是 *spec* 和 *tickets*：spec 是决策和修复它的目的地，[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 是到达那里的执行步骤。如果你转向，删除未完成的工单并保留 spec。

**为什么规范被贴上 `ready-for-agent` 标签？我不想 agent 基于它实现。**

这个标签意味着"无需进一步分类"——文档足够完整，agent 可以基于它工作。这是一个输入指定，而非工作命令。但如果你运行轮询 `ready-for-agent` 的 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agent，它们看不到这个区别，而且它们会很乐意尝试在一次运行中构建整个 spec，而不是领取工单切片。这是这个技能报告最多的粗糙边缘。在改变之前，在你的 AFK agent 的提示中明确排除父 spec，或者在 `/to-tickets` 运行后去掉标签。

**为什么不直接从 grilling 到 `/to-tickets`，跳过 spec？**

通常你应该——spec 只有在多会话工作中才值得这一步。它的价值在于工单是一次性的，而 spec 不是：每个工单的大小适合一个新鲜的上下文窗口，会被删除或关闭，而 spec 作为它们背后推理的唯一存活位置保留下来。在单次会话变更中这买不到任何东西，而且你还多付了一个 [model](https://www.aihero.dev/ai-coding-dictionary/model) 可能漂移的综合步骤。走 grilling → `/implement`。

**我刚完成了 wayfinder 地图。应该给它什么？**

主地图问题——`/to-spec #<map_issue>`，而不是单个决策工单。[wayfinder](https://aihero.dev/skills-wayfinder) 产出决策而非交付物，散布在地图上；`to-spec` 是将它们折叠成一个可构建文档的步骤。把地图直接循环到 `/implement` 就丢弃了这个折叠。

**规范是给我审查的，还是只给 agent 的？**

主要是给 agent 的，而且读起来也是如此——完整、密集、参考密集。你值得看的部分是接缝和不在范围内章节，因为这两个地方是错误决策最容易捕捉、以后发现代价最贵的地方。从头到尾读整个文档是人们真正的抱怨，而且没有摘要模式：诚实的答案是，如果规范让你惊讶，那是 grilling 太浅，而不是规范太长。

**工单开始后规范是保持冻结的，还是让 agent 重写它？**

没有东西让它保持同步，所以实际上它是你在那一刻所知的快照，而且它在实现教会你新东西的第一时间就会过时。把工作交付后就把它当作一次性的。意在超越它存活的是你的 `CONTEXT.md` 和你的 ADR——如果实现中学到的东西值得保留，它应该去那里，而不是编辑后的 spec。

**我的工作是重构或模块边界，而不是功能。模板适合吗？**

不太适合，这是一个已知的限制。模板严重依赖用户故事，这对架构工作来说是错误的形状——你会为真正关于接口和不变量的决策写出没人要求的用户故事。依靠实现决策和测试决策章节，让持久的架构调用通过 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 作为 ADR 落地，而不是试图让规范承载它们。

**它会检查跟踪器中是否有相关工作，或引用它尊重的 ADR 吗？**

两个都不会。它会阅读并尊重它所接触区域覆盖的 ADR，但不会链接它们，也不会在起草之前搜索跟踪器中是否有重叠的问题——所以一个 spec 可能会悄悄复制别人已经提交的工作。如果这个区域很忙，你自己先搜索跟踪器。

**`/to-tickets` 读不了我的 spec——它一直在截断。**

非常大的 spec 可能会超出跟踪器问题能干净返回的内容，而且没有本地副本可以回退。修复方法是上下文卫生：不要在 `/to-spec` 和 `/to-tickets` 之间[清理](https://www.aihero.dev/ai-coding-dictionary/clearing)或[压缩](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个窗口中运行它们，spec 就根本不需要重新获取。

## 它生效的标志

- 它开始编写，而不是向你提出新一轮问题。
- 它在编写之前把接缝交给你，并提出尽可能少的接缝。
- 它用你项目的名词返回，而不是通用的产品管理样板。
- 其中的每个决策都是你能记得做过的。没有为了填满章节而发明的内容。
- 不在范围内章节中有真实的东西——你拒绝的东西通常是页面上最有用的行。

## 在体系中的位置

`to-spec` 是主构建链中的一个步骤，且只在其多会话分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

上游邻居是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它做这个技能只记录的决策工作，以及 [wayfinder](https://aihero.dev/skills-wayfinder)，其完成的地图就在这里合并到链上。下游，[to-tickets](https://aihero.dev/skills-to-tickets) 将规范切分为示踪弹工单，供 [implement](https://aihero.dev/skills-implement) 构建。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
