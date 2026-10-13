# Tailwind v4 规范（CSS-first，当前主版本）

> 本文件为 **Tailwind v4** 版本特有约定（CSS-first 配置、Oxide 引擎），须与版本通用的 `general-quality.md` 配合使用。v3 配置驱动见 `tailwind3.md`。

## 一、CSS-first 配置

- 引入方式单行化：CSS 中 `@import "tailwindcss";`，**无需** `tailwind.config.js` 与 `@tailwind base/components/utilities` 三段指令
- 遗留配置可临时用 `@config "../../tailwind.config.js";` 兼容，迁移完成后移除

```css
/* ✅ v4 入口 CSS */
@import "tailwindcss";

@theme {
  --color-primary: #2563eb;
  --color-surface: #ffffff;
  --font-sans: "Inter", system-ui, sans-serif;
}
```

## 二、@theme 令牌

- `@theme` 中定义的变量**自动生成工具类**：`--color-primary` → `text-primary` / `bg-primary` / `border-primary`；`--spacing` 驱动全部间距刻度
- 主题变量即 CSS 变量：运行时可直接引用 `var(--color-primary)`，暗色模式只覆盖变量值
- 命名用语义层（`primary` / `danger` / `surface`），不暴露原始色名

## 三、自动内容检测

- v4 自动扫描项目内模板文件，**无需** `content` 配置
- 需要显式排除（如 `node_modules` 内示例）用 `@source not "../legacy";`；二进制/生成目录默认已忽略

## 四、引擎与工程

- Oxide 引擎：全量构建速度大幅提升；`lightningcss` 承担前缀与压缩，无需另配 autoprefixer
- 新增原生变体与单位支持：容器查询（`@container` / `@min-*`）、`size-*`、`text-nowrap` 等，优先用原生工具类替代自写 CSS

## 五、v3 → v4 迁移清单

1. 执行官方迁移工具：`npx @tailwindcss/upgrade`
2. 配置迁移：`tailwind.config.js` 的 `theme.extend` → `@theme`；`darkMode: 'class'` → `@custom-variant dark`
3. CSS 变量形态：令牌即 `var(--…)`，检查 JS 中读取 Tailwind 配置值的代码改为读取 CSS 变量
4. 边缘差异：`shadow-sm` → `shadow-xs` 等刻度改名（迁移工具自动处理，人工复核视觉）
