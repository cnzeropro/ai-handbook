# Tailwind v3 规范（配置驱动）

> 本文件为 **Tailwind v3** 版本特有约定（JS 配置驱动），须与版本通用的 `general-quality.md` 配合使用。v4（CSS-first）见 `tailwind4.md`。

## 一、配置文件

- 配置在 `tailwind.config.js` / `tailwind.config.ts`；`content` 数组必须**准确覆盖**所有含类名的模板文件——漏扫 = 类缺失，多扫 = 产物膨胀

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx,vue}'],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        primary: { DEFAULT: '#2563eb', … },
        surface: { light: '#ffffff', dark: '#0d1117' },
      },
    },
  },
};
```

- `content` 禁用 `"./**/*"` 式全量通配（扫描产物与依赖，产物膨胀且慢）

## 二、主题扩展与令牌

- 颜色 / 间距 / 圆角 / 阴影在 `theme.extend` 中令牌化，命名用**语义层**（`primary` / `danger` / `surface`），不暴露原始色名（禁 `blue-500` 覆盖为品牌色这种语义漂移）
- 扩展后工具类自动生成：`text-primary`、`bg-surface-dark`
- 覆盖默认刻度（`theme.spacing` 整体替换）需谨慎——优先 `extend` 增量

## 三、暗色模式策略

- `darkMode: 'media'`（跟随系统，零 JS）或 `'class'`（手动切换，需在根元素挂 `dark` 类）——按产品需求选定并全局统一
- 令牌化后 dark 变体只覆盖变量，组件内不写两套色值（见 `general-quality.md` 五）

## 四、自定义扩展

- 自定义工具类用插件 `addUtilities` / `matchUtilities`（带值参数），集中注册；业务组件内禁 `@apply`
- 第三方组件深度覆盖可用重要修饰符 `!` 前缀（如 `!text-primary`）——极少数场景，注释原因

## 五、v3 → v4 迁移提示

- 迁移工具：`npx @tailwindcss/upgrade`；主要差异是**配置方式**（JS → CSS-first）而非工具类写法
- 迁移清单与 v4 写法见 `tailwind4.md`
