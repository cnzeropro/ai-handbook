---
name: "css-coding-standard"
description: "CSS 编码规范：命名与组织（BEM 默认）、选择器与优先级、属性顺序、布局（Flex/Grid）、响应式、主题变量与性能。当编写、重构或审查 CSS/SCSS 样式代码、编写 Vue/React 组件样式时使用。凡涉及 CSS 代码编写与修改的任务均应主动使用。"
---

# CSS 编码规范

本 skill 提供 CSS 语言层面的编码规范，融合 Google HTML/CSS Style Guide、CSS Guidelines（Harry Roberts）、MDN 与 web.dev 最佳实践，适用于任意 CSS/SCSS——包括框架组件内的样式（Vue SFC `style` 块、CSS Modules 等）。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。
>
> 与其他 skill 的关系：标记结构见 `html-coding-standard`；脚本行为见 `js-coding-standard`；框架层组件设计见 `vue-coding-standard` / `react-coding-standard`。

## 何时使用

- 编写、重构或审查任意 CSS/SCSS（全局样式、组件样式、Vue SFC `style` 块）
- 确定命名方案、属性顺序、布局方案、响应式与主题实现时
- 代码审查时作为样式规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 CSS 及预处理器产物（SCSS/Less 的通用规则部分）
- 框架组件内样式：Vue SFC `style` 块、CSS Modules、内联样式（动态样式按框架约定）

### 不适用（边界）

- 标记结构（参见 `html-coding-standard` skill）
- 交互逻辑与脚本行为（参见 `js-coding-standard` skill）
- 组件设计与状态（参见 `vue-coding-standard` / `react-coding-standard` skill）
- 工具类框架（Tailwind 等）的类名组合约定——使用此类框架时**从其官方约定**，本 skill 的命名/属性规则仅约束自写样式
- 缩进、换行、属性行序强制化等纯排版格式，遵循项目工具链（Stylelint/Prettier）配置

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- 命名：class 小写连字符（`.model-card`），按**用途**命名（`.warning` 而非 `.red`）；**BEM 为默认方案**（`block__element--modifier`），Tailwind/CSS Modules 项目从其约定
- 选择器：类选择器为主；扁平化优先级，嵌套最多 2~3 层；禁 `!important`（第三方覆盖场景除外并注明）；不用 `@import`（用 `<link>`）
- 属性顺序：布局（display/position/盒模型）→ 排版（font/color/text）→ 视觉（background/border/shadow/animation）；项目有 Stylelint 插件排序时以工具链为准
- 布局：Flex/Grid 优先，禁 float 布局与 inline-block hack；兄弟间距用 `gap` 而非 `margin` 兄弟选择器；全局 `box-sizing: border-box`
- 响应式：移动优先（`min-width` 断点）；断点用语义 token（`--bp-md` 等或 SCSS 变量）禁散落魔法数；字号间距用相对单位（rem）
- 主题与令牌：颜色、间距、圆角、阴影 token 化为自定义属性；dark mode 用 `prefers-color-scheme` + 变量切换
- 书写细节：`0` 不带单位；颜色用 token/变量或小写 hex；`url()` 带引号；能用简写用简写；禁行内样式与 `<style>` 散落（动态样式按框架约定）
- 版本适配：CSS 无整版概念、按模块演进；以项目 browserslist / 目标浏览器为准决定现代特性（容器查询、`:has` 等）使用范围，兼容处理交给构建链（autoprefixer 等）而非手写 hack

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：命名与组织、选择器与优先级、属性顺序、布局与盒模型、响应式、主题令牌、性能各章完整规则
- `references/style-examples.md` — 编码风格完整示例与市面常见正反例速查（BEM 全解、居中方案、dark mode、z-index 管理等）
