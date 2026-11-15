# Issue 跟踪器：GitLab

本仓库的 Issue 和 PRD 作为 GitLab issues 存在。所有操作均使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI。

## 约定

- **创建 issue**：`glab issue create --title "..." --description "..."`。多行描述使用 heredoc。传递 `--description -` 打开编辑器。
- **阅读 issue**：`glab issue view <number> --comments`。使用 `-F json` 获取机器可读输出。
- **列出 issues**：`glab issue list -F json`，配合适当的 `--label` 过滤器。
- **评论 issue**：`glab issue note <number> --message "..."`。GitLab 将评论称为"notes"。
- **应用/移除标签**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多个标签可以用逗号分隔或重复标志。
- **关闭**：`glab issue close <number>`。`glab issue close` 不接受关闭评论，所以先用 `glab issue note <number> --message "..."` 发布解释，然后再关闭。
- **Merge requests**：GitLab 将 PR 称为"merge requests"。使用 `glab mr create`、`glab mr view`、`glab mr note` 等——与 `gh pr ...` 形状相同，用 `mr` 替换 `pr`，用 `note`/`--message` 替换 `comment`/`--body`。

从 `git remote -v` 推断仓库——`glab` 在克隆内运行时会自动处理。

## Merge Request 作为分诊面

**MR 作为请求面：no。** _（如果本仓库将外部 merge requests 视为功能请求，则设为 `yes`；`/triage` 会读取此标志。）_

设为 `yes` 时，MR 与 issues 走相同的标签和状态流程，使用 `glab mr` 等价命令：

- **阅读 MR**：`glab mr view <number> --comments` 以及 `glab mr diff <number>` 获取 diff。
- **列出外部 MR 进行分诊**：`glab mr list -F json`，然后仅保留作者不是项目成员/所有者的 MR（贡献者的 MR，而非维护者进行中的工作）。
- **评论/标签/关闭**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。

与 GitHub 不同，GitLab 对 issues 和 MRs 分别编号，所以只要知道维护者指的是哪个面，`#42` 就是明确的。

## 当技能说"发布到 issue 跟踪器"时

创建一个 GitLab issue。

## 当技能说"获取相关工单"时

运行 `glab issue view <number> --comments`。

## 寻路操作

由 `/wayfinder` 使用。**地图**是一个单独的 issue，**子**issues 作为工单。

- **地图**：一个标记为 `wayfinder:map` 的 issue，包含 Notes / Decisions-so-far / Fog 正文。`glab issue create --label wayfinder:map`。（在具有原生 epic 的 GitLab 层级中，epic 可以持有地图；标记的 issue 适用于所有层级。）
- **子工单**：一个 issue，在其描述顶部带有 `Part of #<map>`，标签为 `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被认领，工单将分配给负责的开发者。
- **阻塞**：GitLab **原生阻塞链接**——规范的、UI 可见的表示。使用 `/blocked_by #<n>` 快速操作添加，作为 note 发布（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生阻塞链接是 Premium/Ultimate 功能；在免费层（或不可用时）回退到在描述顶部添加 `Blocked by: #<n>, #<n>` 行。当每个阻塞者都被关闭时，工单解除阻塞。
- **前沿查询**：`glab issue list -F json` 限定到地图的子工单，剔除任何有开放阻塞者的——指向开放 issue 的原生 `blocked_by` 链接（`glab api projects/:id/issues/:iid/links`），或 `Blocked by` 行中的开放 issue——或有分配者的；按地图顺序第一个胜出。
- **认领**：`glab issue update <n> --assignee @me`——会话的第一次写入。
- **解决**：`glab issue note <n> --message "<answer>"`，然后 `glab issue close <n>`，然后将上下文指针（gist + 链接）追加到地图的 Decisions-so-far。
