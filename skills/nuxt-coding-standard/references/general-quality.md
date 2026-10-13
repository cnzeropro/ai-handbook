# Nuxt 通用质量规范

> 本文件为 nuxt-coding-standard 的**版本通用**质量规范（Nuxt 2 与 3/4 通用部分）。写法规约见分册：Nuxt 3/4 见 `nuxt3-4.md`，Nuxt 2 遗留见 `nuxt-legacy.md`。来源：nuxt.com 官方文档。完整正反例见 `style-examples.md`（同目录）。Vue 组件层规则遵循 `vue-coding-standard`。

## 目录

- 一、目录约定与自动导入
- 二、页面与组件
- 三、数据获取
- 四、状态管理
- 五、服务端能力（server/ 与 Nitro）
- 六、渲染模式与性能
- 七、安全与配置

## 一、目录约定与自动导入

- 使用官方目录：`pages/`（文件路由）、`components/`（自动导入）、`composables/`（`useXxx` 自动可用）、`utils/`、`layouts/`、`plugins/`、`server/`（API 与工具）、`public/`（静态资源）；Nuxt 4 默认 `app/` 目录聚合
- **不自造平行目录体系**（如 `api/`、`services/` 顶层目录）；数据访问放 `composables`（客户端包装）与 `server/utils`（服务端共享）
- 自动导入：`ref` / `computed` / `useFetch` / `useState` 与自建 composables/components 自动可用
- 团队二选一并统一：依赖自动导入，或显式 `import`（显式更利于跳转与 tree-shaking 把控；自动导入是 Nuxt 默认体验）

## 二、页面与组件

- `pages/` 文件路由：`pages/models/[id].vue` → `/models/:id`；嵌套路由用同名 `<NuxtPage>` 父子结构
- 布局放 `layouts/`，页面 `definePageMeta({ layout: 'default' })` 指定
- 组件名 PascalCase、多词；`components/` 内按域建子目录（自动导入以目录为前缀可读性更好）
- 页面级中间件用 `definePageMeta({ middleware: 'auth' })`；全局中间件放 `middleware/` 并 `.global` 后缀

## 三、数据获取

- 页面/组件的**注水数据**用 `useFetch` / `useAsyncData`，且**必须显式 key**（SSR 载荷去重与客户端复用依赖它）：

```vue
<script setup lang="ts">
// ✅ 显式 key + 传递响应式参数
const { data: model, error } = await useFetch(`/api/models/${id}`, {
  key: `model-${id}`,
});
</script>
```

- `$fetch` 仅用于客户端事件触发的请求（提交表单等）；它不参与 SSR 注水
- 禁在 `useEffect`/`onMounted` 里拉首屏数据（SSR 白取，客户端重复请求）
- 同一数据多处使用：同 key 复用（`useAsyncData(key, fn)` 去重）

## 四、状态管理

- 跨组件共享状态用 `useState('key', init)`（SSR 安全、按 key 单例）：

```ts
const count = useState('counter', () => 0);
```

- 复杂领域状态用 Pinia（Nuxt 官方集成，模块自动导入）；**禁模块顶层可变单例**——服务端所有请求共享同一模块实例，会造成跨请求数据污染（Nuxt 第一大坑）：

```ts
// ❌ 模块级可变单例：SSR 下所有用户共享
export const cache = { items: [] };

// ✅ 每请求独立（useState / Nitro storage / Pinia）
const cache = useState<Model[]>('model-cache', () => []);
```

## 五、服务端能力（server/ 与 Nitro）

- API 放 `server/api/models.get.ts` / `models.post.ts`（文件名即方法）；`server/utils/` 共享服务端逻辑
- 处理器校验入参（h3 `readValidatedBody` + zod）、鉴权先行——server API 是公开端点

```ts
// server/api/models/index.post.ts
export default defineEventHandler(async (event) => {
  const session = await requireAuth(event);
  const body = await readValidatedBody(event, CreateModelSchema.parse);
  const model = await db.insertModel(session, body);
  return model;
});
```

- 秘密与配置：`runtimeConfig` 分层——`public`（客户端可见）与私有（仅服务端）；禁把密钥放 `public`

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    public: { apiBase: '/api' },
    dbUrl: '',            // 私有层：NODE_ENV 注入
  },
});
```

## 六、渲染模式与性能

- 默认 SSR；渲染模式按路由声明 `routeRules`（`/docs/**: { prerender: true }`、`/admin/**: { ssr: false }`）
- 非关键组件用 `Lazy` 前缀自动按需加载（`<LazyHeavyChart />`）
- 图片用 `@nuxt/image`（自动格式/尺寸/懒加载）；payload 精简：大对象不整包注水（`experimental: { payloadExtraction }` 按项目）
- 谨慎大依赖进 `plugins/`（插件在每次请求/启动执行）；能 tree-shake 的库按需 import

## 七、安全与配置

- `v-html` / `innerHTML`：用户内容必须消毒（与 vue skill 同规则）
- middleware 鉴权（`defineNuxtRouteMiddleware`），未登录重定向登录页；server API 侧同样校验（前端中间件不是安全边界）
- CSP 等安全头在 `nuxt.config` `security` / Nitro 层统一配置
- 依赖升级跟随 Nuxt 大版本（Nuxt 4 目录变更见 `nuxt3-4.md`）
