---
name: "tailwind-coding-standard"
description: "Tailwind CSS 编码规范，按版本组织（v4 CSS-first / v3 配置驱动）：类名组织与排序、组件抽取边界、动态类名安全、响应式与暗色变体、设计令牌。当编写、重构或审查使用 Tailwind 的界面代码时使用。凡涉及 Tailwind 样式编写的任务均应主动使用。"
---

# Tailwind CSS 编码规范

本 skill 提供 Tailwind CSS 编码规范，按版本组织（v4 CSS-first 当前主版本 / v3 配置驱动仍在用），融合 Tailwind 官方文档与社区维护最佳实践（CSS Guidelines 精神），适用于任意使用 Tailwind 的项目（React / Vue / 原生 HTML）。标记语义见 `html-coding-standard`，基础 CSS 语言规则见 `css-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查使用 Tailwind 工具类的界面代码（React/Vue 组件、模板、页面）
- 设计类名组织方案（长类名换行、抽取组件、条件类名组合）
- 处理响应式、暗色模式、主题令牌时
- v3 升级 v4 的迁移评估

## 适用范围

### 适用对象（本规则适用于）

- 任意引入 Tailwind 的项目：React / Vue / 原生 HTML
- v4（CSS-first 当前主版本）与 v3（配置驱动，仍在大量使用）双版本

### 不适用（边界）

- 组件逻辑与状态（参见 `js-coding-standard` / `vue-coding-standard` / `react-coding-standard` skill）
- 标记语义与可访问性（参见 `html-coding-standard` skill）
- 自写原生 CSS 的语言级规则（参见 `css-coding-standard` skill；本项目若为"Tailwind + 少量自定义 CSS"混合，自定义部分仍遵循 css skill）
- 不使用 Tailwind 的项目（遵循项目既有样式方案）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **完整类名字面出现**：工具类必须在源码中完整字面出现，禁动态拼接（`text-${color}-500` ❌）——扫描器无法识别；动态需求用完整类名映射表
- **类名顺序约定**：布局 → 盒模型 → 排版 → 视觉 → 杂项；项目启用 `prettier-plugin-tailwindcss` 时以自动排序为准
- **抽取边界**：相同类组合重复 ≥ 3 次抽取为组件；抽取优先级"组件化 > `@apply`"，`@apply` 仅限第三方覆盖等极少数场景并注明
- **任意值克制**：`w-[347px]` 类任意值是设计失控信号——优先间距/颜色刻度与令牌
- **条件类名组合**：复杂条件用 `clsx` + `tailwind-merge`（按项目选型），不堆长三元
- **响应式移动优先**：基础样式为移动端，`sm:` / `md:` / `lg:` 向上增强
- **变体前缀顺序统一**：约定"响应式 → 暗色 → 交互状态"（`sm:dark:hover:…`）或项目既有顺序，全局一致不混用
- **可访问性**：仅视觉隐藏用 `sr-only`；禁裸删 `focus-visible` 焦点样式；语义仍由标签承担（配合 html skill）

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| Tailwind v4（当前主版本） | CSS-first：`@import "tailwindcss"` + `@theme` 令牌即 CSS 变量；自动内容检测；无需配置文件 | `references/tailwind4.md` |
| Tailwind v3（仍在用） | JS 配置驱动：`tailwind.config.js`（content 扫描 / theme.extend / darkMode 策略 / 插件） | `references/tailwind3.md` |

> 升级：官方提供 `npx @tailwindcss/upgrade` 迁移工具；v3 → v4 的主要差异在**配置方式**（JS 配置 → CSS-first）与主题令牌形态（配置值 → CSS 变量），日常工具类写法基本一致。

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：类名组织与排序、抽取边界、任意值克制、动态类名安全、变体顺序、可访问性
- `references/tailwind4.md` — v4 写法规约：CSS-first 配置、@theme 令牌、自动内容检测、迁移工具
- `references/tailwind3.md` — v3 写法规约：JS 配置、content 扫描、darkMode 策略、插件扩展
- `references/style-examples.md` — 编码风格完整示例与市面常见正反例速查（组件抽取、动态类名、响应式暗色、@apply 边界）
