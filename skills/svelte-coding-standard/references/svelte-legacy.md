# Svelte 4（遗留环境）规范

> 适用：Svelte 4.x 遗留项目。仅在维护既有代码或项目明确要求时使用；新项目与新组件一律 Svelte 5 Runes（见 `svelte5.md`）。同一项目保持写法一致，不混用两代语法。

## 一、写法基线

- **响应式赋值**：顶层 `let` 声明即响应式；重新赋值触发更新（对象变异需整体替换或 `$invalidate` 由赋值触发）

```svelte
<script>
  let keyword = '';
  let models = [];
  $: filtered = models.filter((m) => m.name.includes(keyword));
  async function fetchAll() {
    models = await modelApi.list();   // 重新赋值触发更新
  }
</script>
```

- **`$:` 响应式语句**：派生数据与副作用混写易出顺序问题——保持声明在消费之前，派生语句内禁副作用
- **事件指令**：`on:click={handler}`，可带修饰符（`|once` / `|preventDefault` / `|stopPropagation`）——遗留项目可用修饰符；新写法迁移后修饰符移入处理器
- **子父通信**：`createEventDispatcher()` 声明式分发事件；组件 props 仍然单向
- **插槽**：`<slot name="…">` + `slot="name"`；默认内容写在 slot 内
- **动态组件**：`<svelte:component this={Comp}>`
- **stores**：跨组件状态用 `writable` / `readable` + `$store` 自动订阅

## 二、a11y 与模板

- 编译器 a11y 警告与 Svelte 5 同一体系：一律修复，抑制须最小范围 + 注释
- `{#each}` / `{#await}` / `{#if}` 块规则与 `general-quality.md` 三一致

## 三、遗留代码最小侵入

- 不顺手把 `let`/`$:` 迁移为 Runes；升级作为独立任务（官方 `npx sv migrate svelte-5` 迁移工具）
- 遗留库中的新组件保持 Svelte 4 写法，与项目一致
- 升级前置检查：仅用官方 API、无深度依赖编译器内部行为的写法，可显著降低迁移成本
