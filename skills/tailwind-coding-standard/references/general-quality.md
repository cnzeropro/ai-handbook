# Tailwind 通用质量规范

> 本文件为 tailwind-coding-standard 的**版本通用**质量规范，适用于 Tailwind v3 与 v4（版本特有配置见 `tailwind3.md` / `tailwind4.md`）。来源：Tailwind 官方文档与 CSS Guidelines 精神。完整正反例见 `style-examples.md`（同目录）。

## 目录

- 一、完整类名字面出现（动态类名安全）
- 二、类名组织与排序
- 三、抽取边界：组件化优先于 @apply
- 四、任意值克制与设计令牌
- 五、响应式与变体顺序
- 六、可访问性
- 七、与自写 CSS 的边界

## 一、完整类名字面出现（动态类名安全）

Tailwind 依赖扫描源码中的**完整字面类名**按需生成 CSS。拼接半截类名是 Tailwind 第一大坑：

```tsx
// ❌ 扫描器识别不到 text-green-500 / text-red-500，产物缺失
function Status({ type }: { type: 'success' | 'error' }) {
  return <span className={`text-${type === 'success' ? 'green' : 'red'}-500`}>…</span>;
}

// ✅ 完整类名映射表：每个类名都字面出现在源码
const statusClass = {
  success: 'text-green-600',
  error: 'text-red-600',
} as const;

function Status({ type }: { type: keyof typeof statusClass }) {
  return <span className={statusClass[type]}>…</span>;
}
```

- 同理禁止：`w-${size}`、`p-${n}`、模板字符串拼前缀、运行时生成类名
- 动态"数量"用内联样式表达（`style={{ width: `${percent}%` }}`——百分比本就是连续值，不是设计刻度）

## 二、类名组织与排序

- 推荐启用 `prettier-plugin-tailwindcss` 自动排序（官方维护），人工无需纠结顺序
- 无插件时按"布局 → 盒模型 → 排版 → 视觉 → 杂项"手写排序，与 css-coding-standard 的属性顺序同构：

```html
<!-- ✅ 布局 → 盒模型 → 排版 → 视觉 -->
<article class="flex flex-col gap-4 w-full p-4 rounded-lg font-medium text-slate-700 bg-white shadow-sm">
```

- 多属性元素逐行一个属性（与框架规则一致），长类名列表换行不挤压

## 三、抽取边界：组件化优先于 @apply

- 相同类组合**重复 ≥ 3 次**：抽取为组件（React 组件 / Vue 组件），调用方传内容——这是 Tailwind 官方推荐的同义反复消除方式
- `@apply` 仅限极少数场景：无法组件化的第三方节点覆盖、全局富文本内容（`.prose` 内元素）——并注释原因；**业务组件内禁用 @apply**（等于退回"类名即 CSS"的老路，丢掉工具类的可维护性）

```vue
<!-- ❌ 三处复制同一串类名 -->
<button class="inline-flex items-center gap-2 rounded-md px-4 py-2 text-sm font-medium bg-blue-600 text-white hover:bg-blue-700">…</button>

<!-- ✅ 抽取为组件 -->
<!-- BaseButton.vue：内部持有这串类名 -->
<BaseButton @click="submit">…</BaseButton>
```

- 抽取时按"变体进 props"设计：`<BaseButton variant="primary" size="sm">`，内部用完整类名映射表

## 四、任意值克制与设计令牌

- 任意值（`w-[347px]`、`text-[#1f2328]`）是设计失控信号：优先刻度（`w-72`、`text-slate-700`）与主题令牌
- 确需任意值：视觉还原稿的确切值、一次性特殊尺寸——并在组件处可追溯；同一任意值出现 ≥ 2 次就应升级为主题令牌
- 主题令牌优先：`text-primary` / `bg-surface` / `border-border`（v4 在 `@theme` 定义，v3 在 `config.theme.extend`）——换肤与暗色只动令牌
- 间距语义化：`gap-4` 优于 `gap-[17px]`；刻度不满足时先问设计能否归位刻度

## 五、响应式与变体顺序

- **移动优先**：无前缀类是移动端基线，`sm:` / `md:` / `lg:` / `xl:` 向上增强
- 变体前缀顺序全局统一（推荐）：**响应式 → 暗色 → 交互状态**：

```html
<!-- ✅ 统一顺序：sm: → dark: → hover: -->
<div class="bg-white sm:bg-slate-50 dark:bg-slate-900 hover:bg-slate-100 sm:dark:hover:bg-slate-800">
```

- `hover:` / `focus-visible:` / `disabled:` / `group-hover:` 优先用状态变体表达，不用 JS 切换类名
- `dark:` 暗色策略（media / class）按产品需求选定并全局统一；令牌化颜色后 dark 只覆盖变量

## 六、可访问性

- 仅视觉隐藏用 `sr-only`（读屏可读、视觉不可见）；`hidden` 是彻底移除
- 禁裸删 `focus-visible` 焦点环（`outline-none` 必须伴随替代焦点样式，如 `focus-visible:ring-2`）
- 交互语义不由样式承担：可点击仍用 `button` / `a`（配合 `html-coding-standard`）

## 七、与自写 CSS 的边界

- 业务样式默认 Tailwind 工具类；仅以下场景写原生 CSS（全局文件或 SFC `style` 块，遵循 `css-coding-standard`）：
  1. 全局 reset 与令牌定义
  2. 第三方组件的深度覆盖（配合 CSS 作用域方案）
  3. 复杂动画 keyframes、选择器无法表达的伪元素组合
- 引入第三方组件库（自带样式体系）时：以该库的定制方案为准，不与 Tailwind 工具类在其内部混战
