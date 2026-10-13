# Svelte 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：Svelte 官方文档与编译器 a11y 警告体系常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. Runes 状态体系完整示例
- 2. props 与双向绑定
- 3. 事件属性写法
- 4. snippet 复用
- 5. 共享状态（.svelte.js）
- 6. a11y 修复示例
- 7. 模板块（each / await）
- 8. 常见正反例速查

## 1. Runes 状态体系完整示例

```svelte
<script lang="ts">
  let keyword = $state('');

  const models: ModelInfo[] = $state([
    { id: 1, name: 'Alpha', status: 'deployed' },
    { id: 2, name: 'Beta', status: 'draft' },
  ]);

  // ✅ 派生：$derived（简单）/ $derived.by（多行）
  const deployedCount = $derived(models.filter((m) => m.status === 'deployed').length);
  const filtered = $derived.by(() => {
    const kw = keyword.trim().toLowerCase();
    return models.filter((m) => m.name.toLowerCase().includes(kw));
  });

  // ✅ effect 只做外部同步 + 清理
  $effect(() => {
    document.title = `模型（${filtered.length}）`;
  });
</script>

<input bind:value={keyword} />
<p>已部署 {deployedCount} 个</p>
```

```svelte
<script lang="ts">
  // ❌ watch 风格：effect 手工同步派生值
  let fullName = $state('');
  $effect(() => {
    fullName = `${first()} ${last()}`;
  });
</script>
```

## 2. props 与双向绑定

```svelte
<script lang="ts">
  // ✅ 解构 + 默认值 + 类型；rest 透传；$bindable 双向
  let {
    model,
    compact = false,
    onOpen,
    ...rest
  }: {
    model: ModelInfo;
    compact?: boolean;
    onOpen?: (id: number) => void;
  } & Record<string, unknown> = $props();

  let { value = $bindable('') } = $props();
</script>

<input bind:value {rest} />

<!-- 父组件 -->
<ModelCard bind:value={keyword} {model} onOpen={(id) => open(id)} />
```

```svelte
<script lang="ts">
  // ❌ 子组件变异 props 深层
  props.model.tags.push('x');
</script>
```

## 3. 事件属性写法

```svelte
<script lang="ts">
  function handleSubmit(e: SubmitEvent) {
    e.preventDefault();
    save(model);
  }
</script>

<!-- ❌ Svelte 4 指令形态（5 中已弃用） -->
<button on:click={open}>打开</button>
<form on:submit|preventDefault={handleSubmit}>…</form>

<!-- ✅ 属性形态；修饰符移入处理器 -->
<button type="button" onclick={open}>打开</button>
<form onsubmit={handleSubmit}>…</form>
```

组件回调 props 用 `onXxx`：

```svelte
<ModelCard onOpen={(id) => open(id)} />
```

## 4. snippet 复用

```svelte
<script lang="ts">
  import ModelCard from './ModelCard.svelte';
  let models = $state<ModelInfo[]>([ … ]);
</script>

{#snippet statusTag(status: ModelStatus)}
  <span class="tag">{status}</span>
{/snippet}

{#each models as model (model.id)}
  <ModelCard {model}>
    {#snippet footer()}
      {@render statusTag(model.status)}
    {/snippet}
  </ModelCard>
{/each}
```

子内容透传用隐式 `children`：

```svelte
<!-- Container.svelte -->
<script lang="ts">
  import type { Snippet } from 'svelte';
  let { children }: { children?: Snippet } = $props();
</script>

<div class="container">
  {@render children?.()}
</div>
```

## 5. 共享状态（.svelte.js）

```ts
// model-store.svelte.js
// ✅ 导出工厂函数（Runes 只能在 .svelte/.svelte.js 中使用）
export function createModelStore() {
  let models = $state<ModelInfo[]>([]);

  return {
    get models() { return models; },
    set models(v: ModelInfo[]) { models = v; },
    get deployedCount() { return models.filter((m) => m.status === 'deployed').length; },
    async fetchAll() { models = await modelApi.list(); },
  };
}
```

```svelte
<script lang="ts">
  // ✅ 应用级单例在入口组装一次；组件内消费
  const store = createModelStore();
</script>
```

❌ 导出全局单例可变量（跨请求共享、SSR 污染）：

```ts
export const store = { models: [] };   // 禁
```

## 6. a11y 修复示例

```svelte
<!-- ❌ div 假按钮 + 抑制警告 -->
<!-- svelte-ignore a11y_click_events_have_key_events, a11y_no_static_element_interactions -->
<div class="btn" onclick={open}>打开</div>

<!-- ✅ 原生语义 -->
<button type="button" class="btn" onclick={open}>打开</button>

<!-- ❌ 图片无 alt / input 无关联 label -->
<img src="/chart.png" />
<input type="email" />

<!-- ✅ -->
<img src="/chart.png" alt="季度销售趋势" />
<label for="email">邮箱</label>
<input type="email" id="email" />
```

## 7. 模板块（each / await）

```svelte
{#each filtered as model (model.id)}
  <ModelCard {model} />
{:else}
  <p class="empty">暂无模型</p>
{/each}

{#await loadModel(id)}
  <p class="loading">加载中…</p>
{:then data}
  <ModelView {data} />
{:catch error}
  <p class="error">{error.message}</p>
{/await}
```

❌ key 缺失（重排/过滤时状态错乱）与空 catch 分支。

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 派生值建 state | `$effect` 同步 | `$derived` / `$derived.by` |
| effect 做派生 | 手工同步 | `$derived`；effect 仅外部同步 |
| 旧事件指令 | `on:click` | `onclick={…}` |
| 修饰符 | `on:submit\|preventDefault` | 处理器内 `e.preventDefault()` |
| 动态组件 | `<svelte:component this>`（5 中可省） | 直接渲染组件/动态导入 |
| 变异 props | `props.model.tags.push` | 回调请求父级不可变更新 |
| 抑制 a11y | `svelte-ignore` 滥用 | 修复语义与键盘可达 |
| 全局样式污染 | 裸 `:global()` | 限定范围的 `:global()` |
| 跨组件状态 | 全局单例可变对象 | `.svelte.js` 工厂函数 |
| each 无 key | `{#each items as item}` | `{#each items as item (item.id)}` |
