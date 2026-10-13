---
name: "nextjs-coding-standard"
description: "Next.js 编码规范，按版本组织（App Router 当前主推 / Pages Router 仍在用）：Server 与 Client 组件边界、数据获取与缓存语义、Server Actions、文件约定与 metadata、环境变量与安全。当编写、重构或审查 Next.js 代码时使用。凡涉及 Next.js 代码编写与修改的任务均应主动使用。"
---

# Next.js 编码规范

本 skill 提供 Next.js 框架层编码规范，以 **App Router**（当前主推，React Server Components 体系）为主线并覆盖 **Pages Router**（仍在用），规则来源为 next.js 官方文档（nextjs.org/docs）与 React 官方 Rules of React。React 组件层规则见 `react-coding-standard`，标记语义见 `html-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查 Next.js 页面、布局、Server/Client 组件、Route Handlers、Server Actions
- 设计数据获取与缓存策略、配置 metadata、处理环境变量
- 代码审查时作为 Next.js 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- App Router 项目（Next 13.4+ 稳定，当前主推）
- Pages Router 项目（仍在用）：保持既有约定，见 `nextjs-pages.md`；迁移按官方增量指南

### 不适用（边界）

- React 组件层规则（参见 `react-coding-standard` skill；本 skill 约束框架层）
- 标记语义与样式（参见 `html-coding-standard` / `css-coding-standard` skill）
- TS 类型层（参见 `ts-coding-standard` skill）
- 其他元框架（Nuxt 参见 `nuxt-coding-standard` skill）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **默认 Server Component**：仅交互/状态/浏览器 API 的子树下沉为 Client Component（`'use client'`）；client 组件尽量放到叶子
- **文件约定**：`page` / `layout` / `loading` / `error` / `not-found` / `route` 专用文件各司其职；`error.tsx` 必须 `'use client'` 且提供重置能力
- **数据获取在服务端**：Server Component 直接 `await fetch` / 查数据库；变更用 Server Actions（`'use server'`）+ `revalidatePath` / `revalidateTag`
- **缓存语义显式化**：fetch 默认缓存语义随版本演进——按页面需求显式声明 `cache: 'force-cache' | 'no-store'` / `next.revalidate`；写后必失效
- **Client 拉数据**：仅限客户端交互触发；初始数据走 server 端，不在 useEffect 里同步 props
- **环境变量**：客户端可见值必须 `NEXT_PUBLIC_` 前缀；秘密只在服务端，禁 `NEXT_PUBLIC_` 携带密钥
- **内置组件优先**：图片 `next/image`、字体 `next/font`、脚本 `next/script`；metadata 用 Metadata API 而非手工 `<head>`
- **路由参数（Next 15+）**：`params` / `searchParams` 为 Promise，需 `await`
- **安全**：`dangerouslySetInnerHTML` 消毒（同 react skill）；服务端 fetch 校验目标 URL（防 SSRF）

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| App Router（当前主推，13.4+ 稳定 / 15 起 params 异步化） | RSC、文件约定、Server Actions、Metadata API | `references/general-quality.md`（通用）+ `references/app-router.md`（版本特性）+ `references/style-examples.md`（正反例） |
| Pages Router（仍在用） | `getServerSideProps` / `getStaticProps` / `pages/api` | `references/pages-router.md` |

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：文件约定、Server/Client 边界、数据获取与变更、缓存语义、环境变量、性能、安全
- `references/app-router.md` — App Router 版本特性：RSC 边界细则、Server Actions、metadata API、params 异步化、middleware
- `references/pages-router.md` — Pages Router（仍在用）：数据获取函数、API Routes、迁移提示
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
