---
name: claude-handoff
description: 将当前对话移交给一个新的后台 agent，让它可以立即接手工作。
argument-hint: "下一个会话将用于什么？"
disable-model-invocation: true
---

撰写当前对话的交接摘要，以便新 agent 能够继续工作。不要将其保存，而是用摘要作为提示词启动一个后台 agent：`claude --bg --name "<描述性名称>" "<交接摘要>"`。它将在当前工作目录中启动并立即返回；用户通过 `claude agents` 管理它。

始终使用 `-n`/`--name` 传入描述性名称（例如 `--name "修复登录 bug"`）——它将在任务列表、会话选择器和终端标题中显示此名称。

在摘要中包含一个"建议技能"（suggested skills）部分，推荐 agent 应调用的技能。

不要重复已记录在其他工件（PRD、计划、ADR、议题、提交、差异）中的内容。通过路径或 URL 引用它们。

隐去任何敏感信息，如 API 密钥、密码或个人身份信息——摘要将成为 agent 的提示词。

如果用户传入了参数，将其视为下一个会话将专注于什么的描述，并相应调整摘要内容。
