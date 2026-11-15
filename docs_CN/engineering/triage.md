## 它做什么

`triage` 处理项目跟踪器上的 issue，将每个 issue 通过一个小型状态机推进——一个分类角色和一个状态角色——最终留下以下三者之一：一份 agent-ready 简报、一个向报告者提出的具体问题，或一个带有记录原因的已关闭 issue。

它仅适用于**你自己未创建**的 issue。原始 bug 报告、新收到的功能请求、未经宣告就到达的外部拉取请求——从外部进入跟踪器的工作，无论报告者以什么形式留下它们。通过 [to-tickets](https://aihero.dev/skills-to-tickets) 生成的 [任务](https://www.aihero.dev/ai-coding-dictionary/ticket) 在构建时就已经是 agent-ready 的，对它们运行 `triage` 至多是浪费精力。规则是明确的：`/triage` 仅用于外来的 issue，不用于你自己创建的 issue。

将它与手动打标签区分开来的第二点是：它会推荐并等待。它会告诉你它的分类和状态判断及推理，以及它在代码库中发现了什么，在你指示之前不会应用任何操作。

## 何时使用

你通过输入 `/triage` 然后用自然语言描述你想要的来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。"给我看看任何需要我关注的东西"、"让我们看看 #42"、"把 #42 移到 ready-for-agent"。

| 你拥有的 | 去哪里 |
| --- | --- |
| 一个充满了其他人原始报告的跟踪器 | `/triage` |
| 一个你自己的模糊构想，没有写下来的东西 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 一个已经确定的对话，需要转化为 [规范](https://www.aihero.dev/ai-coding-dictionary/spec) | [to-spec](https://aihero.dev/skills-to-spec) |
| 一个要拆分为 agent-ready 工单的规范 | [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一个已确认的需要找到根因而非打标签的 bug | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 前置条件

`triage` 会读写你的 issue 跟踪器，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先配置好该跟踪器及其标签词汇。下面的角色名称是**规范术语**；你跟踪器中的标签字符串可能有所不同，映射关系由 setup 提供。如果你的跟踪器已经精确使用了规范名称，就无需映射也无需设置。

跟踪器配置还决定是否将外部拉取请求视为请求表面，以及谁被视为外部。该标志默认关闭，不再是一个设置问题——如果你希望 PR 在范围内，在 `docs/agents/issue-tracker.md` 中翻转它即可。

## 状态机

每个经过分类的条目最终携带恰好一个分类角色和一个状态角色。两个分类：`bug`（某些东西坏了）和 `enhancement`（新功能或改进）。五个状态：

| 状态 | 含义 |
| --- | --- |
| `needs-triage` | 你需要评估它。未打标签的 issue 通常首先落在这里。 |
| `needs-info` | 等待报告者回复。他们回复后回到 `needs-triage`。 |
| `ready-for-agent` | 已完整指定，附有 agent 简报。一个 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agent 可以接手。 |
| `ready-for-human` | 同样的简报，外加为什么不能委托的原因——判断、外部访问、手动测试。 |
| `wontfix` | 已关闭，记录了原因。 |

这就是全部词汇，"恰好一个状态角色"的不变量保证了查询的简洁性。这也是该 [skill](https://www.aihero.dev/ai-coding-dictionary/skill) 中被请求最多的领域：用户曾要求增加第六个状态用于已指定但被其他 issue 阻塞的工作、用于由未来触发器门控的 `deferred` 工作，以及用于终止状态的 `implemented`。这些都没有发布。见下面的问题。

`wontfix` 分为三种情况，区别很重要，因为只有其中一种会写入知识库：

| 关闭原因 | 会发生什么 |
| --- | --- |
| 已经实现 | 一条评论，指向它已经存在的位置。不会向 `.out-of-scope/` 写入任何内容——这是一个已构建的功能，而非被拒绝的功能，把它归档在那里会污染去重检查。 |
| 被拒绝的 bug | 礼貌地解释，然后关闭。 |
| 被拒绝的增强 | 在 `.out-of-scope/` 中创建一个文件，从关闭评论中链接，然后关闭。 |

`.out-of-scope/` 是每个被拒绝的**概念**一个 markdown 文件，而不是每个 issue 一个，写成简短的设计文档而非数据库行：被拒绝了什么、为什么，以及每个请求过它的 issue。`triage` 在评估任何内容之前会读取整个目录，按概念而非关键词匹配——"night theme" 会匹配 `dark-mode.md`。当它命中匹配时，会展示旧的决策并询问你是否仍然持相同看法，而不是从头开始重新审理该请求。

## 先验证，再写简报

在任何 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 之前，`triage` 会检查声明是否实际成立。对于 bug，它会根据报告者的步骤复现它。对于 PR，它会检出分支并运行相关测试。然后它报告以下三种结果之一：已确认，附带代码路径；无法复现；或细节不足无法尝试，这本身就是最强的 `needs-info` 信号。

它在同一次遍历中对代码库运行另外两个检查——**冗余**（这是否已经实现，按领域概念搜索而非报告者的措辞？）和**先前拒绝**（`.out-of-scope/` 是否已经说过不？）。两者代价都很低，并且命中时都会产生 `wontfix`。

所有这些的存在都是为了打磨一个产物：**agent 简报**，即 issue 移动到 `ready-for-agent` 时发布的结构化评论。一旦发布，简报就是合约，原始报告只是上下文。简报的编写是为了**持久**而非精确，因为一个 issue 可能在 `ready-for-agent` 中停留数周，而代码在此期间已经变化。因此它们命名类型、签名和行为合约，从不使用文件路径或行号。已确认的复现所写出的简报远比猜测写出的更可靠。

## PR 是附带代码的 issue

当跟踪器将外部拉取请求视为请求表面时，它们会经过同一台机器——相同的分类、相同的状态、相同的转换。状态只是针对 diff 读取：`ready-for-agent` 意味着已附加简报，agent 应该对代码采取下一步行动；`ready-for-human` 意味着已准备好让人来合并。PR 上的简报描述了对现有 diff 还需要做什么，而不是如何从零开始构建整个东西。

发现表面只列出*外部* PR，因为协作者的飞行中分支不属于分类工作。该过滤器仅用于发现——明确命名一个 PR 它就会被打分类，无论谁写的。一个粗糙的边缘：GitHub 模板的外部 PR 列出命令向 `gh pr list` 请求了一个 `authorAssociation` 字段，而 `gh` 并未暴露该字段，因此该命令按原样直接失败（[#468](https://github.com/mattpocock/skills/issues/468)）。

## 常见问题

**我运行了 `/to-spec` 和 `/to-tickets`，现在那些工单就坐在那里未分类。我需要对它们运行 `/triage` 吗？**
不需要。它们已经是 agent-ready 的——`to-tickets` 在发布时会应用 `ready-for-agent` 标签，正是为了让 AFK runner 无需再次遍历就能拾取它们。遇到这个问题的用户运行了规范流程，看到输出上带有 `needs-triage`，发现他们的 AFK runner 忽略了所有内容。`triage` 是从外部进入的工作的入口匝道；规范流程是你自己发起的工作的通道。它们在 `ready-for-agent` 处汇合，而非在此之前。

**现在有了 `to-spec` → `to-tickets` → `implement` 流程，`triage` 还有意义吗？**
只有当你有入站工作时才有意义。`triage` 早于该主干线，做的是不同的工作：它是其他人提交的报告的通道。如果你跟踪器中的所有内容都来自你自己的规划，你很少会打开它。如果你维护任何公开的项目，或者你的团队向你提交 bug，它就是前门。主要用途是接收来自外部贡献者的 issue 的开源仓库。

**agent 尝试应用 `ready-for-agent`，但 `gh` 说该标签不存在。**
已知的开放 bug（[#616](https://github.com/mattpocock/skills/issues/616)）。`setup-matt-pocock-skills` 将标签词汇写入 `docs/agents/triage-labels.md`，但不会在你的跟踪器中创建这些标签。使用 `gh label create` 或跟踪器的 UI 自己一次性创建这五个状态标签和两个分类标签，之后就没事了。issue 中链接了一个社区修复分支，但尚未合并。

**五个状态不够——阻塞的、延迟的、已实现的怎么办？**
这是该 skill 上被提交最多的缺口，表现为三种形态。一个已完整指定但等待另一个 issue 关闭的问题（[#139](https://github.com/mattpocock/skills/issues/139)）——报告者的抱怨是 `ready-for-agent` 在那里"技术上正确"但具有误导性，所以 agent 拾取后会撞墙。由触发器门控的未来工作，有意但尚不可操作（[#297](https://github.com/mattpocock/skills/issues/297)）。以及一个"已实现，等待验证"的终止状态，没有它的话 AFK runner 会重新排队已完成的工单。Matt 已同意阻塞情况是真实存在的，但对名称尚未决定（`blocked` 还是 `paused`）。这些都没有发布。人们使用的变通方法是在分类旁边使用仓库本地的额外标签，这保持了规范状态槽位被诚实的东西占据，代价是 skill 不知道它。一个社区衍生版本更进一步，添加了 `needs-slicing`、`tracking` 和工作量标签——这可行，但那是他们的，不是 skill 的。

**这和 `/diagnosing-bugs` 有什么区别？**
这里的验证步骤故意保持浅层——足以回答"这是真的吗，大致位于哪里"，而不是找到根因。当 bug 无法根据报告者的步骤在几分钟内复现时，诚实的做法是 `needs-info`，或者如果你现在想追查就用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。两个 skill 的文本目前都没有提到对方；一个用户发现了这个接缝，它仍然是开放的。

**我能把它指向整个待办列表然后让它跑吗？**
你可以这样要求，但请注意它读了什么。"显示需要注意的内容"这一遍是一个廉价的列表，用于*选择*——你挑选一个，然后它收集你所选那个的完整 [上下文](https://www.aihero.dev/ai-coding-dictionary/context)。一次性跨二十个 issue 运行，agent 可能会悄悄回退到以这个廉价列表作为其证据基础，这会返回 issue 正文但不包括评论。一个用户恰好遇到了这种情况：三个 issue 已经带有"已修复，建议关闭"的评论，但三个都得到了新的 agent 简报。如果你想要批量处理，请明确说明每个 issue 必须读取评论。

**它支持 Linear 或 GitHub Issues 以外的工具吗？**
支持——跟踪器是配置，而非硬编码假设，人们通过 `linear` CLI 对它运行 Linear，以及 GitLab 和 `.scratch/` 下的纯 markdown 文件。一个常见的拆分是 Linear 用于 issue 和规划，GitHub 用于代码和 PR：说"issue tracker"的 skill 映射到 Linear，说"PR"的 skill 映射到 GitHub。在本地 markdown 跟踪器上，有一个开放的模板 bug，生成的文件可能将验收标准携带两次，一次在顶层，一次在 agent 简报内（[#200](https://github.com/mattpocock/skills/issues/200)）。

## 它正在工作的标志

- 它接触的每个条目都以恰好一个分类角色和一个状态角色结束——绝不为零，也绝不会出现两个状态冲突。
- 它会给你一个带有推理的建议并停止，而不是重新打标签然后继续。
- bug 被复现了，或 PR 被检出并运行了，然后才到达 `ready-for-agent`。
- 它编写的简报命名了类型和行为，不包含文件路径和行号。
- 一个六个月前被拒绝的请求又回来了，它指出了这一点并引用了旧原因，而不是重新分类。
- 它发布的每条评论都以 `> *This was generated by AI during triage.*` 开头

## 它的位置

`triage` 是一个**入口匝道**，不是主链中的一个步骤。主流程从你自己的想法出发——grill、spec、tickets、implement、review——而 `triage` 是到达的工作的并行通道。它们合并到同一个地方：一个带有简报的标记为 `ready-for-agent` 的 issue，[implement](https://aihero.dev/skills-implement) 会像处理来自 [to-tickets](https://aihero.dev/skills-to-tickets) 的工单一样拾取它。当一个请求在写简报之前需要 sharpen 时，`triage` 会同时运行 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling)，一次一轮问答，这样决策就会落入 `CONTEXT.md` 和 ADR 中。当你不确定自己在哪个通道时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
