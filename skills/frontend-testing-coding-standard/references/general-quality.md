# 前端测试通用质量规范

> 本文件为 frontend-testing-coding-standard 的**版本通用**质量规范（Vitest/Jest、Playwright/Cypress 通用部分）。来源：Testing Library 官方指导原则、Vitest 与 Playwright 官方文档。完整正反例见 `style-examples.md`（同目录）。被测代码写法规约见各语言/框架 skill。

## 目录

- 一、测试策略与金字塔
- 二、测行为，不测实现
- 三、用例设计（FIRST 与 AAA）
- 四、断言规范
- 五、mock 边界
- 六、异步与稳定性（防 flaky）
- 七、测试数据与清理
- 八、覆盖率的态度

## 一、测试策略与金字塔

| 层级 | 对象 | 数量占比（参考） | 特点 |
| --- | --- | --- | --- |
| 单元测试 | 纯函数、hooks/composables、工具类 | 最多 | 快、稳、定位准 |
| 组件测试 | 单个组件的用户可见行为（Testing Library） | 居中 | 渲染真实 DOM |
| E2E | 用户金路径（登录、下单主流程） | 最少 | 慢、最易 flaky |

- E2E 只做**金路径冒烟**（核心业务能跑通），不覆盖全部分支——成本最高且最脆
- 静态检查（TS/ESLint）算金字塔的"底座"，能靠类型与 lint 拦住的错误不写测试
- 测试随 bug 补：每个线上/审查发现的 bug，先写复现测试再修

## 二、测行为，不测实现（Testing Library 核心原则）

- 断言**用户可见的结果**：渲染出的文本、可交互性、角色与状态；不断言组件内部 state、私有方法、调用细节
- 判断标准："重构实现（不改行为）后，测试必须仍然全绿"——需要改测试的实现重构是坏测试的信号

```ts
// ❌ 测实现：依赖内部 state 与方法名
expect(wrapper.vm.internalCount).toBe(1);
expect(spyOnPrivateFormat).toHaveBeenCalled();

// ✅ 测行为：用户视角
render(ModelCounter);
await userEvent.click(screen.getByRole('button', { name: /增加/ }));
expect(screen.getByText('1')).toBeVisible();
```

## 三、用例设计（FIRST 与 AAA）

- **FIRST**：Fast（快）/ Independent（相互独立）/ Repeatable（可重复）/ Self-validating（自校验）/ Timely（及时）
- **AAA**：Arrange（准备）→ Act（操作）→ Assert（断言），空行分段
- 一个 `it` 只测一个主题；新场景新 `it`，禁一个用例串多断言主题
- 测试标题表达"被测行为 + 预期"（`增加一次后计数显示 1`）；跟随项目语言约定，无约定用英文
- 参数化重复用例：`it.each` / `test.each` 表驱动

```ts
describe('formatMoney', () => {
  it.each([
    [0, '¥0.00'],
    [1234.5, '¥1,234.50'],
  ])('金额 %d 格式化为 %s', (input, expected) => {
    expect(formatMoney(input)).toBe(expected);
  });
});
```

## 四、断言规范

- 断言**具体预期值**，禁 `toBeTruthy()` / `toBeFalsy()` 模糊断言（能写 `toBe(false)` 就别写 falsy）
- 禁无断言测试（只执行不断言的"冒烟"用例应有明确命名与理由）
- 一次断言一个主题；错误信息断言包含关键上下文而非整段堆栈

```ts
// ❌ 模糊
expect(result).toBeTruthy();

// ✅ 具体
expect(result.status).toBe('deployed');
```

## 五、mock 边界

- **只 mock 外部依赖**：网络请求、时间、随机数、存储、第三方 SDK——被测对象的内部函数不 mock
- mock 数据必须与真实形状一致（最好从真实响应抓取后裁剪），"方便起见"的假形状会让测试通过而线上爆炸
- 禁过度 mock：mock 越多，测试越接近"测试 mock 本身"
- 模块 mock 最小化：只 mock 被测单元直接依赖的外部模块，不 mock 传递依赖链

```ts
// ❌ mock 被测内部方法
vi.spyOn(counter, 'increment');

// ✅ mock 外部网络
vi.spyOn(api, 'fetchModels').mockResolvedValue([{ id: 1, name: 'Alpha' }]);
```

## 六、异步与稳定性（防 flaky）

- 组件测试：异步断言用 `findBy*` / `waitFor`，**禁 `sleep(1000)`** 类手动等待
- E2E：用 web-first 断言自动等待（见 `playwright-e2e.md`），禁 `page.waitForTimeout`
- 时间相关逻辑用 fake timers（`vi.useFakeTimers`），禁真实 sleep
- 渲染期禁随机数/时间戳（SSR 与快照同理）；需要时注入固定值

```ts
// ❌ 手动等待：慢且 flaky
await new Promise((r) => setTimeout(r, 1000));
expect(screen.getByText('已加载'));

// ✅ 自动等待出现
expect(await screen.findByText('已加载')).toBeVisible();
```

## 七、测试数据与清理

- 测试数据用**工厂函数**构造（`createModel(overrides)`），禁在多个用例复制大字面量
- 测试间隔离：每个用例独立准备数据；Testing Library 自动 cleanup（`globals: true` 时）；自定义渲染封装统一清理与 Provider 注入
- 禁依赖执行顺序（第一个用例写入、第二个用例读取）

```ts
// ✅ 工厂函数：默认值 + 覆盖
function createModel(overrides: Partial<ModelInfo> = {}): ModelInfo {
  return { id: 1, name: 'Alpha', status: 'draft', ...overrides };
}
```

## 八、覆盖率的态度

- 覆盖率是**参考指标**不是目标：80% 无意义的覆盖不如关键路径的精准断言
- 看"关键业务路径是否被测试"与"回归是否有防护"，不为凑数写无断言或浅断言测试
- 新代码（bug 修复）先写复现测试——回归防护的价值高于覆盖率数字
