# Skill 机制

[`writing-for-agents`](SKILL.md) 的 skill 特定分支：当文档是 skill 时有什么不同——frontmatter、调用选择和路由器 skill。关于编写它的一切其他内容在 `SKILL.md` 的通用参考中。

## 调用

两种选择，权衡两种负担：

- **model-invoked（模型调用）** skill 保留 `description`，以便 agent 可以自主触发它——其他 skill 也可以访问它。你仍然可以输入它的名字：模型调用总是*包含*用户访问；描述只增加 agent 发现，从不移除人类的。描述是 skill 的顶级 context pointer（上下文指针），被迫始终加载——永久的 context load 以换取可发现性。一个内容全是参考的 model-invoked skill 也是共享参考的一个归属地：另一个 skill 可以调用它，所以多个 skill 需要的参考放在一处。机制：省略 `disable-model-invocation`，并编写面向模型的描述，携带触发分支（`SKILL.md` 中的指针编写规则完全适用）。
- **user-invoked（用户调用）** skill 从 agent 的访问范围中剥离描述：只有人类输入其名字才能调用它，其他 skill 也不行。零 context load，但它消耗 cognitive load——你是必须记住它存在的索引。机制：设置 `disable-model-invocation: true`；`description` 变成面向人类的——一行摘要，剥离触发列表。

仅在 agent 必须自行访问 skill 或其他 skill 必须访问时才选择 model-invocation。如果它只通过手动触发，让它成为 user-invoked，不花 context load。

两个 user-invoked skill 都需要的共享参考不能存在于任一 skill 中——没有描述，谁也无法触发另一个。把它推到 skill 系统之外的普通文件中：任何 skill 都可以指向的外部参考。

## 按调用拆分

拆分的调用切面（序列切面在 `SKILL.md` 中）：当你有一个独立的引导词应该单独触发它时——你在 prompt 中实际使用的触发词——或者其他 skill 必须访问它时，拆分出一个 model-invoked skill。你为新的始终加载的描述支付 context load，所以这种独立访问必须值得这个代价。

## 路由器 skill

当 user-invoked skill 的数量多到你记不住时，堆积的 cognitive load 可以通过**router skill（路由器 skill）** 来解决：一个 user-invoked skill 命名其他 skill 以及何时使用每个 skill，这样人类只需要记住一个 skill 而不是多个。它只能暗示，永远无法触发它们：user-invoked skill 没有描述，所以除了人类之外什么都无法访问它们。
