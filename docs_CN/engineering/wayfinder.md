## 它做什么

`wayfinder` 将一个超出单个 agent [会话](https://www.aihero.dev/ai-coding-dictionary/session) 所能容纳的工作——一个你能说出**目的地**却还看不清路线的构想——绘制成共享的**地图**，以**决策任务**的形式放在你的任务跟踪器上，然后逐一解决它们，直到道路畅通。

它只做规划，不做实现。每个任务都承载着一个问题，其解决结果是一个决策，而非待执行的构建切片；当所有需要决策的事项都已确定、可以有人去构建东西时，地图才算完成。正是这条规则将 wayfinder 任务与普通的实现[任务](https://www.aihero.dev/ai-coding-dictionary/ticket) 区分开来，也是 agent 最常违反的规则。当地图清空后，wayfinder 就交棒；它不会继续进入代码编写阶段。

## 何时使用

你通过输入 `/wayfinder` 来调用——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。

这是整套技能中最重量级、最密集的[工作流](https://www.aihero.dev/ai-coding-dictionary/flow)，因此触发条件很窄：工作的规模必须确实超出单个 agent 会话能容纳的范围，而且到达目的地的路线必须模糊不清。分界线很清晰：单会话规划用 `/grill-with-docs`，多会话规划用 `/wayfinder`。

| 你面对的情况 | 运行什么 |
| --- | --- |
| 一次坐定就能搞定的范围明确的功能 | [grill-me](https://aihero.dev/skills-grill-me)，或有代码库时用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 绿地项目，或跨多个会话的构建，路线尚不清晰 | `/wayfinder` |
| 决策已经做完的线程 | [to-spec](https://aihero.dev/skills-to-spec)——直接跳过地图 |
| 已清空的 wayfinder 地图 | 先 [to-spec](https://aihero.dev/skills-to-spec)，再 [to-tickets](https://aihero.dev/skills-to-tickets) 和 [implement](https://aihero.dev/skills-implement) |
| 已经膨胀过大的现有会话 | 说"hand off to `/wayfinder`"——[handoff](https://aihero.dev/skills-handoff) 既能从会话桥接到地图，也能从地图桥接出来 |

绿地并非必要条件。Wayfinder 经常用于遗留和半成品代码库，而且在那里 arguably 更加锋利，因为大量迷雾是"这里已经是什么情况"而非"我们应该做什么"。

## 前置条件

地图及其任务都存放在仓库的任务跟踪器上，因此 wayfinder 需要 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 铺设的跟踪器接线。这一步会写入一个"Wayfinding 操作"部分，描述地图、其子任务、[阻塞边](https://www.aihero.dev/ai-coding-dictionary/blocking-edges) 和前沿查询如何针对 GitHub、GitLab 或本地 markdown 表达。Wayfinder 通过 `CLAUDE.md` / `AGENTS.md` 中的指针解析该文档，而非固定路径；如果完全没有配置跟踪器，则回退到本地 markdown 文件。

跟踪器不是装饰品。[阻塞](https://www.aihero.dev/ai-coding-dictionary/blocking) 让前沿在跟踪器自身的 UI 中可视化呈现，而没有原生依赖链接的跟踪器——比如自托管的 Gitea——会使 wayfinder 退化为从地图文本中推断阻塞关系，这能用但需要更密切的监督。

## 地图、迷雾和前沿

**地图**是一个标有 `wayfinder:map` 的单个任务；它的子任务就是它的子 issue。它是一个**索引，而非存储**——一个决策只存在于一个地方，即它的任务，地图只做摘要并链接。一个会话以低分辨率加载地图，按需放大到单个任务，这正是地图能持续增长而不会让每个会话都为其完整历史付费的原因。

地图上有四样东西：

- **目的地**——到达这张地图终点时是什么样子。命名目的地是绘制地图的第一步，在任何任务存在之前就要完成，因为目的地决定了每个任务衡量的范围边界。
- **已有决策**——每个已关闭任务一行，链接到细节实际所在之处。
- **尚未明确**——**战争迷雾**。你能看出即将到来的决策，但还无法精确表述。区分迷雾和任务的标准是：你*现在*能否精确陈述这个问题，而不是你能否回答它。解决一个任务会清除它前方的迷雾，并将现在可明确的内容升级为新任务。
- **超出范围**——被排除在目的地之外的工作。迷雾只会*朝着*目的地聚集，所以超出范围的工作被关闭且永远不会升级。

**前沿**是开放的、未被阻塞的、未被认领的任务——已知区域的边缘。一个会话在做任何工作之前通过将自己分配给任务来认领它，因此被分配者*就是*认领标记，并发会话会跳过它。任务在整个过程中始终通过名称引用，从不使用裸的 `#42`；一堵 issue 编号墙在叙述中是不可读的。

## 四种决策任务类型

每个任务都带有一个 `wayfinder:<type>` 标签，要么是 **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)**——与[能代表自己的人](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)协作——要么是 **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)**，由 agent 单独驱动。HITL 任务只能通过实时交流来解决；自己回答自己的[无情访谈](https://www.aihero.dev/ai-coding-dictionary/grilling)问题的 agent 已经破坏了它。

| 类型 | 模式 | 何时使用 | 解决方式 |
| --- | --- | --- | --- |
| `grilling` | HITL | 默认类型。问题可以通过讨论来解决。 | [grilling](https://aihero.dev/skills-grilling) 加上 [domain-modeling](https://aihero.dev/skills-domain-modeling)，在新会话中 |
| `prototype` | HITL | "它应该长什么样"或"它应该如何表现"——讨论无法解决的问题。 | [prototype](https://aihero.dev/skills-prototype)，构建产物作为资产链接到任务上 |
| `research` | AFK | 工作目录之外的事实阻塞了一个决策。 | 一个 [research](https://aihero.dev/skills-research) [子agent](https://www.aihero.dev/ai-coding-dictionary/subagent)，在绘制地图时触发，在 `research/<name>` [分支](https://www.aihero.dev/ai-coding-dictionary/branch) 上并行烧尽 |
| `task` | 两者皆可 | 没有需要决定的，但有手工工作阻塞了决策——配置访问权限、注册服务、移动数据以便看到它的形状。 | agent 单独完成力所能及的部分，否则给人一个精确的检查清单 |

`task` 是唯一一种*做*而非决定的类型，它的存在价值在于解除决策的阻塞——而不是交付目的地的一部分。这是实践中最容易出错的类型：agent 会把它理解为实现步骤，开始在地图中编写产品代码。

Research 是*每个会话一个任务*原则的唯一例外。

## 常见问题

**这和 `/grill-with-docs` 有什么区别？我应该先运行哪个？**

是会话数量，不是项目规模。`/grill-with-docs` 是单会话规划；wayfinder 是多会话规划。如果你能在一次对话中容纳整个事情，grilling 是更便宜也更好的工具，而 wayfinder 对这种情况来说确实更慢更密集。社区已经沉淀出的简略说法：只有当工作无法塞进单个会话时，wayfinder 才有意义。这是迄今为止被问得最多的 wayfinder 问题，而且一直被问，因为描述没有告诉你你自己的工作在那条线上的位置——你必须自己判断会话数量。

**当它要求"目的地"时，是指这次会话的终点还是一切的终点？**

整张地图——整个地图的目的地，不仅仅是初始会话。这个问题读起来有歧义，因为 wayfinder 按定义就是一个多会话工具，所以以会话为范围的答案永远说不通。典型的目的地包括要交棒的[规范](https://www.aihero.dev/ai-coding-dictionary/spec)、规划开始前要锁定的决策、概念验证，或就地完成的变更比如数据迁移。

**地图已经清空了。为什么我还需要 `/to-spec` 和 `/to-tickets`——wayfinder 不是已经写了规范并创建了任务吗？**

不是。Wayfinder 的任务是决策任务，到地图关闭时它们也都已关闭。剩下的是一张充满已链接决策的地图，这不是构建计划。[to-spec](https://aihero.dev/skills-to-spec) 将这些已链接决策折叠成一份规范——`/to-spec #<map_issue>`——而 [to-tickets](https://aihero.dev/skills-to-tickets) 将其切片为[示踪弹](https://www.aihero.dev/ai-coding-dictionary/tracer-bullet)实现任务。直接从地图循环到 [implement](https://aihero.dev/skills-implement) 会跳过折叠步骤，扔掉已链接的细节。只有在工作确实很小的情况下才直接进入实现。有人运行精简管线并报告有效；额外的那两步买到的是一份明确的规范产物，审阅者或同事可以阅读，你越不是单人作战这越重要。

**我的 agent 在 wayfinder 会话中间开始编写生产代码了。**

这是该技能被报告最多的失败模式，背后有一个真正的漏洞。Wayfinder 的"只规划不做实现"默认值可以在地图的 **Notes** 中被覆盖——但 Notes 是 agent 写的，所以约束和豁免都在被约束方拥有的同一个文件里。有用户目睹 agent 在自己的 Notes 中写入"本地图携带执行"，然后在后续会话中把它读回当作自己的许可，在活跃服务器上构建。目前没有硬性的技能内停止机制来处理"我是说默认情况"。在那之前：阅读任何不是你绘制的地图的 Notes，把实现放在它自己的会话中，并将任何看起来像构建切片的 `wayfinder:task` 视为类型错误。

**我绘制了 27 个任务，等做到第十三个时，剩下的已经说不通了。**

这是一个真实的、反复被报告的结果，来自一线报告的原话。Wayfinder 的默认本能是全面规划，而一个后面任务建立在前面任务会推翻的假设之上的地图，正是该技能被指责的瀑布陷阱。有两件事可以对抗它。将地图范围限定在有界目的地，而不是整个产品——实践者一致报告限定在一个明确定义的 epic 内的地图表现比 sprawling 的"实现 V1"更好，而且规划非常大的东西本来就不是目标——交付小增量才是。还有 aggressively 使用 [prototype](https://www.aihero.dev/ai-coding-dictionary/prototyping)：路线之所以能保持实时，正是因为不确定性在实现依赖它之前就被廉价的具体产物冲刷掉了。Wayfinder 是"prototypemaxxing"，不是"planmaxxing"。

**我可以并行处理多个任务吗？**

前沿的构建就是为了让你看到哪些可以领取，而[阻塞边](https://www.aihero.dev/ai-coding-dictionary/blocking-edges)的存在就是为了保证并行工作在纸面上是安全的。实践中一次一个是更安全的默认值。同时做两个 grilling 任务的用户会在一个会话中被问到他刚在另一个会话中回答过的问题，因为这些会话不共享[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。prototype 任务上也有一个已知缺口：有报告称 agent 构建了三个 UI 变体，自己选了一个，然后关闭了任务——选择权是你的，而技能目前没有说得足够响亮。如果你确实要并行运行，先自己审查依赖图。

**我必须用 GitHub Issues 吗？**

不一定——任何任务跟踪器都可以。GitHub 是支持最好的路径，因为它原生的子 issue 和阻塞关系让前沿不需要打开地图就能可见；GitLab、Linear、Jira 和本地 markdown 也都有人用。两个坦诚的局限。没有原生阻塞功能的跟踪器意味着依赖图从文本中推断，需要手动校正。而本地 markdown 把产物放在仓库里，这不推荐：把这些材料存到仓库中往往导致意外的持久化。开源维护者遇到的是相反的问题——公共跟踪器里塞满了 agent 生成的规划任务——但他们也倾向于选择本地 markdown。

**Grilling 太累人了。每个问题都有三段长。**

这是对 wayfinder 最尖锐的实时投诉，而且尚未解决。一位用户的分解：冗长本身导致决策疲劳，而且长度剥离了*为什么*要问这个问题，所以随着地图变长，你会丢失从决策到决策的链条。这种冗长看起来像是当前 [模型](https://www.aihero.dev/ai-coding-dictionary/model) 集合的属性，而非技能的属性，目前还没有修复落地。实践中流传的缓解措施：使用较低的[推理努力](https://www.aihero.dev/ai-coding-dictionary/effort)，并在全局 `CLAUDE.md` 中放入一条直白的指令。无论如何，准备好在这里投入真正的思考——wayfinder 要求你做的思考量不是缺陷，而是它存在的大部分意义。

**一个我已经关闭的决策后来发现是错的。我应该编辑旧任务还是创建新任务？**

没有官方指导，而 agent 的本能帮不上忙：它倾向于围绕坏决策做设计而不是挑战它，所以你必须手动引导。有效做法是明确告诉 wayfinder 发生了什么变化——它会更新地图、修改受影响的任务、在已关闭的任务上添加评论。地图中间的 scope 变化是可以恢复的。但你*设计*成会变的地图就是一个范围气味。

**`decision-mapping` 去哪了？**

就是这个技能，在 v1.1 中重命名为 `wayfinder`，以 `/wayfinder` 调用。"Decision map" 是行话，而且也不准确，因为四种任务类型中只有一种真正是决策。这次重构给了技能一个统一的词汇表——目的地、战争迷雾、前沿、地图——而不是一个叠上去的自创术语。但单元保留了"decision"这个词：**决策任务**就是 wayfinder 任务的名称，正是为了防止人们把它当作实现任务来读。

## 工作正常的表现

- 在创建任何任务之前，目的地就已经写下并达成共识。
- 每个开放的任务读起来都是一个问句。任何读起来像"构建 X"的任务要么是类型错误，要么属于地图的下游。
- 你可以看着跟踪器就知道哪些任务可以领取，不需要打开地图——这就是前沿通过原生阻塞自行呈现。
- 一个会话解决一个任务，将答案作为解决评论发布，关闭任务，然后在地图的*已有决策*上留下一行。然后停止。
- **尚未明确**随时间缩小。一块升级为任务的迷雾会从该部分消失，而不是同时存在于两个地方。
- 当初始的广度优先 grill 没有发现任何迷雾时，技能会停止并告诉你这个工作足够小，可以跳过地图。
- 完成地图的会话将你导向一份规范，而不是一个 pull request。

## 它的位置

`wayfinder` 是一个**情境性入口匝道**，不是默认的前门。grill 主导的构思 → 交付链条仍然是大多数工作开始的地方；wayfinder 是当构想太大、一次会话装不下时你爬上去的东西，它在 [to-spec](https://aihero.dev/skills-to-spec) 处合并回那条链条，因为清空后的地图是交棒而非构建。

在底层，它大多是其他技能戴着 wayfinder 的调度帽子在运作：[grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 解决默认的任务类型，[prototype](https://aihero.dev/skills-prototype) 解决讨论无法解决的任务，[research](https://aihero.dev/skills-research) 作为子agent 运行，所以它的阅读内容不会落到你的会话里。[handoff](https://aihero.dev/skills-handoff) 是进出的桥梁——从一个膨胀过大的对话进入地图，从一个会话中途出现的支线任务离开地图。其他任何事情，[ask-matt](https://aihero.dev/skills-ask-matt) 在整个集合上做路由。
