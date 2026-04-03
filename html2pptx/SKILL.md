---
name: html2pptx
description: Convert HTML files to high-fidelity, editable PowerPoint (PPTX) presentations using browser automation. Use this skill whenever the user wants to turn HTML slides into PPTX, export web-based slide decks to PowerPoint, batch-convert multiple HTML pages into a single presentation, or needs to produce a .pptx from any HTML content — even if they don't explicitly mention "html2pptx". Also trigger if the user has already generated HTML slide files and now needs the PPTX conversion step.
---

# HTML to PPTX Converter

This skill converts HTML files into high-fidelity, text-editable PowerPoint presentations. It uses a bundled browser-based converter (`scripts/converter.html`) powered by the `dom-to-pptx` library. The conversion preserves vector shapes, gradients, shadows, fonts, and even canvas-rendered charts — all as editable PPTX elements, not screenshots.

## When to Use

- User has one or more `.html` slide files and wants a `.pptx`
- User asks to "export to PowerPoint" from HTML content
- User has generated HTML slides (e.g. via `pptx-factory` or manually) and needs the final conversion step
- User wants to batch-convert a folder of HTML files into a single presentation

## Prerequisites

- **Browser automation tools** must be available (e.g. `browser_subagent`, `navigate_page`, `evaluate_script`)
- The converter files are bundled at `scripts/converter.html` and `scripts/dom-to-pptx.bundle.js` (relative to this skill)

## Important: Read `references/html-spec.md` Before Writing HTML

If you are **generating** HTML slides (not just converting user-provided ones), you **must** read `references/html-spec.md` first. It documents exactly which CSS features work, which get filtered, naming conflicts to avoid, and structural requirements for optimal conversion results. Skipping this will produce broken or degraded output.

---

## Conversion Workflow

### Step 1: Locate the Converter

The converter is a single-page web app at `scripts/converter.html` (adjacent to this SKILL.md). It loads `dom-to-pptx.bundle.js` from the same directory.

Resolve the **absolute path** to `scripts/converter.html` relative to this skill's location. You'll use it as a `file://` URL.

### Step 2: Collect HTML Slide Content

Gather the HTML content to convert. This can come from:
- Files on disk (read them with `view_file` or a file-reading tool)
- HTML the user pasted directly
- HTML you just generated in a previous step

Each piece of HTML content becomes one slide in the final PPTX.

### Step 3: Open the Converter in a Browser

Use the browser automation tool to navigate to the converter:

```
file:///absolute/path/to/skills/html2pptx/scripts/converter.html
```

Wait for the page to fully load. The converter's console will print "PPTX Factory" initialization messages when ready.

### Step 4: Inject Slides via JavaScript API

The converter exposes a global JavaScript API for automation. Use `evaluate_script` (or equivalent) to call these functions:

#### Option A: Batch Load (Recommended for Multiple Slides)

```javascript
// Build the slides array from your collected HTML content
const slidesData = [
  { content: '<html>...slide 1 HTML...</html>', name: 'slide_01.html' },
  { content: '<html>...slide 2 HTML...</html>', name: 'slide_02.html' },
  // ... more slides
];

// Inject all slides at once
window.batchLoadSlides(slidesData);
```

The function returns the total number of loaded slides.

#### Option B: Single Slide

```javascript
window.loadHtmlContent('<html>...your HTML...</html>', 'my_slide.html');
```

### Step 5: Trigger the Conversion

```javascript
window.triggerConversion('My_Presentation.pptx');
```

This starts the async conversion. The converter will:
1. Parse each HTML slide (handling full documents with `<head>`, `<style>`, `<script>`)
2. Inject styles and scripts into a hidden workspace
3. Wait for external resources (fonts, chart libraries, images) to load
4. Pre-process overflow containers to preserve text editability
5. Call `dom-to-pptx.exportToPptx()` to generate the PPTX
6. Auto-download the file

### Step 6: Wait for Completion

After triggering conversion, **poll the status** until it reports success:

```javascript
window.getStatus();
// Returns strings like:
//   "⏳ 处理幻灯片 1/3: slide_01.html"
//   "⏳ 正在生成 PPTX 文件..."
//   "✅ 高保真导出成功！"
//   "❌ 导出失败: ..."
```

Poll every 2–3 seconds. The conversion typically takes 3–15 seconds depending on slide count and complexity (external scripts, canvas charts, font loading all add time).

### Step 7: Report to User

Once you see the success status:
- Tell the user the PPTX has been generated
- The file is auto-downloaded by the browser (check the default downloads folder)
- If you know the download path, provide it

---

## Available API Functions

| Function | Description |
|----------|-------------|
| `window.loadHtmlContent(html, name)` | Add a single slide |
| `window.batchLoadSlides(slides[])` | Batch add slides (array of `{content, name}`) |
| `window.triggerConversion(filename)` | Start PPTX export with given filename |
| `window.clearSlides()` | Remove all loaded slides |
| `window.getStatus()` | Get the current status message string |
| `window.getLoadedCount()` | Get the number of loaded slides |

---

## HTML Authoring Quick Reference

When **writing** HTML for conversion (not just converting user-provided files), follow these critical rules. For the full specification, read `references/html-spec.md`.

### Must Do
- Root container: `width: 1920px; height: 1080px; position: relative; overflow: hidden;`
- Use `id="slide-container"` on the root for auto-detection
- Declare `text-align` explicitly (don't rely on browser defaults)
- Load external fonts with `crossorigin="anonymous"` on `<link>` tags
- Use inline `<svg>` instead of `<img src="*.svg">`
- Set `animation: false` on chart libraries (Chart.js, ECharts)
- Put critical styles on class/ID selectors, not `body{}`, `html{}`, or `*{}`

### Must Not Do
- Don't use `position: fixed`
- Don't use `mix-blend-mode` or `backdrop-filter`
- Don't use `.header` as a class name (conflicts with converter)
- Don't put key styles in `body {}` or `* {}` (they get stripped)

### Supported Visual Effects
| Effect | Support |
|--------|---------|
| `linear-gradient` | ✅ Full (vector SVG) |
| `box-shadow` | ✅ Full |
| `filter: blur()` | ✅ Soft-edge |
| `border-radius` | ✅ Auto-calculated |
| Flexbox / Grid | ✅ Via computed positions |
| `<canvas>` charts | ✅ Exported as HD images |
| `mix-blend-mode` | ❌ No PPT equivalent |
| `backdrop-filter` | ❌ No PPT equivalent |

---

## Troubleshooting

### Charts not rendering
External scripts (Chart.js, ECharts) need time to load and render. The converter waits up to 8 seconds per external script. Ensure:
- Chart library is loaded via `<script src="...">` in `<head>`
- Chart initialization script sets `animation: false`
- Canvas has explicit width/height

### Text alignment is wrong
The converter strips `body{}`, `html{}`, `*{}` CSS rules. If your text-align is set there, it gets lost. Always set `text-align` on the specific element or its container class.

### Fonts falling back to Arial
The `<link>` tag for Google Fonts must include `crossorigin="anonymous"`. Without it, font auto-embedding fails silently.

### Styles not applying
Avoid generic class names like `.header`, `.container` which conflict with the converter's own UI. Use descriptive names like `.slide-header`, `.slide-content`.

---

## Example: Full Automation Flow

```
1. Read HTML files from output/html/ directory
2. Open browser to file:///path/to/skills/html2pptx/scripts/converter.html
3. Build JSON array of {content, name} from the HTML files
4. evaluate_script: window.batchLoadSlides(DATA)
5. evaluate_script: window.triggerConversion('Final_Report.pptx')
6. Poll: evaluate_script: window.getStatus()  (every 2s until "✅")
7. Report success to user
```
