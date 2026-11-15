Quickstart:

```bash
npx skills add mattpocock/skills --skill=research
```

```bash
npx skills update research
```

[Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/research)

## 它做什么

`research` 通过阅读拥有答案的源头来回答问题，并留下一个带有引用的 Markdown 文件。它只基于**一手资料（primary sources）**——官方文档、源代码、规范、第一方 API——绝不基于这些资料的二手转述，因此它保存的内容可以追溯到权威来源，而不是摘要的摘要。

## 何时使用

输入 `/research`，或者当任务变成跑腿的阅读工作时代理会自动触发它。

当下一步是*查找某些信息*——API 的行为方式、规范的实际内容、某个说法是否成立——并且你不想因为阅读而耽误自己的工作时使用。如果想通过访谈而不是阅读来完善计划，请使用 [grilling](https://aihero.dev/skills-grilling)；如果想通过一次性代码探索构建什么，请使用 [prototype](https://aihero.dev/skills-prototype)。

## 委托的跑腿工作

关键特点是阅读作为**后台代理**运行。你继续工作；它去追踪每个主张的一手来源，然后将一个带有引用的 Markdown 文件放到仓库中存放此类笔记的位置。研究是你委托的跑腿工作，而不是你外包的思考——你得到一份可以回应的文档，并附有其来源。

## 在体系中的位置

随时可用的独立工具，为思考型 skill 提供输入：它生成的文件是可以据以深入探讨、规划或设计的东西，因此它位于 [grilling](https://aihero.dev/skills-grilling) 和 [to-prd](https://aihero.dev/skills-to-prd) 等工作流程的上游，而不是在构建链中。要查看完整地图，请参见 [ask-matt](https://aihero.dev/skills-ask-matt)。
