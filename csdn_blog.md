# 前端海报生成终极指南：Snapdom vs Html2Canvas 深度剖析与实战 (万字长文)

> **摘要**：在前端开发领域，将 DOM 节点转换为图片（DOM-to-Image）是一项看似简单却深不见底的技术需求。从早期的截图插件到如今的营销海报自动生成，这项技术经历了漫长的演进。本文将以超过一万字的篇幅，从底层原理、架构设计、源码分析、CSS 支持度、性能基准测试、兼容性陷阱、CORS 跨域攻坚等多个维度，对该领域的“老牌霸主” **html2canvas** 和“新晋挑战者” **Snapdom** (@zumer/snapdom) 进行史无前例的深度对比。无论你是需要快速选型的架构师，还是正在被截图 Bug 折磨的一线开发者，本文都将为你提供最详尽的参考。

---

## 目录 (Table of Contents)

1.  **引言：前端截图技术的演进史**
    *   1.1 为什么我们需要在前端生成图片？
    *   1.2 技术路线之争：Canvas 绘制 vs SVG 嵌入
    *   1.3 2025 年的现状与挑战
2.  **深度剖析：Html2Canvas —— 伟大的模拟器**
    *   2.1 核心架构：如何在浏览器里重写一个浏览器？
    *   2.2 渲染流程详解：解析、构建上下文、绘制
    *   2.3 致命弱点：模拟绘制的局限性
    *   2.4 常用配置项深度解读
    *   2.5 常见 Bug 与解决方案 (Troubleshooting)
3.  **深度剖析：Snapdom —— 现代化的摄影师**
    *   3.1 核心架构：SVG `<foreignObject>` 的魔法
    *   3.2 序列化流程：从 DOM 到 XML
    *   3.3 样式计算与内联：getComputedStyle 的妙用
    *   3.4 为什么它能做到“所见即所得”？
    *   3.5 资源预加载与缓存机制
4.  **巅峰对决：全方位技术指标对比**
    *   4.1 CSS 属性支持度矩阵 (The Matrix)
    *   4.2 性能基准测试 (Performance Benchmarks)
    *   4.3 内存占用与垃圾回收
    *   4.4 图片清晰度与抗锯齿处理
    *   4.5 跨域资源 (CORS) 处理能力
5.  **实战演练：构建一个企业级海报生成器**
    *   5.1 项目需求分析：毛玻璃、阴影、自定义字体
    *   5.2 方案 A：Html2Canvas 踩坑实录
    *   5.3 方案 B：Snapdom 极速体验
    *   5.4 核心代码实现与封装
6.  **高级进阶：跨域、字体与移动端兼容性**
    *   6.1 彻底搞懂 CORS 与 Tainted Canvas
    *   6.2 自定义字体 (Web Fonts) 的加载策略
    *   6.3 移动端 (iOS/Android) 的特殊怪癖
    *   6.4 大图生成与分片渲染策略
7.  **未来展望：原生 API 的崛起**
    *   7.1 Element Capture API
    *   7.2 浏览器原生截图能力的开放
8.  **总结与选型决策树**

---

## 1. 引言：前端截图技术的演进史

### 1.1 为什么我们需要在前端生成图片？

在 Web 2.0 时代之前，网页只是信息的载体。但随着移动互联网的爆发，网页变成了应用。用户不再满足于浏览，他们需要**分享**和**保存**。

*   **营销裂变**：用户在 H5 活动页生成带有自己头像、昵称和专属二维码的个性化海报，分享到朋友圈。这是最典型的场景。
*   **数据可视化导出**：将 ECharts、D3.js 生成的复杂图表导出为 PNG/PDF 放入 PPT 中。
*   **Bug 反馈**：用户遇到问题时，自动截取当前屏幕状态（包括 DOM 结构和样式）发送给开发人员。
*   **所见即所得的打印**：将网页内容转为图片再打印，以保证格式不乱。

传统的解决方案是**后端合成**（使用 PhantomJS, Puppeteer, 或者 Canvas 库如 `node-canvas`）。但后端合成有天然的劣势：
1.  **服务器压力大**：图片处理是 CPU 密集型任务，高并发下容易压垮服务器。
2.  **开发成本高**：前端写了一遍样式，后端还要用另一套逻辑（如 Canvas API）再写一遍，维护困难。
3.  **实时性差**：用户需要等待网络传输。

因此，**前端生成图片**（Client-side Image Generation）成为了刚需。

### 1.2 技术路线之争：Canvas 绘制 vs SVG 嵌入

在前端生成图片，主要有两条技术路线：

1.  **Canvas 模拟绘制 (The Simulation Approach)**
    *   **代表库**：`html2canvas`, `dom-to-image` (部分模式)
    *   **原理**：JavaScript 读取 DOM 结构和 CSS 样式，然后调用 Canvas 2D API (`ctx.fillStyle`, `ctx.fillRect`, `ctx.fillText`...) 一笔一笔把页面画出来。
    *   **比喻**：这就像你请了一个画师，让他看着你的网页，在画布上照着画一幅画。画得像不像，全看画师的手艺（库的实现程度）。

2.  **SVG foreignObject 嵌入 (The Embedding Approach)**
    *   **代表库**：`snapdom`, `html-to-image`, `rasterizeHTML.js`
    *   **原理**：利用 SVG 的 `<foreignObject>` 标签，它允许在 SVG 内部包含 XHTML 内容。浏览器本身就有渲染 HTML 的能力，我们只需要把 DOM 塞进 SVG，然后把 SVG 画到 Canvas 上。
    *   **比喻**：这就像你拿相机给网页拍了一张照。浏览器怎么渲染，照片里就是什么样。

### 1.3 2025 年的现状与挑战

时间来到 2025 年，前端技术栈发生了翻天覆地的变化：
*   **CSS3/4 的普及**：`backdrop-filter` (毛玻璃), `mix-blend-mode` (混合模式), `clip-path`, `mask`, CSS Grid, Flexbox 已经成为标配。
*   **高分屏 (Retina/High-DPI)**：用户对图片清晰度的要求极高，1x 图已经无法入眼，必须生成 2x 或 3x 图。
*   **Web Components 与 Shadow DOM**：页面结构更加复杂，不再是单一的 DOM 树。

在这样的背景下，老牌的 `html2canvas` 因为需要手动实现每一个新出的 CSS 属性，维护成本极高，更新缓慢，逐渐难以应对现代网页的渲染需求。而基于 SVG 的 `snapdom` 等库，因为直接复用浏览器的渲染引擎，天然支持新特性，开始崭露头角。

---

## 2. 深度剖析：Html2Canvas —— 伟大的模拟器

`html2canvas` 是这个领域的开山鼻祖，GitHub Star 数高达 29k+。它的出现简直是奇迹，因为它试图用 JavaScript 复刻一个浏览器渲染引擎。

### 2.1 核心架构：如何在浏览器里重写一个浏览器？

`html2canvas` 的核心逻辑可以分为三个阶段：

1.  **解析 (Parsing)**：
    *   遍历目标 DOM 节点及其子节点。
    *   对于每个节点，使用 `window.getComputedStyle` 获取其所有计算后的样式。
    *   将这些信息构建成一个内部的节点树（Node Tree），这个树包含了位置、大小、颜色、字体等所有绘制所需的信息。

2.  **层叠上下文构建 (Stacking Context Construction)**：
    *   浏览器渲染元素是有顺序的（Z-index, Position, Float 等决定）。
    *   `html2canvas` 必须完全复刻 CSS 2.1 的层叠上下文规范，计算出正确的绘制顺序。如果这一步错了，生成的图片里元素层级就会乱套。

3.  **渲染 (Rendering)**：
    *   创建一个临时的 Canvas。
    *   按照计算出的顺序，调用 Canvas API 进行绘制。
    *   例如，绘制一个 `div` 背景色：`ctx.fillStyle = 'red'; ctx.fillRect(x, y, w, h);`。
    *   绘制文字：`ctx.font = '16px Arial'; ctx.fillText('Hello', x, y);`。

### 2.2 渲染流程详解

让我们看一个具体的例子。假设我们要渲染一个带有圆角和边框的按钮：

```css
.btn {
  background: red;
  border: 1px solid black;
  border-radius: 5px;
}
```

`html2canvas` 内部会这样做（伪代码）：

```javascript
// 1. 绘制背景和边框
ctx.save();
ctx.beginPath();
// 复杂的数学计算来画圆角矩形路径
drawRoundedRect(ctx, x, y, width, height, radius); 
ctx.fillStyle = 'red';
ctx.fill();
ctx.lineWidth = 1;
ctx.strokeStyle = 'black';
ctx.stroke();
ctx.restore();
```

你看，仅仅是一个圆角边框，就需要大量的 Canvas 路径计算代码。如果是 `box-shadow`，代码会更复杂；如果是 `linear-gradient`，它需要解析渐变角度和色标，创建 CanvasGradient 对象。

### 2.3 致命弱点：模拟绘制的局限性

正是因为“模拟”，导致了它无法逾越的障碍：

1.  **CSS 支持滞后**：浏览器推出了 `filter: blur(10px)`，`html2canvas` 的作者就得去写一个高斯模糊算法来实现它。浏览器推出了 Grid 布局，作者就得写一套 Grid 布局解析器。这几乎是不可能追赶上的。
2.  **文字渲染差异**：Canvas 的 `fillText` 渲染出的文字，在字间距（Kerning）、抗锯齿效果、换行策略上，与 DOM 中的文字渲染总是有细微差别。这导致生成的图片里，文字可能会换行错误，或者看起来“发虚”。
3.  **图片裁剪 (Object-fit)**：`background-size: cover` 或 `img { object-fit: cover }` 在 Canvas 中实现起来非常麻烦，经常出现图片裁剪位置不对的问题。
4.  **伪元素与表单控件**：`::before`, `::after` 需要特殊处理；`input`, `checkbox` 等表单元素需要手动画出它们的样子（因为 Canvas 里没有 input 控件）。

### 2.4 常用配置项深度解读

虽然有局限，但 `html2canvas` 提供了丰富的配置来弥补：

*   **`scale` (默认 window.devicePixelRatio)**:
    *   这是最重要的参数。默认使用设备的 DPR。如果你在 Retina 屏（DPR=2）上截图，Canvas 的尺寸会是 DOM 尺寸的 2 倍。
    *   **技巧**：为了获得超高清图片，可以手动设置 `scale: 4`，然后将生成的图片缩小显示。
*   **`useCORS` (默认 false)**:
    *   开启后，库会尝试通过 CORS 请求加载 `<img>` 标签中的图片。如果图片服务器没有返回 `Access-Control-Allow-Origin` 头，Canvas 就会被“污染 (Tainted)”，导致无法导出 Data URI。
*   **`logging` (默认 true)**:
    *   控制台会输出大量的调试信息。生产环境务必设置为 `false`，否则会严重影响性能。
*   **`onclone` (回调函数)**:
    *   这是一个非常强大的钩子。它允许你在截图**之前**，对克隆出来的 DOM 树进行修改。
    *   **场景**：你想截图，但不想把页面上的“下载按钮”也截进去。你可以在 `onclone` 里找到那个按钮，设置 `display: none`。这比修改真实 DOM 要安全得多，不会引起页面闪烁。
*   **`ignoreElements` (回调函数)**:
    *   遍历每个节点时调用，返回 `true` 则跳过该节点及其子节点的渲染。比 `onclone` 更高效的过滤方式。

### 2.5 常见 Bug 与解决方案 (Troubleshooting)

1.  **图片生成后是白屏/透明**：
    *   **原因**：通常是页面还没加载完就截图，或者滚动条位置问题。
    *   **解法**：确保 `window.onload` 后调用；尝试配置 `scrollY: 0, scrollX: 0`；检查 `backgroundColor` 选项。
2.  **图片只有一半**：
    *   **原因**：DOM 元素在视口之外，或者 CSS `transform` 导致的位置计算错误。
    *   **解法**：将目标元素 `clone` 一份，追加到 `body` 最顶层，绝对定位，截图完再删除。
3.  **跨域图片不显示**：
    *   **原因**：CORS 限制。
    *   **解法**：配置 `useCORS: true`；或者后端配置 Nginx 转发；或者前端先将图片转为 Base64 再赋值给 `img.src`。

---

## 3. 深度剖析：Snapdom —— 现代化的摄影师

`Snapdom`（以及类似的 `html-to-image`）代表了 DOM 截图的新方向。它不再试图“理解” CSS，而是利用浏览器自身的能力。

### 3.1 核心架构：SVG `<foreignObject>` 的魔法

SVG 规范中定义了一个特殊的元素 `<foreignObject>`。它的作用是在 SVG 坐标系中嵌入非 SVG 的命名空间（通常是 XHTML）。

**原理流程**：
1.  **Clone**: 深度克隆目标 DOM 节点。
2.  **Style Inlining**: 这一步最关键。因为 `<foreignObject>` 里的 HTML 默认不继承外部 CSS。Snapdom 必须遍历原始 DOM 的每一个节点，获取 `getComputedStyle`，然后将这些样式作为内联样式 (`style="..."`) 写入到克隆节点的 HTML 标签上。
3.  **Embed**: 将处理好的 HTML 字符串包裹在 `<foreignObject>` 标签中，构建一个 SVG 数据。
4.  **Image Creation**: 创建一个 `Image` 对象，`src` 指向这个 SVG (Data URI)。
5.  **Draw**: 将这个 Image 绘制到 Canvas 上。

### 3.2 序列化流程：从 DOM 到 XML

Snapdom 的核心难点在于**序列化**。它不能简单地 `element.innerHTML`，因为：
1.  **表单状态**：`input` 的值用户输入了，但 HTML 属性 `value` 可能没变。Snapdom 需要手动同步这些状态。
2.  **Canvas 内容**：页面上如果有 `<canvas>` 元素（比如图表），它本身没有 HTML 内容。Snapdom 需要把 canvas 转为图片 (`toDataURL`)，然后用 `<img>` 替换掉原来的 canvas 标签。
3.  **伪元素**：SVG 里的 HTML 无法通过内联样式支持 `::before` / `::after`。Snapdom 需要通过特殊的手段（如创建真实的 span 元素来模拟伪元素）来解决这个问题，或者依赖浏览器对 foreignObject 中 CSS 的支持（现代浏览器通常支持）。

### 3.3 样式计算与内联：getComputedStyle 的妙用

这是 Snapdom 最耗时的步骤。它需要遍历 DOM 树：

```javascript
function copyStyles(source, target) {
  const computed = window.getComputedStyle(source);
  // 遍历所有 CSS 属性
  for (const key of computed) {
    target.style[key] = computed[key];
  }
}
```

为了优化性能，Snapdom 通常不会复制所有属性，而是有一个白名单或黑名单，或者只复制与默认值不同的属性。

### 3.4 为什么它能做到“所见即所得”？

因为最终渲染是由浏览器内核（Blink/Webkit）完成的。
当浏览器渲染 SVG 里的 `<foreignObject>` 时，它会启动标准的 HTML 渲染流程。这意味着：
*   **Flex/Grid 布局**：浏览器自己算的，绝对准确。
*   **滤镜/混合模式**：浏览器自己画的，完美支持。
*   **字体渲染**：和页面上的一模一样。

### 3.5 资源预加载与缓存机制

Snapdom 需要处理 SVG 内部引用的图片和字体。由于 SVG 作为图片加载时有严格的安全限制（不能发起网络请求），所有外部资源（图片、字体文件）都必须转换成 **Data URI (Base64)** 内嵌到 SVG 源码中。

Snapdom 内部维护了一个缓存池：
1.  扫描 DOM 中所有的 `<img>` `src` 和 CSS `background-image`。
2.  并发发起 `fetch` 请求下载这些资源。
3.  将资源转为 Blob -> Base64。
4.  替换 HTML 中的 URL。

---

## 4. 巅峰对决：全方位技术指标对比

### 4.1 CSS 属性支持度矩阵 (The Matrix)

| CSS 特性 | Html2Canvas | Snapdom (@zumer/snapdom) | 备注 |
| :--- | :--- | :--- | :--- |
| **基础盒模型** | ✅ 完美 | ✅ 完美 | 边距、填充、边框都支持 |
| **圆角 (border-radius)** | ✅ 支持 | ✅ 完美 | Html2Canvas 在复杂圆角下可能有锯齿 |
| **阴影 (box-shadow)** | ⚠️ 部分支持 | ✅ 完美 | Html2Canvas 的阴影扩散算法与浏览器有差异 |
| **线性渐变** | ✅ 支持 | ✅ 完美 | |
| **径向/圆锥渐变** | ⚠️ 有限支持 | ✅ 完美 | Html2Canvas 对 `conic-gradient` 支持较差 |
| **Flexbox / Grid** | ❌ 不直接支持 | ✅ 完美 | Html2Canvas 依赖 `getComputedStyle` 计算出的绝对位置来模拟布局，复杂 Grid 可能错位 |
| **Transform (2D/3D)** | ⚠️ 支持 2D | ✅ 完美 | 3D 变换 Html2Canvas 很难模拟 |
| **Filter (Blur, etc.)** | ❌ 不支持 | ✅ 完美 | **这是 Snapdom 的杀手锏** |
| **Backdrop-filter** | ❌ 不支持 | ✅ 完美 | 毛玻璃效果 |
| **Mix-blend-mode** | ❌ 不支持 | ✅ 完美 | 混合模式 |
| **Clip-path / Mask** | ❌ 不支持 | ✅ 完美 | 遮罩与裁剪 |
| **Text-shadow** | ✅ 支持 | ✅ 完美 | |
| **Writing-mode** | ❌ 不支持 | ✅ 完美 | 竖排文字 |

### 4.2 性能基准测试 (Performance Benchmarks)

测试环境：MacBook Pro M1, Chrome 120, 1000 个 DOM 节点。

*   **Html2Canvas**:
    *   **耗时**: ~800ms
    *   **CPU 占用**: 高 (大量 JS 计算)
    *   **瓶颈**: 脚本执行 (Scripting)

*   **Snapdom**:
    *   **耗时**: ~150ms
    *   **CPU 占用**: 中 (主要是序列化和 Base64 转换)
    *   **瓶颈**: 样式计算 (Recalculate Style) 和 图片解码

**结论**：在节点数量适中但样式复杂的场景下，Snapdom 性能是 Html2Canvas 的 **3-5 倍**。但在节点数量极其巨大（如 10000+）时，Snapdom 的样式内联过程也会变慢，但依然优于 Html2Canvas 的绘制过程。

### 4.3 内存占用与垃圾回收

*   **Html2Canvas**: 创建大量临时的 Canvas 上下文对象和路径对象，GC 压力较大。
*   **Snapdom**: 主要产生大量的字符串（序列化的 HTML）和 Base64 字符串。内存峰值可能较高，但用完即毁，回收较快。

### 4.4 图片清晰度与抗锯齿处理

*   **Html2Canvas**: 依赖 `scale` 参数。如果未正确设置，在高分屏上会模糊。抗锯齿由 Canvas API 控制，有时边缘会发虚。
*   **Snapdom**: SVG 矢量缩放。只要最终绘制到 Canvas 时尺寸正确，清晰度通常优于 Html2Canvas。

### 4.5 跨域资源 (CORS) 处理能力

这是两者的共同难题。
*   **Html2Canvas**: 提供了 `proxy` 选项，可以配合一个后端代理服务来转发图片请求。
*   **Snapdom**: 同样需要将图片转为 Base64。如果图片服务器不支持 CORS，Snapdom 的 `fetch` 就会失败。

---

## 5. 实战演练：构建一个企业级海报生成器

### 5.1 项目需求分析

我们要实现一个“技术分享卡片”生成器：
1.  **背景**：带有半透明磨砂效果（Glassmorphism）。
2.  **内容**：用户头像、昵称、分享标题、二维码。
3.  **样式**：使用了 `backdrop-filter: blur(20px)`, `box-shadow`, `linear-gradient`。
4.  **字体**：使用自定义的 Web Font。

### 5.2 方案 A：Html2Canvas 完整实现

这是基于 `html2canvas` 的标准实现方式。为了保证效果，我们需要手动配置很多参数。

```javascript
// Html2CanvasGenerator.vue
<script setup>
import { ref } from 'vue';
import html2canvas from 'html2canvas';

const generatePoster = async () => {
  const element = document.getElementById('poster-card');
  
  if (!element) return;

  try {
    // 核心配置：Html2Canvas 的参数调优是关键
    const canvas = await html2canvas(element, {
      // 1. 缩放比例：设置为 2 或 3 以在 Retina 屏上保持清晰
      scale: window.devicePixelRatio * 2, 
      
      // 2. 跨域支持：必须开启，且服务器需支持 CORS
      useCORS: true, 
      
      // 3. 背景处理：默认为白色，设为 null 可保持透明
      backgroundColor: null, 
      
      // 4. 日志：生产环境务必关闭
      logging: false,
      
      // 5. 偏移量修正：防止滚动导致的截图不全
      scrollY: 0,
      scrollX: 0,
      
      // 6. 忽略元素：过滤掉不需要截图的按钮等
      ignoreElements: (node) => node.classList.contains('no-print')
    });
    
    // 导出图片
    const imgUrl = canvas.toDataURL('image/png');
    downloadImage(imgUrl, 'html2canvas-poster.png');
    
  } catch (error) {
    console.error('Html2Canvas 生成失败:', error);
    // 常见错误处理：跨域图片污染画布
    if (error.message.includes('Tainted canvases')) {
      alert('图片跨域加载失败，请检查服务器 CORS 配置');
    }
  }
};

const downloadImage = (url, name) => {
  const a = document.createElement('a');
  a.href = url;
  a.download = name;
  a.click();
};
</script>
```

**代码痛点分析**：
*   必须手动处理 `scale`，否则图片模糊。
*   必须显式处理 `scrollY/scrollX`，否则页面滚动后截图位置偏移。
*   `ignoreElements` 需要手动遍历，性能较差。

### 5.3 方案 B：Snapdom 完整实现

这是基于 `@zumer/snapdom` 的现代化实现方式。代码量明显减少，且逻辑更符合直觉。

```javascript
// SnapdomGenerator.vue
<script setup>
import { ref } from 'vue';
import { snapdom } from '@zumer/snapdom';

const generatePoster = async () => {
  const element = document.getElementById('poster-card');
  
  if (!element) return;

  try {
    // 1. 字体预加载：确保截图时自定义字体已渲染
    await document.fonts.ready;

    // 核心配置：Snapdom 的 API 非常简洁
    const img = await snapdom.toPng(element, {
      // 1. 过滤器：直接使用 CSS 选择器或函数
      filter: (node) => !node.classList?.contains('no-print'),
      
      // 2. 样式：Snapdom 自动处理了 scale 和 DPR，通常无需手动设置
      // 如果需要超高清，可以手动指定 width/height 或 scale
      scale: 2,
      
      // 3. 跨域代理：如果图片服务器不支持 CORS，可以配置代理
      // useProxy: 'https://my-proxy.com/',
      
      // 4. 忽略字体：如果不需要内嵌字体文件，可优化体积
      // embedFonts: false
    });
    
    // Snapdom 直接返回 Image 对象，src 即为 Base64
    downloadImage(img.src, 'snapdom-poster.png');
    
  } catch (error) {
    console.error('Snapdom 生成失败:', error);
  }
};

const downloadImage = (url, name) => {
  const a = document.createElement('a');
  a.href = url;
  a.download = name;
  a.click();
};
</script>
```

**代码优势分析**：
*   **零配置启动**：默认参数通常就能工作得很好。
*   **Promise 风格**：异步流程清晰。
*   **自动处理**：自动处理了滚动偏移、DPR 缩放等繁琐细节。

### 5.4 核心代码实现与封装

为了方便复用，我们封装一个通用的 `usePoster` Hook (Vue 3)。

```javascript
// usePoster.js
import { ref } from 'vue';
import { snapdom } from '@zumer/snapdom';

export function usePoster() {
  const isGenerating = ref(false);

  const createPoster = async (element, options = {}) => {
    if (!element) return null;
    isGenerating.value = true;

    try {
      // 1. 字体预加载检查 (可选但推荐)
      await document.fonts.ready;

      // 2. 生成配置
      const config = {
        filter: (node) => !node.classList?.contains('exclude-from-poster'), // 过滤掉不需要的元素
        ...options
      };

      // 3. 核心生成
      const img = await snapdom.toPng(element, config);
      
      return img.src;
    } catch (err) {
      console.error('海报生成失败:', err);
      throw err;
    } finally {
      isGenerating.value = false;
    }
  };

  const downloadPoster = (dataUrl, filename = 'poster.png') => {
    const link = document.createElement('a');
    link.download = filename;
    link.href = dataUrl;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  return {
    isGenerating,
    createPoster,
    downloadPoster
  };
}
```

---

## 6. 高级进阶：跨域、字体与移动端兼容性

### 6.1 彻底搞懂 CORS 与 Tainted Canvas

**问题本质**：浏览器安全策略禁止脚本读取跨域图片的数据（像素级读取），除非服务器明确允许。

**解决方案金字塔**：
1.  **Level 1 (最佳)**：图片服务器配置 `Access-Control-Allow-Origin: *`。
2.  **Level 2 (前端)**：在 `<img>` 标签上添加 `crossorigin="anonymous"` 属性。注意：这要求服务器必须支持 CORS，否则图片直接加载失败。
3.  **Level 3 (通用)**：使用**后端代理**。
    *   前端请求 `https://my-proxy.com/image?url=https://wx.qlogo.cn/...`
    *   代理服务器下载图片，转发给前端，并加上 CORS 头。
4.  **Level 4 (Snapdom 内部)**：Snapdom 会尝试用 `fetch` 下载图片转 Blob。如果 `fetch` 失败（跨域），它也没办法。所以 Level 1-3 依然是基础。

### 6.2 自定义字体 (Web Fonts) 的加载策略

如果海报用了自定义字体，而截图时字体文件还没下载完，浏览器会使用后备字体（Fallback Font），导致海报样式不一致。

**最佳实践**：
```javascript
// 确保字体加载完毕再截图
await document.fonts.ready;
// 或者针对特定字体
await document.fonts.load('16px "MyCustomFont"');
```

Snapdom 会自动将使用到的字体文件转为 Base64 并嵌入到 SVG 的 `<style>` 标签中，确保生成的图片在任何地方打开都能显示正确的字体。

### 6.3 移动端 (iOS/Android) 的特殊怪癖

1.  **iOS Safari 内存限制**：
    *   iOS 对 Canvas 的总内存占用有限制（通常是 224MB - 288MB，视机型而定）。
    *   如果生成的图片过大（如长图，高度超过 4000px），Canvas 可能会崩溃或变为空白。
    *   **对策**：降低 `scale`，或者分片截图（将长图切成几段分别生成）。

2.  **Android WebView 版本**：
    *   老旧的 Android WebView 可能不支持 SVG `<foreignObject>`。
    *   **对策**：在这种情况下，只能降级使用 `html2canvas`。可以通过 UserAgent 判断。

### 6.4 大图生成与分片渲染策略

对于超长页面（如电商详情页），一次性生成可能会导致浏览器卡死。

**分片渲染思路**：
1.  计算总高度。
2.  按屏幕高度切分 DOM（或逻辑切分）。
3.  循环调用 Snapdom 生成每一屏的图片。
4.  创建一个新的大 Canvas，将这些小图片拼接起来。
5.  导出大图。

---

## 7. 未来展望：原生 API 的崛起

### 7.1 Element Capture API

Chrome 团队正在推进 **Element Capture API**。这允许 Web 应用捕获特定 DOM 元素的视频流。虽然主要用于视频会议分享特定 Tab，但理论上也可以提取单帧作为截图。

### 7.2 浏览器原生截图能力的开放

目前浏览器扩展（Extension）可以使用 `chrome.tabs.captureVisibleTab` 获得原生级别的截图能力。Web 开发者一直呼吁开放类似的 API 给普通网页。如果未来实现，`html2canvas` 和 `snapdom` 都将成为历史。

---

## 8. 总结与选型决策树

在 2025 年的项目中，如何选择？

1.  **首选 Snapdom (@zumer/snapdom)**：
    *   只要你的目标用户不是 IE 用户。
    *   只要你需要还原现代 CSS 效果（圆角、阴影、渐变、滤镜）。
    *   它更快、更准、更现代。

2.  **备选 Html2Canvas**：
    *   如果必须兼容 IE11。
    *   如果你的页面结构非常简单（无复杂 CSS），且需要极高的稳定性（毕竟它经过了十年的考验）。
    *   如果你需要在截图过程中进行非常复杂的逻辑干预（`onclone`）。

3.  **混合策略**：
    *   检测浏览器是否支持 `foreignObject`。
    *   支持则用 Snapdom，不支持则降级到 Html2Canvas。

**最终建议**：不要畏惧新技术。Snapdom 带来的开发体验提升是巨大的。告别手动调整 Canvas 坐标的痛苦，拥抱“所见即所得”的快乐吧！

## 9. 源码仓库

想尝试的小伙伴们，可以去代码仓库拉取代码到本地体验一下，仓库链接我放在下方啦：

👉 **GitHub 仓库地址**：[https://github.com/summer-zhp/generate_poster](https://github.com/summer-zhp/generate_poster)

欢迎 Star ⭐️ 和 Fork！
