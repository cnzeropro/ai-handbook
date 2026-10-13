# Next.js App Router 规范（当前主推）

> 本文件为 **App Router**（RSC 体系）版本特有约定，须与版本通用的 `general-quality.md` 配合使用。Pages Router 见 `pages-router.md`。适用 Next 13.4+；Next 15 起 `params` / `searchParams` 为 Promise 需 await。

## 目录

- 一、文件约定 —— page / layout / loading / error / not-found / route / template
- 二、Server Components 与边界 —— 'use client'、children 组合
- 三、数据获取与缓存 —— fetch 语义、revalidate
- 四、Server Actions —— 变更与表单
- 五、metadata —— 静态与动态
- 六、params / searchParams —— Next 15 异步化
- 七、middleware 与边缘

## 一、文件约定

| 文件 | 职责 | 注意 |
| --- | --- | --- |
| `page.tsx` | 路由唯一 UI 入口 | 禁同目录无 page 只放组件 |
| `layout.tsx` | 嵌套布局（保留状态、不重渲染） | 根 layout 必含 `<html>`/`<body>` |
| `loading.tsx` | 段级 Suspense 骨架 | 配合流式 |
| `error.tsx` | 段级错误 UI | 必须 `'use client'`，含 `reset()` 重置按钮 |
| `not-found.tsx` | 404 UI | `notFound()` 抛出即渲染 |
| `route.ts` | API 端点（GET/POST…） | 与 page 同目录冲突 |
| `template.tsx` | 每次导航重渲染的布局 | 默认用 layout |

- 组织：`_components/`、`_lib/` 私有目录 colocate；`(marketing)` 路由组做路径无关分组

## 二、Server Components 与边界

- 默认 Server：直接 `async` 组件 + `await` 数据；不写 hooks/事件/浏览器 API
- `'use client'` 标边界文件；交互件下沉叶子；**children 组合模式**保持 server 渲染：

```tsx
// ✅ server 页面组合 client 交互件
// app/models/page.tsx（server）
import { ModelsTable } from './_components/models-table';        // server：直查库
import { ModelsToolbar } from './_components/models-toolbar';    // client：交互

export default async function Page() {
  const models = await getModels();
  return (
    <ModelsToolbar>
      <ModelsTable models={models} />     {/* server children 传入 client 布局 */}
    </ModelsToolbar>
  );
}
```

- Server → Client 只传可序列化 props；敏感数据（密钥、多余字段）不进 client props

## 三、数据获取与缓存

- server 端 `fetch` 显式声明缓存意图，不依赖默认值记忆：

```ts
// 静态内容：构建期缓存
fetch('https://…/config', { cache: 'force-cache' });
// 实时数据：每次请求取新
fetch('https://…/quotes', { cache: 'no-store' });
// 周期再验证
fetch('https://…/models', { next: { revalidate: 60, tags: ['models'] } });
```

- 同一 URL 的 fetch 在单次渲染内自动去重（Request Memoization）
- 数据库直查（ORM）不经过 fetch 缓存——用 `unstable_cache`（或升级缓存方案）+ tag 失效
- 写操作后失效：`revalidatePath('/models')` / `revalidateTag('models')`

## 四、Server Actions

- `'use server'` 文件集中放写操作；或内联于 client 表单
- 入参出参必须可序列化；视为公开端点：鉴权 + zod 校验 + 错误返回值

```ts
// app/models/actions.ts
'use server';

export async function createModel(formData: FormData) {
  const session = await auth();
  if (!session) return { ok: false as const, error: 'unauthorized' };
  const parsed = CreateModelSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { ok: false as const, error: 'invalid input' };
  await db.insertModel(parsed.data);
  revalidatePath('/models');
  return { ok: true as const };
}
```

```tsx
// ✅ 表单直挂 action + useActionState 反馈
const [state, action] = useActionState(createModel, { ok: true, error: '' });
<form action={action}>…</form>
```

## 五、metadata

```ts
// 静态
export const metadata: Metadata = { title: '用户中心 - 示例系统', description: '…' };

// 动态（按数据生成）
export async function generateMetadata({ params }): Promise<Metadata> {
  const model = await getModel((await params).id);
  return { title: `${model.name} - 示例系统` };
}
```

- 文件约定资源：`favicon.ico` / `opengraph-image.tsx` / `sitemap.ts` / `robots.ts`

## 六、params / searchParams（Next 15 异步化）

```ts
// Next 15+：Promise 化，需 await
export default async function Page({ params, searchParams }: {
  params: Promise<{ id: string }>;
  searchParams: Promise<{ q?: string }>;
}) {
  const { id } = await params;
  const { q } = await searchParams;
}
```

（Next 14 及之前为同步对象；升级时逐页改造。）

## 七、middleware 与边缘

- `middleware.ts` 用于鉴权重定向、A/B、安全头；保持轻量（每次请求执行）
- 边缘运行时禁 Node 专属 API（按 runtime 声明选择 `edge` / `nodejs`）
