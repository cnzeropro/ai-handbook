# Vue 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：Vue 官方 Style Guide、Vue 3 官方文档常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. SFC 完整骨架正反例
- 2. Priority A 五规则示例（多词名/props/key/v-if 与 v-for/v-html）
- 3. 命名体系完整示例（文件名/Base 前缀/模板命名/props 大小写）
- 4. 元素属性顺序完整示例
- 5. 组合式 API 正反例（computed vs watch、ref/reactive、defineModel、组合式函数、清理）
- 6. 组件通信完整示例（单向数据流、Pinia）
- 7. 常见正反例速查

## 1. SFC 完整骨架正反例

```vue
<!-- ✅ 块顺序 script → template → style；script setup；类型化 props/emits；scoped -->
<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  model: ModelInfo
  compact?: boolean
}>()

const emit = defineEmits<{
  (e: 'open', id: number): void
}>()

const statusText = computed(() =>
  props.model.status === 'deployed' ? '已部署' : '未部署',
)
</script>

<template>
  <article class="model-card" :class="{ 'model-card--compact': compact }">
    <h3 class="model-card__title">{{ model.name }}</h3>
    <p class="model-card__status">{{ statusText }}</p>
    <button type="button" @click="emit('open', model.id)">查看</button>
  </article>
</template>

<style scoped>
.model-card { … }
</style>
```

```vue
<!-- ❌ 块顺序混乱、裸全局样式、props 数组式、无 emits 声明 -->
<template>
  <div class="card" @click="onOpen">{{ model.name }}</div>
</template>

<style>
.card { color: red; } /* 全局污染 */
</style>

<script>
export default {
  props: ['model'], // 无类型
  methods: {
    onOpen() {
      this.$parent.$emit('open', this.model.id) // 隐式父子通信（Priority D）
    },
  },
}
</script>
```

## 2. Priority A 五规则示例

多词组件名（官方 A-1）：

```vue
<!-- ❌ 与 HTML 元素冲突风险 -->
<Todo />
<!-- ✅ -->
<TodoItem />
```

详细 props（官方 A-2）：

```ts
// ❌ 仅原型阶段
const props = defineProps(['status', 'modelId'])

// ✅
const props = defineProps<{
  status: 'syncing' | 'synced' | 'error'
  modelId: number
}>()
```

keyed v-for（官方 A-3）：

```vue
<!-- ❌ index 作 key：删除/排序时状态错乱 -->
<li v-for="(todo, index) in todos" :key="index">{{ todo.text }}</li>

<!-- ✅ 稳定业务 id -->
<li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
```

v-if 与 v-for 不同元素（官方 A-4）：

```vue
<!-- ❌ Vue 3 中 v-if 先执行，user 不存在，直接报错 -->
<li v-for="user in users" v-if="user.isActive" :key="user.id">{{ user.name }}</li>

<!-- ✅ 过滤用 computed -->
<li v-for="user in activeUsers" :key="user.id">{{ user.name }}</li>

<!-- ✅ 条件隐藏：v-if 上移容器 -->
<ul v-if="shouldShowUsers">
  <li v-for="user in users" :key="user.id">{{ user.name }}</li>
</ul>
```

v-html 与用户内容（官方 A-5）：

```vue
<!-- ❌ XSS -->
<div v-html="comment.body"></div>

<!-- ✅ 纯文本插值；确需富文本先消毒（如 DOMPurify） -->
<p>{{ comment.body }}</p>
```

## 3. 命名体系完整示例

```
components/
|- BaseButton.vue          # 基础组件 Base 前缀
|- BaseTable.vue
|- BaseIcon.vue
|- ModelCard.vue           # 业务组件 PascalCase、多词
|- ModelList.vue
```

```vue
<!-- ✅ 模板 PascalCase；props camelCase 声明、kebab-case 传递 -->
<ModelCard
  :model-id="currentId"
  :status-text="statusText"
  compact
/>
```

```ts
// ❌
const props = defineProps({ modelId: Number, statusText: String }) // 声明用 kebab 了
// ✅ 声明 camelCase
const props = defineProps<{ modelId: number; statusText: string }>()
```

## 4. 元素属性顺序完整示例

`is` → `v-for` → 条件指令 → `id` → `ref`/`key` → `v-model` → 自定义指令 → 属性 → 事件 → `v-html`：

```vue
<MyList
  v-for="item in items"
  :key="item.id"
  ref="listItem"
  v-model:focus-id="focusId"
  v-highlight="item.tag"
  :title="item.title"
  :compact="item.compact"
  @select="onSelect"
/>
```

## 5. 组合式 API 正反例

computed 优先于 watch：

```ts
// ❌ watch 模拟派生数据
const fullName = ref('')
watch([first, last], () => {
  fullName.value = `${first.value} ${last.value}`
})

// ✅
const fullName = computed(() => `${first.value} ${last.value}`)
```

ref 默认，reactive 受限：

```ts
// ❌ reactive 整体替换丢响应
const state = reactive({ list: [] })
function reset() {
  state = reactive({ list: fetchAll() }) // 编译错误，且解构已失响应
}

// ✅
const list = ref<ModelInfo[]>([])
function reset() {
  list.value = fetchAll()
}
```

defineModel 替代手工 modelValue 对：

```ts
// ❌
const props = defineProps<{ modelValue: string }>()
const emit = defineEmits<{ (e: 'update:modelValue', v: string): void }>()
function onInput(v: string) {
  emit('update:modelValue', v)
}

// ✅ 3.4+
const model = defineModel<string>()
```

组合式函数与清理：

```ts
// ✅ useXxx：输入 ref/getter，返回 refs；副作用配对清理
export function usePolling(fn: () => void, interval = 5000) {
  let timer: number | undefined
  onMounted(() => {
    timer = window.setInterval(fn, interval)
  })
  onUnmounted(() => window.clearInterval(timer))
}
```

## 6. 组件通信完整示例

单向数据流 + emit 上行：

```vue
<!-- ❌ 子组件直接改 props 对象属性 -->
<script setup>
function removeTag(tag) {
  props.model.tags.splice(props.model.tags.indexOf(tag), 1) // 变异了父级数据
}
</script>

<!-- ✅ emit 请求，父级改数据 -->
<script setup lang="ts">
const props = defineProps<{ model: ModelInfo }>()
const emit = defineEmits<{ (e: 'remove-tag', tag: string): void }>()
function removeTag(tag: string) {
  emit('remove-tag', tag)
}
</script>
```

Pinia 领域 store：

```ts
// ✅ state 状态 / getters 派生 / actions 写操作
export const useModelStore = defineStore('model', () => {
  const models = ref<ModelInfo[]>([])
  const deployedCount = computed(() => models.value.filter((m) => m.status === 'deployed').length)
  async function fetchAll() {
    models.value = await modelApi.list()
  }
  return { models, deployedCount, fetchAll }
})
```

## 7. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| v-for 无 key / index key | `v-for="t in ts"` | `:key="t.id"` |
| v-if + v-for 同元素 | 过滤写在 v-if | computed 过滤 / v-if 上移 |
| 修改 props | `props.model.x = 1` | emit 请求父级变更 |
| watch 模拟 computed | watch 双赋值 | `computed` |
| 全局样式泄漏 | 裸 `<style>` | `scoped` / module / BEM |
| 巨型模板表达式 | 三元嵌套 + 方法链 | 拆 computed |
| 隐式父子通信 | `this.$parent.xxx` | 显式 props / emits |
| 状态对象绕过 Pinia | 直接改导出的可变对象 | store actions |
| 手写 v-model 对 | modelValue + update:* | `defineModel` |
| 忘记清理副作用 | `onMounted` 里 `setInterval` 不清 | `onUnmounted` 配对清理 |
