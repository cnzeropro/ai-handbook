---
name: "frontend-testing-coding-standard"
description: "前端测试编码规范（Vitest/Jest 单测与组件测试 + Testing Library + Playwright E2E）：测试策略与金字塔、用例设计（FIRST/AAA）、Testing Library 查询优先级、mock 边界、异步稳定性与 E2E 选择器。当编写、重构或审查前端测试代码时使用。凡涉及前端测试代码编写与修改的任务均应主动使用。"
---

# 前端测试编码规范

本 skill 提供前端测试编码规范（领域专项），覆盖 **单元/组件测试（Vitest，Jest 差异注记）+ Testing Library + E2E（Playwright，Cypress 差异注记）**，规则来源为 Testing Library 官方指导原则、Vitest 与 Playwright 官方文档。被测代码的写法规约见各语言/框架 skill（`js-coding-standard` / `vue-coding-standard` / `react-coding-standard` 等）。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。

## 何时使用

- 编写、重构或审查前端测试：纯函数单测、组件测试（React/Vue/Angular/Svelte）、E2E 用例
- 判断"该不该测、测到哪一层、怎么 mock"时
- 处理测试稳定性（flaky）与测试数据管理时
- 代码审查时作为测试规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 前端项目测试：Vitest（当前主流）/ Jest、Testing Library（React/Vue/Angular/Svelte 适配）、Playwright（E2E 当前主流）/ Cypress 差异注记
- 工具小版本无破坏性写法差异，故不设版本分册（与 html/css 同理）；基线 Vitest 3.x / Jest 29+ / Playwright 1.4x+

### 不适用（边界）

- 被测代码本身的写法规约（参见对应语言/框架 skill：`js` / `ts` / `vue` / `react` / `angular` / `svelte` / `nextjs` / `nuxt`）
- 后端测试体系（JUnit 等，不在前端范围）
- 测试框架的构建与配置细节逐项规定（跟随项目脚手架；本 skill 只约束原则，见 `vitest-jest.md` 配置基线）

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- **测行为，不测实现**（Testing Library 核心原则）：断言用户可见的结果（渲染文本、角色、状态），不断言内部 state、私有方法调用次数
- **测试金字塔**：单测为主、组件/集成测试居中、E2E 只做金路径冒烟——E2E 成本最高、最易 flaky
- **FIRST**：Fast / Independent / Repeatable / Self-validating / Timely；测试之间禁相互依赖与共享可变状态
- **AAA 结构**：Arrange-Act-Assert 分段清晰；一个 it 只测一个主题
- **用户视角查询**：Testing Library 按 `getByRole` → `getByLabelText` → `getByText` → … → `getByTestId` 优先级查询；Playwright 同哲学（`getByRole` 优先）
- **禁 sleep**：异步等待一律 `waitFor` / `findBy`（组件测试）或 web-first 断言（E2E 自动等待）；`sleep(1000)` 是 flaky 之源
- **mock 边界**：只 mock 外部依赖（网络、时间、随机、存储）；禁 mock 被测对象内部；mock 数据必须与真实形状一致
- **事件交互用 `user-event`** 替代 `fireEvent`（模拟真实交互链路）
- **确定性**：禁渲染期随机/时间戳；时间用 fake timers 控制
- **覆盖率是参考指标**：不追 100%，看关键路径与回归防护

## 二、版本与基线

| 工具 | 定位 | 详细规范 |
| --- | --- | --- |
| Vitest（当前主流） | 单测与组件测试运行器，Vite 项目天然集成 | `references/vitest-jest.md`（含 Jest 差异注记） |
| Testing Library | 组件测试查询与交互（React/Vue/Angular/Svelte 适配） | `references/vitest-jest.md` |
| Playwright（当前主流） | E2E | `references/playwright-e2e.md`（含 Cypress 差异注记） |

> Jest（29+）与 Vitest API 高度同构（`jest.` → `vi.`），差异注记内联；Cypress 项目对照 Playwright 规则映射（选择器优先级与 web-first 断言哲学相同）。

## References（完整规范与代码示例）

- `references/general-quality.md` — 版本通用规范：测试策略与金字塔、用例设计、断言、mock 边界、稳定性、数据与清理、覆盖率态度
- `references/vitest-jest.md` — 单测与组件测试写法规约：组织与命名、Testing Library 查询优先级、user-event、mock、快照慎用、配置基线
- `references/playwright-e2e.md` — E2E 写法规约：适用边界、定位器优先级、web-first 断言、稳定性、CI
- `references/style-examples.md` — 编码风格完整示例与常见正反例速查
