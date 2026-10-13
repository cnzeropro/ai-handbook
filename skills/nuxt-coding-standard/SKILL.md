---
name: "nuxt-coding-standard"
description: "Nuxt 编码规范（Nuxt 3/4 当前 / Nuxt 2 遗留）：目录约定与自动导入、数据获取（useFetch/useAsyncData）、server/ API 与 Nitro、useState 状态、渲染模式与性能、runtimeConfig 与安全。当编写、重构或审查 Nuxt 代码时使用。凡涉及 Nuxt 代码编写与修改的任务均应主动使用。"
---

# Nuxt 编码规范

本 skill 提供 Nuxt 框架层编码规范，以 **Nuxt 3/4**（Nitro 引擎、自动导入体系）为当前版本并覆盖 **Nuxt 2**（遗留），规则来源为 Nuxt 官方文档（nuxt.com/docs）。Vue 组件层规则见 `vue-coding-standard`，标记语义见 `html-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查 Nuxt 页面、组件、composables、server API（`server/`）
- 设计数据获取（SSR 数据注水）、渲染模式（SSR/SSG/SPA）、runtimeConfig
- 代码审查时作为 Nuxt 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- Nuxt 3 / 4 项目（当前版本；Nuxt 4 主要变更目录结构 `app/`）
- Nuxt 2 遗留项目：保持既有约定，见 `nuxt-legacy.md`；升级按官方迁移指南

### 不适用（边界）

- Vue 组件层规则（参见 `vue-coding-standard` skill；Runes 等不相关）
- 标记语义与样式（参见 `html-coding-standard` / `css-coding-standard` skill）
- TS 类型层（参见 `ts-coding-standard` skill）
- 其他元框架（Next.js 参见 `nextjs-coding-standard` skill）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **目录即约定**：`pages/`（文件路由）、`components/`（自动导入）、`composables/`、`utils/`、`server/`（Nitro API 与工具）、`public/`——不自造平行体系
- **自动导入**：`ref`/`computed` 与 `useXxx` 组合式函数自动可用；团队可选显式导入，但**全项目统一**
- **数据获取必须带 key**：`useFetch` / `useAsyncData` 显式 key（SSR 注水与去重依赖它）；客户端事件触发的请求用 `$fetch`
- **SSR 一致性**：服务端与客户端渲染结果必须一致，防 hydration mismatch（随机值/时间戳/`Math.random` 禁在渲染期）
- **状态用 `useState`**：SSR 安全的跨组件状态；**禁模块顶层共享可变状态**（服务端跨请求污染——Nuxt 第一大坑）
- **服务端能力**：`server/api/*.get|post.ts` 文件路由（Nitro）；秘密进 `runtimeConfig` 的非 `public` 层
- **渲染模式按路由声明**：`routeRules` 指定 SSR / 预渲染 / SPA fallback；默认 SSR
- **安全**：`v-html` 消毒；`runtimeConfig.private` 秘密禁入 `public` 层；middleware 鉴权

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| Nuxt 3 / 4（当前） | Nitro、自动导入、`app/` 目录（Nuxt 4）、`useFetch` 体系 | `references/general-quality.md`（通用）+ `references/nuxt3-4.md`（写法规约）+ `references/style-examples.md`（正反例） |
| Nuxt 2（遗留） | Options API、asyncData/fetch 钩子、Vuex | `references/nuxt-legacy.md` |

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：目录约定、自动导入、数据获取、状态、服务端能力、渲染模式、安全
- `references/nuxt3-4.md` — Nuxt 3/4 写法规约：app/ 目录、useFetch 选项、routeRules、Nitro 细节、Nuxt 4 变更
- `references/nuxt-legacy.md` — Nuxt 2 遗留环境：写法基线、与 Nuxt 3 差异对照、最小侵入原则
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
