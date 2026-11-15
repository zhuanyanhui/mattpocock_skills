## 它做什么

`resolving-merge-conflicts` 逐个 hunk 地处理进行中的 git merge 或 rebase，然后运行项目自己的检查，最后以 commit 完成操作。

它拒绝将冲突当作纯文本问题来处理。在触碰任何 hunk 之前，它会追溯每一方回到其 **[一手资料](https://www.aihero.dev/ai-coding-dictionary/primary-source)** —— commit message、PR、原始 issue —— 因此它是在两个意图之间做选择，而不是在两个文本块之间做选择，并且在它们兼容的地方尽可能保留双方。当它们确实不兼容时，它会选择符合 merge 声明目标的那一方，并命名所做的权衡。它不会发明任何新行为来掩盖冲突，`--abort` 也不是它拥有的选项：merge 总是进行到完成的 commit。

## 何时使用它

输入 `/resolving-merge-conflicts`，或者当任务匹配时 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 会自动调用它。

当 git 已经因无法自行解决的冲突而停下时使用它。它只针对你眼前的冲突，不涉及冲突任何一方的其他内容：

| 你的情况 | skill |
| --- | --- |
| 正在 merge 或 rebase 中，树中存在冲突标记 | 这个 |
| merge 已完成，某处现在出现你看不出来原因的行为异常 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| 规划如何拆分工作以减少分支碰撞 | 都不适用 —— 见下方关于并行工作的问题 |

## 一手资料优先于 `ours` 和 `theirs`

这个 skill 要消灭的失败模式是通过标志解决冲突：`--ours`、`--theirs`，或者手工删除看起来不太重要的代码块，这样标记消失了，构建也能编译通过。这种解决方式语法上可能完美，却仍然会默默丢弃某人故意做的变更。

你无法保留你没有读过的意图。所以工作从历史开始 —— commits、PRs、[工单](https://www.aihero.dev/ai-coding-dictionary/ticket) —— 然后才进入 diff。循环中存在另一个步骤也是出于同样的原因：skill 会找到仓库自己的 [自动化检查](https://www.aihero.dev/ai-coding-dictionary/automated-check) 并在 commit 前运行它们，因为 merge 是 git 中最容易产生同时满足两个分支却通不过任何一方测试的代码的地方。

## 常见问题

**Claude Code 本身已经能很好地解决冲突了。为什么这还需要一个 skill？**

附加值在于"找到一手资料"和"运行反馈循环"这两个步骤，否则每次都必须手工提示。一个未被提示的 agent 通常会仅从 diff 中产生一个看似合理的解决然后就停下了。skill 的价值在于它不让 agent 跳过的两个步骤 —— 阅读每一方为什么存在，以及事后运行检查。这比一个好的 [model](https://www.aihero.dev/ai-coding-dictionary/model) 只多出微弱的优势，而这正是设计意图：至少有一位读者预测过这是一个完整的 skill，而随着模型的改进，它会变成一个无操作指令。

**为了避免冲突，我一开始就应该让并行 agent 远离相同文件吗？**

大部分情况下不需要。在并行任务间划分文件的成本超过了它节省的部分，因为 agent 处理合并冲突的能力已经足够强，权衡并不像看起来那么严酷。唯一值得保持的纪律是先做大型重构。在十个分支已经从它分叉之后，一个大型重命名才落地的情况，代价仍然很高。

关于并行工作树，来自用户报告的一个注意事项：当兄弟 [会话](https://www.aihero.dev/ai-coding-dictionary/session) 各自在自己的工作树中构建一个工单时，合并回来最好由编写了变更的那个会话来完成，因为它已经知道意图。最后把所有人的冲突集中到一个 agent 上，恰恰丢弃了这个 skill 第二步必须去重新构建的 [上下文](https://www.aihero.dev/ai-coding-dictionary/context)。

**为什么从不使用 `--abort`？**

放弃会丢弃已完成的解决工作，下次你尝试时仍然会回到同样的冲突，毫无改变。这个 skill 是为 merge 必然要发生的情况而写的。如果你已经决定它不应该发生，那是在调用之前就要做的决定，而不是循环内的一个分支。

## 它在工作中的标志是

- agent 在解决冲突时向你引用 commit message、PR 或 issue，而不仅仅是 diff hunks。
- 每个 hunk 最终都包含双方的行为，或者有一条明确说明丢弃了什么以及为什么的注释。
- 结果中没有任何东西是两个分支都不包含的。
- 类型检查、测试和格式化在 commit *之前* 就已经找到并运行通过，而不是在你注意到某些东西出问题之后。
- 你最终得到一个干净的工作树，操作已完成 —— 包括多 commit rebase 中剩余的每个 commit。

## 它的位置

一个随时可调用的独立 skill，不依赖任何其他 skill：它在 git 停滞时启动，在工作树干净且已 commit 时结束。它唯一的真正邻居是 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)，当 merge 干净地解决但合并后的代码出现异常时接手 —— 这是诊断问题，不是冲突问题。它完全脱离了构思到交付的主流程，所以 [ask-matt](https://aihero.dev/skills-ask-matt) 是了解它前后运行什么的地图。
