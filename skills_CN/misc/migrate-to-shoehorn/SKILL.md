---
name: migrate-to-shoehorn
description: 将测试文件从 `as` 类型断言迁移到 @total-typescript/shoehorn。当用户提到 shoehorn、想要在测试中替换 `as`、或需要部分测试数据时使用。
---

# 迁移到 Shoehorn

## 为什么用 shoehorn？

`shoehorn` 让你在测试中传递部分数据，同时保持 TypeScript 满意。它用类型安全的替代方案替换 `as` 断言。

**仅限测试代码。** 永远不要在生产代码中使用 shoehorn。

`as` 在测试中的问题：

- 被训练不要使用它
- 必须手动指定目标类型
- 双重 as（`as unknown as Type`）用于故意错误的数据

## 安装

```bash
npm i @total-typescript/shoehorn
```

## 迁移模式

### 大对象，仅需少数属性

之前：

```ts
type Request = {
  body: { id: string };
  headers: Record<string, string>;
  cookies: Record<string, string>;
  // ...另外 20 个属性
};

it("gets user by id", () => {
  // 只关心 body.id，但必须伪造整个 Request
  getUser({
    body: { id: "123" },
    headers: {},
    cookies: {},
    // ...伪造所有 20 个属性
  });
});
```

之后：

```ts
import { fromPartial } from "@total-typescript/shoehorn";

it("gets user by id", () => {
  getUser(
    fromPartial({
      body: { id: "123" },
    }),
  );
});
```

### `as Type` → `fromPartial()`

之前：

```ts
getUser({ body: { id: "123" } } as Request);
```

之后：

```ts
import { fromPartial } from "@total-typescript/shoehorn";

getUser(fromPartial({ body: { id: "123" } }));
```

### `as unknown as Type` → `fromAny()`

之前：

```ts
getUser({ body: { id: 123 } } as unknown as Request); // 故意错误类型
```

之后：

```ts
import { fromAny } from "@total-typescript/shoehorn";

getUser(fromAny({ body: { id: 123 } }));
```

## 何时使用每种

| 函数            | 使用场景                                   |
| --------------- | ------------------------------------------ |
| `fromPartial()` | 传递仍能通过类型检查的部分数据              |
| `fromAny()`     | 传递故意错误的数据（保留自动补全）          |
| `fromExact()`   | 强制完整对象（稍后可与 fromPartial 互换）   |

## 工作流

1. **收集需求** - 询问用户：
   - 哪些测试文件有引起问题的 `as` 断言？
   - 它们是否涉及大对象，其中只有部分属性重要？
   - 他们是否需要传递故意错误的数据进行错误测试？

2. **安装并迁移**：
   - [ ] 安装：`npm i @total-typescript/shoehorn`
   - [ ] 查找包含 `as` 断言的测试文件：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
   - [ ] 将 `as Type` 替换为 `fromPartial()`
   - [ ] 将 `as unknown as Type` 替换为 `fromAny()`
   - [ ] 从 `@total-typescript/shoehorn` 添加导入
   - [ ] 运行类型检查以验证
