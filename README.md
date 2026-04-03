# AI-PPTX-Agent-Skills

本项目提供了一套完全跑在本地、**零 API 成本**的“文本到企业级高保真 PPT”的人工智能自动化生成管线。
这是一组专门为大语言模型（LLM）与 Agent 系统设计的 **技能组 (Skills)**。

系统包含两个协同工作的模块：`pptx-creator` (大脑：生成呈现式高质量网页卡片) 与 `html2pptx` (引擎：无损转化 PPTX)。支持最高标准的“企业级管理层汇报”视觉规范。

---

## 🌟 核心特性
- 📴 **100% 离线转换引擎**：彻底摒弃依赖云端转换 API。系统内部包含基于 `dom-to-pptx` 构建的微型无框架前端引擎，保护企业隐私数据，不走任何外网转换！
- 🎨 **四套高管级视觉风格**：
  1. 🇨🇭 **瑞士国际主义风 (Swiss Style)**：极致网格、粗实线点缀，标准企业红绿配色。
  2. 🏛️ **顶尖咨询风 (Consulting)**：纯粹的直角块排布、深海军蓝调头，经典 MBB 分析感。
  3. 📰 **财经研报风 (Financial Times)**：羊皮纸色背景、优雅衬线体、报刊分栏结构。
  4. 💫 **流光手稿风 (Aurora Manuscript)**：充满现代科技前沿感的留白白皮书，带极简流光漫射背景。
- 📊 **图表与代码原生支持**：得益于底层的强大重绘能力，由 AI 在 HTML 中直接编写的 Canvas 动态图表（如 Chart.js, ECharts）可在 8 秒内转化为 PPTX 内的高清矢量/位图展示。
- ✨ **高信息密度规范**：专为**呈现式/阅读版**研报设计，拒绝一页屏幕只显示三个大字的废话排布。

## ⚙️ 架构与工作流 (Workflow)

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
   - 打开项目内嵌的 `converter.html`（底层依赖 dom-to-pptx）前端页面。
   - 利用 puppeteer_evaluate 注入上方生成的 HTML 代码。
   - 轮询获取完成状态，并利用浏览器原生的下载能力，落盘出精准无损的 .pptx。
        |
        v
4. 🎉 交付用户最高保真的企业汇报 PPTX
```

## 🛠️ 安装与环境依赖 (Dependencies)

这个项目是供你的 AI (如 Claude, 深度集成 MCP 工具链的终端等) 直接挂载并运行的。

要使这套能力能够正常运作，部署环境必须具备以下四个条件：

1. **MCP (Model Context Protocol) 兼容的终端**
   你的模型客户端（例如 Claude Desktop App 或者是自定义的 CLI）需要支持读取和执行本地文件形态的 Skills。
   
2. **chrome-devtools-mcp 服务器**
   为了能够让 Agent 打开转换工具并未我们完成导出，必须在 MCP 配置文件中挂载基于 Chrome 的 DevTools MCP 服务。
   
3. **Google Chrome 浏览器**
   你的执行端（电脑/服务器）必须安装了 Chrome 浏览器（推荐版本 146+），并开启了 CDP 调试端口以配合 MCP。
   
4. **底层转换库引用许可**
   项目内的 `html2pptx/scripts/converter.html` 已经打包了离线版的所有 JS 逻辑。请确保代理引擎能够直接访问并渲染此 `file:///...` 文件协议。无需开启任何 Node.js/Python HTTP 服务器。

## 🚀 推荐 Prompt 试用语句

把这两个文件夹放进机器人的知识库和技能槽中后，向它发送：
> “这是一份《中国风力发电2023总成报告》。请运用你的 PPT 构建技能帮我做一份报告。我想要现代科技公司的流光手稿风格。每一页都必须有清晰的主标题和提供干货洞察的副标题。”

---
*Created by next-generation PPT Agentic AI.*
