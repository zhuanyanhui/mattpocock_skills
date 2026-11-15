---
name: setup-ts-deep-modules
description: 将 dependency-cruiser 接入 TypeScript 仓库，使每个包成为深层模块——实现隐藏在子文件夹中，仅能通过入口点文件访问。
disable-model-invocation: true
---

# 设置 TS 深层模块

将此仓库中的每个包变成**深层模块**（deep module）：少量行为对外暴露，背后是大量实现。包的公共表面是其**入口点**（entry points）——位于包根目录的文件——而子文件夹中的所有内容都是隐藏的。本技能安装 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 和相应的规则，使入口点成为唯一的访问途径，然后验证规则确实生效。

关于术语（深层模块、接口、接缝、深度），请运行 `/codebase-design` 技能——全程使用其语言。

## 此规则强制实施的形态

```
src/packages/
  <name>/
    index.ts        ← 入口点（公开）。从外部导入此处。
    client.ts       ← 另一个入口点。包可以暴露**多个**入口点。
    lib/            ← 实现：对外部隐藏，内部可以自由互相导入。
    tests/          ← 同目录的测试和 fixture（子文件夹，因此是私有的）。
```

公共表面是包的**根文件**——而非某个指定的 `index.ts`。按照约定，实现放在 `lib/` 中，测试放在 `tests/` 中，使每个包具有相同的两文件夹结构。但规则本身是通用的：**任何**子文件夹中的**任何**内容都是私有的，因此你无需扩展配置来添加新文件夹。

四个规则，均为 `error` 级别：

1. **入口点边界（Entry-point boundary）**——包外部的代码（应用代码或其他包）只能导入该包的入口点（其根文件），绝不能导入其子文件夹中的任何内容。
2. **包内自由（Intra-package freedom）**——包自身的文件可以自由互相导入。
3. **通过入口点的测试（Tests through the entry points）**——`<pkg>/tests/` 下的文件可以导入任何包的入口点及其自身的 `tests/` fixture，但绝不能导入任何包的子文件夹内部（即使是自己的也不行）。跨包的集成测试是可以的；深层导入则不行。
4. **无循环（No cycles）**——无依赖循环。

**入口点，而非桶文件（barrel）。** 因为公共表面是**每个**根文件，一个包可以暴露多个小型入口点（`index.ts`、`client.ts`、`server.ts`），而不是将所有内容通过一个巨大的 `index.ts` 集中导出。不鼓励重新导出整个子树的桶文件——保持入口点小而精，将实现隐藏在子文件夹中。

层级关系（哪些包可以依赖哪些包）是**另一个不同**的关注点，保留为配置中的注释占位符，供本仓库自行填写。

## 步骤

### 1. 检测环境

- **包管理器** —— `pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则为 npm。在后续每条命令中使用检测到的管理器（`pnpm`/`yarn`/`npm run`/`bunx`）。
- **包根目录** —— 如果存在 `src/` 使用 `src/packages`，否则使用 `packages`。如果仓库已有其他明显的约定，与用户确认选择。
- **已有配置** —— 检查是否存在 `.dependency-cruiser.*` 文件。如果存在，**不要**覆盖：合并四个规则及其选项，并告知用户你添加了什么。

**完成标志：** 包管理器、包根目录和已有配置状态均已明确。

### 2. 安装 dependency-cruiser

使用检测到的包管理器将 `dependency-cruiser` 安装为 devDependency。

**完成标志：** `dependency-cruiser` 已出现在 `devDependencies` 中。

### 3. 编写配置

将 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。将 `PACKAGES_ROOT` 设置为步骤 1 中检测到的根目录。规则基于路径深度且与扩展名无关，因此无需其他调整。

**完成标志：** `.dependency-cruiser.cjs` 已存在，包含正确的 `PACKAGES_ROOT` 和四个禁止规则。

### 4. 接入检查流程

- 添加 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
- 将其并入仓库的伞式检查命令——即已在运行类型检查的命令（例如 `check`/`ci`/`validate` 脚本）。**不要**修改 `tsconfig` 或添加路径别名。
- 如果没有伞式脚本，添加 `lint:boundaries` 并告知用户在 CI 中包含它。

**完成标志：** `lint:boundaries` 已存在，并与类型检查在同一命令中运行。

### 5. 搭建示例包

创建可提交的 `<packages-root>/example/` 作为可复制的模板：

- `index.ts` —— 入口点。导出一个委托给内部文件的函数（使包明显是**深层的**，而非简单的透传）。
- `lib/impl.ts` —— **子文件夹**中的内部文件，由 `index.ts` 导入，外部不可访问。
- `tests/example.test.ts` —— 仅导入 `../index`（入口点），并对公共函数进行断言。

告知用户这是一个可供复制或删除的起始模板。

**完成标志：** 示例包已存在，通过根入口点暴露其行为，并将 `impl` 隐藏在子文件夹中。

### 6. 验证规则生效

这是整个技能的完成标准——一个在违规时不报错的配置是毫无价值的。

1. 运行 `lint:boundaries`。它必须在干净的示例上**通过**。
2. 临时向 `tests/example.test.ts` 添加一个深层导入（例如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries` —— 它必须**失败**，报 `tests-through-entrypoints`。
3. 还原深层导入。再运行一次——它必须**通过**。

**完成标志：** 你已观察到一次通过、一次深层导入失败、再次通过。如果步骤 2 没有失败，说明规则未正确接入——在完成前修复。

### 7. 记录约定

在**包文件夹**中（`<packages-root>/README.md`）编写 `README.md`——紧邻其管理的包——涵盖：`src/packages/<name>/` 布局（根目录的入口点、`lib/` 用于实现、`tests/` 用于测试），"只能通过包的入口点（其根文件）导入"，以及如何运行 `lint:boundaries`。**明确禁止桶文件（barrel）**——应暴露多个小型入口点，而不是通过一个 index 重新导出整个子树。保持简短：一个可复制的代码片段加上每条规则一段说明。

然后在仓库的 agent 指令文件中添加**指向它的上下文指针**——如果存在 `CLAUDE.md`，则加入其中；否则加入 `AGENTS.md`（如果两者都不存在，则创建 `AGENTS.md`）。一行即可，例如 `包是深层模块——在添加或导入之前请参阅 [src/packages/README.md](./src/packages/README.md)。` 这样 agent 才能发现边界规则，而不是撞上它。

**完成标志：** `<packages-root>/README.md` 已存在并禁止桶文件，且仓库的 `CLAUDE.md`/`AGENTS.md` 已链接到它。

## 备注

- 配置文件中的 `$1` 反向引用（dependency-cruiser 的分组匹配）使包能够访问自身内部而外部无法访问——不要将其展平为每个包的单独规则。
- 公开与私有由**深度**决定：包的根文件是入口点；子文件夹中的任何内容都是私有的。约定的子文件夹是 `lib/`（实现）和 `tests/`，但规则并未硬编码它们——任何子文件夹都是私有的，因此新文件夹无需更改配置。添加入口点只需添加一个根文件——无需桶文件。
- 包是**扁平的**：根目录下一层直接子级。包的内部可以任意深度嵌套；一个包不能包含另一个包。
- 使用 `.cjs`（而非 `.js`），这样配置中的 `module.exports` 即使在 `"type": "module"` 的仓库中也能正常工作。
