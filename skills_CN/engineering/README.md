# 工程

我每天用于代码工作的技能。

## 用户调用

只有当你输入时才能访问（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

- **[ask-matt](./ask-matt/SKILL.md)** —— 询问哪种技能或流程适合你的情况。本仓库中用户调用技能的路由器。
- **[grill-with-docs](./grill-with-docs/SKILL.md)** —— 盘问环节，同时构建项目的领域模型，实时优化术语并更新 `CONTEXT.md` 和 ADR。
- **[triage](./triage/SKILL.md)** —— 通过分类角色的状态机推动问题流转。
- **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.md)** —— 扫描代码库寻找深化机会，以可视化 HTML 报告呈现候选方案，然后就你选择的一个进行盘问。
- **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.md)** —— 为工程技能配置此仓库（问题跟踪器、分类标签、领域文档布局）。在使用其他工程技能之前，每个仓库运行一次。
- **[to-spec](./to-spec/SKILL.md)** —— 将当前对话转换为规格说明并发布到问题跟踪器。无需面试——仅综合你已经讨论过的内容。
- **[to-tickets](./to-tickets/SKILL.md)** —— 将任何计划、规格说明或对话分解为一组跟踪子弹工单，每个工单声明其阻塞边——作为文本写入本地文件，或作为真实跟踪器上的原生阻塞链接。
- **[implement](./implement/SKILL.md)** —— 构建由规格说明或一组工单描述的工作，在预先约定的接缝处驱动 `/tdd`，并在提交前以 `/code-review` 收尾。
- **[wayfinder](./wayfinder/SKILL.md)** —— 规划一个超出单个 Agent 会话容量的大块工作，作为问题跟踪器上的共享探索工单地图——一次解决一个，直到通往目的地的路径清晰。

## 模型调用

模型或用户可访问（丰富的触发措辞，使模型能够主动调用它们）。

- **[prototype](./prototype/SKILL.md)** —— 构建一次性原型来回答设计问题——单个可分享的 HTML 文件用于状态/逻辑问题，或几个从同一路由切换的截然不同的 UI 变体。
- **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)** —— 针对顽固 bug 和性能回归的有纪律的诊断循环：构建一个能在这个 bug 上报错的反馈循环 → 最小化 → 假设 → 仪器化 → 修复 → 回归测试。
- **[research](./research/SKILL.md)** —— 基于高可信度的一手资料调查问题，并将发现结果保存为仓库中带引用的 Markdown 文件，以后台 Agent 方式运行。
- **[tdd](./tdd/SKILL.md)** —— 测试驱动开发，采用红-绿-重构循环。一次一个垂直切片地构建功能或修复 bug。
- **[domain-modeling](./domain-modeling/SKILL.md)** —— 主动构建和优化项目的领域模型——对照术语表质疑术语，用边界情况场景进行压力测试，并实时更新 `CONTEXT.md` 和 ADR。
- **[codebase-design](./codebase-design/SKILL.md)** —— 设计深度模块的共享规范和词汇：小型接口背后的大量行为，放置在干净的接缝处，可通过该接口测试。
- **[code-review](./code-review/SKILL.md)** —— 自固定点以来的差异的双轴审查：**标准**（是否遵循仓库的编码标准，加上 Fowler 坏味道基线？）和**规格**（是否忠实实现了原始问题/规格说明？），作为并行子 Agent 运行，以免相互污染。
- **[resolving-merge-conflicts](./resolving-merge-conflicts/SKILL.md)** —— 逐块处理正在进行的 git 合并或变基冲突，根据追踪到每一边一手资料的意图进行解决，然后完成操作——绝不 `--abort`。
- **[wizard](./wizard/SKILL.md)** —— 生成交互式 bash 向导，引导人类完成只有他们才能执行的步骤：配置基础设施、设置凭证或 CI 密钥、引导浏览陌生的第三方仪表盘，或运行一次性迁移或切换。
