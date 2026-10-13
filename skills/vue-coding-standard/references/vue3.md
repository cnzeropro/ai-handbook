# Vue 3 规范（当前主版本）

> 本文件为 **Vue 3** 写法规约（Composition API、`<script setup>`、Pinia 等），须与版本通用的 `general-quality.md`（官方 Style Guide A-D）配合使用。Vue 2 遗留环境见 `vue-legacy.md`。

## 目录

- 一、组合式 API 约定 —— ref/reactive、computed 优先、watch 与清理、组合式函数、define 系列、provide/inject、生命周期清理
- 二、组件通信与状态 —— defineModel、Pinia、attrs 透传
- 三、性能与安全 —— 异步组件、KeepAlive、v-memo、shallowRef、生产构建

## 一、组合式 API 约定

### 1. ref 与 reactive

- 默认 `ref`：通用、可整体替换、解构不失响应
- `reactive` 仅用于关系紧密的成组局部状态；**不可整体替换**（解构与重赋值均失响应）

### 2. computed 优先于 watch

- 派生数据一律 `computed`（缓存、声明式、可测试）
- `watch` 用于副作用：请求、持久化、操作 DOM；注意 `immediate` 语义与清理函数

```js
// ❌ watch 模拟 computed
const fullName = ref('')
watch(() => [first.value, last.value], () => {
  fullName.value = `${first.value} ${last.value}`
})

// ✅
const fullName = computed(() => `${first.value} ${last.value}`)
```

### 3. 组合式函数（composables）

- 可复用有状态逻辑抽为 `useXxx`（`useModelList` / `usePermission`）
- 输入接受 ref/getter，返回 refs 对象；内部用到的定时器/监听器在 `onUnmounted`/`onScopeDispose` 清理

### 4. define 系列

- `defineProps` 类型化（泛型式优先，见 `general-quality.md` A-2）
- `defineEmits<{ (e: 'change', value: string): void }>()` 声明全部事件
- v-model 用 `defineModel`（3.4+），替代 `modelValue` prop + `update:modelValue` 手工对

### 5. provide / inject

- 仅用于组件库与真正的深层上下文（主题、当前用户、i18n），业务数据流走 props + events
- provide 时用 `InjectionKey` 保证类型安全；提供方负责值的只读性（`readonly`）

### 6. 生命周期配对清理

`onMounted` 中注册的定时器 / 事件监听 / 订阅，必须在 `onUnmounted`（或 `onScopeDispose`）清理，防内存泄漏。

## 二、组件通信与状态

- **单向数据流**：props 下行、events 上行；子组件不得修改 props（含数组/对象的深层变更）
- **v-model**：`const model = defineModel<string>()`；多 v-model 用具名参数
- **Pinia**：一个领域一个 store（`useModelStore`）；state 仅存状态、getters 派生、actions 写操作；组件不绕过 store 直接改后端数据对象

```ts
// ✅ Pinia 领域 store
export const useModelStore = defineStore('model', () => {
  const models = ref<ModelInfo[]>([])
  const deployedCount = computed(() => models.value.filter((m) => m.status === 'deployed').length)
  async function fetchAll() {
    models.value = await modelApi.list()
  }
  return { models, deployedCount, fetchAll }
})
```

- **事件命名**：kebab-case（`@model-change`）；原生事件优先 `$attrs` 透传，不手动二次 emit

## 三、性能与安全

- 路由级组件懒加载（`() => import('./views/ModelList.vue')`）；重组件 `defineAsyncComponent`
- `KeepAlive` 用于条件缓存（配合 `include` 白名单）；`v-memo` 仅在确认渲染瓶颈后使用
- 深层大对象用 `shallowRef` + 手动触发，避免全量深层响应式开销
- `v-html` 仅限完全可信内容（见 `general-quality.md` A-5）；生产构建关闭 devtools 与 sourcemap 外泄
