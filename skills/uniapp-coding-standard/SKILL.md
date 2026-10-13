---
name: "uniapp-coding-standard"
description: "uni-app 编码规范（Vue 3 语系，多端条件编译）：pages.json 与页面路由、三层生命周期、条件编译集中隔离、API 封装（uni.xxx）、样式与 rpx/安全区、easycom 组件、状态管理与分包包体优化。当编写、重构或审查 uni-app 代码时使用。凡涉及 uni-app 代码编写与修改的任务均应主动使用。"
---

# uni-app 编码规范

本 skill 提供 uni-app 框架层编码规范（**Vue 3 语系**，多端条件编译：H5 / 微信等各家小程序 / App），规则来源为 uni-app 官方文档（uniapp.dcloud.net.cn）。Vue 组件层规则见 `vue-coding-standard`（Runes 等无关概念不适用），微信原生小程序项目见 `miniprogram-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查 uni-app 页面、组件、跨端 API 调用
- 处理多端差异（条件编译）、平台 API 封装、分包与包体优化
- 代码审查时作为 uni-app 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- uni-app Vue 3 语系项目（H5 / 微信及各家小程序 / App，Vue 3 + Vite 工程）
- Vue 2 语系遗留项目：保持既有写法一致，Options 语法差异参考 `vue-coding-standard` 的 `vue-legacy.md` 思路

### 不适用（边界）

- Taro（React 语系跨端）：遵循 Taro 官方文档与 `react-coding-standard`
- 微信原生小程序（参见 `miniprogram-coding-standard` skill）
- Vue 组件层细节（参见 `vue-coding-standard` skill；TS 类型层参见 `ts-coding-standard` skill）
- 标记语义与样式基础规则（参见 `html-coding-standard` / `css-coding-standard` skill；小程序端标签集为各端组件）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **pages.json 驱动**：页面注册、tabBar、分包 `subPackages`、导航栏样式——路由零代码化，禁手写路由表
- **条件编译集中隔离**：平台差异（`#ifdef MP-WEIXIN / H5 / APP-PLUS`）统一封装到适配层（`services/platform.ts` 或平台目录），业务代码少散落 `#ifdef`；禁整文件跨平台复制两份
- **三层生命周期**：应用（`onLaunch` / `onShow`）/ 页面（`onLoad` 参数解析、`onShow`、`onReady`、`onUnload` 清理）/ 组件（Vue 3 标准生命周期）；页面钩子从 `@dcloudio/uni-app` 引入
- **API 统一封装**：`uni.request` 封装拦截器（baseUrl、token、错误码、401 跳登录）；平台差异 API（定位/支付/分享）按端适配后对外统一签名
- **样式与适配**：rpx 尺寸；安全区 `env(safe-area-inset-*)`；条件编译样式注释块仅限差异行
- **easycom**：`components/组件名/组件名.vue` 自动注册，免手写 import；三方库（uni-ui）按需引入
- **状态管理**：Pinia（Vue 3）；登录态 token 管理（刷新与失效跳转）统一封装
- **包体与性能**：小程序端分包、图片压缩与 CDN、长列表、首屏数据在 `onLoad` 触发而非 `onReady` 之后

## 二、版本与基线

| 维度 | 说明 |
| --- | --- |
| Vue 语系 | Vue 3（Composition API）为基线；Vue 2 遗留项目保持 Options 写法一致（差异参考 `vue-coding-standard` 的 `vue-legacy.md`），不混用 |
| 工程形态 | HBuilderX 可视化工程与 CLI 工程（vue-cli/vite）皆可；CI 场景推荐 CLI 工程 |
| 平台基线 | 各端（H5 / 各家小程序 / App）能力差异以条件编译 + 官方兼容表裁决；不支持的 API 在适配层降级 |

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：pages.json 与页面、条件编译、三层生命周期、API 封装、样式适配、easycom、状态、包体性能、平台合规
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查（条件编译隔离、request 封装、分包、安全区）
