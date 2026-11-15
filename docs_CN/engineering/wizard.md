## 功能概述

`wizard` 生成一个交互式 bash 脚本，逐步引导人工完成某项手动流程——对接第三方服务、执行一次性迁移、将项目从状态 A 迁移到状态 B。它会打开每个 URL，说明该点击和复制什么，捕获返回内容，并将其写入 `.env` 文件和 GitHub Actions 密钥中。

[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 负责编写脚本，但从不运行它。由你在自己的机器上运行。因此 wizard 不是一份让你照着做的指令清单——它是一个能驱动流程并保存状态的程序，你的工作就是点击、粘贴和按 Enter。

## 何时使用

你可以输入 `/wizard`，agent 也可以自行调用它。当它遇到必须由你来完成的步骤——无法生成的密钥、无法点击的仪表盘——它会为你构建一个 wizard，而不是把指令写在聊天里然后被滚动掉。

当下一步阻碍你的是走一遍仪表盘时，就该用它：

| 场景 | wizard 会做什么 |
| --- | --- |
| 新开发者需要在应用启动前配置六个服务 | 按顺序打开每个仪表盘，捕获密钥，写入 `.env` 和 CI |
| 一次性迁移需要按特定顺序切换开关 | 将不可逆的步骤放在确认关卡之后 |
| 项目需要从状态 A 迁移到状态 B，只需一次 | 引导完成迁移并报告未能完成的事项 |
| 你正准备把这些步骤写进 README | 改为编写可执行版本，不会悄无声息地过期 |

不要用它来*决定*构建什么；那是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 和 [to-spec](https://aihero.dev/skills-to-spec) 的工作。

## 前置条件

生成 wizard 无需任何前置条件。它编写的脚本运行在 bash 上，并在某个 stage（阶段）设置 GitHub 密钥或变量时使用 `gh`。如果 `gh` 缺失或未认证，该 stage 会发出警告，末尾的总结会告诉你需要手动设置什么，而不是让整个运行失败。

## stage（阶段）

一个 **stage（阶段）** 是集中在一个屏幕上的一项聚焦任务。脚本会在 stage 之间清屏，所以超出屏幕的 stage 会丢失滚动掉的部分。你需要按依赖顺序编写各个 stage，并设置 `TOTAL_STAGES`，这会驱动进度显示。

范围界定在写下任何一行代码之前完成。[skill](https://www.aihero.dev/ai-coding-dictionary/skill) 会先读取仓库，而不是冷启动提问：`.env*`、`docker-compose*`、框架配置，以及 `.github/workflows/` 中的每个 `secrets.*` / `vars.*` 引用——每个都是 wizard 必须产出的值。然后它会向你展示有序的 stage 列表以供确认，只有在这之后才会将每个 stage 映射到人工操作的具体路径（"Dashboard → Developers → API keys → Reveal test key → 复制"）。对于它不知道当前 UI 的部分，它会询问你或查阅文档，而不是编造点击步骤。

对于每个捕获的值，范围界定会确定它的去向：

| 目标 | 何时 |
| --- | --- |
| 仅 `.env` | 仅本地开发需要，CI 不需要 |
| GitHub 密钥 | CI 会读取，且是敏感信息 |
| GitHub 变量 | CI 会读取，且是公开信息 |
| `.env` 和密钥两者 | 本地开发和 CI 都需要 |
| 无处 | 该 stage 是纯操作——切换了一个开关或升级了一个计划 |

## 模板已解决用户体验问题

[模板](https://github.com/mattpocock/skills/blob/main/skills/engineering/wizard/template.sh) 自带完整体验：进度与剩余时间、确认关卡、跨平台 URL 打开（包括 WSL）、密钥的隐藏输入、幂等的 `.env` upsert、`gh secret` / `gh variable` 写入，以及末尾总结所有跳过的内容。`STAGES` 标记以上的部分是一个固定的库代码，在每个 wizard 中完全相同，从不手动编辑。一致性正是其核心价值所在。你只需要界定流程范围并编写各个 stage。

编写 wizard 的 agent 永远不会端到端地运行它，因为它会打开浏览器并等待人工输入。它改为进行静态验证：`bash -n`、可用时的 `shellcheck`，以及追踪每个值是否落到了范围界定指定的位置，且每个 `set_secret` 名称都匹配 CI 中真实的 `secrets.*` 引用。据此设定你的预期——第一次运行由你来进行，而那次运行就是测试。

## 默认为一次性

| 你拥有的 | 该如何处理脚本 |
| --- | --- |
| 一次性迁移、个人配置、不会重复的过渡 | 保存到 scratch 或 `scripts/` 路径，运行它，然后删除 |
| 仓库的下一个人也会需要的配置流程 | 提交它并在 README 中链接，这样他们运行脚本而不是重新问 agent |

## 常见问题

**我的 API 密钥会出现在 model 的上下文中吗？**

不会。agent 只是写一个脚本，不会运行它。你自己运行脚本，它通过隐藏终端输入捕获密钥并直接写入 `.env` 或 `gh secret`。wizard 是一个 CLI，model 并未与它连接。一个注意事项：这仅适用于 wizard 在运行时捕获的值。如果你在界定流程范围时将密钥粘贴到聊天中，它就会像其他粘贴文本一样出现在 [context](https://www.aihero.dev/ai-coding-dictionary/context) 中。

**我能回去修改填错的值吗？**

运行中途不行。没有后退按钮——stage 只能向前运行，第 3 阶段填错了就得 Ctrl-C 重新运行。重新运行是有意设计得很廉价：任何已写入 `.env` 的值都会作为默认值提供给你，所以你可以直接按 Enter 跳过填对的 stage，只重新输入填错的那个。这在发布周就有人提过，此后一直未解决："很喜欢！不过有个问题——有没有办法回去修改已经输入的内容？"

还有一个相关的未修复 bug。在 `ask` 提示中按方向键会插入 `^[[D` / `^[[C` 而不是移动光标，因为提示使用的是 `read -r` 而非 Readline（[issue #741](https://github.com/mattpocock/skills/issues/741)）。退格键能用，方向键不行。删除到错误处，而不是把光标移进去。

**它知道我已经配置了什么吗？**

部分知道，而且不如发布时的反应所假设的那么多。它会在提问前先读取仓库——你的 `.env` 文件、`docker-compose`、框架配置、CI 中的 `secrets.*` 引用——所以它只针对确实缺失的值进行范围界定，而不是像 README 那样从零开始。它不会做的是检查第三方服务。如果你的 `.env` 中已有某个密钥，wizard 会将其作为默认值提供给你，按 Enter 就保留；如果你已经创建了 Stripe 账户但没有保存密钥，wizard 仍然会把你引导到对应的仪表盘。

**它在工作流中的位置是什么——在 grilling 和 spec 之后吗？**

没有固定位置。它是一个独立技能，不是链式步骤。常见的猜测是 `/grill-with-docs → /to-spec → /wizard`，这个顺序没问题，但触发条件是一个手动流程的出现，这可以发生在任何阶段：开始之前、构建中途、或者交付之后很久。它也可以作为发现工具——范围界定能暴露任务的隐藏前置条件，比如你还没想到的三个 API 密钥，在你投入工作之前就让你看到。

**它在 Claude Code 之外能用吗？**

生成产物可以，无条件地：它是一个普通的 bash 脚本，不关心是哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 生成了它。skill 本身是模型调用的，所以它在各处都有列出——在 Claude Code 中输入 `/wizard`，在 Codex 中输入 `$wizard`，或者直接描述你卡住的配置场景。模型调用也让它避开了 [#693](https://github.com/mattpocock/skills/issues/693) 的问题，即 Claude 桌面版和网页版会从 [model](https://www.aihero.dev/ai-coding-dictionary/model) 的列表中丢失*用户调用*的 skill 并报告未安装。

**它以前不是用户调用的吗？**

以前确实是。现在改为模型调用，所以当 agent 遇到必须由你完成的步骤时，会自行调用它。你以前能做的操作都不会失效——模型调用*增加*了 agent 的调用能力，不会移除你的调用能力，所以 `/wizard` 的行为和以前完全一样。变化的是它消除的那种失败模式：agent 在构建中途遇到凭据墙，然后在聊天里丢下六个编号步骤让你手动跟着做。

**它以前在 `in-progress/` 里——现在在哪？**

`engineering/`，从 v1.2 开始。它已从 beta 分类毕业，现在随插件一起发布，所以会和其余推广分类中的 skill 一起到达，不再需要单独安装。它的行为在毕业时没有改变。

## 以下情况说明它正常工作

- 在脚本存在之前，你会看到一个有序的 stage 列表，每个 stage 产生的值，并被要求确认。
- 每个 URL 都会在询问该页面的值之前打开。永远不会让你粘贴还没去获取的东西。
- 密钥输入是盲打的。没有任何敏感信息会回显到你的滚动历史中。
- 每个 stage 适配一屏内容。你需要的内容都不会滚动掉。
- Ctrl-C 后重新运行会从你离开的地方继续，已保存的值会作为默认值提供。
- 最后的屏幕列出它写入了什么，并单独列出它未能完成、需要你手动收尾的事项。

## 它在工作流中的位置

`wizard` 是一个随时可调用的独立技能，位于自动化止步、必须由人工点击的交界处。与它最接近的是 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills)，因为两者都是为了让仓库进入可用状态——前者配置这套 skill 集合，而 `wizard` 为其他一切生成配置流程。它也与 [implement](https://aihero.dev/skills-implement) 搭配使用：当构建完成了一个需要凭据或手动切换的功能时，wizard 就是完成人工半部分的方式。当你不确定哪个 skill 适合当前场景时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
