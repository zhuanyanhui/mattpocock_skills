## 功能

`codebase-design` 固定了你用来设计模块的词汇：**module**（模块）、**interface**（接口）、**depth**（深度）、**seam**（接缝）、**adapter**（适配器）、**leverage**（杠杆效应）、**locality**（局部性）。它对每个术语都给出了精确定义，禁止使用模糊的替代词（"component"、"service"、"API"、"boundary"），并阐明了由此衍生的少数几项原则。

它是一份参考，而非流程。没有要运行的循环，没有它产出的制品，也没有它会向你提问的检查点。每个涉及设计的 skill 都会借用它的词汇；单独使用时，它只给你语言然后就停下。这是你在调用它之前需要知道的，因为一个没有流程也没有停止规则的 skill，如果你把一个 [session](https://www.aihero.dev/ai-coding-dictionary/session) 指向它然后说"开始"，它会自己即兴发挥一套——参见下面的问题。

## 何时使用

输入 `/codebase-design`，或者当设计任务符合条件时 agent 会自动调用它。

当你已经知道要重新设计哪些代码，并且需要思考它的形态时调用它：接缝放在哪里、接口能缩到多小、一次提取是否值当它带来的开销。当你需要裁决一个词到底是什么意思时，也是调用它的时候。

有几个 skill 与它位置相近。你需要哪一个取决于实际问题是什么：

| 问题 | skill |
|---|---|
| 单个模块的形态——它的接口、它的接缝、它的深度 | `codebase-design` |
| *领域的词汇*——"account"有三种含义，两个人对"cancellation"的理解不同 | [domain-modeling](https://aihero.dev/skills-domain-modeling) |
| 你还不知道要重新设计*哪个*模块 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)——找出候选的调查工具 |
| 你希望设计被审视，而不只是被命名 | [grilling](https://aihero.dev/skills-grilling) |
| 有一个具体的行为要构建，你希望测试能经受住重构 | [tdd](https://aihero.dev/skills-tdd) |

## 词汇表

词汇表就是 skill 本身。每个术语都与其他术语对照定义，并且每个术语都附带了它所替代的词。

| 术语 | 含义 | 不要说 |
|---|---|---|
| **Module（模块）** | 任何具有接口和实现的东西。刻意保持规模无关——可以是一个函数、一个类、一个包、一个跨层级的切片。 | unit、component、service |
| **Interface（接口）** | 调用者正确使用它所需知道的一切：类型签名，加上不变量、顺序约束、错误模式、所需配置、性能特征。 | API、signature |
| **Depth（深度）** | 接口处的杠杆效应——调用者或测试每学习一单位接口就能行使多少行为。**深（Deep）**：小接口背后有大量行为。**浅（Shallow）**：接口几乎与实现一样复杂。 | — |
| **Seam（接缝）** | Michael Feathers 的术语：你可以在不编辑该位置的情况下改变行为的地方。它是接口的*位置*，而把它放在哪里是一个独立的决策，与它背后放什么无关。 | boundary |
| **Adapter（适配器）** | 在接缝处满足接口的具体事物。命名的是一个角色而非物质——内存中的 fake 和 Postgres repo 都是适配器。 | — |
| **Leverage（杠杆效应）** | 调用者从深度中获得的：每学习一单位接口就能获得更多能力。 | — |
| **Locality（局部性）** | 维护者从深度中获得的：变更、bug 和验证集中在一个地方。一处修复，处处修复。 | — |

深度刻意*不*定义为实现行数与接口行数之比，那是 Ousterhout 自己的定义。那个度量会鼓励往实现里注水。这里改用深度即杠杆效应。

## 四项原则

- **深度是接口的属性，而非实现的属性。** 一个深模块内部可以由小型可互换部分构建而成。它们只是不暴露给调用者。一个模块可以有供自身测试使用的内部接缝，以及在接口处的一个外部接缝。
- **删除测试。** 想象删除这个模块。如果复杂度消失了，它只是一个透传。如果复杂度在 N 个调用者处重新出现，它就值当了。
- **接口即测试表面。** 调用者和测试跨越同一个接缝。如果你想*越过*接口去测试，这个模块的形态就不对。
- **一个适配器意味着假设的接缝。两个适配器意味着真实的接缝。** 在有什么东西真正跨越接缝变化之前，不要切出接缝。单适配器的接缝只是间接层。

还有两个支撑文件走得更远，skill 会按需读取它们而非一开始就读取。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 对候选的依赖进行分类——进程内、本地可替代、远程但自有、真正的外部——因为分类决定了深化后的模块如何跨接缝进行测试。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 启动并行的 [sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent) 为同一模块生成三个或更多截然不同的接口，然后在深度、局部性和接缝放置上对它们进行比较。

## 常见问题

**我到底怎么在 TypeScript 中构建一个深模块？**

这是关于这个 skill 被问得最多的问题，而 skill 本身并不回答它。它定义了深模块*是什么*；它没有说如何阻止一个随意的 import 越过接口去访问。[Issue #458](https://github.com/mattpocock/skills/issues/458) 直截了当地提出了："假设我们对接口很满意，它隐藏了细节等等。但我们怎么强制执行呢？我觉得如果没有 lint 或明确的护栏，人类和 LLM 都会逐渐让它变得混乱。" Matt 在那个帖子中的回答给了三个选项：用 class 或 IIFE 包装它，接受 class 会变得巨大；在 monorepo 中把它做成一个 package，接受 monorepo 工具链；或者使用像 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 这样的 linter 来禁止绕过接口的 import。他还曾单独表示 Effect 是最好的机制，dependency-cruiser 是第二好的。仓库的 `in-progress/` 分类中有一个 `setup-ts-deep-modules` skill，它建立了 `src/packages/<name>/index.ts` 约定，但它是一个 beta 通道的 skill，没有文档页面，也没有随附的 lint 规则。

**我把一个 session 指向它，它烧掉了 100k [token](https://www.aihero.dev/ai-coding-dictionary/token) 重新设计了我从未要求过的东西。**

这是已知问题，已记录在 [issue #449](https://github.com/mattpocock/skills/issues/449)。这个 skill 是模型调用的，并自称为词汇表，但其中没有任何东西能硬性阻止 agent 把它当作可运行的流程来对待。当被告知"恢复 /codebase-design 并推动开放决策"时，agent 找到了它能找到的最像行动的内容——`DESIGN-IT-TWICE.md` 中的并行 sub-agent——重新探索了前一个 session 已经映射过的代码，并在提问之前跑了很远。这里不存在驱动 skill 所具备的任何护栏（检查点、一次一个问题、禁止自动推进），因为参考文档没有这些。变通方法是命名一个驱动 skill，让本 skill 位于其下：`/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`，将 `codebase-design` 作为词汇表。该 issue 仍处于开放状态。

**`design-an-interface` 去哪了？还有 `/interface-design` skill 吗？**

`design-an-interface` 已被移除并吸收到本 skill 中。没有丢失任何内容：它的"设计两次"技术——并行 sub-agent 生成截然不同的设计，源自 Ousterhout——作为 `DESIGN-IT-TWICE.md` 放在这里。另外，有几个人要求一个专门的 `/interface-design` skill 来承载深模块/薄接口的理念；这个理念已经存在于本 skill 中，没有计划单独设立 skill。如果你是冲着这两个名字来的，这就是你要找的页面。

**这不就是个文件结构约定——文件夹、barrel 文件、feature 切片吗？**

不是，而且这个 skill 在反复的反对意见中一直坚持这条线。[Issue #95](https://github.com/mattpocock/skills/issues/95) 提议将形式化的分形树文件结构作为深模块的具体实现；回复是两者正交——"深模块关注的是接口设计以及通过严格接口进行访问，与文件系统长什么样无关。用这种方式完全有可能仍然得到浅模块。" #458 中也出现了同样的情况："我觉得你可能把模块的概念和文件系统绑得太紧了。文件系统当然可以对模块形态提供有用的提示，但构建深模块并不一定要用到文件系统。" 词汇表有意将 **module** 定义为规模无关的。

**`tdd` 真的用了这套词汇吗？**

现在用了。很长一段时间里它没有用。曾经在 `tdd` 内部的内联深模块说明在 v1.0 中被移除，转而使用这个共享 skill，但替换它们的指针从未被添加——所以 `tdd` 自己定义了"seam"且没有引用任何东西。这个缺口已补上：指针现在已经放在 skill 中，当接口形态是开放问题而非测试时调用。`tdd` 仍然拥有"seam"作为你*测试*所在的边界；本 skill 拥有它背后的模块形态。

**设计两次模式在 Claude Code 之外也能用吗？**

不能 cleanly 使用。`DESIGN-IT-TWICE.md` 说的是"使用 Agent tool 并行生成 3+ 个 sub-agent"，这是 Claude Code 以自己的名字命名的 [tool](https://www.aihero.dev/ai-coding-dictionary/tool)。仓库为其他 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)（包括 Codex）也发布了元数据，那些可能在这个名字下什么都不暴露——所以并行设计阶段不如 skill 的元数据暗示的那样可移植。记录在 [issue #564](https://github.com/mattpocock/skills/issues/564)，开放中。

**我能往词汇表里加自己的概念吗——connascence、module secrets、[progressive disclosure](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)？**

确实有人提议过这些。[Issue #180](https://github.com/mattpocock/skills/issues/180) 添加了 Parnas 的 module secrets 和 Page-Jones 的 connascence 作为命名层来描述*什么*在接缝处泄露了，并附上了可用的 diff；[issue #303](https://github.com/mattpocock/skills/issues/303) 提议在实现内部使用渐进式披露，这样在公共接口处很深的模块在下面不会变成一整块无差别的板砖。两者都开放且未合并。已发布的词汇表刻意保持精简，skill 本身就说明了保持精简的原因：一致的语言才是全部目的，而一个没人一致使用的术语比没有术语更糟。

## 生效的标志

- 设计对话不再产出"component"、"service"和"boundary"这些词，开始产出"module"、"interface"和"seam"。
- 有人能指着一个提议的提取，不加犹豫地说它是否通过了删除测试。
- 一个提议的接缝同时命名了第二个适配器，而不只是第一个。
- 对接口的讨论涵盖不变量、顺序和错误模式——而不仅仅是类型签名。
- 调用它不会启动一个 session。如果 agent 仅凭 `/codebase-design` 就开始读文件并提出重构，它就把参考当成了驱动。

## 定位

`codebase-design` 是一个**随时可调用的独立 skill**，是工程 skill 之下的底层词汇，而非任何链中的步骤。它最近的邻居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，那是*问题域*词汇的并行参考，而非模块形态——两者通常需要一起使用，因为要给一个深模块取好名字需要两者兼备。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是另一个：它调查代码库找出深化候选，并用这个词汇表写出每一个，所以它找到模块，而本 skill 是你用来设计它的工作台。当你不确定哪个 skill 或工作流合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
