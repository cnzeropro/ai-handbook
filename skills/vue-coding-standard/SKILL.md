---
name: "vue-coding-standard"
description: "Vue 编码规范，按版本组织（Vue 3 组合式 API 现行 / Vue 2 Options 遗留分册）：SFC 结构与命名、props/emits 约定、v-for/v-if 使用、组合式 API 与 computed、组件通信与 Pinia 状态、性能与 XSS 安全。当编写、重构或审查 Vue 代码、设计 Vue 组件时使用。凡涉及 Vue 代码编写与修改的任务均应主动使用。"
---

# Vue 编码规范

本 skill 提供 Vue 框架层编码规范，覆盖 Vue 3 组合式与 Vue 2 Options 两代写法体系，规则主干来自 Vue 官方 Style Guide（Priority A/B/C/D 四级）并补充组合式 API 开发约定。标记结构与样式分别见 `html-coding-standard` / `css-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。
>
> 示例说明：示例以占位业务名（Model 等）演示，实际开发替换为项目业务名。

## 何时使用

- 编写、重构或审查 Vue 3 组件（SFC、组合式函数、指令）
- 设计组件 API（props/emits/v-model/expose）或组织组件通信与状态
- 维护 Vue 2 遗留项目（写法基线见 `vue-legacy.md`）
- 代码审查时作为 Vue 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- Vue 3.x 项目：SFC（`<script setup>`）、模板、组合式函数（composables）、Pinia 状态
- Vue 2 遗留项目：已 EOL，保持既有 Options API 风格一致即可，不套用本 skill 的组合式约定；升级以官方迁移指南为准

### 不适用（边界）

- 模板中的标记语义与属性写法（参见 `html-coding-standard` skill）
- 样式写法（参见 `css-coding-standard` skill；scoped 方案本 skill 另有约束）
- JS 语言级风格（参见 `js-coding-standard` skill）
- TypeScript 类型层规范（参见 `ts-coding-standard` skill；组件级类型约定见 `vue3.md`）
- React 组件（参见 `react-coding-standard` skill）

## 一、版本与基线

1. **Vue 3 + `<script setup>` 为默认**；新组件一律组合式 API
2. 同一项目**不混用** Options API 与 Composition API；遗留 Options 代码重构时应整体迁移
3. 不使用已废弃/撤回的实验特性（如 Reactivity Transform）
4. 跟随项目 Vue/生态版本（Vue、Vue Router、Pinia 大版本对齐），禁越级使用更高版本语法

| 版本 | 写法体系 | 详细规范 |
| --- | --- | --- |
| Vue 2（遗留，已 EOL） | Options API、data 函数、Vuex | `references/vue-legacy.md` |
| Vue 3（当前主版本） | Composition API + `<script setup>`、Pinia | `references/general-quality.md`（通用）+ `references/vue3.md`（写法规约） |

## 二、官方 Style Guide 规则精要（Priority A/B/C/D）

完整规则与正反例见 `references/general-quality.md`，完整示例见 `references/style-examples.md`：

**Priority A（Essential，防错必须遵守）：**

- 组件名多词（`TodoItem` 而非 `Todo`），防与 HTML 元素冲突
- props 必须详细定义：至少类型（`defineProps<{ status: string }>()` 或对象校验式）
- `v-for` 必带 `key`（组件上强制；用稳定业务 id，慎用 index）
- `v-if` 与 `v-for` 禁止同元素（Vue 3 中 `v-if` 先执行，迭代变量不存在直接报错；过滤用 computed，隐藏用容器）
- 禁止对用户提供的内容使用 `v-html`（XSS）
- 组件样式必须作用域化：`scoped` / CSS Modules / BEM 类名约定（组件库优先类名方案）

**Priority B（Strongly Recommended）：**

- 一组件一文件；SFC 文件名统一 PascalCase（`ModelCard.vue`）
- 基础展示组件统一 `Base` 前缀（`BaseButton`），只含元素/其他基础组件，不含全局状态
- 模板中组件用 PascalCase；props 声明用 camelCase、模板传递用 kebab-case
- 多属性元素逐行一个属性；模板属性/文本统一双引号；指令简写统一（`:` 与 `@`）
- 模板只放简单表达式，复杂逻辑进 computed

**Priority C（Recommended）：**

- 元素属性顺序：`is` → `v-for` → `v-if/v-else-if/v-else` → `v-show/v-cloak/v-pre` → `id` → `ref`/`key` → `v-model` → 自定义指令 → 普通属性 → 事件 `@` → `v-html`
- SFC 顶层块顺序：`<script>` → `<template>` → `<style>`（项目统一即可，优先官方顺序）
- 组件自闭合（`<ModelCard />`）；多属性块之间空行

**Priority D（Use with Caution）：**

- 谨慎：scoped 样式中的元素选择器、隐式父子通信（依赖 `this.$parent` / attrs 透传）、绕过单向数据流的全局状态写法

## 三、组合式 API 约定（开发实践）

- 状态默认 `ref`；`reactive` 仅用于成组局部状态（不可整体替换，易失响应）
- 派生数据用 `computed`，**优先于** `watch`；`watch` 用于副作用（请求/持久化/操作 DOM），注意清理
- 可复用逻辑抽为组合式函数：`useXxx` 命名、返回 refs 对象、输入用 ref/getter
- `defineProps` 类型化、`defineEmits` 声明事件、`defineModel` 实现 v-model（3.4+）
- `provide`/`inject` 仅用于组件库与深层上下文（主题/当前用户），业务数据流用 props + events
- 生命周期对应：`onMounted` / `onUnmounted` 配对清理定时器、监听器

## 四、组件通信与状态

- 单向数据流：props 下行、events 上行；子组件不改 props（需变更时 emit 或用 defineModel）
- 全局状态用 Pinia：一个 store 一个领域（`useModelStore`），state/getters/actions 分层；组件内不直接改 store 之外的全局对象
- 事件命名 kebab-case（`@model-change`）；原生事件透传优先 `v-bind="$attrs"` 而非手动转发

## 五、性能与安全

- 路由级组件异步加载（`defineAsyncComponent` / 路由懒加载）；大列表虚拟滚动
- `KeepAlive` 缓存条件列表场景；`v-memo` 仅用于确认瓶颈后优化
- `v-html` 仅限完全可信内容；生产构建禁用 devtools 与 sourcemap 暴露
- 避免深层响应式大对象（`shallowRef` 按需）

## References（完整规范与代码示例）

- `references/general-quality.md` — 官方 Style Guide 四级规则完整版（A/B/C/D），版本通用（Vue 2/3 均适用）
- `references/vue3.md` — Vue 3 写法规约：组合式 API 约定、defineModel、Pinia、性能与安全
- `references/style-examples.md` — SFC 完整骨架正反例、A 级五规则示例、属性顺序、命名体系、常见正反例速查
- `references/vue-legacy.md` — Vue 2 遗留环境：Options API 基线、Vue 3 差异对照、最小侵入原则
