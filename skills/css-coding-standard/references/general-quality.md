# CSS 通用质量规范

> 本文件为 css-coding-standard 的通用质量规范，适用于任意 CSS/SCSS 及框架组件内样式。规则来源：Google HTML/CSS Style Guide、CSS Guidelines（Harry Roberts）、MDN、web.dev。完整正反例见 `style-examples.md`（同目录）。

## 一、命名与组织

- class 名小写连字符：`.model-card`、`.user-avatar`；禁驼峰/下划线混用（项目既有约定除外）
- 按**用途与语义**命名，不按外观：`.warning` 优于 `.red`、`.primary-btn` 优于 `.blue-btn`——外观变了名字就是谎言
- **BEM 为默认命名方案**：`block__element--modifier`（`.model-card__title--highlight`）
  - 收益：天然作用域化、避免全局冲突、删除安全（一眼看出归属）
  - 使用 Tailwind / CSS Modules / styled-components 等方案的项目**从其官方约定**，不强行套 BEM
- 禁用 ID 选择器做样式挂载（ID 留给锚点与 JS）；样式挂载一律 class
- 文件组织：按组件/页面拆分，公共层分离（`variables` / `base` / `mixins` / `utilities`）；避免单一巨型 `main.css`

```css
/* ❌ 按外观 + 无归属 */
.red-font-small { color: #c00; font-size: 12px; }
#mainBox .list div span { … }

/* ✅ BEM + 语义 */
.model-card__status--warning {
  color: var(--color-danger);
  font-size: var(--font-size-sm);
}
```

## 二、选择器与优先级

- 类选择器为主；保持优先级**扁平**——高优先级是维护噩梦的根源
- 嵌套（SCSS）最多 2~3 层，超出说明组件边界有问题
- 禁止过度限定：`.nav ul li a.nav-link` → `.nav-link`（限定只表达必要约束）
- 禁 `!important`：它不能解决优先级问题，只能终结级联。唯一例外：覆盖第三方库且无其他手段，必须注释说明
- 不用 `@import`（串行加载阻塞），统一 `<link>` 引入
- 谨慎使用通配 `*` 与属性选择器 `[class^="…"]`（匹配开销与意外命中）
- 全局 reset 只做一层（现代方案 `*, *::before, *::after { box-sizing: border-box; }` + `margin: 0` 归零，或直接采用 modern-normalize）

```css
/* ❌ 优先级通胀 */
#app .content .list ul li a.item.active { color: red; }
.card .title { color: blue !important; }

/* ✅ 扁平 */
.card__title--active { color: var(--color-primary); }
```

## 三、属性顺序与书写

- 属性按"影响范围从大到小"分组书写（团队可用 Stylelint 插件强制）：
  1. 布局：`display` / `position` / `top` / `flex` / `grid` / `gap`
  2. 盒模型：`width` / `height` / `margin` / `padding` / `border`
  3. 排版：`font` / `line-height` / `color` / `text-align`
  4. 视觉：`background` / `border-radius` / `box-shadow` / `opacity` / `transition` / `animation`
- 能用简写用简写：`margin: 0 auto`、`border: 1px solid var(--color-border)`
- `0` 不带单位（`margin: 0`）；数值小数前导 0 写不写（`.5` / `0.5`）以项目 Stylelint 为准
- 颜色：优先令牌变量；字面量用小写 hex 短形式（`#fff`）；需要透明度用 `rgb(... / 50%)` 现代语法
- `url("…")` 带引号
- 禁行内样式与散落 `<style>`：动态计算样式按框架约定（`:style` / styled），静态样式一律进样式文件

```css
/* ❌ 顺序杂乱、魔法值 */
.card {
  color: #333;
  box-shadow: 0 2px 8px #0002;
  width: 300px;
  border-radius: 4.327px;
  display: flex;
}

/* ✅ */
.card {
  display: flex;
  width: var(--card-width);
  color: var(--color-text);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}
```

## 四、布局与盒模型

- 全局 `box-sizing: border-box`（见 reset），宽度计算所见即所得
- 一维排列用 Flex，二维网格用 Grid；**禁** float 布局、inline-block 排版 hack
- 兄弟元素间距用 `gap`（Flex/Grid 原生支持），替代 `margin` + 末项清除 hack
- 居中方案按场景：水平 `margin-inline: auto`；水平垂直 `display: grid; place-items: center`
- 垂直居中文字场景用 `line-height` 仅限单行按钮类；其余用 flex 对齐
- `z-index` 分层管理：定义令牌刻度（如 `--z-dropdown: 100` / `--z-modal: 200` / `--z-toast: 300`），禁散落魔法数（9、99、9999）

```css
/* ❌ */
.card + .card { margin-top: 27px; }
.modal { z-index: 9999; }

/* ✅ */
.card-list {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-4);
}
.modal { z-index: var(--z-modal); }
```

## 五、响应式设计

- **移动优先**：基础样式为移动端，向上用 `min-width` 断点增强
- 断点定义为语义令牌（SCSS 变量或自定义属性），全局共用一组：`sm(640) / md(768) / lg(1024) / xl(1280)`（数值按项目设计系统），禁散落魔法数
- 字号、间距优先相对单位 `rem` / `em`；容器自适应用 `%` / `fr` / `minmax`
- 文本不设固定高度；图片媒体声明宽高 + `max-width: 100%` 防溢出
- 新趋势按需采用：容器查询（`@container`）用于组件级响应式，优于全局断点

```css
/* ❌ 桌面优先 + 魔法断点 */
.card { width: 300px; }
@media (max-width: 767px) { .card { width: 100%; } }

/* ✅ 移动优先 + 令牌断点 */
.card { width: 100%; }
@media (min-width: var(--bp-md)) {
  .card { width: var(--card-width); }
}
```

## 六、主题令牌与 Dark Mode

- 设计令牌统一为 `:root` 自定义属性：颜色（语义层：`--color-primary` / `--color-danger`，不暴露原始色值命名如 `--blue`）、间距、圆角、阴影、z-index
- Dark mode：令牌化是前提——`@media (prefers-color-scheme: dark)` 中仅覆盖变量值，不写第二套选择器
- 组件内禁止绕过令牌硬编码色值（一次性图表色等例外，需注释）

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1f2328;
}
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0d1117;
    --color-text: #e6edf3;
  }
}
```

## 七、性能与工程

- 选择器开销：从右向左匹配——最右端用类，避免以 `*` / 标签链结尾的长选择器
- 谨慎高开销效果：大面积 `box-shadow` 动画、`filter: blur` 连续动画、无限 `transition: all`（明确列出属性：`transition: opacity 0.2s`）
- 布局抖动：动画优先用 `transform` / `opacity`（合成层），避免动画 `width` / `top`
- 组件级隔离按项目选型：Vue scoped / CSS Modules / Shadow DOM；全局样式只放真正全局的（reset、令牌、工具类）
- 删除死样式：重构时同步清理无引用样式（可用工具扫描），不留"以后可能用"
