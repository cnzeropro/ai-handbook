# E2E 写法规约（Playwright）

> 本文件为 **E2E** 层写法规约（Playwright 为当前主流；Cypress 对照注记），须与版本通用的 `general-quality.md` 配合使用。组件/单测层见 `vitest-jest.md`。

## 目录

- 一、E2E 适用边界 —— 金路径冒烟，不覆盖全业务
- 二、定位器优先级 —— 与 Testing Library 同哲学
- 三、web-first 断言 —— 自动等待，禁手动 sleep
- 四、稳定性 —— 独立、隔离、克制重试
- 五、工程化 —— webServer、并行、trace、多浏览器
- 六、禁测三方依赖

## 一、E2E 适用边界

- E2E 只做**用户金路径冒烟**：登录 → 核心业务 → 关键结果。成本最高、最慢、最易 flaky
- 全部分支与边界在单测/组件测试层覆盖，不在 E2E 重复
- 每条 E2E 用例对应一条真实用户旅程，命名按旅程描述

## 二、定位器优先级

与 Testing Library 同哲学——**用户视角定位**：

```ts
// ✅ 优先级：getByRole → getByLabelText → getByText → getByTestId 兜底
await page.getByRole('button', { name: /提交/ }).click();
await page.getByLabel('用户名').fill('alpha');
await page.getByTestId('model-list').isVisible();
```

- 兜底 `getByTestId` 与组件测试共用 `data-testid` 属性，值语义化（`model-list` 而非 `test1`）
- 禁 CSS/XPath 长链选择器（实现耦合，改版即碎）；禁 `page.$` / element handle

## 三、web-first 断言

- 断言用 `expect(locator).toBeVisible()` 等 web-first 断言——**自动等待**直到条件满足或超时

```ts
// ❌ 手动 sleep：慢且 flaky
await page.click('button');
await page.waitForTimeout(1000);
expect(await page.textContent('.result')).toContain('成功');

// ✅ 自动等待 + web-first 断言
await page.getByRole('button', { name: /提交/ }).click();
await expect(page.getByText('提交成功')).toBeVisible();
```

- 禁 `page.waitForTimeout`；确需等待网络响应用 `waitForResponse` / `waitForURL` 明确等待目标
- 断言具体预期值；接口校验用 `expect(response).toBeOK()`

## 四、稳定性

- **测试独立**：每个用例自带数据与前置（`beforeEach` 或 API 造数），禁用例间依赖与执行顺序假设
- 数据隔离：每次运行创建独立测试数据（前缀 + 随机），禁共享真实数据；测试后清理（API 删除或独立测试账号）
- 禁依赖第三方真实服务（第三方支付/短信用 mock/沙箱）
- 重试克制：CI `retries: 2` 只为消化偶发，重试通过即记录为疑似 flaky 并排期修复
- 避免 `networkidle` 等脆弱等待条件；长任务等待具体业务标志

## 五、工程化

- `playwright.config.ts`：`webServer` 启动本地服务（CI 无需手工起服务）；`workers` 并行；`trace: 'on-first-retry'`；`retries` 仅 CI
- 多浏览器按需配置 projects（Chromium 必跑，WebKit/Firefox 按覆盖需求）
- 测试文件按旅程组织（`e2e/auth.spec.ts` / `e2e/model-flow.spec.ts`）

```ts
export default defineConfig({
  webServer: { command: 'npm run dev', url: 'http://localhost:5173', reuseExistingServer: !process.env.CI },
  use: { baseURL: 'http://localhost:5173', trace: 'on-first-retry' },
  retries: process.env.CI ? 2 : 0,
});
```

## 六、禁测三方依赖

- 不测试第三方服务自身行为（支付成功页样式、地图渲染细节）——mock/沙箱后只断言**我方集成点**
- 第三方跳转与回调：mock 回调链路断言我方处理逻辑
