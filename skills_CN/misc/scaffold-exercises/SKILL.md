---
name: scaffold-exercises
description: 创建包含章节、题目、解答和讲解的练习目录结构，通过 lint 检查。当用户想要搭建练习框架、创建练习存根或设置新的课程章节时使用。
---

# 搭建练习框架

创建能通过 `pnpm ai-hero-cli internal lint` 检查的练习目录结构，然后使用 `git commit` 提交。

## 目录命名

- **章节**（Sections）：`exercises/` 内的 `XX-section-name/`（例如 `01-retrieval-skill-building`）
- **练习**（Exercises）：章节内的 `XX.YY-exercise-name/`（例如 `01.03-retrieval-with-bm25`）
- 章节编号 = `XX`，练习编号 = `XX.YY`
- 名称使用短横线命名法（小写、连字符）

## 练习变体

每个练习至少需要以下子文件夹之一：

- `problem/` - 学生工作区，包含待办事项（TODO）
- `solution/` - 参考实现
- `explainer/` - 概念材料，无待办事项

在创建存根时，默认使用 `explainer/`，除非计划另有指定。

## 必需文件

每个子文件夹（`problem/`、`solution/`、`explainer/`）需要一个 `readme.md`，要求：

- **不为空**（必须有实际内容，即使只是单个标题行也可以）
- 没有断开的链接

在创建存根时，创建一个包含标题和描述的极简 readme：

```md
# 练习标题

此处的描述
```

如果子文件夹有代码，还需要一个 `main.ts`（超过 1 行）。但对于存根，仅包含 readme 的练习也可以。

## 工作流

1. **解析计划** - 提取章节名称、练习名称和变体类型
2. **创建目录** - 为每个路径 `mkdir -p`
3. **创建存根 readme** - 每个变体文件夹一个 `readme.md`，包含标题
4. **运行 lint** - `pnpm ai-hero-cli internal lint` 以验证
5. **修复错误** - 迭代直到 lint 通过

## Lint 规则摘要

linter（`pnpm ai-hero-cli internal lint`）检查：

- 每个练习有子文件夹（`problem/`、`solution/`、`explainer/`）
- 至少存在 `problem/`、`explainer/` 或 `explainer.1/` 之一
- 主子文件夹中存在 `readme.md` 且非空
- 没有 `.gitkeep` 文件
- 没有 `speaker-notes.md` 文件
- readme 中没有断开的链接
- readme 中没有 `pnpm run exercise` 命令
- 除非仅包含 readme，否则每个子文件夹需要 `main.ts`

## 移动/重命名练习

重新编号或移动练习时：

1. 使用 `git mv`（而非 `mv`）重命名目录 - 保留 git 历史
2. 更新数字前缀以保持顺序
3. 移动后重新运行 lint

示例：

```bash
git mv exercises/01-retrieval/01.03-embeddings exercises/01-retrieval/01.04-embeddings
```

## 示例：从计划创建存根

给定如下计划：

```
Section 05: Memory Skill Building
- 05.01 Introduction to Memory
- 05.02 Short-term Memory (explainer + problem + solution)
- 05.03 Long-term Memory
```

创建：

```bash
mkdir -p exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer
mkdir -p exercises/05-memory-skill-building/05.02-short-term-memory/{explainer,problem,solution}
mkdir -p exercises/05-memory-skill-building/05.03-long-term-memory/explainer
```

然后创建 readme 存根：

```
exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer/readme.md -> "# Introduction to Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/explainer/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/problem/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/solution/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.03-long-term-memory/explainer/readme.md -> "# Long-term Memory"
```
