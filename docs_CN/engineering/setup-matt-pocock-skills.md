# setup-matt-pocock-skills

## 它做什么

`setup-matt-pocock-skills` 针对一个仓库回答三个问题——任务存放在哪里、[triage](https://aihero.dev/skills-triage) 标签叫什么、领域文档放在哪里——并将答案记录为 `docs/agents/` 下的 markdown 文件。

这些文件是仓库之间唯一不同的东西。skill 本身在所有仓库中都是相同的；它们在运行时读取 `docs/agents/issue-tracker.md` 并按照其中的指示执行。这就是为什么这套 skill 不绑定于 GitHub，以及为什么没有任何 skill 文件需要编辑来指向其他地方。用"将 skill 链接到自定义任务跟踪器"来调用它，可以配合任何你能以编程方式连接的东西，无需对 skill 做任何修改。

它是一个 prompt 驱动的 skill，而不是确定性脚本。它会读取你的 `git remote`、现有的 `CLAUDE.md`、现有的 `CONTEXT.md`，提出它找到的内容，并在写入任何内容之前等待你确认。

## 什么时候使用它

通过输入 `/setup-matt-pocock-skills` 来调用——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己主动调用它。它被刻意标记为不可调用，所以没有其他 skill 能替你触发它。

在每个仓库中、在任何其他 engineering skill 首次使用之前运行一次。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 开始猜测你的任务存放在哪里，或者应用了你的跟踪器不存在的标签，说明还没有在这里完成设置。项目进行到一半的仓库也完全可以运行它；skill 会读取已有的内容，之前的工作不会白费。

## 前置条件

它会写入运行它所在的仓库：

| 它写入的内容 | 位置 |
| --- | --- |
| `issue-tracker.md` | `docs/agents/` |
| `domain.md` | `docs/agents/` |
| `triage-labels.md` | `docs/agents/`，仅在安装了 `triage` skill 时 |
| 一个 `## Agent skills` 区块 | 已存在的 `CLAUDE.md` / `AGENTS.md` 中的某一个 |

所有写入的内容都是已 commit 的 markdown。没有用户级或全局模式：配置存在于仓库中，所以每个仓库都有自己的副本。

## 三个决策

它在每个部分以推荐答案开头，并跳过已经确定的探索内容。大多数运行只需两次确认即可完成。

| 决策 | 它提议的内容 | 什么时候真正询问 |
| --- | --- | --- |
| **任务跟踪器** | 匹配你的 `git remote` 的那个 | 始终询问——这是唯一一个真正的选择 |
| **Triage 标签** | 保留五个规范名称（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`） | 仅在安装了 `triage` skill 时 |
| **领域文档** | 单上下文：根目录下一个 `CONTEXT.md` 加上 `docs/adr/` | 仅在检测到 monorepo 信号时，此时它会提供多上下文的 `CONTEXT-MAP.md` |

跟踪器选项：

| 选项 | 任务存放位置 | 需要 |
| --- | --- | --- |
| **GitHub** | 仓库的 GitHub Issues | `gh` CLI |
| **GitLab** | 仓库的 GitLab Issues | `glab` CLI |
| **本地 markdown** | 本仓库 `.scratch/<feature>/` 下的文件 | 无——完全不需要远程 |
| **其他** | 你指定的任何地方 | 由你用一段话描述工作流 |

前三个作为模板内置于 skill 中，开箱即用。本地 markdown 是一等选项，不是备选：没有远程的单人项目完全受支持。有一点值得重复强调：如果你在使用 GitHub，就不要使用本地 markdown。它们是替代关系，不是叠加关系。

"其他"也不是一个空壳。这正是 Jira、Linear、Azure DevOps 和 Beads 都能工作的原因：你描述工作流，skill 将你的文本记录到 `docs/agents/issue-tracker.md` 中，下游 skill 遵循这段文本来执行。社区已经有人做到了——一个基于 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 的 Jira 变体、一个模仿 `gh` 形状的 Gitea CLI、一个手工构建的本地仪表板。

## 常见问题

**我必须使用 GitHub 吗？**

不必。GitHub、GitLab 和 `.scratch/` 下的本地 markdown 都作为现成模板提供，其他任何东西都可以通过"其他"路径使用。这是记录中被问得最多的问题，大致是这些说法：*"硬绑定到 github"*、*"我能用 GitLab / Jira 吗"*、*"Azure DevOps 呢"*。每次的回答都是：跟踪器是一个设置答案，不是 skill 属性。

**更新 skill 后需要重新运行它吗？**

在 v1.1 之后被直接问到过，Matt 的回答是肯定的。skill 自己的结束信息更温和——它告诉你只有在切换跟踪器或重新开始时才需要重新运行。两种说法都有道理，差距的原因是真实的：种子模板在各版本之间会变化，所以由旧版本写入的 `docs/agents/issue-tracker.md` 可能会与现在读取它的 skill 不一致。如果下游 skill 开始做一些与文档描述不同的事情，重新运行是成本最低的修复方式。

**它写入了 `CLAUDE.md`，但我用的是 Codex。**

这是一个已知的 gap，仍然开放。文件选择规则是"如果 `CLAUDE.md` 存在就编辑它，否则编辑 `AGENTS.md`"——它检查哪个文件存在，而不是哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 在运行。一个从 Claude Code 遗留了 `CLAUDE.md` 的仓库，它的 `## Agent skills` 区块会出现在 Codex 永远不读取的地方。有两种变通方法流传：手动将区块移到 `AGENTS.md`，或者让 `AGENTS.md` 作为规范文件，让 `CLAUDE.md` 变成指向它的一行引用。如果两个文件都不存在，skill 会询问你创建哪一个而不是自行决定，这让那些以为它应该直接做决定的人感到困惑。

**它没有创建我的 triage 标签。**

它不会创建。`docs/agents/triage-labels.md` 是一个*映射*——它告诉 `/triage` 你的跟踪器中的哪些字符串对应五个规范角色。它不会运行 `gh label create`。在一个全新的 GitHub 仓库中，标签确实还不存在，这已经被作为 bug 提交过不止一次。两点补充：

- 如果你的跟踪器已经使用了规范名称，映射就是一个恒等表，没有什么需要配置的。这是预期的常见情况，不是遗漏的步骤。
- [wayfinder](https://aihero.dev/skills-wayfinder) 的 `wayfinder:map` 和 `wayfinder:<type>` 标签也不会在这里创建，而且 `gh issue create --label <missing>` 在标签不存在时会直接失败而不是创建标签。在 GitHub 仓库上首次运行 wayfinder 之前手动创建它们。

**我能在这里配置其他 skill 的行为吗——[grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 频率、问题格式、语气？**

不能。它只配置三样东西：跟踪器、标签、文档布局。曾有人直接请求让它成为每个用户偏好的配置中心，而一贯的回答是 skill 保持有主见：*"配置即死。"* 偏好应该作为普通指令放在你的 `CLAUDE.md` 中，每个 skill 都会读取它。

**我能把配置放在 `~/.claude` 而不是 commit 到每个仓库吗？**

目前不行。有一个来自跨多个仓库运行 skill 的人提出的开放请求正是这个需求，目前不存在用户级模式。每个仓库都携带自己的 `docs/agents/`。

**一个配置其他 skill 的 skill 会不会很奇怪？**

一个长期存在的抱怨说是的，原话大致是：*"有一个 skill 来设置其他 skill 让我觉得不太对——这意味着 LLM 在配置它自己的 skill。"* 这个权衡是真实存在且被承认的：替代设置步骤的方案是将跟踪器指令复制到每个涉及任务的 skill 中。输出是可检查、可编辑的 markdown，这就是缓解措施——你可以阅读它写入的每个文件并手动修改，日常调整就是这样做的，而不是再运行一次。

## 正常工作的标志

- `docs/agents/issue-tracker.md` 和 `docs/agents/domain.md` 存在，如果安装了 `triage` 则还有 `triage-labels.md`。
- 你的 harness 实际读取的指令文件中出现 `## Agent skills` 章节，其中有一行摘要指向这些文件中的每一个。
- 它提议的跟踪器与你实际使用的远程仓库匹配，标签字符串与你跟踪器中真实存在的标签匹配。
- 之后，`/to-tickets` 发布时不再询问你任务存放在哪里，`/triage` 应用标签而不是凭空发明。
- skill 文件本身没有任何改动。如果 setup 编辑了某个 `SKILL.md`，说明出了问题。

## 它在整体中的位置

`setup-matt-pocock-skills` 是 engineering flow 的**一次性设置**，是所有其他内容所依赖的前提条件，而不是链路中的一个步骤。它的邻居就是它的读者：[triage](https://aihero.dev/skills-triage)，应用这里写入的标签词汇；[to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets)，发布到此处命名的跟踪器；以及 [wayfinder](https://aihero.dev/skills-wayfinder)，它读取同一跟踪器文件中的"Wayfinding operations"部分来了解地图和子 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 如何存储。它记录的领域文档布局是 [domain-modeling](https://aihero.dev/skills-domain-modeling) 后续填充的——它懒惰地创建 `CONTEXT.md` 和 ADR，仅在某个术语或决策真正被确定时才创建，所以设置后仓库处于空状态是预期的。关于下一步该使用哪个 skill，[ask-matt](https://aihero.dev/skills-ask-matt) 负责路由整个集合。
