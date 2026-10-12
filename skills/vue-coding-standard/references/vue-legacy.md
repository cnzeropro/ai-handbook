# Vue 2（遗留环境）规范

> 适用：Vue 2.x 遗留项目（官方已于 2023-12-31 EOL）。仅在维护既有代码或项目明确要求时使用；新项目与新组件一律 Vue 3 + Composition API（见 `general-quality.md`）。同一项目内保持 Options API 风格一致，不混用。

## 一、写法基线

- **Options API**：组件选项按官方推荐顺序组织：`name` → `components` → `props` → `data` → `computed` → `watch` → 生命周期钩子（`beforeCreate` → … → `destroyed`）→ 方法
- `data` 必须是**函数**并返回对象（防多实例共享引用）
- 派生数据用 `computed`，禁在 `data` 中冗余存储或用 `watch` 手工同步
- `v-for` 必带 key、`v-if` 与 `v-for` 禁同元素（Vue 2 中 `v-for` 优先级更高，同样禁止同元素）、禁对用户内容 `v-html`——官方 A 级规则与 Vue 3 通用
- 组件样式作用域化（`scoped` / CSS Modules / BEM）与 Vue 3 一致
- 事件名用 kebab-case：`this.$emit('model-change')`
- 全局状态用 Vuex（遗留项目既有约定优先；不引入 Pinia——Pinia 官方不支持 Vue 2 官方维护线）

```js
export default {
  name: 'ModelCard',
  props: {
    model: { type: Object, required: true },
  },
  data() {
    return { expanded: false };
  },
  computed: {
    statusText() {
      return this.model.status === 'deployed' ? '已部署' : '未部署';
    },
  },
  methods: {
    handleOpen() {
      this.$emit('open', this.model.id);
    },
  },
};
```

## 二、Vue 3 差异对照（升级参考）

| Vue 3 | Vue 2 等价写法 |
| --- | --- |
| `<script setup>` + `defineProps` | `props` 选项 |
| `ref` / `computed` 组合式 | `data` / `computed` 选项 |
| `emits` 选项显式声明 | `$emit` 隐式（无声明校验） |
| Pinia | Vuex |
| `defineModel` | `value` prop + `input` 事件 |
| 多根节点（Fragment） | 单根节点限制 |
| `onMounted` 等组合式钩子 | `mounted` 等选项钩子 |
| `Teleport` 内置 | 第三方 portal-vue |

## 三、遗留代码最小侵入

- 维护期**不顺手**把 Options 组件改写为 Composition API；升级作为独立任务执行
- Vue 2 项目内的新组件仍用 Options API，与项目整体保持一致，不越级使用 Vue 3 语法
- 升级路径：官方迁移指南 + `@vue/compat` 兼容构建逐步迁移； Vuex → Pinia 在 Vue 3 迁移完成后进行
