# Next.js 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：nextjs.org 官方文档与 react.dev 同源原则）。规则正文见 `SKILL.md` 与各版本分册；本文件供代码审查对照与深入学习。

## 目录

- 1. 页面骨架（layout / page / loading / error）
- 2. Server 与 Client 边界（children 组合模式）
- 3. 数据获取与 Server Action 表单
- 4. 缓存与失效
- 5. metadata 与 SEO
- 6. 环境变量
- 7. Pages Router 对照
- 8. 常见正反例速查

## 1. 页面骨架（layout / page / loading / error）

```tsx
// app/models/layout.tsx —— 嵌套布局（保留状态）
export default function ModelsLayout({ children }: { children: React.ReactNode }) {
  return (
    <section>
      <ModelsNav />
      {children}
    </section>
  );
}

// app/models/page.tsx —— server 页面：直查数据
export default async function ModelsPage() {
  const models = await getModels();
  return <ModelsTable models={models} />;
}

// app/models/loading.tsx —— 段级骨架
export default function Loading() {
  return <ModelsSkeleton />;
}

// app/models/error.tsx —— 必须 'use client' + reset
'use client';
export default function ModelsError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div role="alert">
      <p>加载失败：{error.message}</p>
      <button type="button" onClick={reset}>重试</button>
    </div>
  );
}
```

❌ 手工 `<head>` 标签、缺 `error.tsx`（错误白屏）、`page` 里堆全部子组件不拆分。

## 2. Server 与 Client 边界（children 组合模式）

```tsx
// ❌ 为用交互件把整页 'use client'（数据获取全部退化到客户端）
'use client';
export default function Page() {
  const [models] = use(fetch('/api/models'));
  return <FilterableTable models={models} />;
}

// ✅ server 获取数据；client 只包交互壳，children 保持 server 渲染
import { ModelsToolbar } from './_components/models-toolbar';   // 'use client'
import { ModelsTable } from './_components/models-table';       // server

export default async function Page() {
  const models = await getModels();
  return (
    <ModelsToolbar>
      <ModelsTable models={models} />
    </ModelsToolbar>
  );
}
```

❌ Server → Client 传函数/类实例（不可序列化）；❌ client 组件内直连数据库。

## 3. 数据获取与 Server Action 表单

```ts
// app/models/actions.ts —— 'use server' 文件集中放写操作
'use server';

import { revalidatePath } from 'next/cache';

export async function createModel(_prev: ActionState, formData: FormData): Promise<ActionState> {
  const session = await auth();
  if (!session) return { ok: false, message: '未登录' };
  const parsed = CreateModelSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { ok: false, message: '输入不合法' };
  await db.insertModel(parsed.data);
  revalidatePath('/models');                    // ✅ 写后失效缓存
  return { ok: true, message: '' };
}
```

```tsx
// app/models/new/page.tsx 内（client 表单）
const [state, action] = useActionState(createModel, { ok: true, message: '' });
<form action={action}>
  <input name="name" required />
  <button type="submit" disabled={isPending}>创建</button>
  {state.message && <p role="alert">{state.message}</p>}
</form>
```

❌ 在 useEffect 中拉首屏数据、❌ Server Action 不鉴权不校验。

## 4. 缓存与失效

```ts
// 显式声明缓存意图
fetch('/api/config', { cache: 'force-cache' });                 // 静态
fetch('/api/quotes', { cache: 'no-store' });                    // 实时
fetch('/api/models', { next: { revalidate: 60, tags: ['models'] } });

// 带 tag 的 ORM 查询（无 fetch 缓存链路时）
import { unstable_cache } from 'next/cache';
const getModels = unstable_cache(
  () => db.listModels(),
  ['models'],
  { revalidate: 60, tags: ['models'] },
);
```

❌ 写操作后忘记 `revalidatePath`/`revalidateTag`（页面显示旧数据）。

## 5. metadata 与 SEO

```ts
// 静态
export const metadata: Metadata = {
  title: '用户中心 - 示例系统',
  description: '管理您的模型与任务。',
};

// 动态
export async function generateMetadata({ params }: { params: Promise<{ id: string }> }): Promise<Metadata> {
  const { id } = await params;
  const model = await getModel(id);
  return { title: `${model.name} - 示例系统`, openGraph: { images: [model.cover] } };
}
```

❌ 手写 `<head><title>`（App Router 用 Metadata API）。

## 6. 环境变量

```ts
// ❌ 密钥加 NEXT_PUBLIC_ 前缀：构建期内联进客户端产物
const secret = process.env.NEXT_PUBLIC_API_SECRET;

// ✅ 秘钥仅服务端读取 + 启动期校验
const { DATABASE_URL } = envSchema.parse(process.env);   // zod 解析，缺失即失败
```

❌ 客户端代码读取 `process.env.DATABASE_URL`。

## 7. Pages Router 对照

```tsx
// pages/models/[id].tsx
export default function ModelPage({ model }: { model: ModelInfo }) { … }

export async function getServerSideProps({ params }) {
  const model = await db.getModel(params.id);
  if (!model) return { notFound: true };
  return { props: { model: JSON.parse(JSON.stringify(model)) } };   // 可序列化
}

// pages/api/models.ts
export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).setHeader('Allow', ['POST']).end();
  // 鉴权 + 校验 + 返回 JSON
}
```

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 整页 'use client' | 为 useState 把页面全 client | 交互下沉叶子 + children 组合 |
| useEffect 拉首屏 | `useEffect(() => fetch(...))` | server 组件直接 await |
| 缓存失控 | fetch 默认语义不审查 | 显式 cache/revalidate + 写后失效 |
| Action 不设防 | 无鉴权无校验的 'use server' | 鉴权 + zod + 可序列化返回 |
| 秘钥泄露 | `NEXT_PUBLIC_` 带密钥 | 服务端专用变量 + 启动校验 |
| params 未 await（15+） | `params.id` | `(await params).id` |
| 裸 img/link/script | `<img>` / `<link>` 字体 | `next/image` / `next/font` / `next/script` |
| 手写 head | `<head><title>` | Metadata API |
| error.tsx 缺 reset / 'use client' | 静态错误页 | client + `reset()` |
| app/pages 双路由混用 | 同路径两处定义 | 单一路由器，逐页迁移 |
