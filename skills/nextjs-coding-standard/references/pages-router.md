# Next.js Pages Router 规范（仍在用）

> 本文件为 **Pages Router** 版本特有约定，须与版本通用的 `general-quality.md` 配合使用。Pages Router 仍在大量项目中使用，但新项目主推 App Router（见 `app-router.md`）；迁移按官方增量指南逐页进行。

## 一、文件约定

- `pages/` 目录即路由：`pages/models/index.tsx` → `/models`；动态路由 `pages/models/[id].tsx`；`pages/_app.tsx` 全局壳、`pages/_document.tsx` 文档结构
- API 路由：`pages/api/*.ts`（默认导出 handler，`req`/`res`）
- 专用组件与页面 colocate，非路由组件放组件目录

## 二、数据获取

| 函数 | 场景 | 说明 |
| --- | --- | --- |
| `getServerSideProps` | 每次请求都需要最新数据/鉴权 | 服务端执行，`props` 进页面；禁内部副作用（除数据获取） |
| `getStaticProps` + `getStaticPaths` | 构建期可定的静态页 | ISR 用 `revalidate` |
| 组件内 `useEffect`/SWR | 客户端交互触发的增量数据 | 首屏数据不走这条 |

```tsx
export default function ModelPage({ model }: { model: ModelInfo }) {
  return <ModelView model={model} />;
}

export async function getServerSideProps({ params }) {
  const model = await db.getModel(params.id);      // 服务端直查
  if (!model) return { notFound: true };
  return { props: { model: JSON.parse(JSON.stringify(model)) } };
}
```

- `props` 必须可序列化（Date/Map 等先转换）；错误返回 `{ notFound: true }` 或重定向

## 三、API Routes

```ts
export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    res.setHeader('Allow', ['POST']);
    return res.status(405).end();
  }
  const parsed = CreateModelSchema.safeParse(req.body);
  if (!parsed.success) return res.status(400).json({ error: 'invalid input' });
  const model = await db.insertModel(parsed.data);
  res.status(201).json(model);
}
```

- 视为公开端点：鉴权 + 校验 + 方法白名单；错误统一 JSON 结构

## 四、样式与 head

- 全局样式在 `_app.tsx` 引入；页面级 CSS Modules（`*.module.css`）
- head 用 `next/head`（title/description/og），等价于 App Router 的 metadata

## 五、与 App Router 并存与迁移

- 过渡期 `app/` 与 `pages/` 可共存（同一路径只能归其一）；逐页迁移
- 迁移对照：`getServerSideProps` → server 组件 await；`pages/api` → `route.ts`；`next/head` → Metadata API
- 每迁完一页删除对应 pages 文件，避免双路由冲突
