# 首页横幅特效与全站背景实现文档

本站首页横幅的两个视觉效果——**Kimi 风格晕染圆球背景**（全站共享）和 **MiMo 风格鼠标揭示文字**（仅首页）——均为自定义实现，不修改 Redefine 主题源码，主题可正常升级。

## 文件结构

```
_config.redefine.yml      # inject 配置：把下面两个文件注入全站页面
source/css/custom.css     # 全部样式：背景层、圆球动画、揭示文字、响应式
source/js/custom.js       # DOM 注入、文字测量布局、鼠标跟随交互
```

`_config.redefine.yml` 中的注入配置：

```yaml
inject:
  enable: true
  head:
    - <link rel="stylesheet" href="/css/custom.css?v=10">
  footer:
    - <script src="/js/custom.js?v=10" defer></script>
```

> **修改 custom.css / custom.js 后必须把 `v=` 数字 +1**，否则浏览器会沿用旧缓存。

## 一、晕染圆球背景（全站）

### DOM 结构

JS 在 `document.body` 末尾注入：

```html
<div class="aurora-global">      <!-- fixed 全屏层 -->
  <div class="aurora-bg">
    <div class="aurora-blob b1"></div>   <!-- 雾蓝 -->
    <div class="aurora-blob b2"></div>   <!-- 樱粉 -->
    <div class="aurora-blob b3"></div>   <!-- 天青 -->
    <div class="aurora-blob b4"></div>   <!-- 淡黄 -->
  </div>
</div>
```

### 核心 CSS 技术

- **圆球本体**：`radial-gradient(circle at ..., 颜色, transparent 65%)` + `border-radius: 50%` + `filter: blur(80px)`，大模糊让边缘完全柔化。
- **交汇晕染**：`mix-blend-mode: multiply`（浅色背景专用；深色底应换 `screen`）。两个圆球重叠时颜色相乘叠加，产生"洇开"的过渡。
- **漂移动画**：三个 `@keyframes`（26s / 32s / 38s 不同时长、不同方向），只动 `transform` 和 `scale`，不触发布局重算，性能友好。

### 层级方案（关键，踩过坑）

```
.aurora-global      position: fixed; z-index: 0     ← 背景层
main.page-container position: relative; z-index: 1  ← 内容层（背景必须透明！）
```

两个必须处理的遮挡：

1. `html, body` 自带 `background: var(--background-color)`，会盖住背景层 → 亮色模式下把 body 背景设为透明。
2. **主题的 `main.page-container` 自带不透明背景**，是内容层真正的遮盖源 → `main.page-container { background: transparent !important; }`。

### 其他细节

- 全站底色统一为 `#f7f6f3`：覆盖 CSS 变量 `--background-color`。因为本文件在 head 中**先于**主题 `style.css` 加载，选择器必须用 `html:not(.dark)` 提高优先级；不能用 `:root`（任何模式都匹配，会误伤暗色模式）。
- 暗色模式下 `html.dark .aurora-global { display: none }`，保持主题原生深色。
- `prefers-reduced-motion` 时圆球动画关闭。

## 二、鼠标揭示文字（首页）

### DOM 结构

JS 向首页横幅的 `.content` 注入 `.reveal-wrap`，内含**两层 SVG**：

```html
<div class="reveal-wrap">
  <svg class="reveal-layer reveal-base">   <!-- 底层：镂空中文，常驻可见 -->
    <text class="reveal-prefix" text-anchor="end">你好，我是&nbsp;</text>
    <text class="reveal-name" text-anchor="start">laimanxi</text>
  </svg>
  <svg class="reveal-layer reveal-top">    <!-- 顶层：实心英文，圆孔内可见 -->
    <text class="reveal-prefix" text-anchor="end">HELLO，I'M&nbsp;</text>
    <text class="reveal-name" text-anchor="start">laimanxi</text>
  </svg>
</div>
```

### 对齐原理：laimanxi 像素级重合

每行拆成"前缀 + 名字"两段 `<text>`：

- 前缀 `text-anchor="end"`（右缘对齐锚点）
- 名字 `text-anchor="start"`（左缘对齐锚点）
- 两层的锚点 x 坐标相同

因此**两层的 laimanxi 从同一 x 坐标起笔，字形、位置完全一致**。布局规则：中文行在横幅中居中，英文行以 laimanxi 为锚，HELLO 部分允许向左超出中文行左缘。

### 镂空描边原理

不用 `-webkit-text-stroke`（细笔画两侧都出线，汉字笔画交汇处会叠出难看的双线），而是：

```css
.reveal-base text {
  fill: var(--banner-bg);      /* 填充色 = 底色 */
  stroke: rgba(30,30,30,0.6);
  stroke-width: 2.2px;
  paint-order: stroke;         /* 先画描边，再画填充 */
  stroke-linejoin: round;
}
```

`paint-order: stroke` 让描边画在填充下层，填充把描边的内半圈盖掉，**只留字形外轮廓一圈干净线条**。

### 揭示圆孔原理

两层各带一个 CSS mask，由三个 CSS 变量驱动（`--mx`/`--my` 圆心、`--r` 半径）：

```css
/* 顶层：只有圆孔内可见 */
.reveal-top {
  mask-image: radial-gradient(circle var(--r) at var(--mx) var(--my), #000 55%, transparent 78%);
}
/* 底层：圆孔内挖空，避免两层叠字 */
.reveal-base {
  mask-image: radial-gradient(circle var(--r) at var(--mx) var(--my), transparent 52%, #000 78%);
}
```

JS 监听横幅的 `mousemove`，用 `requestAnimationFrame` 做线性插值缓动（位置 0.18、半径 0.12），圆孔平滑跟随；`mouseleave` 时半径缓动收缩为 0，圆孔消失。

### JS 布局测量（为什么不能纯 CSS）

SVG 尺寸由 JS 实测后显式设定，否则百分比宽度在收缩布局父容器中会算出错误值：

1. `getComputedTextLength()` 测出前缀和名字的真实渲染宽度
2. 计算 `width` / `height` / `viewBox` 并设置到 SVG 上
3. 英文前缀更宽时，`viewBox` 向左扩展 `leftOver`，并用 `margin-left: -leftOver/2` 补偿，保证中文行居中
4. **字体是异步加载的**：必须用 `document.fonts.load(...)` 显式请求加载完成后重新测量（`fonts.ready` 可能在字体使用前就已 resolve，按回退字体测量会导致裁切）；另加 `resize` 监听和 1.2s 延时兜底

### 遮罩坐标系

`--mx`/`--my` 必须相对 **SVG 自身**（`base.getBoundingClientRect()`）计算，不能相对 wrap——窄屏缩放或容器留白时两者不一致，圆孔会偏移。

## 三、降级与兼容

| 场景 | 行为 |
|---|---|
| 触屏设备（`hover: none` / `pointer: coarse`） | 静态模式：中文、英文上下两行各自居中，无鼠标交互 |
| `prefers-reduced-motion` | 圆球静止 + 静态双行文字 |
| 窄屏 | SVG `max-width: 100%` + `viewBox` 按比例整体缩小，文字永不裁切 |
| swup 单页导航 | 监听 `swup:contentReplaced` 等事件重新初始化，回首页效果不丢 |
| 暗色模式 | 圆球层隐藏，主题原生深色不受影响 |

## 四、字体

- 中文：**Noto Sans SC 900**（最粗字重），经 `fonts.loli.net`（谷歌字体国内镜像）按字形分片加载
- 英文：**Chillax-Variable**（Redefine 主题自带，零额外加载）

## 五、踩坑记录（排障备忘）

1. **主题 rem 基准约 10px**：`6rem` 实际只有 60px，字号相关一律用 `px`/`vw`。
2. **inject 的 CSS 先于主题 style.css 加载**：同优先级必输，要么提高选择器优先级，要么 `!important`。
3. **hexo server 从内存路由提供页面**：直接改 `public/` 下的文件不会生效，必须改 `source/` 重新生成。
4. **localhost 统计数字离谱是正常的**：vercount 按域名记账，本地测试共用 "localhost" 账本；线上域名数据真实。
5. **SVG 文字裁切**：没有 `viewBox` 的 SVG 在收缩布局中宽度会算错；务必 JS 实测后显式设置尺寸。
6. **缓存**：custom.css/js 改名不现实，用 URL 版本号参数（`?v=N`）强制刷新。
