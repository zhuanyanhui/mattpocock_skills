## 作用

`wait-what` 是你在某条消息没传达清楚时输入的命令。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 会重新表述它刚才说的话，补上你缺失的上下文，用通俗的中文来写，并使用你项目 `CONTEXT.md` 中的词汇。

这个 skill 只有三行。这是设计，不是未完成的草稿。对抗冗长的 skill 会因膨胀而失败：一个四百行的"简洁 skill"仍然会让[模型](https://www.aihero.dev/ai-coding-dictionary/model)冗长，因为模型读的是篇幅，不是恳求。这个 skill 只携带一个精确的引导词，再无其他。

## 何时使用

你通过输入 `/wait-what` 来调用它。agent 不会自行使用它，也不应该这样做。只有你知道自己什么时候跟不上思路了。

在你注意到自己开始略读的那一刻就用。agent 已经漂入了它自创的行话、堆砌了五个缩写词，或者解释了一个你从未见过前提的决策。它修复的是你已经在进行的对话。要想从一开始就阻止行话出现，使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它会预先构建共享语言。

## 名字就是机制

引导词是 **wait**（等等）。"Be concise"（简洁一些）是针对 agent 输出的指令，模型会通过删减词汇来遵守，反而让你更跟不上。**Wait** 关注的是*你的*状态。它在说：理解在这里断了。一个听到"简短一点"的 agent 会写出电报。一个听到"等等，你把我弄丢了"的 agent 会退后一步，重新解释。

这种差异就是整个 skill 的核心。每一个流行的修复冗长的方案都在描述*输出*：`/tldr`、`/no-fluff`、`/talk-normal`。模型会矫枉过正到原始人式的表达——更短了，但并没有更清晰。描述*听者*的状态则同时要求两件事：更少的词 **加上** 你缺失的上下文。

skill 说的是重新表述 **那个（that）**，而不是"最后一条消息"。让你困惑的通常比一个段落更大，所以 agent 自行决定要回溯多远。

## 它接入你已经有的语言

正文复用了你全局 `CLAUDE.md` 和项目 `CONTEXT.md` 中已有的引导词。ASD-STE100 简化技术英语设定了语域。通用语言提供了名词。skill、`CLAUDE.md` 和 `CONTEXT.md` 使用相同的 [token](https://www.aihero.dev/ai-coding-dictionary/token)，所以调用它并不是一个新指令。它只是提醒了 agent 已经同意的一个指令。

如果你没有 `CONTEXT.md`，skill 仍然有效。你只是失去了领域词汇那一半。

## 它在起作用的标志是

- 重新表述后的内容**更短更清晰**，而不是更短更生硬。
- 它补上了你缺失的前提，而不只是删除词汇。
- 项目名词替代了自创的名词。`CONTEXT.md` 中的术语回来了。
- 你可以连续使用两次，它不会退化成简短干瘪。

## 它在哪里发挥作用

你可以在任何时候、任何对话中、任何其他 skill 内使用 `wait-what`。它在事后修复一条消息。真正的解法是预先约定好共享语言，那就是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：一场 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话，在过程中运行 [domain-modeling](https://aihero.dev/skills-domain-modeling)，这样你们双方使用的词汇就会进入你的 `CONTEXT.md`。如果你不确定哪个 skill 适合当前场景，[ask-matt](https://aihero.dev/skills-ask-matt) 会帮你路由。
