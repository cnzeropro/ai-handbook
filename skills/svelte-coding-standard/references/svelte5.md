# Svelte 5 规范（Runes，当前主版本）

> 本文件为 **Svelte 5** 版本特有约定（Runes 体系），须与版本通用的 `general-quality.md` 配合使用。Svelte 4 遗留环境见 `svelte-legacy.md`。

## 目录

- 一、Runes 状态体系 —— $state / $derived / $effect
- 二、props 与双向绑定 —— $props / $bindable / rest props
- 三、事件属性 —— onclick 标准属性形态
- 四、snippet —— 复用标记与 children
- 五、共享状态 —— .svelte.js 工厂函数
- 六、effect 边界与生命周期

## 一、Runes 状态体系

### 1. $state

本地响应式状态；对象/数组自动深响应（proxy）：

```svelte
<script lang="ts">
  let keyword = $state('');
  let models: ModelInfo[] = $state([]);
</script>
```

类字段同样直接 `$state`；纯展示转换禁用 `$state` 冗余存储（用 `$derived`）。

### 2. $derived / $derived.by

派生数据一律 `$derived`（简单表达式）或 `$derived.by`（多行逻辑）；**只读、无副作用**：

```svelte
<script lang="ts">
  // ❌ 手工同步派生值
  let fullName = $state('');
  $effect(() => { fullName = `${first()} ${last()}`; });

  // ✅
  const fullName = $derived(`${first()} ${last()}`);

  // 多行派生用 $derived.by
  const filtered = $derived.by(() => {
    const kw = keyword.value.trim().toLowerCase();
    return models.filter((m) => m.name.toLowerCase().includes(kw));
  });
</script>
```

### 3. $effect

仅用于**与外部系统同步**（订阅、日志、操作 canvas/document）；状态派生禁用（用 `$derived`）；事件驱动的逻辑放事件处理器：

```svelte
<script lang="ts">
  // ✅ 同步外部系统 + 返回清理函数
  $effect(() => {
    const timer = setInterval(refresh, 5000);
    return () => clearInterval(timer);
  });
</script>
```

## 二、props 与双向绑定

```svelte
<script lang="ts">
  // ✅ 解构 + 默认值 + 类型标注；rest props 透传
  let {
    model,
    compact = false,
    onOpen,
    ...rest
  }: {
    model: ModelInfo;
    compact?: boolean;
    onOpen?: (id: number) => void;
  } = $props();

  // ✅ 双向绑定字段
  let { value = $bindable('') } = $props();
</script>
```

- 子组件不得变异 props 深层内容；变更通过回调 props 请求父级
- 需要子组件回写表单值时才用 `$bindable`，其余一律单向

## 三、事件属性

Svelte 5 用标准 DOM 属性形态（不再用 `on:` 指令）；修饰符改由处理器或包装函数实现：

```svelte
<!-- ❌ Svelte 4 指令形态（5 中已弃用） -->
<button on:click={open} on:click|once={init}>…</button>

<!-- ✅ 属性形态 -->
<button type="button" onclick={open}>…</button>
<form onsubmit={(e) => { e.preventDefault(); submit(); }}>…</form>
```

组件回调 props 用 `onXxx`（见 `general-quality.md` 五）。

## 四、snippet

复用标记用 snippet（替代 slot）；子内容透传用隐式 `children`：

```svelte
<script lang="ts">
  import ModelCard from './ModelCard.svelte';
</script>

{#snippet header(text: string)}
  <h3 class="title">{text}</h3>
{/snippet}

<ModelCard {model}>
  {#snippet footer()}      <!-- 具名片段传给子组件 -->
    <footer>…</footer>
  {/snippet}

  {@render header(model.name)}
  {@render children()}     <!-- 透传调用方内容 -->
</ModelCard>
```

简单插槽场景仍可用 `<slot />`（兼容形态），新代码统一 snippet。

## 五、共享状态

`.svelte.js` 模块导出**工厂函数**（内部用 Runes），跨组件复用响应式状态：

```ts
// model-store.svelte.js
export function createModelStore() {
  let models = $state<ModelInfo[]>([]);
  const deployed = $derived(models.filter((m) => m.status === 'deployed'));

  return {
    get models() { return models; },
    set models(v) { models = v; },
    get deployed() { return deployed; },
    async fetchAll() { models = await modelApi.list(); },
  };
}
```

- 导出工厂而非单例实例（避免跨请求共享）；应用级单例在入口组装一次

## 六、effect 边界与生命周期

- `$effect` 依赖自动跟踪；返回函数即清理
- 需要在 DOM 更新前读取的用 `$effect.pre`；调试用 `$inspect`
- `onMount` / `onDestroy` 仍可用；`onMount` 中的清理函数返回值语义与 `$effect` 一致
- 无关状态读取用 `untracked()` 隔离，减少意外重跑
