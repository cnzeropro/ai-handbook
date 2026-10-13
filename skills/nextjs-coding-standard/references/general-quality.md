# Next.js 通用质量规范

> 本文件为 nextjs-coding-standard 的**版本通用**质量规范（App Router 与 Pages Router 通用部分）。版本特有约定见分册：App Router 见 `app-router.md`，Pages Router 见 `pages-router.md`。来源：nextjs.org 官方文档与 react.dev Rules of React。完整正反例见 `style-examples.md`（同目录）。React 组件层规则遵循 `react-coding-standard`。

## 目录

- 一、项目结构与文件约定
- 二、Server 与 Client 组件边界
- 三、数据获取与变更
- 四、环境变量
- 五、内置组件与性能
- 六、安全

## 一、项目结构与文件约定

- **路由器二选一不混用**：`app/`（App Router）或 `pages/`（Pages Router）只保留一种（`app` 可渐进共存迁移，目标态唯一）
- 目录与文件命名 kebab-case；组件与路由就近 colocate（非路由文件放路由目录不会被当成路由）
- 私有与专用文件（App Router）：`page` / `layout` / `loading` / `error` / `not-found` / `route` / `template`；下划线前缀 `_folder` 私有目录、`(group)` 路由组
- 一个路由一段职责：页面组装放 `page`，可复用 UI 拆组件，数据访问函数集中在 `lib`/`data` 层而非散落组件

## 二、Server 与 Client 组件边界

- **默认 Server**：无交互、无浏览器 API、无状态的组件都是 Server Component——代码不下发、可直接访问数据库
- 仅以下需求下沉 `'use client'`：state / 事件处理器 / lifecycle / 浏览器 API / 自定义 Hook
- `'use client'` 标在**边界文件顶部**，其子导入全部成为 client——边界越靠叶子越好
- Server → Client 传 props 必须可序列化（禁函数/类实例/复杂对象）；需要交互包装时用 **children 组合模式**（server 端渲染 children 传入 client 布局组件）
- Client 组件可嵌套 Server 子组件（作为 children/props 传入）；Server 组件内禁 hooks / 事件 / 浏览器 API

```tsx
// ❌ server 组件中使用交互
export default function Page() {
  const [open, setOpen] = useState(false);   // hooks 只能在 client
  return <button onClick={() => setOpen(true)}>…</button>;
}

// ✅ server 页面组合 client 交互件 + server children
export default function Page() {
  return (
    <DialogProvider>
      <ModelTable />                          {/* server：直查数据库 */}
      <OpenDialogButton />                    {/* client：交互下沉叶子 */}
    </DialogProvider>
  );
}
```

## 三、数据获取与变更

- **初始数据在服务端获取**（server 组件直接 await），不把"首屏数据获取"塞进 `useEffect`（与 react skill 同源原则）
- **变更用 Server Actions**（`'use server'`）：表单 `action`、按钮过渡（`useTransition` / `useActionState`）；返回可序列化数据
- 写操作后必须失效缓存：`revalidatePath('/models')` / `revalidateTag('models')`
- client 端仅做交互触发的获取（路由处理程序 `/api` 或 Server Action），不直连数据库
- 错误处理：服务端报错进 `error.tsx` 展示并上报；Action 返回可序列化错误态而非抛到客户端崩溃

## 四、环境变量

- 服务端专用秘密（`DATABASE_URL` / API Key）：**不加** `NEXT_PUBLIC_` 前缀，只在 server 上下文可读
- `NEXT_PUBLIC_*` 会在构建期**内联进客户端产物**——任何秘密禁用该前缀
- 校验环境变量：启动/构建期用 schema（如 zod）解析 `process.env`，缺失即失败（fail fast）

## 五、内置组件与性能

- 图片：`next/image`（自动优化/懒加载/防 CLS）；禁裸 `<img>`（特殊场景注明）
- 字体：`next/font`（自托管 + 零布局偏移）；禁渲染阻塞的 `<link>` 字体
- 脚本：`next/script` 并按需选 `strategy`（`afterInteractive` 默认 / `lazyOnload` / `beforeInteractive`）
- metadata：App Router 用 Metadata API（`export const metadata` / `generateMetadata`），不手写 `<head>`
- 流式渲染：慢段用 `<Suspense>` + `loading.tsx`；包体用 `next/dynamic` 按需加载
- 长列表虚拟滚动、重组件按需加载（同 `react-coding-standard` 性能章）

## 六、安全

- 服务端 fetch 校验目标 URL（防 SSRF：禁拼接用户输入直取）；出站请求白名单域名
- 响应头安全基线（CSP / X-Frame-Options / HSTS）在 `next.config` 或 middleware 统一配置
- Server Action / Route Handler 视为公开端点：鉴权、输入校验、速率控制一样不能少（不是"内部函数"）
- 用户输入渲染消毒（同 `react-coding-standard` 六）
