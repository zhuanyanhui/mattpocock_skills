# Issue 跟踪器：本地 Markdown

本仓库的 Issue 和规范（你可能将规范称为 PRD）作为 `.scratch/` 中的 markdown 文件存在。

## 约定

- 每个功能一个目录：`.scratch/<feature-slug>/`
- 规范文件为 `.scratch/<feature-slug>/spec.md`
- 实现 issue 为每个工单一个文件，位于 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，从 `01` 开始编号——永远不要使用单个合并的工单文件
- 分诊状态记录在每个 issue 文件顶部的 `Status:` 行（参见 `triage-labels.md` 了解角色字符串）
- 评论和对话历史在文件底部以 `## Comments` 标题追加

## 当技能说"发布到 issue 跟踪器"时

在 `.scratch/<feature-slug>/` 下创建新文件（如需要则创建目录）。

## 当技能说"获取相关工单"时

读取指定路径的文件。用户通常会直接传递路径或 issue 编号。

## 寻路操作

由 `/wayfinder` 使用。**地图**是一个文件，每个**子**文件对应一个工单。

- **地图**：`.scratch/<effort>/map.md`——Notes / Decisions-so-far / Fog 正文。
- **子工单**：`.scratch/<effort>/issues/NN-<slug>.md`，从 `01` 开始编号，正文中包含问题。`Type:` 行记录工单类型（`research`/`prototype`/`grilling`/`task`）；`Status:` 行记录 `claimed`/`resolved`。
- **阻塞**：靠近顶部的 `Blocked by: NN, NN` 行。当它列出的每个文件都处于 `resolved` 状态时，工单解除阻塞。
- **前沿**：扫描 `.scratch/<effort>/issues/` 中开放、未阻塞、未认领的文件；按编号第一个胜出。
- **认领**：在进行任何工作之前设置 `Status: claimed` 并保存。
- **解决**：在 `## Answer` 标题下追加答案，设置 `Status: resolved`，然后将上下文指针（gist + 链接）追加到 `map.md` 中地图的 Decisions-so-far。
