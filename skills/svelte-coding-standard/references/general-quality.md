# Svelte 通用质量规范

> 本文件为 svelte-coding-standard 的**版本通用**质量规范，同时适用于 Svelte 4 与 Svelte 5（示例以 Svelte 5 语法呈现）。版本特有约定见分册：Svelte 5 见 `svelte5.md`，Svelte 4 遗留见 `svelte-legacy.md`。来源：Svelte 官方文档与编译器内置 a11y 警告体系。完整正反例见 `style-examples.md`（同目录）。

## 目录

- 一、组件结构与命名
- 二、a11y：修复编译器警告，而非抑制
- 三、模板块规范（each / await / key）
- 四、样式作用域
- 五、props 与事件（版本通用部分）
- 六、与外部系统协作

## 一、组件结构与命名

- 单文件组件 `.svelte`，文件名 PascalCase（`ModelCard.svelte`）；`.svelte.js` / `.svelte.ts` 用于共享响应式状态模块
- 一个组件一个职责；超过约 300 行考虑按子组件 / snippet 拆分
- 共享状态模块导出**工厂函数**（`createModelStore()`），而非全局单例可变量（与 vue3 状态约定同构）

## 二、a11y：修复编译器警告，而非抑制

Svelte 编译器内置 a11y 检查（click-events-have-key-events、a11y-missing-attribute 等）——**警告是规范本身**：

- 一律修复（加 `role`、键盘事件、`alt`、`label` 等），不写 `<!-- svelte-ignore … -->` 抑制
- 确需抑制时：范围最小（只作用于该行）+ 注释说明原因

```svelte
<!-- ❌ 抑制而不修复 -->
<!-- svelte-ignore a11y_click_events_have_key_events -->
<div onclick={open}>…</div>

<!-- ✅ 用原生元素与语义 -->
<button type="button" onclick={open}>…</button>
```

- 图片必带 `alt`（装饰图 `alt=""`）；交互元素可达键盘；表单控件关联 `label`——细则同 `html-coding-standard`

## 三、模板块规范（each / await / key）

- `{#each}` 有稳定 key（列表会重排/过滤时必须）：

```svelte
{#each models as model (model.id)}
  <ModelCard {model} />
{/each}
```

- 异步块 `{#await}` 三分支完整表达 loading / 成功 / 失败；错误分支不得为空

```svelte
{#await modelPromise}
  <p>Loading…</p>
{:then data}
  <ModelView {data} />
{:catch error}
  <p class="error">{error.message}</p>
{/await}
```

- 内容不渲染用 `{#if}`；同一条件组用 `{:else if}` 链，禁重复否定条件

## 四、样式作用域

- 组件内 `<style>` 默认 scoped（编译器加 hash 类），放心写组件样式
- `:global()` 是逃生舱：必须限定选择器范围（如 `:global(.dark) .card`），禁裸 `:global()` 全局污染；全局样式统一放入入口样式文件
- 样式语言规则遵循 `css-coding-standard`（若使用 Tailwind 则遵循 `tailwind-coding-standard`）

## 五、props 与事件（版本通用部分）

- 单向数据流：props 下行、回调上行；子组件不得修改 props 对象的深层内容
- 组件对外 API 最小化：不透传无关 props；透传用 rest props（`let { …rest } = $props()`，Svelte 5）
- 事件回调命名（Svelte 5 属性形态）：对外回调 `onXxx`（`onSelect`），组件内处理器 `handleXxx`（`handleSelect`）

## 六、与外部系统协作

- 请求 / 订阅 / 定时器：在 `onMount` / `$effect` 中启动，必须返回清理函数
- 浏览器 API 访问注意 SSR（`browser` 判断），不在组件顶层直接访问 `window`
