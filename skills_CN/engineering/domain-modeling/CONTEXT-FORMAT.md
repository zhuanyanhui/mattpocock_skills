# CONTEXT.md 格式

## 结构

```md
# {上下文名称}

{一到两句话描述这个上下文是什么以及为什么存在。}

## Language

**Order**:
{对该术语的一到两句话描述}
_避免使用_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_避免使用_: Bill, payment request

**Customer**:
A person or organization that places orders.
_避免使用_: Client, buyer, account
```

## 规则

- **要有立场。** 当同一概念存在多个词汇时，选择最佳的一个，并将其他词列在 `_Avoid_`（避免使用）下。
- **保持定义精炼。** 最多一到两句话。定义它*是什么*，而不是它*做什么*。
- **仅包含此项目上下文特有的术语。** 通用编程概念（超时、错误类型、工具模式）不属于这里，即使项目广泛使用了它们。在添加术语之前，先问：这是此上下文独有的概念，还是通用编程概念？只有前者才应该纳入。
- **当自然聚类出现时，在子标题下对术语分组。** 如果所有术语都属于一个内聚的领域，扁平列表即可。

## 单上下文 vs 多上下文仓库

**单上下文（大多数仓库）：** 一个 `CONTEXT.md` 放在仓库根目录。

**多上下文：** 仓库根目录的 `CONTEXT-MAP.md` 列出所有上下文、它们的位置以及它们之间的关系：

```md
# Context Map

## Contexts

- [Ordering](./src/ordering/CONTEXT.md) — receives and tracks customer orders
- [Billing](./src/billing/CONTEXT.md) — generates invoices and processes payments
- [Fulfillment](./src/fulfillment/CONTEXT.md) — manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them to start picking
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched` events; Billing consumes them to generate invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

技能会推断适用哪种结构：

- 如果存在 `CONTEXT-MAP.md`，读取它以查找上下文
- 如果只有根目录的 `CONTEXT.md`，则为单上下文
- 如果两者都不存在，在解析到第一个术语时惰性地在根目录创建 `CONTEXT.md`

当存在多个上下文时，推断当前话题与哪个上下文相关。如果不明确，直接询问。
