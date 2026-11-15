---
name: setup-pre-commit
description: 在当前仓库中设置 Husky 预提交钩子，包含 lint-staged（Prettier）、类型检查和测试。当用户想要添加预提交钩子、设置 Husky、配置 lint-staged、或添加提交流程中的格式化/类型检查/测试时使用。
---

# 设置预提交钩子

## 设置的内容

- **Husky** 预提交钩子
- **lint-staged** 在所有暂存文件上运行 Prettier
- **Prettier** 配置（如果缺失）
- 预提交钩子中的 **typecheck** 和 **test** 脚本

## 步骤

### 1. 检测包管理器

检查 `package-lock.json`（npm）、`pnpm-lock.yaml`（pnpm）、`yarn.lock`（yarn）、`bun.lockb`（bun）。使用存在的那个。如果不明确，默认为 npm。

### 2. 安装依赖

作为 devDependencies 安装：

```
husky lint-staged prettier
```

### 3. 初始化 Husky

```bash
npx husky init
```

这会在 package.json 中创建 `.husky/` 目录并添加 `prepare: "husky"`。

### 4. 创建 `.husky/pre-commit`

写入此文件（Husky v9+ 不需要 shebang）：

```
npx lint-staged
npm run typecheck
npm run test
```

**调整**：将 `npm` 替换为检测到的包管理器。如果仓库在 package.json 中没有 `typecheck` 或 `test` 脚本，省略这些行并告知用户。

### 5. 创建 `.lintstagedrc`

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

### 6. 创建 `.prettierrc`（如果缺失）

仅在不存在 Prettier 配置时创建。使用以下默认值：

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

### 7. 验证

- [ ] `.husky/pre-commit` 存在且可执行
- [ ] `.lintstagedrc` 存在
- [ ] package.json 中的 `prepare` 脚本为 `"husky"`
- [ ] `prettier` 配置存在
- [ ] 运行 `npx lint-staged` 以验证其工作

### 8. 提交

暂存所有更改/创建的文件，并使用以下消息提交：`Add pre-commit hooks (husky + lint-staged + prettier)`

这将会运行新的预提交钩子——一个好的冒烟测试来验证一切正常。

## 注意事项

- Husky v9+ 不需要在钩子文件中添加 shebang
- `prettier --ignore-unknown` 跳过 Prettier 无法解析的文件（图片等）
- 预提交先运行 lint-staged（快速，仅暂存文件），然后运行完整的类型检查和测试
