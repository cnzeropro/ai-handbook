---
name: "ts-coding-standard"
description: "TypeScript 编码规范（TS 5.x 基线）：类型设计原则、any 与 unknown 策略、类型收窄与守卫、泛型、枚举替代方案、类型断言与工具类型、模块与命名。当编写、重构或审查 TypeScript 代码、设计类型与泛型时使用。凡涉及 TypeScript 代码编写与修改的任务均应主动使用。"
---

# TypeScript 编码规范

本 skill 提供 TypeScript 类型层编码规范（**TS 5.x 基线**），融合 TypeScript 官方 Handbook（Do's & Don'ts）、Google TypeScript Style Guide 与 TSConfig Reference，适用于任意 TypeScript 代码（前端框架、Node 后端、类库）。JS 运行时语法规则见 `js-coding-standard`，两者配合使用。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。
>
> 与其他 skill 的关系：JS 运行时写法见 `js-coding-standard`；框架层组件设计见 `vue-coding-standard` / `react-coding-standard`（组件级类型约定随附于对应分册）；标记与样式见 `html-coding-standard` / `css-coding-standard`。

## 何时使用

- 编写、重构或审查任意 TypeScript 代码（类型定义、泛型、类型守卫、工具类型）
- 设计领域模型类型（状态建模、判别联合）或公共 API 类型
- 处理 any / unknown / 断言 / 类型报错时
- 代码审查时作为类型规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 TypeScript 代码：前端（Vue / React / 原生 TS）、Node.js 后端、类库与类型声明
- TS 5.x 语言环境（TS 小版本无破坏性写法差异，故不设版本分册；4.x 遗留项目注意实验性 decorators 与 5.x 的差异）

### 不适用（边界）

- JS 运行时语法风格（参见 `js-coding-standard` skill；本 skill 的代码示例同时遵循它）
- 框架层组件设计（参见 `vue-coding-standard` / `react-coding-standard` skill；组件级类型约定随附于对应分册）
- tsconfig 各编译项的取值逐条规定：跟随项目与构建链，本 skill 只约束"应开启什么类别"的原则（见 `references/general-quality.md` 一）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **strict 全家桶为默认**：`strict: true` 起步；禁用 `any`，外部输入用 `unknown` + 类型收窄
- **禁 namespace**：用 ES Modules 与文件划分（Google TS Guide 明令）；类型/命名冲突用 import 重命名解决
- **禁 enum**：用 `as const` 对象或字面量联合类型（数值枚举存在隐式转数字与逆映射陷阱，const enum 有隔离编译问题）
- **类型收窄**：优先用可收窄写法（`typeof` / `instanceof` / `in` / 判别联合 / 自定义 `is` 谓词），让编译器替你判断，而非断言
- **断言克制**：`as` 仅用于编译器无法得知的边界（DOM、第三方缺类型）；禁 `as any` 与双重断言；非空断言 `!` 需注释不变量
- **抑制报错用 `@ts-expect-error` + 原因注释**（行被移除时它会报错），不用 `@ts-ignore`
- **interface 与 type 分工**：对象形状用 `interface`（可继承、可合并），联合 / 映射 / 条件 / 工具类型用 `type`；同一声明不混用
- **工具类型优先**：`Partial` / `Pick` / `Omit` / `Record` / `ReturnType` 等内置类型不手写
- **命名**：类型 PascalCase；接口不加 `I` 前缀；泛型参数用描述名或单大写 `T`
- **type-only 导入导出**：仅类型的导入用 `import type`，导出类型用 `export type`，帮助编译器与打包器裁剪

## 二、版本与基线

| 版本 | 说明 | 详细规范 |
| --- | --- | --- |
| TS 5.x（基线） | 当前主版本；本 skill 全部规则以 5.x 为准 | `references/general-quality.md`（通用规则）+ `references/style-examples.md`（正反例） |
| TS 4.x（遗留） | 无独立分册：写法差异集中在实验性 decorators（5.x 已标准化）；迁移以官方迁移指南为准 | 按基线规则审查 |

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：tsconfig 原则、类型设计、any/unknown、收窄与守卫、断言、枚举替代、interface/type、泛型、模块与命名、工具类型
- `references/style-examples.md` — 编码风格完整示例与市面常见正反例速查（判别联合建模、枚举替代、断言克制、泛型设计等）
