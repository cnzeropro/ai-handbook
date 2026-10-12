# Vue 通用质量规范

> 本文件为 vue-coding-standard 的通用质量规范，基线 **Vue 3 + `<script setup>`**。一～四章主干来自 Vue 官方 Style Guide（按 Priority A/B/C/D 分级，标注“官方 A/B/C/D”），五～七章为组合式 API 开发约定。完整示例见 `style-examples.md`（同目录）。

## 目录

- 一、Priority A：Essential —— 多词组件名、详细 props、keyed v-for、v-if 与 v-for、v-html、组件作用域样式
- 二、Priority B：Strongly Recommended —— 一组件一文件、文件命名、Base 前缀、模板命名、props 大小写、多属性换行、引号、指令简写
- 三、Priority C：Recommended —— 元素属性顺序、SFC 块顺序、自闭合、模板简单表达式、复杂 computed 拆分
- 四、Priority D：Use with Caution —— scoped 元素选择器、隐式父子通信、非 flux 状态管理
- 五、组合式 API 约定 —— ref/reactive、computed 优先、watch 与清理、组合式函数、define 系列、provide/inject、生命周期清理
- 六、组件通信与状态 —— 单向数据流、defineModel、Pinia、attrs 透传
- 七、性能与安全 —— 异步组件、KeepAlive、v-memo、v-html 与 XSS、shallowRef、生产构建

## 一、Priority A：Essential（防错规则，无条件遵守）

### 1. 组件名多词（官方 A）

组件名至少两个单词（`TodoItem` 而非 `Todo`），根组件 `App` 除外——防与现有/未来 HTML 元素冲突：

```vue
<!-- ❌ -->
<Todo />
<!-- ✅ -->
<TodoItem />
```

### 2. 详细 props 定义（官方 A）

提交的代码 props 必须至少声明类型；带校验更佳——既文档化 API，又能在开发期告警：

```ts
// ❌ 仅原型阶段可接受
const props = defineProps(['status'])

// ✅
const props = defineProps<{
  status: 'syncing' | 'synced' | 'error'
}>()

// ✅ 运行时校验式
const props = defineProps({
  status: {
    type: String,
    required: true,
    validator: (value) => ['syncing', 'synced', 'error'].includes(value),
  },
})
```

### 3. v-for 必带 key（官方 A）

组件上 `key` 强制必带；元素上也应使用稳定业务 id（排序动画、焦点保持等依赖 object constancy）。慎用 `index` 作 key——列表顺序变化时会错乱复用：

```vue
<!-- ❌ -->
<li v-for="todo in todos">{{ todo.text }}</li>

<!-- ✅ -->
<li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
```

### 4. 禁止 v-if 与 v-for 同元素（官方 A）

Vue 3 中 `v-if` 优先级高于 `v-for`，迭代变量尚不存在即被求值，直接报错：

```vue
<!-- ❌ -->
<li v-for="user in users" v-if="user.isActive" :key="user.id">…</li>

<!-- ✅ 过滤：computed -->
<li v-for="user in activeUsers" :key="user.id">…</li>
const activeUsers = computed(() => users.filter((u) => u.isActive))

<!-- ✅ 条件隐藏：v-if 上移容器 -->
<ul v-if="shouldShowUsers">
  <li v-for="user in users" :key="user.id">…</li>
</ul>
```

### 5. 禁止对用户提供内容使用 v-html（官方 A）

XSS 注入直通车。确需渲染富文本时，先经服务端或 DOMPurify 类库消毒：

```vue
<!-- ❌ userContent 来自用户输入 -->
<div v-html="userContent"></div>
```

### 6. 组件样式作用域化（官方 A）

应用中除 `App` 与布局组件外，所有组件样式必须作用域化：`scoped`、CSS Modules 或 BEM 类名约定三选一；**组件库**优先类名方案（便于外部覆盖、避免高优先级）：

```vue
<!-- ❌ 裸全局样式 -->
<style>
.button { border: none; }
</style>

<!-- ✅ scoped -->
<style scoped>
.button { border: none; }
</style>

<!-- ✅ BEM + 前缀类名 -->
<style>
.c-Button--close { background-color: red; }
</style>
```

## 二、Priority B：Strongly Recommended

1. **一组件一文件**（官方 B）：需要构建拼接时，每个组件独立 `.vue` 文件
2. **SFC 文件名统一 PascalCase**（官方 B）：`ModelCard.vue`，与 JS 引用方式一致；`my-component.vue`/`myComponent.vue` 禁用
3. **基础组件 `Base` 前缀**（官方 B）：`BaseButton` / `BaseTable` / `BaseIcon`——只含元素、其他基础组件与三方 UI 组件，**不含全局状态**；字母序排列时自然聚组，且业务组件（`ButtonSubmit`）消费它们
4. **模板中组件名 PascalCase**（官方 B）：`<ModelCard />`；DOM 模板内用 kebab-case
5. **props 命名**（官方 B）：声明用 camelCase（`statusText`），模板传递用 kebab-case（`status-text`）
6. **多属性逐行**（官方 B）：属性超过一行容纳量时，每个属性独立一行

```vue
<ModelCard
  :title="model.title"
  :status="model.status"
  compact
/>
```

7. **模板统一双引号**（官方 B）：`:width="x"`、`class="card"`
8. **指令简写统一**（官方 B）：一律 `:` 代替 `v-bind:`、`@` 代替 `v-on:`，不混用

## 三、Priority C：Recommended

1. **元素属性顺序**（官方 C，与 eslint-plugin-vue 一致）：
   `is` → `v-for` → `v-if` / `v-else-if` / `v-else` → `v-show` / `v-cloak` / `v-pre` → `id` → `ref` / `key` → `v-model` → 自定义指令 → 普通属性 → 事件 `@` → `v-html` / `v-text`
2. **SFC 顶层块顺序**（官方 C）：`<script>` → `<template>` → `<style>`；项目统一即可，默认从官方
3. **组件自闭合**（官方 C）：`<ModelCard />`；多属性块之间留空行
4. **模板只放简单表达式**（官方 C）：`{{ model.title }}` 可以；三元与方法调用链复杂时移入 computed
5. **复杂 computed 拆分**（官方 C）：一个 computed 做一件事；能复用、可读、可测

## 四、Priority D：Use with Caution

1. **scoped 中的元素选择器**（官方 D）：`.card p { }` 会命中子组件根元素且性能差——给元素加类名，用类选择器
2. **隐式父子通信**（官方 D）：禁依赖 `this.$parent` 链或仅靠 `$attrs` 副作用传递数据——显式 props/emits 才是契约
3. **非 flux 状态管理**（官方 D）：业务全局状态进 Pinia；模块级可变导出对象 + 各处直接改写的模式禁用

## 五、组合式 API 约定

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

- `defineProps` 类型化（泛型式优先，见 A-2）
- `defineEmits<{ (e: 'change', value: string): void }>()` 声明全部事件
- v-model 用 `defineModel`（3.4+），替代 `modelValue` prop + `update:modelValue` 手工对

### 5. provide / inject

- 仅用于组件库与真正的深层上下文（主题、当前用户、i18n），业务数据流走 props + events
- provide 时用 `InjectionKey` 保证类型安全；提供方负责值的只读性（`readonly`）

### 6. 生命周期配对清理

`onMounted` 中注册的定时器 / 事件监听 / 订阅，必须在 `onUnmounted`（或 `onScopeDispose`）清理，防内存泄漏。

## 六、组件通信与状态

- **单向数据流**：props 下行、events 上行；子组件不得修改 props（含数组/对象的深层变更）
- **v-model**：`const model = defineModel<string>()`；多 v-model 用具名参数
- **Pinia**：一个领域一个 store（`useModelStore`）；state 仅存状态、getters 派生、actions 写操作；组件不绕过 store 直接改后端数据对象
- **事件命名**：kebab-case（`@model-change`）；原生事件优先 `$attrs` 透传，不手动二次 emit

## 七、性能与安全

- 路由级组件懒加载（`() => import('./views/ModelList.vue')`）；重组件 `defineAsyncComponent`
- `KeepAlive` 用于条件缓存（配合 `include` 白名单）；`v-memo` 仅在确认渲染瓶颈后使用
- 深层大对象用 `shallowRef` + 手动触发，避免全量深层响应式开销
- `v-html` 仅限完全可信内容（见 A-5）；生产构建关闭 devtools 与 sourcemap 外泄
