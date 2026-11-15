---
name: setup-matt-pocock-skills
description: 为此仓库配置工程技能——设置问题跟踪器、分类标签词汇表和领域文档布局。在其他工程技能首次使用前运行一次。
disable-model-invocation: true
---

# 设置 Matt Pocock 的技能

搭建工程技能所需的每个仓库的配置：

- **问题跟踪器（Issue tracker）**——问题存放的位置（默认为 GitHub；本地 markdown 也开箱即用）
- **分类标签（Triage labels）**——五个标准分类角色使用的标签字符串
- **领域文档（Domain docs）**——`CONTEXT.md` 和 ADR 存放的位置，以及读取它们的消费规则

这是一个提示驱动的 skill，而不是确定性的脚本。探索、展示发现、与用户确认、然后写入。

## 流程

### 1. 探索

查看当前仓库以了解其起始状态。读取存在的内容；不要做假设：

- `git remote -v` 和 `.git/config`——这是 GitHub 仓库吗？是哪个？
- 仓库根目录的 `AGENTS.md` 和 `CLAUDE.md`——它们存在吗？其中是否已有 `## Agent skills` 章节？
- 仓库根目录的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
- `docs/adr/` 和任何 `src/*/docs/adr/` 目录
- `docs/agents/`——此 skill 先前的输出是否已存在？
- `.scratch/`——表明本地 markdown 问题跟踪器约定已在使用的标志
- 是否安装了 `triage` skill？（此 skill 文件夹旁边的 `triage` skill 文件夹，或可用技能列表中的 `triage`。）这决定是否运行 B 部分。注意`triage`可能被配置为不自动触发，检测时需同时检查符号链接形式的技能，避免遗漏。
- Monorepo 信号——`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或包含自己 `src/` 的 `packages/*`。仅在真正的大型多包仓库中存在；它们的缺失意味着单上下文（single-context），这适用于几乎所有仓库。

### 2. 展示发现并询问

总结存在什么以及缺少什么。然后按顺序处理各部分——一个部分，一个回答，然后下一个。

每个部分以推荐答案开头，以便用户可以一句话接受。仅在选择确实有分支时才提供一行解释；当探索已经解决时完全跳过该部分（当 `triage` 未安装时跳过 B 部分，当没有 monorepo 时跳过 C 部分）。

**A 部分 — 问题跟踪器。**

> 说明："问题跟踪器"是此仓库的问题存放位置。像 `to-tickets`、`triage`、`to-spec` 和 `qa` 这样的技能会读写它——它们需要知道是调用 `gh issue create`、在 `.scratch/` 下写入 markdown 文件，还是遵循你描述的其他工作流。选择你实际为此仓库跟踪工作的位置。

默认姿态：这些技能是为 GitHub 设计的。如果 `git remote` 指向 GitHub，建议使用 GitHub。如果 `git remote` 指向 GitLab（`gitlab.com` 或自托管主机），建议使用 GitLab。否则（或如果用户偏好），提供：

- **GitHub**——问题存在于仓库的 GitHub Issues 中（使用 `gh` CLI）
- **GitLab**——问题存在于仓库的 GitLab Issues 中（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **本地 Markdown**——问题作为文件存在于仓库的 `.scratch/<feature>/` 目录下（适用于个人项目或无远程仓库）
- **其他**（Jira、Linear 等）——让用户用一段话描述工作流；skill 会将其记录为自由格式的文字

将选择记录在 `docs/agents/issue-tracker.md` 中。GitHub 和 GitLab 模板带有"PRs as a request surface"标志，默认**关闭**——保持关闭且不要提及；希望将外部 PR 纳入分类队列的用户稍后可以在文件中翻转该标志。

**B 部分 — 分类标签词汇表。** 如果 `triage` skill 未安装（探索已告知你），则完全跳过此部分——未安装的 skill 不需要标签。

如果已安装，只问一个问题：

> 您想保留默认的分类标签吗？（推荐：**是**）

默认值是五个标准角色，每个标签字符串等于其名称：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。回答**是**，按原样写入。只有当用户说"否"时——通常是因为他们的跟踪器已经使用了其他名称（例如 `bug:triage` 代替 `needs-triage`）——收集覆盖值，以便 `triage` 应用现有标签而不是创建重复标签。

**C 部分 — 领域文档。** 默认为**单上下文（single-context）**——仓库根目录下的一个 `CONTEXT.md` + `docs/adr/`。这适用于几乎所有仓库；无需询问直接写入。

仅当探索发现 monorepo 信号时，才提供**多上下文（multi-context）**——根目录的 `CONTEXT-MAP.md` 指向每个上下文的 `CONTEXT.md` 文件。然后确认他们想要哪种布局。

### 3. 确认并编辑

向用户展示以下草稿：

- 要添加到 `CLAUDE.md` / `AGENTS.md`（两者中正在编辑的那个）中的 `## Agent skills` 块（选择规则参见第 4 步）
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后一个仅在 `triage` 已安装时）

在写入前让他们编辑。

### 4. 写入

**选择要编辑的文件：**

- 如果 `CLAUDE.md` 存在，编辑它。
- 否则如果 `AGENTS.md` 存在，编辑它。
- 如果两者都不存在，询问用户要创建哪一个——不要替他们选择。

当 `CLAUDE.md` 已经存在时，永远不要创建 `AGENTS.md`（反之亦然）——总是编辑已经存在的那个。

如果所选文件中已存在 `## Agent skills` 块，原地更新其内容，而不是附加一个重复块。不要覆盖用户对周围章节的编辑。

该块：

```markdown
## Agent skills

### Issue tracker

[问题跟踪位置的一行摘要]。参见 `docs/agents/issue-tracker.md`。

### Triage labels

[标签词汇表的一行摘要]。参见 `docs/agents/triage-labels.md`。

### Domain docs

[布局的一行摘要——"single-context" 或 "multi-context"]。参见 `docs/agents/domain.md`。
```

仅当 `triage` 已安装并且 B 部分已运行时，才包含 `### Triage labels` 子块并写入 `docs/agents/triage-labels.md`。否则，两者都省略。

然后使用此 skill 文件夹中的种子模板作为起点编写文档文件：

- [issue-tracker-github.md](./issue-tracker-github.md) —— GitHub 问题跟踪器
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md) —— GitLab 问题跟踪器
- [issue-tracker-local.md](./issue-tracker-local.md) —— 本地 markdown 问题跟踪器
- [triage-labels.md](./triage-labels.md) —— 标签映射（仅在 `triage` 已安装时）
- [domain.md](./domain.md) —— 领域文档消费规则 + 布局

对于"其他"问题跟踪器，使用用户的描述从头编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告知用户设置已完成，以及哪些工程技能现在将从这些文件中读取。提一下他们以后可以直接编辑 `docs/agents/*.md` —— 只有在想要切换问题跟踪器或从头开始时才需要重新运行此 skill。
