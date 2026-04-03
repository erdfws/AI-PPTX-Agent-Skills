# dom-to-pptx HTML 编写规范

> 按照以下规范编写 HTML，可获得最佳的 PPT 转换效果。

---

## 1. 尺寸与容器

根容器必须满足以下条件：

- **固定宽高**：推荐 `width: 1920px; height: 1080px`，或等比的 `1000px × 562px`。保持 16:9 比例，库会自动缩放至标准 PPT 页面。
- **定位方式**：设置 `position: relative`。
- **不可见元素不会被处理**：`display: none` 或 `visibility: hidden` 的元素会被跳过，不要用它们隐藏需要导出的内容。
- **智能溢出裁剪 (Overflow)**：根容器可加 `overflow: hidden`。对于内层容器的 `overflow: hidden`，转换工具会自动识别并裁剪纯装饰性的溢出元素（如背景图块），同时保留文字内容的可编辑性。

### 扩展支持：完整文档结构
转换工具（v1.4.0+）已完全支持解析标准的完整 HTML 文档。
- 可直接在 `<head>` 中使用 `<style>` 编写 CSS，或引入 `<link rel="stylesheet">`。
- **`<head>` 中的 `<script>` 标签**也会被提取并执行（如 CDN 引入的 Chart.js / ECharts），依赖库在 body 脚本之前加载。
- 不再强求将所有样式写为内联样式（inline-style）。

> ⚠️ **CSS 安全注意**：注入到转换器时，`body{}`、`html{}`、`*{}` 这三类全局选择器的样式规则会被**自动过滤**（防止污染转换器自身界面）。因此：
> - 不要在 `body {}` 或 `* {}` 中写幻灯片关键样式，它们不会生效。
> - 布局/字体/颜色等样式请写在根容器（如 `#slide-container`）或具体类选择器上。

```html
<!-- ✅ 推荐结构 -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap"
        rel="stylesheet" crossorigin="anonymous">
  <style>
    /* ✅ 写在具体选择器上，不会被过滤 */
    #slide-container {
      width: 1920px;
      height: 1080px;
      position: relative;
      overflow: hidden;
      background: #ffffff;
      font-family: 'Inter', sans-serif;
    }
    /* ❌ 以下写法会被自动过滤，不要依赖 */
    /* body { font-family: 'Inter'; }  */
    /* * { box-sizing: border-box; }   */
  </style>
</head>
<body>
  <div id="slide-container">
    <!-- 内容 -->
  </div>
</body>
</html>
```

---

## 2. 布局与定位

- **Flexbox 和 Grid 均可使用**。库不读取 CSS 布局规则本身，而是读取浏览器渲染后每个元素的最终位置（`getBoundingClientRect`），因此布局方式不受限制。
- **绝对定位（`position: absolute`）可以使用**，Z-index 层级按 DOM 顺序保留。
- **文本对齐**：`text-align` 属性会被忠实读取。如果你需要左对齐标题，**请显式声明** `text-align: left`，不要依赖浏览器默认值。这样即使 CSS 后续合并也不会被意外覆盖。
- **Flex 居中检测**：如果一个文本容器的 CSS 同时包含 `display: flex` 和 `justify-content: center`，库会将其解读为居中对齐。如果不是期望效果，请避免在文本容器上使用 flex 居中。

---

## 3. Canvas 与动态图表 (Chart.js / ECharts)

转换工具支持将 HTML `<canvas>` 元素高保真导出为 PPT 图片。这使得导出动态渲染的数据图表成为可能。

### 3.1 编写要求
- 确保 `<script>` 图表库通过 CDN 正常引入（如 Chart.js, ECharts）。
- 图表渲染逻辑应在 HTML 加载时自动执行。
- 工具会提供**最长 8 秒钟**的等待时间，确保脚本完全加载并完成 Canvas 渲染。

### 3.2 限制
- 导出后的图表在 PPT 中为静态高清图片，交互功能（如提示框 Tooltip、鼠标悬停）不会保留。
- 保证包含 Canvas 的外部容器不要使用会导致库退化为光栅化的极端负值布局。

## 4. 字体

### 3.1 外部字体（Google Fonts 等）

`<link>` 标签必须加 `crossorigin="anonymous"`，否则字体自动嵌入会失败，PPT 中会 fallback 成 Arial。

```html
<!-- ✅ 正确 -->
<link
  href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap"
  rel="stylesheet"
  crossorigin="anonymous"
/>

<!-- ❌ 错误：缺少 crossorigin -->
<link href="https://fonts.googleapis.com/css2?family=Roboto&display=swap" rel="stylesheet" />
```

### 3.2 手动指定字体（CORS 限制时）

如果字体 URL 因 CORS 限制无法自动检测，通过 `fonts` 选项手动传入：

```javascript
await exportToPptx('#slide-container', {
  fileName: 'output.pptx',
  fonts: [
    {
      name: 'Roboto',
      url: 'https://fonts.gstatic.com/s/roboto/v30/KFOmCnqEu92Fr1Mu4mxK.woff2',
    },
  ],
});
```

### 3.3 最稳妥的方案

将字体文件部署到自己的服务器，配置正确的 CORS 头，或使用 base64 内联字体。

---

## 4. 图片

### 4.1 外链图片需要 CORS 头

外部图片服务器必须返回：

```
Access-Control-Allow-Origin: *
```

否则圆角处理（Canvas `source-in` 合成）无法访问像素，圆角效果会降级跳过。

### 4.2 建议方案

将图片代理到自己的域名下，或使用 base64 内联：

```html
<!-- ✅ base64 内联，无 CORS 问题 -->
<img src="data:image/png;base64,iVBORw0KGgo..." style="border-radius: 50%;" />

<!-- ✅ 自己域名下的图片 -->
<img src="/assets/avatar.png" style="border-radius: 50%;" />

<!-- ⚠️ 外链图片需确认目标服务器有 CORS 头 -->
<img src="https://images.unsplash.com/photo-xxx?w=64&h=64" />
```

### 4.3 圆角图片

`border-radius` 圆角效果会自动处理（Canvas 去白边光晕），无需额外配置，只需保证图片可被 CORS 访问。

---

## 5. CSS 类名与选择器

在使用转换工具（dist-web-app）时，幻灯片的 CSS 会被注入到转换器的同一文档中。这可能导致 **CSS 类名冲突**。

### 5.1 避免使用的通用类名

以下类名已被转换器自身使用，幻灯片中应避免使用或必须显式声明所有关键属性：

| 类名 | 转换器用途 | 冲突风险 |
|------|-----------|----------|
| `.container` | 页面主容器 | 低 |
| `.header` | 页面标题区（含 `text-align: center`） | ⚠️ **高** |
| `.card` | 功能面板 | 中 |
| `.btn` | 按钮样式 | 低 |

### 5.2 最佳实践

```css
/* ✅ 明确声明所有关键属性，不依赖默认值 */
.header {
    text-align: left;        /* 显式声明！ */
    margin-bottom: 50px;
    color: #1A1A1A;
}

/* ✅ 使用更具有描述性的类名 */
.slide-header { ... }
.slide-title { ... }

/* ✅ 或使用 ID 选择器提高特异性 */
#slide-container .header { text-align: left; }
```

---

## 6. 视觉效果

| 效果 | 支持情况 | 说明 |
|------|----------|------|
| `linear-gradient` | ✅ 完全支持 | 转换为矢量 SVG，包括多色节点、角度、透明度 |
| `box-shadow` | ✅ 完全支持 | 直角坐标转换为 PPT 极坐标，1:1 还原 |
| `filter: blur()` | ✅ 支持 | 转换为 PPT soft-edge 效果 |
| `border-radius` | ✅ 支持 | 精确计算圆角百分比 |
| `text-transform` | ✅ 支持 | uppercase / lowercase |
| `letter-spacing` | ✅ 支持 | |
| `mix-blend-mode` | ❌ 不支持 | PPT 格式无对应特性，会被忽略 |
| `backdrop-filter` | ❌ 不支持 | PPT 无对应概念，慎用作关键视觉 |
| CSS 动画 / transition | ❌ 不适用 | 捕获静态状态，动画无意义 |

**建议**：如果 `mix-blend-mode` 是设计的关键视觉，改用固定颜色值替代。

---

## 7. SVG 与图标

### 6.1 SVG 优先使用内联写法

内联 `<svg>` 元素的处理效果远好于 `<img src="xx.svg">`：

```html
<!-- ✅ 推荐：内联 SVG -->
<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
  <path d="M12 2L2 7l10 5 10-5-10-5z" fill="#6366f1"/>
</svg>

<!-- ⚠️ 不推荐：img 引用，可能丢失细节 -->
<img src="icon.svg" width="24" height="24" />
```

### 6.2 SVG 保持矢量

如果需要导出后在 PPT 中继续编辑 SVG 图表，开启 `svgAsVector` 选项：

```javascript
await exportToPptx('#slide-container', {
  svgAsVector: true,
});
```

在 PowerPoint 中右键 SVG 图形 → "转换为形状"（或"组合 → 取消组合"）即可编辑。

### 6.3 FontAwesome 等字体图标

字体图标依赖字体文件加载，需确保字体已正确加载（可在 `fonts` 选项中手动传入字体 URL），否则图标会显示为方块。

---

## 8. 调用示例（完整参考）

```javascript
import { exportToPptx } from 'dom-to-pptx';

await exportToPptx('#slide-container', {
  fileName: 'presentation.pptx',

  // 字体自动嵌入（默认开启）
  autoEmbedFonts: true,

  // 如果自动检测失败，手动补充字体
  fonts: [
    { name: 'MyFont', url: 'https://your-domain.com/fonts/myfont.woff2' },
  ],

  // 推荐：将溢出处理交由 App 层预处理，保持文本可编辑
  overflowHandling: 'ignore',

  // 必须：开启 Canvas 元素导出支持（支持图表）
  exportCanvas: true,
  canvasCorsPolicy: 'placeholder',

  // 保留 SVG 为矢量（适合含图表的幻灯片）
  svgAsVector: true,

  // 不自动下载，返回 Blob 自行处理（如上传到服务器）
  // skipDownload: true,
});
```

**多页导出：**

```javascript
const slides = document.querySelectorAll('.slide');
await exportToPptx(Array.from(slides), { fileName: 'multi-slide.pptx' });
```

---

## 9. 快速检查清单

在运行导出前，对照以下清单：

### 结构与容器
- [ ] 根容器有固定宽高（推荐 1920×1080 或 1000×562）
- [ ] 根容器设置了 `position: relative`
- [ ] 根容器使用 `id="slide-container"` 或 `.slide` 或 `[data-slide]` 以便自动检测
- [ ] 需要导出的元素均为可见状态（没有 `display: none`）

### CSS 样式
- [ ] 关键样式写在具体类/ID 选择器上，**不要写在 `body{}`、`html{}`、`*{}` 中**
- [ ] `text-align` 等对齐属性**显式声明**，不依赖浏览器默认值
- [ ] 避免使用 `.header` 等与转换器冲突的通用类名（或显式覆盖所有属性）
- [ ] 没有依赖 `mix-blend-mode` 或 `backdrop-filter` 做关键视觉

### 资源加载
- [ ] 外部字体的 `<link>` 标签加了 `crossorigin="anonymous"`
- [ ] 外链图片的服务器配置了 `Access-Control-Allow-Origin: *`
- [ ] SVG 图形使用内联写法，而非 `<img src="*.svg">`

### 动态内容
- [ ] Chart.js / ECharts 等库在 `<head>` 中通过 `<script src="...">` 引入
- [ ] 图表初始化脚本设置 `animation: false` 以确保立即渲染完成
- [ ] 所有脚本可在 **8 秒内** 完成加载和渲染
