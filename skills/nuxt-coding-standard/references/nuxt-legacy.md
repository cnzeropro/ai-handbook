# Nuxt 2（遗留环境）规范

> 适用：Nuxt 2.x 遗留项目。仅在维护既有代码或项目明确要求时使用；新项目一律 Nuxt 3/4（见 `nuxt3-4.md`）。同一项目保持写法一致，不混用两代语法。

## 一、写法基线

- **Options API + Nuxt 2 专属钩子**：`asyncData`（页面组件，SSR 数据注水）、`fetch()` 钩子（可访问 this）、`head()`（meta）
- 目录：`pages/` / `layouts/` / `middleware/` / `plugins/` / `store/`（Vuex 模块）/ `static/`（静态资源，Nuxt 3 改名 public）
- 全局状态用 Vuex（或遗留项目的既有方案）；新代码不引入新状态库形态

```js
export default {
  name: 'ModelList',
  async asyncData({ $axios, params }) {
    const models = await $axios.$get(`/api/models`, { params });
    return { models };
  },
  head() {
    return { title: '模型列表 - 示例系统' };
  },
};
```

- 配置：`nuxt.config.js`（`head` / `buildModules` / `modules`）；`@nuxtjs/axios` / `@nuxtjs/auth-next` 等模块按项目既有约定

## 二、与 Nuxt 3 差异对照（升级参考）

| Nuxt 3/4 | Nuxt 2 |
| --- | --- |
| `useFetch` / `useAsyncData` | `asyncData` / `fetch()` 钩子 |
| `useState` / Pinia | Vuex `store/` |
| `runtimeConfig` | `publicRuntimeConfig` / `privateRuntimeConfig` |
| Nitro `server/api` | `pages/api` 不存在（Express 中间件集成） |
| `app/` 目录（Nuxt 4） | 根目录平铺 |
| Vue 3 Composition API | Vue 2 Options API |
| Vite 构建 | webpack |

## 三、遗留代码最小侵入

- 不顺手做语法迁移（Options → Composition、Vuex → Pinia）；升级作为独立任务
- 遗留库中的新页面/组件与项目风格一致；安全修复不引入 Nuxt 3 专属 API
- 升级路径：官方 Nuxt Bridge（`@nuxt/bridge`）兼容层先行，再迁 Nuxt 3/4；迁移前盘点依赖的 Nuxt 2 模块是否有 Nuxt 3 等价物
