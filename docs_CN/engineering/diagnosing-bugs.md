# diagnosing-bugs

## 它做什么

`diagnosing-bugs` 对一个棘手的 bug 或性能回归执行六个阶段的诊断：构建复现、最小化、排列假设、仪表化、编写回归测试后修复、清理。

在存在**紧**反馈循环之前，它不会让 agent 形成理论——一个已经运行过一次、针对*这个* bug 能变红、修复后能变绿的命名命令。编码 agent 接到 bug 报告时的默认行为是阅读代码并猜测；这个 skill 阻止了这一点。如果不存在能变红的命令，就没有 Phase 2。这一道关卡就是这个 skill 存在的意义。它之后的所有内容——二分查找、假设验证、仪表化——在信号存在之后都是机械性的。

## 何时使用

输入 `/diagnosing-bugs`，或者当任务符合条件时 agent 会自动调用它——它是 model-invoked 的，在"diagnose"、"debug this"或某物损坏、抛出异常、失败、变慢的报告中触发。

在以下棘手情况时使用它：一个经不起初步查看的 bug、一个间歇性故障、一个在两个已知正常状态之间悄悄潜入的回归。它的设计就是重型的，不适合你想在一条消息中得到答案的问题。

| 你的情况 | 去哪里 |
| --- | --- |
| 一个可以描述为症状的具体缺陷 | 本 skill |
| 一个已知前后状态的慢端点或时序回归 | 本 skill——它有一个性能分支（测量基线，然后二分查找） |
| "这个代码库的瓶颈在哪里？"——没有具体症状 | 不是本 skill。它诊断一个已知的失败，不做审计 |
| 来自他人的原始 bug 报告，尚未确认或整理 | 先用 [triage](https://aihero.dev/skills-triage) |
| 一次性代码来回答设计问题，而非追踪缺陷 | [prototype](https://aihero.dev/skills-prototype) |
| 先测试后构建计划行为 | [tdd](https://aihero.dev/skills-tdd) |
| 没有好的接缝来锁定 bug | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)——本 skill 本身就会 handoff 到那里 |

## 紧循环就是这个 skill 的核心

Phase 1 获得了不成比例的努力，因为它是唯一一个困难的阶段。这个 skill 提供了一个构建循环的方式阶梯，大致按偏好排序：

1. 在任何能触达 bug 的接缝处编写一个失败的测试。
2. 针对运行中的开发服务器的 curl 或 HTTP 脚本。
3. 带有 fixture 输入的 CLI 调用，与已知正常的快照进行 diff。
4. 一个无头浏览器脚本，对 DOM、控制台或网络进行断言。
5. 重放捕获——保存的请求、载荷或事件日志，在隔离状态下通过代码路径运行。
6. 一次性测试工具：系统的最小子集，一次函数调用。
7. 属性或模糊循环，用于"有时输出不正确"的情况。
8. 可以交给 `git bisect run` 的二分查找测试工具。
9. 差异循环——相同输入，旧版本对比新版本。
10. 最后手段：[人机循环](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop) bash 脚本。该 skill 附带了 `scripts/hitl-loop.template.sh` 来实现这一点：agent 运行脚本，你按照终端中的提示操作，你的答案会作为可解析的输出返回。

*一个*循环不是目标。**紧**才是：快速（几秒钟）、确定性（每次运行结果相同）、精准（断言你的确切症状，而非"没有崩溃"），以及 agent 可以无人值守运行。一个 30 秒的不稳定循环几乎和不循环一样糟糕。对于一个偶尔才会出现的 bug，目标不是干净的复现，而是**更高的复现率**——循环触发、并行化、增加压力、注入休眠，直到不稳定的频率足够高以便调试。

当确实无法构建一个循环时，它被指示停止并说明情况，列出它尝试过的内容，并向你请求 [environment](https://www.aihero.dev/ai-coding-dictionary/environment) 访问权限、捕获的 artifact，或添加临时仪表化的许可。它不应该继续进行假设。

## 阶段之间的关卡

这些阶段是关卡，而不是检查清单。每个阶段在某个特定条件满足之前都拒绝打开。

| 关卡 | 必须为真的条件 |
| --- | --- |
| 进入 Phase 2 | 一个命名命令，已运行并粘贴了输出，能够针对这个 bug 变红 |
| 进入 Phase 3 | 复现已复现*并*已最小化——剩余的每个元素都是支撑性的 |
| 进入 Phase 4 | 存在 3-5 个可证伪的已排序假设，每个假设都陈述了其预测，在测试之前展示给你 |
| 进入 Phase 5 | 探针映射到具体预测，一次一个变量，每条调试日志都带有 `[DEBUG-a4f2]` 风格的标签，这样清理只需一次 grep |
| 完成 | 原始复现不再复现，仪表化已移除，并且被证实正确的假设写入了 commit 消息 |

Phase 5 有一个值得了解的逃生通道。回归测试在修复之前编写，但前提是存在一个**正确的接缝**——测试能在调用点处练习真实的 bug 模式。当唯一可用的接缝太浅时，skill 被要求如实说明，而不是编写一个给出虚假信心的测试。这种缺失本身就是发现，也是将事后分析路由到 `improve-codebase-architecture` 的原因。

## 常见问题

**它在我只想要一个直接答案的快速问题上触发了。**
这是该 skill 被报告最多的问题，而且是真实的。尤其在 GPT-5.6-Sol 上，用户报告它在对问题的简单描述上就会触发："模型触发了相当正式的 diagnosing-bugs skill。然后它继续构建复现场景——通常会构建一个价值有限的模拟场景——然后再给我回复或建议。这导致了相当大的回复延迟。" 四个不同的人报告了同样的情况，见 [issue #578](https://github.com/mattpocock/skills/issues/578)。接受的修复是先用更轻量的方法，只在问题需要时才升级到更重量的方法，但这个改动尚未落地。该 skill 是针对 Claude Code 的调用行为校准的；具有更低激活阈值的 [model](https://www.aihero.dev/ai-coding-dictionary/model) 会过度触发它。在它升级之前，实际的解决方法是说清你想要的（"直接回答这个问题，不要诊断"），或者在 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 中禁用对它的 model-invoked。

**我能把它指向一个代码库，问性能问题在哪里吗？**
不能。它诊断一个你已经能命名的失败。它的性能分支是针对有症状的回归——建立基线测量，然后二分查找，先测量后修复——而不是主动扫描。针对主动版本的 skill 曾被 [提议并关闭](https://github.com/mattpocock/skills/issues/431)；目前没有这样的 skill。

**它在写修复之前会停下来问我吗？**
不会。只有 Phase 3 有一个人工检查点——排序后的假设列表在任何测试之前展示给你，如果你不在，它会按自己的排序继续进行。仪表化和修复之间没有关卡，所以 agent 可能在你同意其根本原因之前就开始编写代码。[Issue #124](https://github.com/mattpocock/skills/issues/124) 请求添加那个关卡，目前仍处于开放状态。如果你想要它，在调用 skill 时说明。

**我已经在这个 bug 报告上运行了 `/triage`。这里的工作是重复的吗？**
部分是，而且两个 skill 都不会承认这一点。正如一位读者所说："Triage 的第 3 步本质上是 diagnosing-bugs Phase 1-2 的一个浅层、有界的实例，但两个文件都没有提及对方。" Triage 做一个有界的"这真的是 bug 吗，表面是什么"的通过；这个 skill 做彻底版本。先运行 triage 并非浪费——它的验证通常会给你 Phase 1 所需的大部分原始材料——但要做好在这里重新彻底做一遍的准备，并且不要期望会有交叉引用告诉你这一点。

**它粘贴的复现输出会泄露密钥吗？**
可能会。该 skill 要求 agent 粘贴调用及其输出，并请求 HAR 文件、日志转储和 core dump 等 artifact。这些都没有通过指令进行清理。[Issue #674](https://github.com/mattpocock/skills/issues/674) 正是提出了这个问题——凭据、令牌、cookie 和个人数据被带入聊天、issue 或 PR——并提出了一个 redaction 护栏。该 issue 处于开放状态且尚未实现。目前将 redaction 视为你的责任，特别是在输出进入任何公开场合之前。

**我的安全扫描器将这个 skill 标记为高风险。**
Snyk 标记了它，而且这个标记是误报。它是该集合中唯一一个随指令一起附带可执行 shell 脚本（`hitl-loop.template.sh`）的 skill。已发布的 `.sh` 加上运行指令加上出站 HTTP 足以触发静态扫描器。该脚本本身大约 30 行 `read -r -p` 提示，暂停等待人工输入。扫描器是在评估能力面，而不是已验证的漏洞利用。

**`/diagnose` 怎么了？**
在 v1.0.0 中重命名为 `/diagnosing-bugs`。旧名称不再存在。你的任何链式调用 `/diagnose` 的东西——一个包装 skill、一个保存的 prompt——都需要更新。

## 如果符合以下情况说明它在工作

- 它在提出任何理论之前先向你展示一个命令及其红色输出。如果理论先到，说明 skill 没有运行。
- 它复现的失败就是你报告的那个，而不是它在过程中找到的附近的失败。
- 它在开始猜测之前缩小复现范围，并且能告诉你为什么每个剩余部分都是支撑性的。
- 在任何假设被测试之前，你会看到一个排序后的 3-5 个假设列表，每个假设都带有你可以证伪的预测。
- 它添加的每条调试日志都带有像 `[DEBUG-a4f2]` 这样的标签，当它宣布完成时，grep 该标签返回为空。
- commit 或 PR 消息指明了哪个假设是正确的。
- 当它无法用测试锁定 bug 时，它会直说，而不是编写一个浅层测试。

## 它如何融入整体

`diagnosing-bugs` 是一个随时可使用的独立 skill。当某物损坏时你进入它，当修复及其回归测试到位后你退出它；它不持有状态，也不需要预先设置。[ask-matt](https://aihero.dev/skills-ask-matt) 会将"Something's broken"路由到这里。

两个邻居很重要。当真正的发现是代码没有接缝来锁定 bug 时，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 会接收 [handoff](https://www.aihero.dev/ai-coding-dictionary/handoff)——这个建议是在修复完成后、拥有更多信息时做出的。[triage](https://aihero.dev/skills-triage) 位于它的上游，用于处理来自他人的原始 bug 报告，并执行相同前两个阶段的较浅版本。
