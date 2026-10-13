# Nuxt 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：nuxt.com 官方文档与 SSR 项目审查常见问题）。规则正文见 `SKILL.md` 与各版本分册；本文件供代码审查对照与深入学习。

## 目录

- 1. 页面与布局骨架
- 2. 数据获取正反例（key、SSR 一致性）
- 3. useState 与模块级单例坑
- 4. server/api 与 runtimeConfig
- 5. 渲染模式 routeRules
- 6. 中间件鉴权
- 7. 常见正反例速查

## 1. 页面与布局骨架

```vue
<!-- app/pages/models/[id].vue -->
<script setup lang="ts">
const route = useRoute()

// ✅ 显式 key + SSR 注水
const { data: model, error } = await useFetch(`/api/models/${route.params.id}`, {
  key: `model-${route.params.id}`,
})

if (error.value || !model.value) {
  throw createError({ statusCode: 404, message: '模型不存在' })
}

definePageMeta({ layout: 'default' })
useHead({ title: () => `${model.value?.name ?? ''} - 示例系统` })
</script>

<template>
  <ModelView v-if="model" :model="model" />
</template>
```

```vue
<!-- app/layouts/default.vue -->
<template>
  <div class="layout">
    <AppNav />
    <main><slot /></main>
  </div>
</template>
```

## 2. 数据获取正反例（key、SSR 一致性）

```vue
<script setup lang="ts">
// ❌ 无 key：SSR 载荷与客户端请求无法对应，重复请求且注水错乱
const { data } = await useFetch('/api/models')

// ❌ 首屏数据塞进 onMounted：SSR 白取，客户端再请求一遍
const models = ref([])
onMounted(async () => {
  models.value = await $fetch('/api/models')
})

// ✅ 显式 key + 查询参数入 key
const page = ref(1)
const { data: models } = await useFetch('/api/models', {
  key: `models-page-${page.value}`,
  query: { page },
})
</script>
```

hydration mismatch 防护：

```vue
<!-- ❌ 渲染期随机/时间戳：server 与 client 结果必然不同 -->
<span>{{ Date.now() }}</span>

<!-- ✅ 渲染在 onMounted 后（client-only 内容用 <ClientOnly> 包裹亦可） -->
<script setup>
const now = ref<number | null>(null)
onMounted(() => { now.value = Date.now() })
</script>
```

## 3. useState 与模块级单例坑

```ts
// ❌ 模块顶层可变单例：SSR 下所有用户共享，跨请求数据污染
export const useCart = () => {
  const items = []                       // 模块级，全服务端共享！
  return { items }
}

// ✅ useState：按请求隔离 + SSR 注水
export function useCart() {
  const items = useState<CartItem[]>('cart-items', () => [])
  function add(item: CartItem) { items.value.push(item) }
  return { items, add }
}
```

复杂领域状态用 Pinia（Nuxt 模块集成，setup store 形态）。

## 4. server/api 与 runtimeConfig

```ts
// server/utils/db.ts —— 服务端共享逻辑（自动导入）
export const db = createDb(useRuntimeConfig().dbUrl)
```

```ts
// server/api/models/index.post.ts
import { z } from 'zod'

const CreateModel = z.object({ name: z.string().min(1) })

export default defineEventHandler(async (event) => {
  const session = await requireAuth(event)                     // 鉴权先行
  const body = await readValidatedBody(event, CreateModel.parse)
  return db.insertModel(session.userId, body)
})
```

```ts
// nuxt.config.ts —— runtimeConfig 分层
export default defineNuxtConfig({
  runtimeConfig: {
    public: { apiBase: '/api' },        // 客户端可见
    dbUrl: '',                          // 私有：仅服务端（env: NUXT_DB_URL）
  },
})
```

```ts
// 调用侧
const config = useRuntimeConfig()
await $fetch(`${config.public.apiBase}/models`)
```

❌ 私有配置放 `public` 层；❌ server 端直接 `import` 含浏览器 API 的模块。

## 5. 渲染模式 routeRules

```ts
export default defineNuxtConfig({
  routeRules: {
    '/':          { prerender: true },                        // 首页预渲染
    '/docs/**':   { swr: 3600 },                              // 增量再生成
    '/admin/**':  { ssr: false },                             // 纯客户端后台
  },
})
```

## 6. 中间件鉴权

```ts
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  const session = useAuthSession()
  if (!session.loggedIn && to.path !== '/login') {
    return navigateTo('/login')
  }
})
```

```vue
<script setup>
definePageMeta({ middleware: 'auth' })
</script>
```

> 前端 middleware 只是体验层跳转，真正的安全边界在 `server/api` 鉴权（server 侧同样校验）。

## 7. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| useFetch 无 key | `useFetch('/api/models')` | 显式唯一 `key` |
| 首屏数据进 onMounted | `onMounted(async () => fetch)` | `await useFetch`（SSR 注水） |
| 模块级可变单例 | `export const cache = {…}` | `useState(key, init)` |
| 渲染期随机值 | `{{ Date.now() }}` | `onMounted` 后更新 / `<ClientOnly>` |
| $fetch 拉首屏 | `onMounted(() => $fetch(…))` | `useFetch` / `useAsyncData` |
| 秘钥进 public | `public: { dbUrl }` | 私有层 `runtimeConfig.dbUrl` |
| v-html 直插 | 用户内容直渲染 | DOMPurify 消毒 |
| 前端中间件当安全边界 | 只做前端跳转 | server API 同样鉴权 |
| 自造目录体系 | 顶层 `api/`、`services/` | 官方目录 + Nitro `server/` |
| 插件塞重依赖 | `plugins/` 引大库全局初始化 | 按需 import / lazy 组件 |
