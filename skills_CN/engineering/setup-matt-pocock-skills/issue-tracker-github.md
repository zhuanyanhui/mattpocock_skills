# Issue 跟踪器：GitHub

本仓库的 Issue 和 PRD 作为 GitHub issues 存在。所有操作均使用 `gh` CLI。

## 约定

- **创建 issue**：`gh issue create --title "..." --body "..."`。多行内容使用 heredoc。
- **阅读 issue**：`gh issue view <number> --comments`，通过 `jq` 过滤评论并同时获取标签。
- **列出 issues**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，配合适当的 `--label` 和 `--state` 过滤器。
- **评论 issue**：`gh issue comment <number> --body "..."`
- **应用/移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：`gh issue close <number> --comment "..."`

从 `git remote -v` 推断仓库——`gh` 在克隆内运行时会自动处理。

## Pull Request 作为分诊面

**PR 作为请求面：no。** _（如果本仓库将外部 PR 视为功能请求，则设为 `yes`；`/triage` 会读取此标志。）_

设为 `yes` 时，PR 与 issues 走相同的标签和状态流程，使用 `gh pr` 等价命令：

- **阅读 PR**：`gh pr view <number> --comments` 以及 `gh pr diff <number>` 获取 diff。
- **列出外部 PR 进行分诊**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，然后仅保留 `authorAssociation` 为 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的 PR（剔除 `OWNER`/`MEMBER`/`COLLABORATOR`）。
- **评论/标签/关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 在 issues 和 PR 之间共享同一个编号空间，所以 bare `#42` 可能是其中任一——用 `gh pr view 42` 解析，失败则回退到 `gh issue view 42`。

## 当技能说"发布到 issue 跟踪器"时

创建一个 GitHub issue。

## 当技能说"获取相关工单"时

运行 `gh issue view <number> --comments`。

## 寻路操作

由 `/wayfinder` 使用。**地图**是一个单独的 issue，**子**issues 作为工单。

- **地图**：一个标记为 `wayfinder:map` 的 issue，包含 Notes / Decisions-so-far / Fog 正文。`gh issue create --label wayfinder:map`。
- **子工单**：作为 GitHub 子 issue 链接到地图的 issue（在子 issue 端点上使用 `gh api`）。如果子 issue 未启用，则将子工单添加到地图正文的任务列表中，并在子工单正文顶部放置 `Part of #<map>`。标签：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被认领，工单将分配给负责的开发者。
- **阻塞**：GitHub **原生 issue 依赖**——规范的、UI 可见的表示。添加边使用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`，其中 `<blocker-db-id>` 是阻塞者的数字**数据库 id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，_不是_ `#number` 或 `node_id`）。GitHub 报告 `issue_dependencies_summary.blocked_by`（仅开放阻塞者——实时门控）。如果依赖不可用，回退到在子工单正文顶部添加 `Blocked by: #<n>, #<n>` 行。当每个阻塞者都被关闭时，工单解除阻塞。
- **前沿查询**：列出地图的开放子工单（`gh issue list --state open`，限定到地图的子 issue / 任务列表），剔除任何有开放阻塞者的（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中有开放 issue）或有分配者的；按地图顺序第一个胜出。
- **认领**：`gh issue edit <n> --add-assignee @me`——会话的第一次写入。
- **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，然后将上下文指针（gist + 链接）追加到地图的 Decisions-so-far。
