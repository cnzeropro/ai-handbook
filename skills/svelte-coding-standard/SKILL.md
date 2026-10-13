---
name: "svelte-coding-standard"
description: "Svelte 编码规范（Svelte 5 Runes 现行 / Svelte 4 遗留分册）：组件结构与命名、Runes 状态体系（$state/$derived/$effect）、props 与双向绑定、事件属性、a11y 编译器警告修复、Snippet 与副作用边界。当编写、重构或审查 Svelte 代码时使用。凡涉及 Svelte 代码编写与修改的任务均应主动使用。"
---

# Svelte 编码规范

本 skill 提供 Svelte 框架层编码规范，覆盖 Svelte 5 Runes 与 Svelte 4 两代写法体系，规则来源为 Svelte 官方文档与编译器 a11y 警告体系。标记语义见 `html-coding-standard`，TS 类型层见 `ts-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查 Svelte 组件（`.svelte` 单文件组件、`.svelte.js` 共享状态模块）
- 设计组件状态（Runes）、props 与双向绑定、事件处理
- 代码审查时作为 Svelte 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- Svelte 5 项目：Runes 语法、snippet、事件属性写法
- Svelte 4 遗留项目：保持既有写法一致（`let` 响应式、`$:` 语句、`on:` 指令），见 `svelte-legacy.md`；升级以官方迁移指南为准

### 不适用（边界）

- 标记语义与属性写法（参见 `html-coding-standard` skill）
- 样式规则（Svelte 默认 scoped；通用 CSS 规则参见 `css-coding-standard` skill）
- JS 运行时语法与 TS 类型层（参见 `js-coding-standard` / `ts-coding-standard` skill）
- SvelteKit 框架约定（路由/服务端约定跟随官方文档，组件层规范适用本 skill）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **修复而非抑制**：编译器 a11y 警告必须修复（`svelte-ignore` 仅限确有理由处并注释）
- **状态用 Runes**：`$state`（响应式状态）、`$derived`（派生优先于手工同步）、`$effect`（仅与外部系统同步，能不用就不用）
- **props**：`let { … } = $props()` 解构 + 默认值 + 类型标注；双向绑定用 `$bindable`
- **事件用属性**：`onclick={handler}`（标准 DOM 属性形态），不用 Svelte 4 的 `on:click`
- **复用标记用 snippet**：`{#snippet}` + `{@render}` 替代 slot（子内容透传仍可用隐式 `children`）
- **组件命名**：PascalCase 文件名（`ModelCard.svelte`）；实例导出默认导出组件
- **共享状态**：`.svelte.js` 模块 + 导出 runes 的工厂函数（`createModelStore`），不用全局单例可变对象
- **样式默认 scoped**：组件内 `<style>` 天然隔离；全局样式显式 `:global()` 并注明范围

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| Svelte 5（当前主版本） | Runes 体系：`$state` / `$derived` / `$effect` / snippet / 事件属性 | `references/general-quality.md`（通用）+ `references/svelte5.md`（写法规约）+ `references/style-examples.md`（正反例） |
| Svelte 4（遗留） | `let` 响应式、`$:` 语句、`on:` 指令、slot、createEventDispatcher | `references/svelte-legacy.md` |

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：组件结构与命名、a11y、模板块（each/await/key）、样式作用域、通用约定
- `references/svelte5.md` — Svelte 5 写法规约：Runes 全解、snippet、事件属性、effect 边界
- `references/svelte-legacy.md` — Svelte 4 遗留环境：写法基线与最小侵入原则
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
