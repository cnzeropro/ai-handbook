# 前端测试编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：Testing Library 官方指导原则、Vitest / Playwright 官方文档与社区审查常见问题）。规则正文见 `SKILL.md` 与各分册；本文件供代码审查对照与深入学习。

## 目录

- 1. 纯函数单测（AAA + it.each）
- 2. 组件测试骨架与查询优先级
- 3. user-event 替代 fireEvent
- 4. mock 正反例（msw / 模块 mock / fake timers）
- 5. 异步等待正反例（findBy vs sleep）
- 6. 工厂函数与测试隔离
- 7. Playwright 金路径 E2E 完整示例
- 8. 常见正反例速查

## 1. 纯函数单测（AAA + it.each）

```ts
// utils/format-money.test.ts
import { describe, expect, it } from 'vitest';
import { formatMoney } from './format-money';

describe('formatMoney', () => {
  // ✅ 表驱动参数化：新边界加一行即可
  it.each([
    [0, '¥0.00'],
    [1234.5, '¥1,234.50'],
    [-0.5, '-¥0.50'],
  ])('金额 %d 格式化为 %s', (input, expected) => {
    // Act + Assert（无外部依赖时 Arrange 可省）
    expect(formatMoney(input)).toBe(expected);
  });

  it('非数字输入抛出类型错误', () => {
    // ✅ 异常断言用函数包裹
    expect(() => formatMoney(Number.NaN)).toThrow(TypeError);
  });
});
```

❌ 反例：无标题语义（`test('works')`）、一个 it 串多主题、无断言。

## 2. 组件测试骨架与查询优先级

React：

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('ModelCounter（用户视角行为）', () => {
  it('点击增加按钮后计数显示 1', async () => {
    // Arrange
    render(<ModelCounter initial={0} />);

    // Act
    await userEvent.click(screen.getByRole('button', { name: /增加/ }));

    // Assert
    expect(screen.getByText('1')).toBeVisible();
  });
});
```

Vue：

```ts
import { render, screen } from '@testing-library/vue';
import ModelCard from '../ModelCard.vue';

it('卡片展示模型名称与状态', () => {
  render(ModelCard, { props: { model: { id: 1, name: 'Alpha', status: 'deployed' } } });
  expect(screen.getByRole('heading', { name: 'Alpha' })).toBeVisible();
  expect(screen.getByText('已部署')).toBeVisible();
});
```

查询优先级（用户视角递减）：`getByRole` → `getByLabelText`（表单）→ `getByText` → … → `getByTestId` 兜底：

```tsx
// ❌ 实现细节查询
screen.querySelector('.submit-btn');
screen.getByTestId('btn-1');

// ✅ 用户视角
screen.getByRole('button', { name: /提交/ });
```

## 3. user-event 替代 fireEvent

```tsx
// ❌ fireEvent：离散事件，绕过焦点与输入链
fireEvent.change(input, { target: { value: 'Alpha' } });

// ✅ 真实交互链路
await userEvent.type(input, 'Alpha');
await userEvent.click(submitButton);
await userEvent.keyboard('{Enter}');
```

`fireEvent` 仅用于 user-event 覆盖不到的场景（如直接触发自定义事件），并注释原因。

## 4. mock 正反例（msw / 模块 mock / fake timers）

网络层用 msw（service worker 层，形状真实、可跨测试复用）：

```ts
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer(
  http.get('/api/models', () => HttpResponse.json([{ id: 1, name: 'Alpha' }])),
  http.post('/api/models', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 2, ...body }, { status: 201 });
  }),
);

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));  // ✅ 未拦截请求直接报错
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

```ts
// ❌ mock 被测内部方法
vi.spyOn(counter, 'increment');

// ✅ mock 外部依赖
vi.spyOn(api, 'fetchModels').mockResolvedValue([{ id: 1, name: 'Alpha' }]);
```

时间相关用 fake timers：

```ts
it('5 秒后自动刷新', async () => {
  vi.useFakeTimers();
  startAutoRefresh();
  vi.advanceTimersByTime(5000);
  expect(refresh).toHaveBeenCalledOnce();
  vi.useRealTimers();
});
```

## 5. 异步等待正反例（findBy vs sleep）

```tsx
// ❌ 手动 sleep：慢且 flaky
render(<ModelList />);
await new Promise((r) => setTimeout(r, 1000));
expect(screen.getByText('Alpha')).toBeVisible();

// ✅ findBy 自动等待出现（可配超时）
expect(await screen.findByText('Alpha')).toBeVisible();

// ✅ waitFor 断言消失/计数
await waitFor(() => expect(screen.queryByRole('progressbar')).not.toBeInTheDocument());
```

E2E（Playwright）同理用 web-first 断言：

```ts
// ❌
await page.waitForTimeout(2000);
// ✅
await expect(page.getByText('提交成功')).toBeVisible();
```

## 6. 工厂函数与测试隔离

```ts
// ✅ 工厂函数：默认值 + 用例覆盖
function createModel(overrides: Partial<ModelInfo> = {}): ModelInfo {
  return { id: Math.random(), name: 'Alpha', status: 'draft', tags: [], ...overrides };
}

it('过滤已部署模型', () => {
  const models = [createModel({ status: 'deployed' }), createModel({ status: 'draft' })];
  expect(filterDeployed(models)).toHaveLength(1);
});
```

- 每个用例独立造数，禁用例间共享可变数据与执行顺序假设
- 测试随机数/时间注入固定值（`vi.useFakeTimers` / 依赖注入）

## 7. Playwright 金路径 E2E 完整示例

```ts
// e2e/model-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('模型管理金路径', () => {
  test('登录后创建模型并出现在列表', async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('用户名').fill('alpha');
    await page.getByLabel('密码').fill('secret');
    await page.getByRole('button', { name: /登录/ }).click();

    await expect(page).toHaveURL(/models/);

    await page.getByRole('button', { name: /新建模型/ }).click();
    await page.getByLabel('模型名称').fill('E2E-Test');
    await page.getByRole('button', { name: /创建/ }).click();

    await expect(page.getByRole('row', { name: /E2E-Test/ })).toBeVisible();
  });
});
```

- 用例对应真实用户旅程；数据独立（前缀 + 随机）；断言用户可见结果
- ❌ `page.waitForTimeout`、依赖第三方真实服务、覆盖全部分支

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 测实现细节 | 断言内部 state / 私有方法 | 断言用户可见结果 |
| 模糊断言 | `toBeTruthy()` | 具体值 `toBe('deployed')` |
| 无断言 | 只执行不校验 | 每用例至少一个有效断言 |
| 手动 sleep | `setTimeout(1000)` 后断言 | `findBy*` / `waitFor` / web-first 断言 |
| fireEvent 直发 | `fireEvent.change` | `userEvent.type` |
| testid 优先 | `getByTestId` 到处用 | `getByRole` 优先，testid 兜底 |
| mock 内部 | spy 被测方法 | 只 mock 外部依赖 |
| mock 形状失真 | 手写简化假数据 | 真实响应裁剪 |
| 快照滥用 | 整组件快照 | 具名断言；快照仅稳定小对象 |
| 用例相互依赖 | 前一用例写、后一用例读 | FIRST：独立造数 |
| 魔法等待 | `waitForTimeout` | auto-waiting / waitForResponse |
| 只测快乐路径 | 无异常分支 | 补错误与边界用例 |
| 覆盖率凑数 | 无断言跑覆盖率 | 关键路径精准断言 |
