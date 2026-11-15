Quickstart:

```bash
npx skills add mattpocock/skills --skill=tdd
```

```bash
npx skills update tdd
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd)

## 它做什么

`tdd` 以测试优先的方式构建功能或修复 bug：先写一个失败的测试，然后只写刚好足够的代码让它通过，接着做下一个行为。它承载着让这个循环产出值得保留的测试的标准——什么样的测试是好的、测试放在哪里、mock 是做什么的，以及悄悄毁掉整个测试套件的三种反模式。

它不会在你尚未商定的接缝处编写任何测试。在任何测试存在之前，它会列出打算在哪些公共边界进行测试，并停下来等你确认，因为测试工作是有限的，而这里正是你把测试花在关键路径上而非每个边缘情况上的地方。另一件要知道的事是，`tdd` 是一个**参考**，而非驱动者。它保存着循环的规则，而其他东西（你自己，或者 [implement](https://aihero.dev/skills-implement)）运行应用这些规则的[会话](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何时使用

输入 `/tdd`，或者当任务合适时 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 会自动调用它——以测试优先的方式构建功能或修复 bug，或者当你说"red-green-refactor"时。

当有一个具体的行为要构建，有输入和可观察的输出，并且你想要能经受住重构的测试时使用它。

| 你的情况 | 去哪里 |
| --- | --- |
| 有定义好的输入和输出的行为——业务逻辑、请求/响应契约、转换、验证 | `tdd` |
| 行为还没有确定 | [to-spec](https://aihero.dev/skills-to-spec)，它还会在任何代码编写之前商定测试接缝 |
| 问题真正在于接口的形状，而非测试 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你有了 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 或 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)，想要完整构建 | [implement](https://aihero.dev/skills-implement)，它在每个工单中驱动 `tdd` |
| 配置、接线、胶水代码、类型注解、直接的 CRUD 委托 | 这里没有完全合适的——见下面的开放缺口 |

最后一行是一个真正的缺口，而不是风格偏好。这个技能决定接缝*在哪里*；但它内部没有任何东西决定一个变更是否值得进行循环。在一个没有独立真实来源可以断言的变更上运行它，你就会得到一个重述实现的测试——正是这个技能本身警告的同义反复反模式，只是从另一个方向到达的。这是 [issue #746](https://github.com/mattpocock/skills/issues/746)，且处于开放状态。在关闭之前，这个判断由你或你的 `CLAUDE.md` 来决定。

## 前置条件

需要安装 [codebase-design](https://aihero.dev/skills-codebase-design)。`tdd` 过去自带深层模块和接口设计笔记；在 v1.0 中这些被删除了，改用共享技能，而 `tdd` 现在依赖它来获取接口设计词汇。没有其他前置条件——这个技能是[无状态的](https://www.aihero.dev/ai-coding-dictionary/stateless)，不写自己的文件。

## 循环，以及它运行的接缝

三个概念承载了这个技能。

**红-绿。** 先写失败的测试，然后只写刚好足够的代码让它通过。不要预判下下个测试。没有重构阶段：它在 2026 年 6 月被移除了，因为 agent 基本上从不执行它，也因为审查和实现工作作为单独的会话更好。重构属于 [code-review](https://aihero.dev/skills-code-review)。

**垂直切片。** 一个接缝、一个测试、一个最小实现，然后重复——第一个周期是一颗**示踪弹**，证明一条路径端到端工作。相反的是水平切片：先写所有测试，再写所有代码。批量测试验证的是*想象的*行为，它们检查的是事物的形状而非用户做了什么，并且在理解实现之前就锁定了测试结构。

**预先商定的接缝。** 接缝是你观察行为的公共边界，不需要触及内部。规则是绝对的：不在未经确认的接缝上编写测试。在完整链中，接缝在更早的 [to-spec](https://aihero.dev/skills-to-spec) 阶段就已经商定——"`/tdd` 被告知只在预先商定的测试接缝上工作，`/code-review` 检查是否只使用了商定的测试接缝。"单独调用时，`tdd` 会直接问你。

它被编写来防止的三种反模式：

| 反模式 | 标志 |
| --- | --- |
| 与实现耦合 | 重命名内部函数时测试失败，尽管行为没有改变。Mock 内部协作者、断言调用次数、使用数据库查询而非接口进行验证。 |
| 同义反复 | 期望值以代码计算它的相同方式计算，所以测试在构造上必然通过。期望值必须来自其他地方——已知正确的字面量、演算示例、规范。 |
| 水平切片 | 在任何实现之前就落地了一批测试。 |

Mock 仅用于系统边界——外部 API、时间、随机数，有时是文件系统或数据库。不用于你自己的模块。

## 常见问题

**为什么它不重构？描述不是说"red-green-refactor"吗？**

因为重构步骤被移除了，但描述没有。移除是有意为之：agent 几乎从不执行它，而且把实现和审查放在单独的会话中效果更好。结果是否仍然符合教科书上的 TDD 定义，不如循环是否产出更好的代码重要。触发短语和正文之间的不匹配已被归档为 [issue #589](https://github.com/mattpocock/skills/issues/589)，仍然开放，所以"red-green-refactor"继续作为触发技能的短语有效。你得到的是红→绿，重构在 [code-review](https://aihero.dev/skills-code-review) 中进行。

**它让我选择测试接缝，我不知道该选哪个。**

这是这个技能报告最多的摩擦点（[issue #607](https://github.com/mattpocock/skills/issues/607)）。提示只按名称列出候选接缝，没有说明每个接缝捕捉到什么或遗漏什么，所以你是在标签之间做选择。目前还没有修复方案。实用的变通方法是在回答之前问 agent 权衡是什么——组件级接缝遗漏了集成接缝捕捉到的什么，以及它慢多少。这也是为什么在 `to-spec` 中预先商定接缝，因为你在那里能看到整个功能，而不是单个提示。

**它在测试之前先写了实现，尽管技能说先红。**

这确实会发生。一位用户逼问 [model](https://www.aihero.dev/ai-coding-dictionary/model)，得到了一个异常诚实的回答："我知道技能说'一次一个测试，看它因正确的原因失败'——我读了。我只是默认了自己的正常习惯。"这个技能的编写就包容了这一点。没有指令能让 agent 100% 遵守，而且强行施压收益甚微——即使循环没有被严格遵守，它仍然值得运行，因为结果总体上仍然更好。如果某个特定切片必须严格遵守，盯着运行而不是相信技能能强制执行。

**它应该先写浏览器测试还是端到端测试吗？**

通常不应该，而且这个技能也不会阻止。一位用户报告 agent 先写了 Playwright 测试，然后花了很长的循环重新运行它并得出结论——*测试*出了问题，而功能还不存在。在你的 `CLAUDE.md` 中配置这一点。浏览器测试太慢，以至于红-绿反馈循环不再值得；在你的仓库的 `CLAUDE.md` 中声明它们在行为工作之后再编写。

**`/tdd` 替代了 `/implement` 或课程的 `/do-work` 吗？**

没有。`/tdd` 文档化方法论；`/implement` 是一个非常简单的 工作→反馈→提交 循环，是 `/do-work` 的直接替代。课程的单个 `/do-work` 步骤现在拆分到了 `/implement`、`/tdd` 和 `/code-review`。如果你在问应该对工单运行哪一个，答案几乎总是 `/implement`。

**深层模块和接口设计指南去哪了？**

在 v1.0 中进入了 [codebase-design](https://aihero.dev/skills-codebase-design)，泛化了，所以多个技能共享一套词汇。`refactoring.md`` 同时离开了；重构现在是 [code-review](https://aihero.dev/skills-code-review) 的工作，那个技能承载了 Fowler 的坏味基线。

**它知道我的其他工单吗？**

不知道。针对单个工单运行时，它会很乐意提出属于兄弟工单的工作，因为它看不到其余的问题图（[issue #129](https://github.com/mattpocock/skills/issues/129)）。Matt 的立场是，这不是 `tdd` 的工作。把 spec 和工单一起传会有帮助；而一开始就把工单大小调对更有帮助。

## 它生效的标志

- 它在任何测试文件存在之前就停下来，命名打算测试的接缝，并等待。
- 一个测试出现，变红，得到刚好足够的代码通过它，然后才出现下一个测试——不是一批测试后跟一批代码。
- 测试名称读起来是能力（"用户可以使用有效的购物车结账"），而不是内部（"checkout 调用 paymentService.process"）。
- 断言中的期望值是你能追溯到规范的字面量，而不是以代码计算它的方式重新计算的值。
- 重命名内部函数在套件中不会破坏任何东西。
- Mock 只出现在外部边界——支付 API、时钟——从不围绕你自己的模块。

## 在体系中的位置

`tdd` 是主链构建步骤内部的引擎，而不是一个独立的步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

[to-spec](https://aihero.dev/skills-to-spec) 预先商定测试接缝，[implement](https://aihero.dev/skills-implement) 在每个工单中驱动 `tdd`，而 [code-review](https://aihero.dev/skills-code-review) 之后检查是否只使用了商定的接缝——并且拥有 `tdd` 不再做的重构工作。它的另一个邻居是 [codebase-design](https://aihero.dev/skills-codebase-design)，`tdd` 使用的接缝和深层模块词汇的共享来源。你也可以直接使用它，只要有具体的行为要构建而没有完整的规范在起作用。当你不确定哪个技能适合你的情况时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
