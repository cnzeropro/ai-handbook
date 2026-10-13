---
name: "miniprogram-coding-standard"
description: "微信小程序原生编码规范：项目结构与文件约定、Page 与 Component 构造器、setData 性能与路径更新、WXML 指令与 WXSS 样式（rpx）、分包与启动性能、网络封装、缓存与安全合规。当编写、重构或审查微信小程序原生代码时使用。凡涉及微信小程序代码编写与修改的任务均应主动使用。"
---

# 微信小程序原生编码规范

本 skill 提供微信小程序原生框架编码规范（WXML / WXSS / JS/TS、Page / Component 构造器），规则来源为微信官方文档（developers.weixin.qq.com/miniprogram）。跨端框架项目见 `uniapp-coding-standard`，JS 语法细节见 `js-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查微信小程序原生代码（页面、组件、分包、云函数调用端）
- 处理 setData 性能、分包启动优化、登录与用户信息授权、内容安全合规时
- 代码审查时作为小程序规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 微信小程序原生项目（JS 或 TS）
- 基础库 2.x 演进体系：新 API 随基础库版本标注可用性，`app.json` 声明最低基础库版本

### 不适用（边界）

- 跨端框架项目 uni-app / Taro（参见 `uniapp-coding-standard` skill）
- Web 浏览器环境（无 DOM/BOM；标记语义与浏览器规范不适用 `html-coding-standard`，其 a11y/语义思想可借鉴）
- 其他小程序平台（支付宝/抖音等，遵循各自官方文档）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **页面四件套同名列同目录**：`index.wxml` / `wxss` / `js` / `json`；`json` 必须存在（可为空对象）；页面在 `app.json` `pages` 注册
- **setData 最小化**：只传变化的数据；用路径更新（`'list[0].status'`）；禁一次性 setData 大对象；高频场景（滚动/输入）节流
- **纯数据字段**：不影响视图的数据用 `pureDataPattern` 纯数据字段，绕过 setData 开销
- **Component 构造器优先**：可复用 UI 与复杂页面均用 `Component` 构造器（behaviors 复用、定义段清晰）
- **WXML**：`wx:for` 必带 `wx:key`（稳定 id）；频繁切换用 `hidden` 而非 `wx:if`；复杂逻辑用 `wxs`，模板内禁长表达式
- **WXSS**：尺寸用 rpx；类名 BEM；组件样式默认隔离（`styleIsolation` 按需调整）
- **生命周期配对清理**：`onUnload` / `detached` 清理定时器与事件监听
- **网络统一封装**：`wx.request` 域名白名单（合法域名HTTPS）；封装拦截器（token、错误码、重试），业务不散调
- **分包**：主包 ≤ 2M；按 tab/特性 `subPackages` 分包；`preloadRule` 预下载；独立分包仅工具页
- **缓存与合规**：本地缓存上限 10MB、敏感信息不落明文；`AppSecret` 禁落前端；用户信息走 `getUserProfile` 授权 + 隐私协议接口

## 二、版本与基线

| 维度 | 说明 |
| --- | --- |
| 基础库 | 持续演进，无传统大版本分册；`app.json` 声明最低基础库，新 API 按官方文档标注的最低版本使用 |
| 渲染引擎 | 默认 WebView；Skyline 为新一代渲染引擎（按需启用并验证兼容性） |
| 语言 | JS 通用规则见 `js-coding-standard`；TS 项目遵循同套约定 |

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：结构与文件、Page/Component、setData 性能、WXML/WXSS、生命周期、网络封装、分包、缓存与合规
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
