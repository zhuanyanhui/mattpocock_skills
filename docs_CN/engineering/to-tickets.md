Quickstart:

```bash
npx skills add mattpocock/skills --skill=to-tickets
```

```bash
npx skills update to-tickets
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-tickets)

## 它做什么

`to-tickets` 接收一个计划、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 或你正在进行的对话，并将其拆分为一组你的问题跟踪器上的 **[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)**。每个工单声明其**阻塞边**——必须在其完成之后才能开始的其他工单。

每个工单都是一颗**示踪弹**：贯穿变更每一层的狭窄但完整的路径——schema、API、UI、测试——在落地的瞬间就可以独立演示。这就是让它与拆分工作的显而易见的方式表现不同的约束：后者是一次切一层，在最后集成。它还将每个工单的大小调整为适合单个新鲜[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)，因为将领取工单的是一个从未见过你的 spec 的[会话](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何时使用

你通过输入 `/to-tickets` 来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会主动调用它。

| 你在哪里 | 运行什么 |
| --- | --- |
| 你有一个 spec 问题，且构建跨多个会话 | `/to-tickets`，或 `/to-tickets #<spec_issue>` |
| 计划只在对话中，从未写过 | `/to-tickets` 直接读取线程——不需要 spec |
| 整个变更适合一个上下文窗口 | [implement](https://aihero.dev/skills-implement)——跳过工单 |
| 还没有做任何决定 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，然后 [to-spec](https://aihero.dev/skills-to-spec) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder) 地图已清晰 | 先用 [to-spec](https://aihero.dev/skills-to-spec) 折叠地图，然后 `/to-tickets` |

`to-tickets` 产出的工单天然是 agent-ready 的。不要在它们上面运行 [triage](https://aihero.dev/skills-triage)——triage 是用于来自其他人的工作。

## 前置条件

`to-tickets` 发布到跟踪器，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须已为此仓库配置了一个跟踪器，以及分类标签词汇。哪种都行：像 GitHub 或 Linear 这样的真实跟踪器，或者 `.scratch/` 下的本地 markdown 文件，开箱即用地支持。

## 示踪弹，而非层

**水平**切片交付变更的一层。在每一层都落地之前没有任何东西工作，而且每个工单的验收标准都必须延伸到另一个工单所有的工作。**垂直**切片——示踪弹——一次交付一条穿过所有层的薄路径，所以它可以独立验证，并且拥有它所触及的一切。

这是人们最常打破的规则，后果有据可查。一个团队运行了一个 26 工单的按层切分的堆栈——语料库、生产者、聚合器、选择器——每个关闭的工单大约有二十次 agent 运行，其中约四分之三是返工。他们自己的事后分析将每个失败类都追溯回水平切片，而非实现。

在发布任何内容之前发生两件事。`to-tickets` 寻找预重构——"先让变更变得容易，再做出容易的变更"——并将该工作排在最前面。然后它把拆分方案作为编号列表呈现出来，并就它向你提问：粒度是否合适，阻塞边是否真实，是否有什么应该合并或拆分。在你批准之前没有任何东西到达跟踪器，而那个提问正是提出异议的地方。

## 阻塞边

边是这个产物的要点。它们有两种读取方式，取决于跟踪器：

| 跟踪器 | 边在哪里 | 如何使用它们 |
| --- | --- | --- |
| 本地 markdown | `.scratch/<feature>/issues/<NN>-<slug>.md` 下每个工单一个文件中的文本，按阻塞项优先编号 | 从上到下，手动 |
| 真实跟踪器（GitHub、Linear） | 原生阻塞链接，或者跟踪器有子工单的地方 | 任何阻塞项都完成的工单处于**前沿**，可以被领取 |

无论哪种方式，边都存在于工单中。媒介只决定是否有任何东西可以并行处理它们。`to-tickets` 产出产物；运行它——一次一个会话，或一个舰队——是你的工作，不是技能的。

## 大规模重构例外

有一种形态打破了示踪弹规则。**大规模重构**是一项机械性变更——重命名列、更改共享符号的类型——其**影响范围**遍布整个代码库，因此一次编辑会破坏数千个调用点，没有任何垂直切片能够保持绿色。

`to-tickets` 将其编排为**展开-收缩**：

- **展开**——在旧形式旁边添加新形式，这样不会破坏任何内容。
- **迁移**——按影响范围大小分批移动调用点（按包、按目录），每批一个工单，每个都由展开阻塞。由于旧形式仍然存在，CI 保持绿色。
- **收缩**——在没有调用者后删除旧形式，作为一个由所有迁移批次阻塞的工单。

即使分批后各批次无法独立保持绿色，它们也共享一个集成分支，并且都阻塞一个最终的集成与验证工单。绿色仅在那里承诺。

## 常见问题

**它为一个三行变更生成了十二个工单。**

过度分解是这个技能报告最多的摩擦，而且跨实践者是一致的：[model](https://www.aihero.dev/ai-coding-dictionary/model) 默认为原子单位，失去了使它们有意义的分组。提问步骤正是为此存在的——要求它合并，它就会。更深层的答案是工单有底线：如果整个变更适合一个上下文窗口，你根本不需要这个技能。直接去 [implement](https://aihero.dev/skills-implement)。

**工单是按层出来的——所有 schema 在一个，所有 API 在另一个。**

这是垂直切片规则写来防止的失败，而且这个技能有时仍然会产生它。在提问步骤通过每个工单问一个问题来捕捉它：完成后我能演示什么？没有答案的工单就是水平切片。出于这个原因，一些人在每个工单中添加"演示路径"行，并报告这会推动模型走向垂直分解。

**在 GitHub 上，工单没有被创建为 spec 问题的子问题。**

已知且未修复。这已在十几次运行和多个模型中被报告，[最完整的是 issue #554](https://github.com/mattpocock/skills/issues/554)，在 Codex 上比在 Claude 上更糟。`gh` 从 v2.94 起原生支持这个：`gh issue create --parent <n>`，以及事后 `gh issue edit <parent> --add-sub-issue <n>`。在跟踪器模板偏好这些之前，在运行后自己连接父链接是可靠的做法。

**"被阻塞于"被写入了问题正文而不是真正的阻塞链接。**

同一类问题，[在 issue #513 中报告](https://github.com/mattpocock/skills/issues/513)，agent 甚至断言 GitHub 没有原生阻塞关系。它有——`gh issue create --blocked-by 12,15`。因为阻塞项首先发布，它们的编号在创建时始终可用。正文文本意在作为没有原生边的跟踪器的回退，而不是默认。

**本地工单去哪了？v1.1 笔记说根级的 `tickets.md`。**

确实如此，那是一个 bug——单个共享文件在并行 agent 写入时也会竞争。本地模式现在在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下为每个工单写入一个文件，按依赖顺序，与本地跟踪器模板已经描述的布局匹配。`NN` 前缀是真正的工单 ID，所以 `/implement 03` 可以工作，而不需要重新输入长标题。

**它在尝试读取我的 spec 时一直在截断。**

非常大的 spec 可能会超出跟踪器问题能干净返回的内容，而且没有本地副本可以回退——agent 然后浪费 [tool call](https://www.aihero.dev/ai-coding-dictionary/tool-call) 重新获取块，永远到不了末尾。不要在 `/to-spec` 和 `/to-tickets` 之间[清理](https://www.aihero.dev/ai-coding-dictionary/clearing)或[压缩](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个上下文窗口中运行它们，spec 就根本不需要被重新获取。

**验收标准什么都没判定——有些在任何工作开始之前就通过了。**

模板要求标准，但没说它们是否能失败，所以这会发生。三种形态反复出现：在基础提交时已经为真的标准、只能由另一个工单所有的工作满足的标准，以及重述请求而非从工件导出的标准。垂直切片防止了大部分——一个交付以前不存在的行为的切片在基础提交时天然是红的——但手动检查是值得的。对于每个标准，命名会显示它为假的观察，并确认它在实现者开始的提交时失败。

**工单已发布。我如何实际运行它们？**

这个技能停在产物处，没有自动分发模式。分发是手动的：看板，计数没有开放阻塞项的工单，然后打开那么多 agent 会话。每个新鲜上下文一个工单，在之间清理。注意 [implement](https://aihero.dev/skills-implement) 在 GitHub 或本地 markdown 中完成时不能可靠地关闭或勾选工单，所以工单的状态由你来更新。

## 它生效的标志

- 每个工单都有"完成后我能演示什么？"的答案——而且答案是行为，而不是一层。
- 列表以编号形式返回给你，每个都有"被阻塞于"行，在任何内容发布之前。
- 最上面的工单没有阻塞项，可以立即开始。
- 工单正文中没有文件路径或行号，除非是原型产生的代码片段。
- 每个工单读起来像一个新鲜会话可以在你不在场的情况下完成的东西。
- 预重构（如果找到的话）排在顺序的前面，而不是混入功能工单。

## 在体系中的位置

`to-tickets` 是主构建链中的一个步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

上游是 [to-spec](https://aihero.dev/skills-to-spec)，它交给你一份已确定的 spec 来切片——在同一个不间断的上下文窗口中保留两者。下游是 [implement](https://aihero.dev/skills-implement)，它在每个新鲜会话中构建一个工单，驱动 [tdd](https://aihero.dev/skills-tdd) 进行测试，并以 [code-review](https://aihero.dev/skills-code-review) 结束。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
