---
name: "angular-coding-standard"
description: "Angular 编码规范（现代 Angular，v17+ 控制流与 Signals 方向）：单一职责与命名、独立组件、依赖注入（inject）、Signals 与响应式、RxJS 使用边界、生命周期、路由懒加载与性能。当编写、重构或审查 Angular 代码时使用。凡涉及 Angular 代码编写与修改的任务均应主动使用。"
---

# Angular 编码规范

本 skill 提供 Angular 框架层编码规范，以**官方 Style Guide**（angular.dev/style-guide）与 angular.dev 官方最佳实践为纲，适用于现代 Angular（v14+ 独立组件起步，v17+ 新控制流与 Signals 为推荐方向）。AngularJS（1.x）是完全不同的框架，不在本 skill 范围。标记语义见 `html-coding-standard`，TS 类型层见 `ts-coding-standard`。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查 Angular 组件、服务、指令、管道
- 设计依赖注入结构、状态方案（Signals / RxJS）、路由与懒加载
- 代码审查时作为 Angular 规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 现代 Angular 项目（v14+；推荐对齐 v17+ 的独立组件、内置控制流、Signals 写法）
- 遗留写法（NgModule、@Input/@Output 装饰器、构造器注入）：维护时保持一致，重构时按下述新写法整体迁移

### 不适用（边界）

- AngularJS（1.x）：完全不同的框架，不适用本 skill
- TS 类型层通用规则（参见 `ts-coding-standard` skill；Angular 特有类型随章提示）
- 模板中的标记语义（参见 `html-coding-standard` skill）与组件样式写法（参见 `css-coding-standard` skill；Angular 的样式封装为 ViewEncapsulation 另行说明）
- 其他框架（参见 `vue-coding-standard` / `react-coding-standard` skill）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **单一职责**：一个文件一个组件/服务/管道；超过 400 行考虑拆分（官方 Style Guide SRP）
- **命名**：文件名点分隔（`user-profile.component.ts`、`user-profile.service.ts`）；类名 PascalCase + 类型后缀（`UserProfileComponent`）；选择器带应用前缀（`app-user-profile`）
- **独立组件为默认**（standalone，v17+ 新项目无 NgModule）；新控制流 `@if` / `@for` 替代 `*ngIf` / `*ngFor`（`@for` 必带 `track`）
- **输入输出现代写法**：`input()` / `output()` 函数式 API，属性标 `readonly`；模板仅模板访问的成员标 `protected`
- **事件处理器按动作命名**：`(click)="saveUserData()"` 而非 `handleClick()`
- **依赖注入**：新代码用 `inject()` 函数（字段初始化场景），服务 `providedIn: 'root'`；令牌用 `InjectionToken`
- **状态与响应式**：新代码优先 Signals（`signal` / `computed` / `effect`）；RxJS 用于事件流与复杂异步编排；`computed` 优先于手工订阅
- **变更检测**：组件默认 `OnPush`（配合 Signals/不可变更新）
- **生命周期保持简单**：`ngOnInit` 只做编排调用具名方法；实现对应接口（`implements OnInit`）
- **样式绑定**：优先 `[class.x]` / `[style.x]` 绑定，优于 `ngClass` / `ngStyle`
- **模板方法保持简单**：复杂逻辑进 `computed` / 组件方法，不在模板内写表达式链
- **安全**：`innerHTML` 必须经 `DomSanitizer` 处理；禁直接拼接注入

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| Angular v17+（当前方向） | 独立组件、`@if`/`@for` 控制流、Signals、`inject()`、函数式守卫 | `references/general-quality.md`（通用）+ `references/style-examples.md`（正反例） |
| v14 ~ v16（过渡） | standalone 可用但无新控制流；维护保持一致，重构按 v17+ 方向整体迁移 | 按通用规则审查 |
| AngularJS 1.x | 完全不同的框架，不在范围 | — |

> 组件后缀：`Component` / `Service` 等类型后缀按项目统一（新版 CLI 允许省略后缀，默认保留以便检索）。

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：单一职责、命名、组件与模板、依赖注入、Signals 与 RxJS 边界、生命周期、路由懒加载、性能、安全
- `references/style-examples.md` — 编码风格完整示例与正反例速查（组件骨架、Signals、DI、控制流、track 等）
