---
name: "react-coding-standard"
description: "React 编码规范，按版本组织（现行 Hooks 写法 / class 组件遗留分册）：Rules of React、Hooks 使用约定、Effect 正确场景、组件设计与状态管理、渲染性能与 XSS 安全。当编写、重构或审查 React 代码、设计 React 组件与 Hooks 时使用。凡涉及 React 代码编写与修改的任务均应主动使用。"
---

# React 编码规范

本 skill 提供 React 框架层编码规范，覆盖现行 Hooks 写法与 class 组件遗留体系，规则主干来自 React 官方 Rules of React（react.dev，“rules not guidelines”——违反即 bug）与官方文档惯用法。标记结构与样式分别见 `html-coding-standard` / `css-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。
>
> 示例说明：示例以占位业务名（Model 等）演示，实际开发替换为项目业务名。

## 何时使用

- 编写、重构或审查 React 组件（函数组件、自定义 Hooks）
- 设计组件状态与数据流、判断 Effect 使用场景、优化渲染性能
- 维护 class 组件遗留代码（写法基线见 `react-legacy.md`）
- 代码审查时作为 React 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- React 17 ~ 19 项目（现行体系）：函数组件、自定义 Hooks、Context
- 遗留 class 组件：保持既有风格一致即可，不套用 Hooks 约定；重构时整体迁移为函数组件

### 不适用（边界）

- 模板中的标记语义与属性写法（JSX 属性名以 react.dev 为准，语义规则参见 `html-coding-standard` skill）
- 样式写法（参见 `css-coding-standard` skill）
- JS 语言级风格（参见 `js-coding-standard` skill）
- Vue 组件（参见 `vue-coding-standard` skill）
- TypeScript 类型层规范：props/事件类型在本 skill 各章随附提示，系统性类型设计参见 `ts-coding-standard` skill

## 一、版本与基线

1. **函数组件 + Hooks 为默认**；新组件一律函数组件，不新增 class 组件
2. **开启 StrictMode 与 `eslint-plugin-react-hooks`**（官方强烈推荐）：开发期暴露不纯渲染与 Hook 依赖问题
3. 同一项目不混用 class 与函数组件；重构整体迁移
4. 跟随项目 React 版本（18/19），禁混用尚未稳定的实验 API

| 版本 | 写法体系 | 详细规范 |
| --- | --- | --- |
| React ≤16 class（遗留） | class 组件 + 生命周期 | `references/react-legacy.md` |
| React（现行体系，17 ~ 19） | 函数组件 + Hooks | `references/general-quality.md`（通用）+ `references/react-modern.md`（写法规约） |

> React 19 变化点：函数组件 `defaultProps` 已废弃（用参数默认值）；`forwardRef` 不再需要（`ref` 作为普通 prop）；Actions / `useActionState` 可用。

## 二、Rules of React 精要（官方“规则而非指南”）

完整规则与正反例见 `references/general-quality.md`，完整示例见 `references/style-examples.md`：

**组件与 Hooks 必须纯粹：**

- 组件是幂等的：同样 props/state/context 输出同样的 JSX
- 副作用不得在渲染期执行——只放在事件处理器或 Effect 中
- props 与 state 不可变：禁直接修改（含数组/对象的深层变更），用不可变更新
- 传给 Hook 的参数与返回值、已写入 JSX 的值，都不得在之后修改

**组件与 Hooks 由 React 调用：**

- 禁止直接调用组件函数（`ModelCard()` ❌，`<ModelCard />` ✅）
- 禁止把 Hook 当普通值传递（Hook 只在组件/自定义 Hook 内调用）

**Rules of Hooks：**

- 只在顶层调用 Hook：禁在循环、条件、嵌套函数中调用，且在提前 return 之前
- 只在 React 函数中调用 Hook：普通 JS 函数中禁用

## 三、Hooks 使用约定

- 派生数据在渲染期计算，**不为派生值建 state**；昂贵计算用 `useMemo`
- “你可能不需要 Effect”：响应事件用事件处理器；prop 变化重置状态用 `key`；数据链式同步改为派生/事件——Effect 只做与 React 外部系统同步
- Effect 必须有完整依赖数组（`exhaustive-deps`）；需要清理的副作用（订阅、定时器、AbortController）配对返回清理函数
- 状态按需选型：局部 `useState`、复杂迁移逻辑 `useReducer`、跨组件就近提升或状态库；`useContext` 传递低频全局上下文
- `useCallback` / `useMemo` 不预防性滥用：仅用于确认瓶颈、引用相等性有语义、或文档要求的场景（React Compiler 启用的项目可省略）

## 四、组件设计与状态

- 组合优于继承：用 children / 组合组件表达复用；单一职责，组件过大拆分
- 状态就近放置：放实际使用的最低层级；多个组件共享时提升至最近公共父级
- 单一数据源：同一数据不在两处 state 各存一份；渲染期可算出的不存储
- 受控组件为默认；非受控（`defaultValue` + ref）仅限文件类与简单表单
- 命名：组件 PascalCase（`ModelCard.tsx`）；事件回调 props `onXxx`（`onSelect`），内部处理器 `handleXxx`（`handleSelect`）；自定义 Hook `useXxx`
- Props 只读；默认值用参数默认值（`{ page = 1 }`），不用已废弃的 `defaultProps`

## 五、渲染性能与安全

- 列表 `key` 用稳定业务 id，禁 index（可重排/过滤列表尤其）
- 条件渲染注意 `&&` 数字陷阱（`count && <List />` 在 count 为 0 时渲染 “0”）
- 避免渲染期创建新数组/对象/函数传给 memo 子组件（确认瓶颈后 `useMemo`/`useCallback`）
- 大列表虚拟滚动（react-window 等，按项目选型）
- `dangerouslySetInnerHTML` 等同 Vue 的 `v-html`：仅限完全可信内容，用户输入必须消毒（XSS）

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：核心原则、组件设计与状态、列表与条件渲染、安全
- `references/react-modern.md` — 现行体系（函数组件 + Hooks）：Rules of React、Hooks 使用约定（Effect 正确场景）、渲染性能、数据获取原则
- `references/style-examples.md` — 组件骨架正反例、Rules 正反例、Effect 误用高频案例（含“你可能不需要 Effect”）、速查表
- `references/react-legacy.md` — class 组件遗留环境：写法基线、Hooks 对照、最小侵入原则
