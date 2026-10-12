# CSS 编码风格完整示例

> 收录各主题完整示例与市面高频正反例（来源：CSS Guidelines、Google HTML/CSS Style Guide、web.dev 常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. BEM 命名完整示例
- 2. 属性顺序完整示例
- 3. 选择器正反例（过度限定与 !important）
- 4. Flex/Grid 布局经典（居中、栅格、gap）
- 5. 响应式断点示例
- 6. 主题变量与 dark mode
- 7. 常见正反例速查

## 1. BEM 命名完整示例

组件卡片从结构到状态的全套命名：

```css
/* Block：组件根 */
.model-card { … }

/* Element：仅属于该组件的子节点（双下划线连接） */
.model-card__title { … }
.model-card__body { … }
.model-card__footer { … }

/* Modifier：状态或变体（双连字符连接） */
.model-card--compact { … }
.model-card__title--warning { … }
```

```html
<article class="model-card model-card--compact">
  <h3 class="model-card__title model-card__title--warning">标题</h3>
  <div class="model-card__body">…</div>
  <footer class="model-card__footer">…</footer>
</article>
```

❌ 常见反例：

```css
.red { color: red; }              /* 按外观命名：变成绿色就是谎言 */
.card .title h3 span { … }        /* 过度限定，优先级通胀 */
.cardTitle { font-size: 14px; }   /* 驼峰混入连字符体系 */
```

## 2. 属性顺序完整示例

按"布局 → 盒模型 → 排版 → 视觉"分组：

```css
.model-card {
  /* 布局 */
  display: flex;
  flex-direction: column;
  position: relative;
  gap: var(--spacing-2);

  /* 盒模型 */
  width: 100%;
  padding: var(--spacing-4);
  border: 1px solid var(--color-border);

  /* 排版 */
  font-size: var(--font-size-md);
  line-height: 1.5;
  color: var(--color-text);

  /* 视觉 */
  background: var(--color-bg);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
  transition: box-shadow 0.2s ease;
}
```

## 3. 选择器正反例（过度限定与 !important）

优先级扁平化：

```css
/* ❌ ID + 深层链：后续无法覆盖，复用为零 */
#app .content ul.model-list li a.link.active { color: var(--color-primary); }

/* ✅ 一层类 */
.model-list__link--active { color: var(--color-primary); }
```

!important 的唯一豁免场景（覆盖第三方且注明原因）：

```css
/* ❌ 用 !important 打架 */
.title { font-size: 18px !important; }

/* ✅ 第三方内联样式无解时，注明原因并隔离作用域 */
.third-party-widget .title {
  /* 覆盖 xx 库 1.4.0 内联样式，无其他手段，升级后移除 */
  font-size: 18px !important;
}
```

## 4. Flex/Grid 布局经典（居中、栅格、gap）

水平垂直居中：

```css
/* ❌ 老三样 */
.center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* ✅ Grid 两行（容器自身控制） */
.center {
  display: grid;
  place-items: center;
}
```

卡片列表：gap 替代 margin 兄弟 hack：

```css
/* ❌ */
.card + .card { margin-top: 16px; }
.card:last-child { margin-top: 0; }

/* ✅ */
.card-list {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-4);
}
```

二维栅格用 Grid：

```css
/* ❌ float 三栏 + clearfix */
/* ✅ */
.grid-3 {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: var(--spacing-4);
}
```

## 5. 响应式断点示例

移动优先 + 语义令牌断点：

```css
/* ❌ 桌面优先 + 散落魔法数 */
.sidebar { width: 260px; }
@media (max-width: 767px)  { .sidebar { display: none; } }
@media (max-width: 1023px) { .sidebar { width: 200px; } }

/* ✅ */
:root {
  --bp-md: 768px;
  --bp-lg: 1024px;
}

.sidebar { width: 100%; }                       /* 移动端基线 */
@media (min-width: var(--bp-md)) {
  .sidebar { width: var(--sidebar-w); }
}
@media (min-width: var(--bp-lg)) {
  .sidebar { width: 260px; }
}
```

## 6. 主题变量与 dark mode

令牌化是 dark mode 的前提——切换主题只改变量，不写第二套选择器：

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1f2328;
  --color-border: #d0d7de;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0d1117;
    --color-text: #e6edf3;
    --color-border: #30363d;
  }
}

/* 组件只消费令牌 */
.card {
  background: var(--color-bg);
  color: var(--color-text);
  border: 1px solid var(--color-border);
}
```

❌ 组件内硬编码色值（dark mode 下必然漏改）：

```css
.card { background: #fff; color: #333; }
```

## 7. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 全局盒模型混乱 | 默认 content-box | `*, *::before, *::after { box-sizing: border-box; }` |
| z-index 军备竞赛 | `9999` / `99999` | 令牌刻度 `--z-modal: 200` |
| 过渡全部属性 | `transition: all 0.3s` | 明确列出 `transition: opacity 0.2s` |
| 动画触发重排 | 动画 `width` / `top` | 动画 `transform` / `opacity` |
| 串行加载 | `@import url(...)` | `<link rel="stylesheet">` |
| 零值带单位 | `margin: 0px` | `margin: 0` |
| 颜色透明度拼接 | `rgba(0,0,0,.5)` 硬编码 | 令牌 / `rgb(0 0 0 / 50%)` |
| ID 做样式挂载 | `#submit-btn { … }` | `.submit-btn { … }` |
| 行内样式散落 | `style="margin-top: 8px"` | class + 令牌间距 |
| 深层嵌套 SCSS | 五层嵌套 | 最多 2~3 层，扁平类名 |
