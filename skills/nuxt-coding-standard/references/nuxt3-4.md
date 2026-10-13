# Nuxt 3/4 规范（当前版本）

> 本文件为 **Nuxt 3/4** 写法规约（Nitro 引擎、自动导入、`app/` 目录），须与版本通用的 `general-quality.md` 配合使用。Nuxt 2 遗留环境见 `nuxt-legacy.md`。

## 目录

- 一、版本阶梯 —— Nuxt 3 与 Nuxt 4 的关系
- 二、app/ 目录结构（Nuxt 4）
- 三、useFetch / useAsyncData 选项
- 四、渲染模式 routeRules
- 五、Nitro 细节 —— server 路由、存储与缓存

## 一、版本阶梯

- **Nuxt 3**：Nitro 引擎、自动导入、组合式数据获取
- **Nuxt 4**（当前推荐新项目）：默认 `app/` 目录聚合（`app/pages` / `app/components` / `app/composables`…），`server/` 保留在根；`useAsyncData`/`useFetch` 返回的 `data` 为更精细的响应式语义（不易误写 null 断言）
- 升级：官方 `nuxi upgrade` + codemod；升级属于独立任务

## 二、app/ 目录结构（Nuxt 4）

```
app/
├── pages/            # 文件路由
├── components/       # 自动导入组件
├── composables/      # useXxx 自动导入
├── utils/
├── layouts/
├── middleware/
├── plugins/
├── app.vue
server/               # 保留在根：Nitro 服务端
├── api/
└── utils/
public/               # 静态资源
nuxt.config.ts
```

## 三、useFetch / useAsyncData 选项

| 选项 | 作用 | 约定 |
| --- | --- | --- |
| `key` | SSR 载荷去重与客户端复用 | **必填且全局唯一**（含参数指纹） |
| `server: false` | 仅客户端执行 | 登录后实时数据等非首屏场景 |
| `lazy` | 不阻塞导航 | 配合 `status` 展示骨架 |
| `immediate: false` | 手动 `execute()` | 搜索、分页 |
| `watch` | 响应式参数变化自动重取 | 分页/筛选 |
| `transform` | 注水前精简 payload | 大列表裁字段，减小注水体积 |
| `default` | 立即给出的默认值 | 防模板空引用 |

```ts
// ✅ key 含参数指纹 + watch 联动分页
const { data, status } = await useFetch('/api/models', {
  key: `models-${page.value}-${keyword.value}`,
  query: { page: page, q: keyword },
  watch: [page, keyword],
});
```

- 同 URL 不同选项的请求：key 必须不同，避免载荷互相覆盖
- 客户端事件触发（提交表单）用 `$fetch`，不用 `useFetch`

## 四、渲染模式 routeRules

按路由声明渲染与缓存策略（Nitro 层生效）：

```ts
export default defineNuxtConfig({
  routeRules: {
    '/':                { prerender: true },            // 首页预渲染
    '/docs/**':         { isr: 3600 },                  // 增量再生成（Nitro）
    '/admin/**':        { ssr: false },                 // 纯 SPA 后台
    '/models/**':       { headers: { 'cache-control': 's-maxage=60' } },
  },
});
```

- 默认 SSR；营销页预渲染、后台 SPA、API 走 Nitro

## 五、Nitro 细节

- `server/api/**.get.ts` / `**.post.ts` 文件路由；`server/routes/` 非约定路由；`server/utils/` 服务端共享逻辑（自动导入）
- 服务端存储/缓存用 Nitro `useStorage()`（`runtimeConfig` 挂载驱动：内存/Redis/文件）
- `defineCachedEventHandler` / `defineCachedFunction` 缓存服务端计算

```ts
// server/api/models/index.get.ts
export default defineEventHandler(async (event) => {
  const query = getQuery(event);
  return db.listModels(query);          // 鉴权中间件已在 server/middleware 统一处理
});
```
