# AI-PPTX-Agent-Skills

[中文说明](#中文) | [English Documentation](#english)

<h2 id="中文">🇨🇳 中文说明</h2>

本项目提供了一套完全跑在本地、**零 API 成本**的“文本到企业级高保真 PPT”的人工智能自动化生成管线。
这是一组专门为大语言模型（LLM）与 Agent 系统设计的 **技能组 (Skills)**。

系统包含两个协同工作的模块：`pptx-creator` (大脑：生成呈现式高质量网页卡片) 与 `html2pptx` (引擎：无损转化 PPTX)。支持最高标准的“企业级管理层汇报”视觉规范。

---

### 🌟 核心特性
- 📴 **100% 离线转换引擎**：彻底摒弃依赖云端转换 API。系统内部包含基于 `dom-to-pptx` 构建的微型无框架前端引擎，保护企业隐私数据，不走任何外网转换！
- 🎨 **四套高管级视觉风格**：
  1. 🇨🇭 **瑞士国际主义风 (Swiss Style)**：极致网格、粗实线点缀，标准企业红绿配色。
  2. 🏛️ **顶尖咨询风 (Consulting)**：纯粹的直角块排布、深海军蓝调头，经典 MBB 分析感。
  3. 📰 **财经研报风 (Financial Times)**：羊皮纸色背景、优雅衬线体、报刊分栏结构。
  4. 💫 **流光手稿风 (Aurora Manuscript)**：充满现代科技前沿感的留白白皮书，带极简流光漫射背景。
- 📊 **图表与代码原生支持**：得益于底层的强大重绘能力，由 AI 在 HTML 中直接编写的 Canvas 动态图表（如 Chart.js, ECharts）可在 8 秒内转化为 PPTX 内的高清矢量/位图展示。
- ✨ **高信息密度规范**：专为**呈现式/阅读版**研报设计，拒绝一页屏幕只显示三个大字的废话排布。

### ⚙️ 架构与工作流 (Workflow)

```text
1. 用户发号施令 (包含市场研报/原始分析数据)
        |
        v
2. Agent 执行 [pptx-creator] 技能
   - 提取业务逻辑、提取核心报表数据
   - 强行反问用户所需的展现风格 (四大风格选一)
   - 遵照复杂的 CSS 逻辑规格，将报告切割成高难度 HTML 排版的若干个模块。
   - 生成本地文件：slide_01.html, slide_02.html ...
        |
        v
3. Agent 跨技能唤起 [html2pptx]
   - 通过 MCP 操控本地的 headless Chrome。
   - 打开项目内嵌的 `converter.html` 前端页面。
   - 利用 puppeteer_evaluate 注入上方生成的 HTML 代码。
   - 轮询获取完成状态，并利用浏览器原生的下载能力，落盘出精准无损的 .pptx。
        |
        v
4. 🎉 交付用户最高保真的企业汇报 PPTX
```

### 🛠️ 安装与环境依赖 (Dependencies)
要使这套能力正常运作，部署环境必须具备以下四个条件：
1. **MCP (Model Context Protocol) 兼容的终端** (如 Claude Desktop app)。
2. **`chrome-devtools-mcp` 服务器** (需配置在 MCP 插件中)。
3. **Google Chrome 浏览器**。
4. **底层转换库引用许可** (Agent 能够直接通过 `file:///` 访问本项目封装的前端页面)。

---

<br>

<h2 id="english">🇺🇸 English Documentation</h2>

This project provides an AI-powered, **zero-API-cost** automated pipeline that converts text into high-fidelity, enterprise-grade PowerPoint (PPTX) presentations, running entirely locally.
This is a set of **Skills** specifically designed for Large Language Models (LLMs) and Agent-based systems.

The system consists of two synergistic modules: `pptx-creator` (The Brain: generates presentation-style high-quality HTML cards) and `html2pptx` (The Engine: performs lossless conversion to PPTX). It strictly adheres to the highest standard "C-level executive reporting" visual guidelines.

---

### 🌟 Core Features
- 📴 **100% Offline Conversion Engine**: Completely abandons reliance on cloud conversion APIs. The system includes an embedded micro frontend engine built on `dom-to-pptx`, safeguarding enterprise data privacy with zero external network traffic!
- 🎨 **Four Executive Visual Styles**:
  1. 🇨🇭 **Swiss Style**: Extreme grid alignment, bold line accents, classic corporate red/green palettes.
  2. 🏛️ **Consulting Style**: Pure orthogonal blocks, deep navy tone, classic MBB (McKinsey/BCG/Bain) analytical feel.
  3. 📰 **Financial Times Style**: Parchment background, elegant serif fonts, newspaper-column layout.
  4. 💫 **Aurora Manuscript**: High-tech white papers with minimalist, iridescent mesh-gradient backgrounds and extreme whitespace.
- 📊 **Native Chart Support**: Thanks to the robust rendering engine, Canvas dynamic charts (like Chart.js, ECharts) coded directly in HTML by the AI are converted into high-definition graphics in the PPTX within 8 seconds.
- ✨ **High Information Density**: Exclusively designed for **Reading/Report-Oriented** documents, rejecting the impractical minimalist layouts.

### ⚙️ Architecture & Workflow

```text
1. User provides prompt (containing market reports / raw analytical data)
        |
        v
2. Agent executes the [pptx-creator] Skill
   - Extracts business logic and core data.
   - Mandatorily asks the user to select one of the four visual styles.
   - Follows complex CSS specifications to generate highly sophisticated HTML chunks.
   - Generates local files: slide_01.html, slide_02.html ...
        |
        v
3. Agent triggers [html2pptx] across skills
   - Controls local headless Chrome via MCP.
   - Opens the embedded `converter.html` frontend page.
   - Injects the generated HTML code using evaluated scripts.
   - Polls for completion and utilizes native browser downloads to save the lossless .pptx file.
        |
        v
4. 🎉 Delivers maximum-fidelity corporate PPTX to the user
```

### 🛠️ Installation & Dependencies
To ensure this pipeline functions correctly, the deployment environment must meet the following computing prerequisites:
1. **MCP (Model Context Protocol) Compatible Interface** (e.g., Claude Desktop).
2. **`chrome-devtools-mcp` Server** (Must be mounted within your MCP configuration).
3. **Google Chrome Web Browser**.
4. **Local Converter Accessibility** (The Agent must be allowed to directly navigate to `file:///` URLs to access the bundled converter page).
