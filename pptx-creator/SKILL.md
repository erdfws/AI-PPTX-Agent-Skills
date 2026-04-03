---
name: pptx-creator
description: A specialized tool for generating high-quality presentation slides (PPTX) tailored for market analysis and management reporting. Features user-selectable styles including Consulting, Financial Times, Tech, and Swiss Design. It generates high-information-density HTML slides strictly conforming to the dom-to-pptx renderer, then delegates to the html2pptx skill for high-fidelity conversion. Trigger this skill whenever the user asks to "make a PPT", "create slides", "presentation from this article", or similar — even if they don't say "pptx" explicitly.
---

# PPTX Creator Skill

## 简介
这是一个专业的 PPT 自动生成核心控制脑，专注于为企业管理层、市场分析汇报等场景生成**呈现式/阅读版（Reading-Oriented）**的高信息密度幻灯片。它完全由用户自定义风格，支持多数据图表排版，并将生成的 HTML 交由底层引擎转换为无损 PPTX。

## 核心能力
- **强行询问机制**：严格遵循用户意志，若用户未指定风格，立刻中断反问用户。
- **呈现式研报基因**：采用 Action Title（核心结论标题）与高密度网格结构。
- **多风格系统**：内置四种专属汇报的视觉规范（顶尖咨询风 / 财经研报风 / 现代科技风 / 瑞士国际主义风格）。
- **自动化转换**：生成的 HTML 自动交由 `html2pptx` 独立 skill 完成高保真 PPTX 转换。

## 依赖
- **html2pptx skill**：负责读取指定 HTML 文件列表并自动操作底层的 Chrome，输出 PPTX。

## 操作流程 (Agent SOP)

### 阶段零：风格拦截与确认 (Crucial Step: User Preference)
1.  **检查用户指令**：判断用户是否在请求中明确指定了 PPT 的视觉风格。
2.  **停下来反问**：如果用户**没有**指明风格，**立刻停止向后执行，向用户提问**："请问您希望这份幻灯片使用哪种视觉风格？目前支持：1. 瑞士国际主义风 (Swiss Style)，2. 顶尖咨询风，3. 财经研报风，4. 流光手稿风。" 必须等待用户明确选择后才可进行下一步。

### 阶段一：需求分析与规划 (Analysis & Planning)
根据输入的内容类型决定生成多少页 HTML 幻灯片。若为长篇报告（路径 B），需先解析并向用户确认结构："识别到X个模块... 将生成对应的幻灯片"，随后进入生成。如果是短文本/数据（路径 A），则直接生成一张。

### 阶段二：HTML 呈现式设计 (Generation)
你必须直接输出完整且可渲染的 HTML 内容，**严格遵守全局技术规格，并按照所选风格编写内联 CSS。**

> 排版底层逻辑 (阅读向规则 - 强制执行)
> 1. **核心标题与洞察 (Header & Insight)**：每页幻灯片顶端的 H1 标题必须是概括性的**短促名词短语**（例如：“风电市场装机容量概括”）。同时，必须在 H1 标题下方紧贴一个独立副标题或提要框（如 `.lead-box`），放置完整分析结论的长句子，从而达到“提纲挈领”的效果。
> 2. **高密度结构 (Grid)**：幻灯片绝大部分必须使用 2栏 或 3栏 的类报纸网格结构布局（使用原生 Flex 或 CSS Grid）。谢绝大面积的空白极简风，以呈现详实分析数据为主。

> ⚠️ 全局技术规格 (针对 dom-to-pptx 转换引擎，所有风格必须严格遵守)
> 1. 文档结构与样式 (Document & CSS)：
>    * 必须输出完整的 HTML 文档，包含 `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`。
>    * CSS 推荐在 `<head>` 中使用 `<style>` 标签集中管理，但**严禁**将关键样式写在 `body{}`、`html{}` 或 `*{}` 全局选择器中（会被转换器强制过滤）。
>    * **避免使用** `.header` 等通用类名以防与转换器样式冲突，建议使用 `.slide-header` 等具体类名。
> 2. 根容器 (Root Container)：
>    * 必须固定宽高：`width: 1920px; height: 1080px;`。
>    * 必须设置：`position: relative; overflow: hidden;` 且推荐使用 `id="slide-container"` 以便检测。
>    * 禁止使用 `position: fixed`。
> 3. 布局与文本处理 (Layout & Text)：
>    * 允许使用 Flexbox、Grid 以及 `position: absolute`。支持 `overflow: hidden` 用于裁剪视觉元素。
>    * **文本对齐必须显式声明**（如 `text-align: left;`），切勿依赖浏览器默认值。
>    * 避免在文本容器上使用 flex 居中（合并使用 display:flex 和 justify-content:center 会导致转换库强行判定居中），文字容器建议用原生块级特性排版。
> 4. 动态图表与依赖 (Charts & Scripts)：
>    * 支持 `<canvas>` 图表。Chart.js/ECharts 等依赖库可直接在 `<head>` 中通过 `<script src="...">` 引入。
>    * **【强制防错】图表渲染和初始化的 JavaScript 代码块，必须放置在 `<body>` 标签即将闭合前的最底部，以保证在执行时相关的 Canvas DOM 绝对存在。**
>    * 图表初始化脚本**必须设置** `animation: false`（禁用动画），以确保转换器在 8 秒捕获窗口内抓取到完整静态图像。
> 5. 资源加载 (CORS)：
>    * 外部字体的 `<link>` 必须带有 `crossorigin="anonymous"`。
>    * 外部图片同理，否则图片圆角等将失效降级；强烈建议使用 Base64。
> 6. 禁用事项：
>    * **严禁**使用 `<img src="*.svg">`，**必须**使用代码内联的 `<svg>` 以保留矢量特性。
>    * **严禁**使用 `mix-blend-mode` 或 `backdrop-filter`。

> 风格对应的 CSS 参数规范 (选择其一映射到你的 HTML <style> 中)
> 
> **选项 1: 瑞士国际主义风 (Swiss Style - 经典)**
> - *色彩*：商务深蓝 (`#003366`) 为主，中性灰为辅，警示红 (`#D73A49`) 与增长绿 (`#28A745`) 为点缀。根容器背景纯白。
> - *字体*：全局统一设置为主流默认字体 `'Microsoft YaHei', sans-serif`（微软雅黑），无需外链引用。H1 使用 Extra Bold，色值为 `#1A1A1A`。
> - *CSS 特征*：核心结论区 (Lead) 使用极浅商务蓝背景 (`#F0F4F8`)，并在左侧用 8px 的深蓝 (`#003366`) 粗实线作为点睛边框。极其强调网格的左对齐。
> 
> **选项 2: 顶尖咨询风 (Consulting Style)**
> - *色彩*：深海军蓝 `#002060` (为主色，用于顶部横幅粗线或背景块)，强调色用暗砖红 `#C00000` 或青蓝 `#00B0F0`，背景纯白 `#FFFFFF`。
> - *字体*：全局统一使用 `'Microsoft YaHei', sans-serif`（微软雅黑），无需外链引用。
> - *CSS 特征*：卡片或模块间采用 `border: 1px solid #D9D9D9;` 隔离隔离框，`border-radius: 0px;`（完全直角），不允许使用任何阴影 box-shadow 以显严谨冷静。
> 
> **选项 3: 财经研报风 (Financial Times Style)**
> - *色彩*：背景绝不可用纯白，必须是浅洋皮纸肤色 `#FDFBF7` 或淡粉肤色 `#FFF1E5`；标题与粗线条使用深墨绿 `#003319` 或酒红色 `#660000`。
> - *字体*：核心中英大标题必须使用经典**衬线字体 (Serif)**（如系统自带的 `Georgia, 'Times New Roman', SimSun`），正文数据块必须使用 `'Microsoft YaHei', sans-serif`（微软雅黑）形成古典与现代的优雅反差，无需外链引用。
> - *CSS 特征*：不需要包围的边框；重点数据区顶部或底部使用稍厚的实体线条分隔如 `border-top: 4px solid #660000;` 模拟老旧财经报纸。
> 
> **选项 4: 流光手稿风 (Aurora Manuscript Style - 白色科技)**
> - *色彩*：极具呼吸感的极简白底，并在背景区域或标题装饰区块使用极淡的流光渐变（如非常微弱的 `linear-gradient` 呈现冷蓝过渡至丁香紫），以呈现无形中的科技流光感。文字主体采用反差极强的纯黑 `#111111`。
> - *字体*：全局统一使用 `'Microsoft YaHei', sans-serif`（微软雅黑），无需外链引用。
> - *CSS 特征*：具有前沿科技白皮书的“手稿”质感，不要使用厚重的外边框去包裹内容卡片。主要通过极致的留白排版、无边框的元素分区以及文字层面的色彩发力（如关键指标的高光渐变文字 `background-clip: text;`）来塑造科技公司的高级表现力。

执行结束要求：所有 HTML 内容代码生成后，必须将它们以 `slide_01.html`, `slide_02.html` 的形式保存至工作区目录下的 `output/PPT_时间戳/html/`。

### 阶段三：自动化触发导出 (Delegation)
在工作区创建完所有指定的 HTML 后，委托给 `html2pptx` skill 控制底层系统完成最后一步：跨进程通信和导出。
读取 `html2pptx` 的 SKILL.md，按它指示操作（例如使用 puppeteer 等）。对于你自身而言任务已全部完成。