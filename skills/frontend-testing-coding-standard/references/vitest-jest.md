# 单测与组件测试写法规约（Vitest / Jest + Testing Library）

> 本文件为**单元与组件测试**层的写法规约（Vitest 为当前主流运行器；Jest 29+ 同构，差异注记），须与版本通用的 `general-quality.md` 配合使用。E2E 见 `playwright-e2e.md`。

## 目录

- 一、工具基线 —— Vitest 与 Jest 差异、globals 决策、environment
- 二、测试组织与命名 —— describe/it、参数化、focus/skip
- 三、Testing Library 查询优先级 —— 八级优先 + 三类查询语义
- 四、user-event 替代 fireEvent
- 五、mock 规约 —— vi.mock 提升、vi.fn、网络 mock、fake timers
- 六、快照测试慎用
- 七、组件测试要点 —— Vue/React/Angular 挂载与查询

## 一、工具基线

- **Vitest 为当前主流**（Vite 原生、ESM、速度快）；Jest 29+ 与其 API 高度同构，差异主要是前缀（`vi.` → `jest.`）与配置格式
- `globals: true`（`describe`/`it` 免导入）与显式导入二选一，**全项目统一**；显式导入更利于类型与跳转
- DOM 环境选 `jsdom`（兼容性最全）或 `happy-dom`（更快）；统一放 `test.environment` 或文件头注释按需覆盖
- 清理：`globals: true` 时 Testing Library 自动 cleanup；否则在 setup 文件 `afterEach(cleanup)`
- 覆盖率 provider：Vitest 用 `@vitest/coverage-v8`；覆盖率仅作参考指标（见 `general-quality.md` 八）

## 二、测试组织与命名

- `describe` 按被测单元分组，`it` 表达"被测行为 + 预期"；标题跟随项目语言约定，无约定用英文
- 参数化用 `it.each` / `test.each` 表驱动，禁复制粘贴多份近似用例
- `it.only` / `it.skip` / `fit` / `xit` **禁提交**（提交前清零，CI 加检测）

```ts
describe('formatMoney', () => {
  it.each([
    [0, '¥0.00'],
    [1234.5, '¥1,234.50'],
    [-1, '-¥1.00'],
  ])('金额 %d 格式化为 %s', (input, expected) => {
    expect(formatMoney(input)).toBe(expected);
  });
});
```

- 测试文件位置与命名跟随项目（`__tests__/` 或同目录 `.test.ts` / `.spec.ts`），与被测文件同名

## 三、Testing Library 查询优先级

**按可访问性优先级选择查询**（与真实用户和读屏用户的感知对齐）：

| 优先级 | 查询 | 适用 |
| --- | --- | --- |
| 1 | `getByRole` | 一切可交互/有角色的元素（`name` 选项即可访问名） |
| 2 | `getByLabelText` | 表单控件 |
| 3 | `getByPlaceholderText` | 仅当 placeholder 是最主要线索 |
| 4 | `getByText` | 非表单的文本内容 |
| 5 | `getByDisplayValue` | 表单当前值 |
| 6 | `getByAltText` | 有 alt 的图片 |
| 7 | `getByTitle` | title 属性（读屏不一致，少用） |
| 8 | `getByTestId` | 兜底：用户不可感知（需 `data-testid`） |

```ts
// ❌ 实现细节查询（class/testid 优先）
screen.querySelector('.submit-btn');
screen.getByTestId('submit-btn');

// ✅ 用户视角查询
screen.getByRole('button', { name: /提交/ });
```

- **三类查询语义**：`getBy*` 同步取（0 个抛错 / 多个抛错）；`queryBy*` 断言"不存在"用（返回 null）；`findBy*` 异步等待用（自动 retry）
- 用 `screen` 查询（免传 container）；TextMatch 优先整串匹配或带 `regex` 标志，禁过于宽泛的正则误命中

## 四、user-event 替代 fireEvent

`@testing-library/user-event` 模拟真实交互链路（焦点、键盘、输入逐字符）；`fireEvent` 是离散事件，绕过中间行为：

```ts
// ❌ fireEvent：不触发焦点与完整输入链
fireEvent.change(input, { target: { value: 'Alpha' } });

// ✅
await userEvent.type(screen.getByLabelText('名称'), 'Alpha');
await userEvent.click(screen.getByRole('button', { name: /提交/ }));
```

- `userEvent.setup()` 放 beforeAll（性能）；事件必然异步，注意 await

## 五、mock 规约

- **vi.mock 自动提升到文件顶部**：工厂内引用外部变量需 `vi.hoisted`；mock 工厂必须返回真实形状
- 只 mock 直接外部依赖（见 `general-quality.md` 五）；网络 mock 优先用 msw（service worker 层，同形且可复用）而非逐个拦截
- `vi.fn` 断言"被调用方式"仅用于边界（回调触发、事件上报），不断言被测内部协作

```ts
// ✅ 网络层 mock（msw）
const server = setupServer(
  http.get('/api/models', () => HttpResponse.json([{ id: 1, name: 'Alpha' }])),
);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

- 时间相关用 fake timers：`vi.useFakeTimers()` + `vi.advanceTimersByTime`；测完 `vi.useRealTimers()`（放 afterEach 统一恢复）

## 六、快照测试慎用

- 快照只用于**稳定且小**的序列化输出（如生成的纯文本）；大组件快照没人看，退化成"改了就 update"
- 禁无脑 `toMatchSnapshot` 组件整树；优先具名断言

## 七、组件测试要点

- Vue：`@testing-library/vue` 的 `render`；Angular：`@testing-library/angular`；React：`@testing-library/react`
- 交互一律 `user-event`；异步 UI 用 `findBy*`；路由/全局 Provider 通过自定义 `render` 封装注入
- 禁断言内部 state / 私有方法；禁为通过测试在组件上加 `data-testid` 以外的实现性钩子（testid 兜底也要语义化）
