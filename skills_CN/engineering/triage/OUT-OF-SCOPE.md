# 超出范围的知识库

仓库中的 `.out-of-scope/` 目录存储被拒绝的功能请求的持久记录。它有两个用途：

1. **组织记忆**——为什么某个功能被拒绝，这样推理不会在 issue 关闭后丢失
2. **去重**——当新 issue 与之前的拒绝匹配时，skill 可以展示先前的决定，而不是重新讨论

## 目录结构

```
.out-of-scope/
├── dark-mode.md
├── plugin-system.md
└── graphql-api.md
```

每个**概念**一个文件，而不是每个 issue 一个文件。多个请求同一内容的 issue 会被归组到一个文件下。

## 文件格式

文件应以轻松、易读的风格编写——更像简短的设计文档，而不是数据库条目。使用段落、代码示例和示例来使推理对首次接触的人清晰有用。

```markdown
# Dark Mode

This project does not support dark mode or user-facing theming.

## Why this is out of scope

The rendering pipeline assumes a single color palette defined in
`ThemeConfig`. Supporting multiple themes would require:

- A theme context provider wrapping the entire component tree
- Per-component theme-aware style resolution
- A persistence layer for user theme preferences

This is a significant architectural change that doesn't align with the
project's focus on content authoring. Theming is a concern for downstream
consumers who embed or redistribute the output.

```ts
// The current ThemeConfig interface is not designed for runtime switching:
interface ThemeConfig {
  colors: ColorPalette; // single palette, resolved at build time
  fonts: FontStack;
}
```

## Prior requests

- #42 — "Add dark mode support"
- #87 — "Night theme for accessibility"
- #134 — "Dark theme option"
```

### 文件命名

使用简短、描述性的 kebab-case 概念名称：`dark-mode.md`、`plugin-system.md`、`graphql-api.md`。名称应足够可识别，让浏览目录的人无需打开文件就能理解被拒绝的内容。

### 编写拒绝理由

理由应当实质性的——不是"我们不想要这个"，而是说明为什么。好的理由应引用：

- 项目范围或理念（"本项目专注于 X；主题是下游关注点"）
- 技术约束（"支持此功能需要 Y，这与我们的 Z 架构冲突"）
- 战略决策（"我们选择使用 A 而不是 B，因为……"）

理由应当经得起时间考验。避免引用临时情况（"我们现在太忙了"）——那些不是真正的拒绝，只是推迟。

## 何时检查 `.out-of-scope/`

在分类期间（步骤 1：收集上下文），读取 `.out-of-scope/` 中的所有文件。评估新 issue 时：

- 检查请求是否匹配已有的超出范围的概念
- 匹配基于概念相似性，而非关键词——"night theme" 匹配 `dark-mode.md`
- 如果匹配，向维护者展示："这类似于 `.out-of-scope/dark-mode.md`——我们之前拒绝是因为[理由]。你还坚持同样的看法吗？"

维护者可能：

- **确认**——将新 issue 添加到现有文件的"Prior requests"列表中，然后关闭
- **重新考虑**——删除或更新超出范围的文件，issue 继续正常分类流程
- **不同意**——issues 相关但有区别，继续正常分类

## 何时写入 `.out-of-scope/`

仅当**增强**（不是 bug）被以 `wontfix` *拒绝*时才写入。这同样适用于增强 PR 和 issue——被拒绝的 PR 在此记录，以防止相同请求以新代码形式再次出现。

当某事因为**已经实现**而被 `wontfix` 关闭时，**不要**写入这里。那是已构建的功能，不是被拒绝的功能；记录它会用假拒绝污染去重检查。相反，关闭评论应指向该功能已存在的位置。

流程：

1. 维护者决定某个功能请求超出范围
2. 检查是否已存在匹配的 `.out-of-scope/` 文件
3. 如果是：将新 issue 追加到"Prior requests"列表
4. 如果否：创建新文件，包含概念名称、决定、理由和首个 prior request
5. 在 issue 上发布评论，说明决定并提及 `.out-of-scope/` 文件
6. 使用 `wontfix` 标签关闭 issue

## 更新或删除超出范围的文件

如果维护者改变了之前被拒绝概念的想法：

- 删除 `.out-of-scope/` 文件
- Skill 无需重新打开旧 issue——它们是历史记录
- 触发重新考虑的新 issue 继续正常分类流程
