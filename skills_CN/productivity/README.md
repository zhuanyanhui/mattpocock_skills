# 生产力

通用工作流工具，非代码特定。

## 用户调用

只有当你输入时才能访问（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

- **[grill-me](./grill-me/SKILL.md)** — 就计划或设计接受无休止的面试，直到决策树的每个分支都得到解决。
- **[handoff](./handoff/SKILL.md)** — 将当前对话压缩成交接文档，以便另一个 agent 可以继续工作。
- **[to-questionnaire](./to-questionnaire/SKILL.md)** — 将一个你无法独自回答的决策转化为问卷，交给他人填写。
- **[teach](./teach/SKILL.md)** — 通过多次会话教会用户一项新技能或概念，使用当前目录作为有状态的教学工作空间。
- **[writing-for-agents](./writing-for-agents/SKILL.md)** — 为 agent 编写文档的参考：使文档可预测的控制点与原则。
- **[writing-great-skills](./writing-great-skills/SKILL.md)** — 编写和编辑技能的参考：使技能可预测的词汇和原则。
- **[wait-what](./wait-what/SKILL.md)** — 停下，最后一条消息没传达清楚——请求用更简单的语言和缺失的上下文重新表述。

## 模型调用

模型或用户可访问（丰富的触发措辞，使模型能够主动调用它们）。

- **[grilling](./grilling/SKILL.md)** — 就计划、决策或想法无休止地面试用户，直到决策树的每个分支都得到解决。
