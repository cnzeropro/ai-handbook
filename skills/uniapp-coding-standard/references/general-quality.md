# uni-app 通用质量规范

> 本文件为 uniapp-coding-standard 的通用质量规范，基线 **Vue 3 语系**（多端：H5 / 微信及各家小程序 / App）。来源：uni-app 官方文档。完整正反例见 `style-examples.md`（同目录）。Vue 组件层规则遵循 `vue-coding-standard`。

## 目录

- 一、pages.json 与页面路由
- 二、条件编译规范（多端差异隔离）
- 三、生命周期（应用 / 页面 / 组件）
- 四、API 统一封装（uni.request 与平台差异）
- 五、样式与多端适配
- 六、组件与 easycom
- 七、状态管理
- 八、性能与包体
- 九、平台合规（登录 / 支付 / 分享 / 内容安全）

## 一、pages.json 与页面路由

- 路由零代码化：页面在 `pages.json` `pages` 注册，导航用 `uni.navigateTo` / `uni.switchTab` / `uni.redirectTo`；路径写绝对形式（`/pages/models/detail`）
- 首页为 `pages` 第一项；tabBar 页面只能用 `uni.switchTab` 切换
- 分包在 `subPackages` 声明（小程序端生效，H5/App 自动合并加载）

```json
{
  "pages": [
    { "path": "pages/index/index", "style": { "navigationBarTitleText": "首页" } },
    { "path": "pages/models/list", "style": { "navigationBarTitleText": "模型列表" } }
  ],
  "subPackages": [
    { "root": "pkg-models", "pages": [ { "path": "detail/detail" } ] }
  ]
}
```

- 页面参数经 `onLoad(options)` 解析；复杂参数编码传递（encodeURIComponent）

## 二、条件编译规范（多端差异隔离）

- **平台差异集中到适配层**：`services/platform.ts`（或 `utils/platform`）对外提供统一签名，内部 `#ifdef` 实现各端差异；业务组件少散落条件编译
- 文件级条件编译用于整平台差异文件（`xxx.h5.ts` / `xxx.mp.ts` 配合解析策略）

```ts
// services/location.ts —— 统一签名，内部分端
export function getLocation(): Promise<LocationResult> {
  // #ifdef H5
  return h5GetLocation();
  // #endif
  // #ifdef MP-WEIXIN || APP-PLUS
  return uniGetLocation();
  // #endif
}
```

- 禁大段业务代码整文件按平台复制两份维护（漂移源头）；`APP-PLUS` 内 plus.* 仅在 App 端条件块使用
- 条件编译是**编译期裁剪**：注释块语法必须准确（`// #ifdef MP-WEIXIN` … `// #endif`），CSS 内用 `/* #ifdef */`

## 三、生命周期（应用 / 页面 / 组件）

| 层级 | 钩子 | 用途 |
| --- | --- | --- |
| 应用 | `onLaunch` / `onShow` / `onHide` | 初始化、登录态恢复、埋点 |
| 页面 | `onLoad(options)` / `onShow` / `onReady` / `onHide` / `onUnload` | 参数解析、每次显示刷新、清理 |
| 组件 | Vue 3 标准生命周期 | `mounted` / `unmounted` |

- Vue 3 组合式：页面钩子从 `@dcloudio/uni-app` 引入（`onLoad` / `onShow` 等）

```vue
<script setup lang="ts">
import { onLoad, onUnload } from '@dcloudio/uni-app';

const id = ref('');
onLoad((options) => { id.value = options.id ?? ''; });
onUnload(() => clearInterval(timer));
</script>
```

- 定时器 / 事件监听（`uni.$on`）在 `onUnload` / `onUnmounted` 配对 `uni.$off` 清理

## 四、API 统一封装（uni.request 与平台差异）

- `uni.request` 统一封装：baseUrl、token 拦截、统一错误码（401 跳登录）、超时与重试；业务禁散调

```ts
// utils/request.ts
const BASE_URL = 'https://api.example.com';

export function request<T>(options: UniApp.RequestOptions): Promise<T> {
  return new Promise((resolve, reject) => {
    uni.request({
      ...options,
      url: BASE_URL + options.url,
      header: { Authorization: `Bearer ${getToken()}`, ...options.header },
      success: (res) => (res.statusCode === 200 ? resolve(res.data as T) : handleBizError(res, reject)),
      fail: (e) => { toastNetworkError(); reject(e); },
    });
  });
}
```

- 平台差异 API（定位 / 支付 / 分享 / 文件）在适配层按端实现，签名统一
- 上传下载用 `uni.uploadFile` / `uni.downloadFile` 封装，进度回调统一

## 五、样式与多端适配

- 尺寸 **rpx**（750 基准；`upx` 已废弃改 rpx）
- 安全区：底部操作条 `padding-bottom: env(safe-area-inset-bottom)`；刘海屏顶部预留
- 条件编译样式仅限差异行；通用样式进公共文件
- 字体图标 / SVG 替代多套切图；主题色令牌化（css 变量）

## 六、组件与 easycom

- 组件放 `components/组件名/组件名.vue` 即自动注册（easycom），免 import；不符合命名规范的组件显式引入
- 三方库（uni-ui）按需引入；自定义组件跨端兼容优先用官方组件与内置组件
- 组件通信与 Vue 3 一致：props 下行、emits 上行（`uni.$emit` 事件总线仅用于跨页面松耦合通知，配对 `$off`）

## 七、状态管理

- Vue 3 项目用 Pinia（持久化插件做登录态恢复）；登录态 token 统一存储（storage + 内存双份）
- 登录态失效（401）统一在 request 拦截器处理：清理 token → 跳登录
- 禁模块顶层可变单例（H5 SSR 场景同 nuxt 坑；小程序/App 单实例风险低但保持一致写法）

## 八、性能与包体

- 小程序端分包（`subPackages`）+ 主包瘦身（图片/大资源上 CDN）
- 图片压缩 + 按端选格式；长列表用 `recycle-list` 思想或分页
- `v-if` 惰性渲染低频块；频繁切换用 `hidden`（H5）与对应端等价方案
- 启动速度：减少 `onLaunch` 同步任务；分包预下载按平台配置

## 九、平台合规（登录 / 支付 / 分享 / 内容安全）

- 登录：微信端 `uni.login` → 服务端 code2Session；App 端按厂商（一键登录/三方）适配层实现
- 支付：`uni.requestPayment` 参数由服务端下发；各端支付渠道差异封在适配层
- 分享：小程序 onShareAppMessage / H5 meta / App 分享 SDK，统一分享内容构造函数
- 内容安全：用户生成内容过各端内容安全接口（微信 `msgSecCheck` 等，App/H5 用服务端方案）
