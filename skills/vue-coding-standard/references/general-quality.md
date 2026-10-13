# Vue 通用质量规范

> 本文件为 vue-coding-standard 的**版本通用**质量规范，同时适用于 Vue 2 与 Vue 3（示例以 Vue 3 语法呈现，规则本身对 Options API 同样适用）。写法规约见分册：Vue 3 见 `vue3.md`，Vue 2 遗留见 `vue-legacy.md`。一～四章主干来自 Vue 官方 Style Guide（按 Priority A/B/C/D 分级，标注“官方 A/B/C/D”）。完整示例见 `style-examples.md`（同目录）。

## 目录

- 一、Priority A：Essential —— 多词组件名、详细 props、keyed v-for、v-if 与 v-for、v-html、组件作用域样式
- 二、Priority B：Strongly Recommended —— 一组件一文件、文件命名、Base 前缀、模板命名、props 大小写、多属性换行、引号、指令简写
- 三、Priority C：Recommended —— 元素属性顺序、SFC 块顺序、自闭合、模板简单表达式、复杂 computed 拆分
- 四、Priority D：Use with Caution —— scoped 元素选择器、隐式父子通信、非 flux 状态管理

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

```js
// ❌ 仅原型阶段可接受
props: ['status']

// ✅ 运行时校验式
props: {
  status: {
    type: String,
    required: true,
    validator: (value) => ['syncing', 'synced', 'error'].includes(value),
  },
}
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

同元素使用时指令优先级导致迭代变量先于定义被求值（Vue 3 直接报错），是高频错误源。过滤用派生数据，条件渲染上移容器：

```vue
<!-- ❌ -->
<li v-for="user in users" v-if="user.isActive" :key="user.id">…</li>

<!-- ✅ 过滤：派生数据 -->
<li v-for="user in activeUsers" :key="user.id">…</li>

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
3. **非 flux 状态管理**（官方 D）：业务全局状态进集中式状态管理（Vue 3 用 Pinia，见 `vue3.md`）；模块级可变导出对象 + 各处直接改写的模式禁用
