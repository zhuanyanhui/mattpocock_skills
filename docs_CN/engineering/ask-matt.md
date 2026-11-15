## 它做什么

`ask-matt` 是本 repo 中 skill 的路由器。你描述自己所处的处境——一个不知从何开始的构想、一堆 incoming 的 bug 报告、一个[会话](https://www.aihero.dev/ai-coding-dictionary/session)已经拖得太久——它会给出匹配的 skill 或 skill 序列，以及该序列中人类决策的位置。

它只做推荐和停止。它不会进行 grilling、编写[规范](https://www.aihero.dev/ai-coding-dictionary/spec)、打开文件或触发它刚刚命名的 skill；你得到的结果是下一步该输入什么，由你来输入。它也是一份本 repo 中 skill 的手写地图，而不是对你已安装内容的扫描，所以它不会把你路由到你自己的 skill 或其他作者的 skill 上。

## 什么时候使用它

你通过输入 `/ask-matt` 来调用它——agent 不会自行调用它。

| 你的处境 | 路由器给出的结果 |
| --- | --- |
| 有一个构想，但不知从何开始 | 主流程的入口，以及构建是否小到可以跳过 spec |
| 来自其他人的 bug 和需求 | [triage](https://aihero.dev/skills-triage) 入口匝道，以及为什么你自己生成的[任务](https://www.aihero.dev/ai-coding-dictionary/ticket)不属于这条路由 |
| 两个 skill 看起来可以互换 | 它们之间的界线，通常是一个具体的测试而非品味问题。[grill-me](https://aihero.dev/skills-grill-me) 还是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 取决于你是否在[工作目录](https://www.aihero.dev/ai-coding-dictionary/working-directory)中；[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 还是 [wayfinder](https://aihero.dev/skills-wayfinder) 取决于工作量是否适合一个会话 |
| 一个长会话，需要就[上下文](https://www.aihero.dev/ai-coding-dictionary/context)做出决策 | 阶段边界处五个选项的有序树 |
| 一个你已经选好的 skill | 没有有用的信息。直接调用那个 skill。 |

## 前置条件

路由器只命名 skill，不安装它们。它指向的一切都必须已安装，推荐才是可执行的，而且它只知道本 repo 中的推广 skill。

依赖任务跟踪器的路由——triage、`to-spec`、`to-tickets`、`implement`——假设 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 已经在 repo 中配置好了任务跟踪器。路由器会愉快地在这些配置完成之前就推荐它们。

## 工作流，而非 skill

这个 skill 给你的思考词是**工作流**（flow）：一条穿*过* skill 的路径，而不是单个 skill。命名你的处境会将你放在工作流的某个步骤上，这与"这里是匹配你关键词的 skill"是不同的答案。存在四种路由，skill 本身完整地携带了它们：

- **主流程**（main flow），从构思到交付。Grill、spec、任务、implement、review，其中有两个分支：当某个问题需要可运行的代码来敲定时有一条 prototype 绕行路线，以及 spec-and-tickets 拆分，只有当构建跨越多个会话时才值得付出这个成本。
- **入口匝道**（on-ramps），用于产生工作然后汇入主流程的处境：incoming 的 bug 报告、某些东西坏了，或者一个太模糊太大、无法在一个会话中容纳的工作量。
- **独立技能**（standalones），脱离所有工作流，因自身的原因被调用——prototype、问卷、你正身处其中的合并冲突。
- **底层词汇层**（a vocabulary layer underneath），两个参考，当问题出在用词而非流程上时，其他 skill 会拉取它们。

## 阶段边界

它交给你的另一个概念是**阶段边界**（phase boundary）。一个[阶段](https://www.aihero.dev/ai-coding-dictionary/phase)是会话中的一块工作——[grilling](https://www.aihero.dev/ai-coding-dictionary/grilling)、实现、QA——两个阶段之间的边界是"我要怎么处理这个上下文？"这个问题唯一该出现的地方。在阶段中间没有什么可决定的：继续，或者把剩下的拆给[subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)。

| 选项 | 何时采用 |
| --- | --- |
| **Continue** | 下一个阶段需要当前阶段逐字保留，或者你还剩[智能区](https://www.aihero.dev/ai-coding-dictionary/smart-zone)。这是唯一能将会话保持为[一手资料](https://www.aihero.dev/ai-coding-dictionary/primary-source)的操作，所以先排除它 |
| **`/clear`** | 你身后的一切都是可丢弃的。最便宜的操作，而且如果你判断错了就是单向的 |
| **[handoff](https://aihero.dev/skills-handoff)** | 有什么东西需要传递：一个新的[harness](https://www.aihero.dev/ai-coding-dictionary/harness)、一个新的目录、一位同事、一个在阶段中间分叉出去的支线任务 |
| **Subagent** | 任务的范围足够窄，可以让你[离开键盘](https://www.aihero.dev/ai-coding-dictionary/afk)去运行 |
| **`/compact`** | 以上都不是。默认选项，而且经常落在这里 |

其中两个经常被搞错，这就是为什么路由器携带的是顺序而非列表。`/handoff` 读起来像是窗口之间的通用桥梁，但其实不是：可移植性是它带来的全部价值。`/compact` 是树的底部而非第一个选项，因为它上面的四个问题每个都更便宜或更精确。

## 常见问题

**不就是按正确顺序列一个 skill 清单吗？**

人们一直在 README 里要这个。这个 skill 就是那个清单——它就是为此而存在的。一个静态表格会说 `wayfinder → to-spec → to-tickets → implement → code-review`，但对大多数情况来说这是错的，因为有意思的部分是分支——有没有代码库、构建是否跨越会话、这个问题能不能通过谈话来解决。诚实的代价是路由器是手工维护的，会滞后于 repo。`/grilling` 和 `/resolving-merge-conflicts` 都在路由器命名它们之前很久就发布了。

**它告诉我一半的 skill 没安装。**

一个已知 bug，尚未修复。大多数路由器路由你经过的 skill 都设置了 `disable-model-invocation: true`，这意味着 harness 会将它们从注入到 agent 上下文中的 skill 列表中排除。agent 把那个列表当作穷尽的，然后报告它们缺失。有一个报告的会话中，它声明整个 spec-and-tickets 工作流不存在，并重新路由到裸 `/grilling` 和 `/tdd`。插件的二十二个 skill 中有十三个带有这个标记，所以这是常见情况而非边缘情况。它们已经安装了。无论如何直接输入斜杠命令，或者检查 `.claude-plugin/plugin.json`，它是判断什么已存在的权威来源。

**它描述了一个 skill 的行为，但那个 skill 并不那样做。**

这也是真实的，也尚未修复。路由器从自己对每个 skill 的一行摘要中回答，而不是从 skill 本身。一份详细的报告追踪了单个会话中的三个实例，包括一个建议跳过 [to-spec](https://aihero.dev/skills-to-spec) 的情况，依据的是"把线程变成 spec"这个 gloss——`to-spec/SKILL.md` 从未被打开。在每个案例中，它都只在用户反驳后才验证，而且从未主动进行过。在那里跳过 `to-spec` 的代价是失去了一次真正的 seam 检查，产出的任务也低估了工作量。当路由器对另一个 skill 做出[关键原因](https://www.aihero.dev/ai-coding-dictionary/load-bearing-reason)的断言时，让它先打开那个 `SKILL.md`。同样的道理也适用于地图完全没有覆盖的问题，比如是否使用[plan mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode)：那个答案是[模型](https://www.aihero.dev/ai-coding-dictionary/model)的推断，不是这里写下的东西。

**为什么是文本而不是编号清单？**

一个合理的抱怨，作为一个开放 issue 提交了，论点说大部分路由是确定性的，叙述式使得它难以扫描。没有什么能阻止你要求压缩形式——"直接给我序列"就能得到序列。文本所承载的是条件那一半：分支、哪里预期有人类决策、哪里在步骤之间 clear 或 compact。一个扁平的清单丢掉的恰恰是这些。

**它能路由我自己的 skill 或其他作者的 skill 吗？**

不能。有三个独立的提案要求路由器读取你本地的 `skills/` 目录并从任何已安装的东西中推荐。`ask-matt` 不是那个。它是一组技能的地图，由手工维护，它对你编写或从别处安装的 skill 一无所知。

**它让我去编辑一个 SKILL.md。**

这个建议经常是正确的，但很少是持久的。有人问它如何让 [implement](https://aihero.dev/skills-implement) 关闭任务，被告知往 skill 里加一行，然后立刻发现了问题：`npx skills update` 会覆盖文件，而且插件安装是只读的。把常驻行为放在你自己的 `CLAUDE.md` 或 `AGENTS.md` 中，或者在调用中说出来。提示级别的适配能经受更新——把流程指向 Linear 而不是 GitHub，或者问它哪些开放任务可以并行运行，都是人们这样做的方式。

**它命名了一个我没有的 skill，或者漏掉了我确实有的。**

在假定它消失了之前先检查 changelog 看是否有重命名。`writing-great-skills` 变成了 [writing-for-agents](https://aihero.dev/skills-writing-for-agents)，没有别名；`to-prd` 变成了 [to-spec](https://aihero.dev/skills-to-spec)；`pathfinder` 变成了 [wayfinder](https://aihero.dev/skills-wayfinder)。四个 skill 被直接退休并融入了吸收它们的 skill：`ubiquitous-language`、`design-an-interface`、`qa` 和 `request-refactor-plan`。反过来的情况就是路由器自己的滞后，如上所述。

## 它在工作的标志是

- 它以命名该输入什么并就此停止来结束，而不是自己动手开始工作。
- 它给出的路由提到了在哪里 clear 或 compact 上下文、在哪里预期你进行 review，而不仅仅是一个 skill 名称列表。
- 当两个 skill 接近时，它说选哪个以及为什么另一个不适合你。
- 它对另一个 skill 的行为做出的任何声明都在 trace 中显示为它读取了那个 skill 的 `SKILL.md`。
- 你在它交回的内容中认出了自己的处境，而不是最接近的通用场景。

## 它的位置

`ask-matt` 是一个**独立路由器**，覆盖整个集合。它从来不是链中的一步；它指向每一条链，而且它是其他文档页面链接回来的节点，这样它们就不必重新绘制这张图。从这里你最常落到 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)（主流程的入口）或 [triage](https://aihero.dev/skills-triage)（针对已到达的工作而非你启动的工作的入口匝道）。

它是它所描述的 skill 的[二级资料](https://www.aihero.dev/ai-coding-dictionary/secondary-source)。当路由器和 `SKILL.md` 有分歧时，`SKILL.md` 是正确的。
