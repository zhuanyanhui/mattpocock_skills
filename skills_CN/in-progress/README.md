# 开发中

Beta。这些技能有意公开——试试看，告诉我哪里会出问题。它们在毕业到稳定分类之前，不会被包含在插件和顶层 README 中，没有文档页面，并且可能随时更改或消失，恕不另行通知。

插件不会自动提供这些技能。你可以直接安装某一个：

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

- **[loop-me](./loop-me/SKILL.md)** — 通过多次会话对自己进行盘问，将想法提炼为可执行的工作流规格，使用当前目录作为有状态的工作空间。用户调用。
- **[writing-beats](./writing-beats/SKILL.md)** — 将文章塑造成节拍式的旅程，自选冒险风格。选择一个起始节拍，只写该节拍，然后转向下一个，直到文章自然结束。
- **[writing-fragments](./writing-fragments/SKILL.md)** — 盘问环节，挖掘你手中的写作碎片—— heterogeneous 的金句素材——并将它们追加到单个文档中，作为未来文章的原始材料。
- **[writing-shape](./writing-shape/SKILL.md)** — 将 markdown 文件中的原材料逐段塑造成文章，在每一步中论证格式选择。
- **[claude-handoff](./claude-handoff/SKILL.md)** — 将当前对话交接给一个全新的后台 agent，通过 `claude --bg` 注入交接摘要后立即接手工作。用户调用。
- **[setup-ts-deep-modules](./setup-ts-deep-modules/SKILL.md)** — 将 dependency-cruiser 接入 TypeScript 仓库，使每个包成为深度模块——实现隐藏在子文件夹中，只能通过入口文件访问，测试也通过入口进行。用户调用。
