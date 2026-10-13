# Tailwind 编码风格完整示例

> 收录各主题完整示例与市面高频正反例（来源：Tailwind 官方文档与社区审查常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 动态类名安全（映射表方案）
- 2. 类名排序与换行
- 3. 组件抽取（variant / size props）
- 4. 响应式与暗色完整示例
- 5. @apply 使用边界
- 6. 任意值升级令牌
- 7. 常见正反例速查

## 1. 动态类名安全（映射表方案）

React：

```tsx
// ❌ 拼接半截类名：扫描器识别不到，产物缺失
const color = type === 'success' ? 'green' : 'red';
<span className={`text-${color}-600`} />

// ✅ 完整类名映射表
const statusClass = {
  success: 'text-green-600',
  error: 'text-red-600',
  pending: 'text-amber-600',
} as const;

<span className={statusClass[type]} />
```

Vue：

```vue
<!-- ❌ -->
<p :class="`text-${type}-600`" />

<!-- ✅ -->
<script setup lang="ts">
const statusClass: Record<Status, string> = {
  success: 'text-green-600',
  error: 'text-red-600',
};
</script>
<template>
  <p :class="statusClass[type]" />
</template>
```

复杂条件组合用 `clsx` + `tailwind-merge`：

```tsx
// ❌ 长三元拼接，冲突类无法合并
<div className={`px-4 py-2 ${primary ? 'bg-blue-600' : 'bg-slate-600'} ${small ? 'text-sm' : 'text-base'}`} />

// ✅
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

<div className={twMerge(clsx(
  'px-4 py-2',
  primary ? 'bg-blue-600' : 'bg-slate-600',
  small && 'text-sm',
))} />
```

## 2. 类名排序与换行

```html
<!-- ❌ 顺序随机、单行挤压 -->
<article class="text-sm shadow-sm flex flex-col bg-white p-4 w-full rounded-lg gap-4 sm:p-6">

<!-- ✅ 布局 → 盒模型 → 排版 → 视觉；多属性逐行 -->
<article
  class="flex w-full flex-col gap-4 rounded-lg bg-white p-4 shadow-sm sm:p-6"
>
  …
</article>
```

（启用 `prettier-plugin-tailwindcss` 后以自动排序为准。）

## 3. 组件抽取（variant / size props）

```tsx
// ❌ 三处复制同一串类名
<button className="inline-flex items-center gap-2 rounded-md px-4 py-2 text-sm font-medium bg-blue-600 text-white hover:bg-blue-700">…</button>

// ✅ 抽取为组件，变体进 props（完整类名映射表）
interface BaseButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md';
  children: React.ReactNode;
}

const variantClass = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'bg-slate-200 text-slate-800 hover:bg-slate-300',
} as const;
const sizeClass = { sm: 'px-3 py-1.5 text-xs', md: 'px-4 py-2 text-sm' } as const;

function BaseButton({ variant = 'primary', size = 'md', children }: BaseButtonProps) {
  return (
    <button
      type="button"
      className={twMerge(clsx(
        'inline-flex items-center gap-2 rounded-md font-medium',
        variantClass[variant],
        sizeClass[size],
      ))}
    >
      {children}
    </button>
  );
}
```

抽取判定：相同类组合重复 ≥ 3 次即抽取；只出现一次的保留内联。

## 4. 响应式与暗色完整示例

移动优先 + 统一变体顺序（响应式 → 暗色 → 状态）：

```html
<!-- ✅ -->
<div
  class="grid gap-4 p-4
         sm:grid-cols-2 sm:p-6
         lg:grid-cols-3
         dark:bg-slate-900
         hover:shadow-md focus-visible:ring-2"
>
  …
</div>
```

暗色令牌化（v4 @theme / v3 config）后组件只消费令牌，dark: 仅出现在令牌定义处：

```css
/* v4：令牌层处理暗色 */
@theme {
  --color-surface: #ffffff;
}
@media (prefers-color-scheme: dark) {
  :root { --color-surface: #0d1117; }
}
```

```html
<!-- ✅ 组件内不再出现 dark:bg-... -->
<div class="bg-surface text-text">…</div>
```

## 5. @apply 使用边界

```css
/* ❌ 业务组件内 @apply：退回"类名即 CSS"老路 */
.card {
  @apply flex flex-col gap-4 rounded-lg bg-white p-4;
}

/* ✅ 第三方节点覆盖：无组件化抓手时的唯一场景，注释原因 */
/* 覆盖 markdown-it 渲染的富文本链接样式，无法加类名 */
.prose a {
  @apply text-primary underline;
}
```

## 6. 任意值升级令牌

```html
<!-- ❌ 魔法任意值 -->
<div class="mt-[17px] w-[347px] text-[#1f2328]">

<!-- ✅ 刻度 + 令牌 -->
<div class="mt-4 w-80 text-primary">
```

同一任意值出现 ≥ 2 次即升级为主题令牌（v4 在 `@theme` 增加变量；v3 在 `theme.extend`）。

## 7. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 动态拼接类名 | `text-${color}-600` | 完整类名映射表 |
| 类名冲突叠加 | 手工长三元 | `clsx` + `tailwind-merge` |
| 组件内 @apply | `.card { @apply … }` | 抽取组件；@apply 仅限第三方覆盖 |
| 裸删焦点环 | `outline-none` | `outline-none focus-visible:ring-2` |
| 魔法任意值 | `w-[347px]` | 刻度 / 令牌 / 确需则注释 |
| 变体顺序混用 | `hover:sm:…` 与 `sm:hover:…` 混杂 | 统一"响应式 → 暗色 → 状态" |
| 视觉隐藏 | `hidden`（读屏也消失） | `sr-only` |
| 响应式桌面优先 | `max-md:` 堆叠 | 移动优先 `sm:` 向上增强 |
| 语义靠颜色 | 红色即错误（色盲不可辨） | 颜色 + 图标/文本（配合 a11y） |
| content 漏扫（v3） | 动态引入的新目录类缺失 | content 数组覆盖模板路径 |
