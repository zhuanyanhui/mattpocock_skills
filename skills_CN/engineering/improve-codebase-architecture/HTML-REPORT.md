# HTML 报告格式

架构评审会渲染为一个位于操作系统临时目录中的独立 HTML 文件。Tailwind 和 Mermaid 都来自 CDN。Mermaid 能可靠地处理图状图表；手工构建的 div 和行内 SVG 则处理更具编辑感的视觉效果（质量图、横截面图）。两者混合使用——不要凡事都依赖 Mermaid，否则看起来会很千篇一律。

## 脚手架

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly:
         dashed seam lines, hand-drawn-feeling arrow heads, etc. */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## 页眉

仓库名称、日期，以及一个简洁的图例：实线框 = 模块，虚线 = 接缝，红色箭头 = 泄漏，加粗深色框 = 深模块。无需介绍段落——直接展示候选方案。

## 候选方案卡片

图表承载核心信息。文字部分应稀疏、简洁，直接使用来自 `/codebase-design` 技能的术语，不做额外解释。

每个候选方案是一个 `<article>`：

- **标题**——简短，命名深化方向（例如"折叠订单接收流水线"）。
- **徽章行**——推荐强度（`Strong` = 翠绿色，`Worth exploring` = 琥珀色，`Speculative` = 石板色），外加依赖分类标签（`in-process`、`local-substitutable`、`ports & adapters`、`mock`）。
- **文件**——等宽字体列表，`font-mono text-sm`。
- **Before / After 图**——核心部分。两列并排。参见下方模式。
- **问题**——一句话。哪里出了毛病。
- **方案**——一句话。做什么改变。
- **收益**——要点列表，每条 ≤6 个词。例如"测试只命中一个接口"、"定价逻辑不再泄漏"、"删除 4 个浅层包装器"。
- **ADR 标注**（如适用）——琥珀色底框内一行文字。

无需大段解释。如果一张图需要配一段文字才能看懂，那就重画这张图。

## 图表模式

选择匹配候选方案的模式。混合使用。不要让每张图看起来都一样——多样性本身就是重点。

### Mermaid 图（依赖 / 调用流程的主力）

当要点是"X 调用 Y 调用 Z，看看这团乱麻"时，使用 Mermaid 的 `flowchart` 或 `graph`。将其包裹在 Tailwind 样式的卡片中，避免显得突兀。使用 classDef 为泄漏边缘标红、深模块标深色。序列图适用于"之前：6 次往返；之后：1 次"的场景。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手工盒子与箭头（当 Mermaid 的布局不配合时）

将模块作为带边框和标签的 `<div>`。箭头作为绝对定位在相对容器上的行内 SVG `<line>` 或 `<path>` 元素。当你希望"之后"的图呈现为一个粗边框深模块、内部灰化的效果时使用——Mermaid 无法以合适的视觉权重渲染这种效果。

### 横截面图（适合展示层次浅度）

堆叠水平色带（`h-12 border-l-4`）来展示一次调用穿过的各层。之前：6 个薄层各干不了多少事。之后：1 个厚色带，标注整合后的职责。

### 质量图（适合"接口与实现一样庞大"的场景）

每个模块两个矩形——一个代表接口表面积，一个代表实现。之前：接口矩形几乎和实现矩形一样高（浅）。之后：接口矩形矮，实现矩形高（深）。

### 调用图折叠

之前：函数调用树呈现为嵌套盒子。之后：同一棵树折叠进一个盒子，内部调用以淡化形式展示其中。

## 样式指南

- 偏向编辑风格，而非企业仪表板风格。宽裕的留白。标题可选衬线字体（`font-serif` 搭配 stone/slate 效果不错）。
- 节制用色：一个强调色（翠绿或靛蓝），红色表示泄漏，琥珀色表示警告。
- 图表高度保持在约 320px，使 Before/After 可以舒适地并排显示而不需滚动。
- 图中模块标签使用 `text-xs uppercase tracking-wider`——它们应读作示意图，而非 UI。
- 唯一脚本是 Tailwind CDN 和 Mermaid ESM 导入。报告本身是静态的——没有应用代码，没有 Mermaid 自身渲染之外的交互性。

## 顶部推荐区

一个较大的卡片。候选方案名称、一句话说明推荐理由，以及指向其卡片的锚链接。就这么简单。

## 语气

中文表达平实、简洁——但架构方面的名词和动词直接来自 `/codebase-design` 技能。简洁不是漂移的借口。

**严格使用：** module、interface、implementation、depth、deep、shallow、seam、adapter、leverage、locality。

**绝不可替换：** component、service、unit（替代 module）· API、signature（替代 interface）· boundary（替代 seam）· layer、wrapper（替代 module，当你指的是 module 时）。

**符合风格的措辞：**

- "Order intake module is shallow — interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in prod, in-memory in tests."

**收益要点**用术语命名所得：*"locality: bugs concentrate in one module"*、*"leverage: one interface, N call sites"*、*"interface shrinks; implementation absorbs the wrappers"*。不要写 *"easier to maintain"* 或 *"cleaner code"*——这些不在术语表中，不配占用篇幅。

不含糊其辞，不铺垫开场，不写"值得注意的是……"。如果一个句子可以做成要点，就做成要点。如果一个要点可以删掉，就删掉。如果一个词不在 `/codebase-design` 术语表中，在发明新词之前先找一个已有的。
