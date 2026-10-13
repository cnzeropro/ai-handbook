# 微信小程序通用质量规范

> 本文件为 miniprogram-coding-standard 的通用质量规范，基线为基础库 2.x（新 API 按官方文档标注最低版本使用）。来源：微信官方文档（developers.weixin.qq.com/miniprogram）。完整正反例见 `style-examples.md`（同目录）。JS 语法规则遵循 `js-coding-standard`。

## 目录

- 一、项目结构与文件约定
- 二、Page 与 Component
- 三、setData 与性能
- 四、WXML 指令与模板
- 五、WXSS 样式
- 六、生命周期与资源清理
- 七、网络请求封装
- 八、分包与启动性能
- 九、缓存与存储
- 十、安全与合规

## 一、项目结构与文件约定

- 页面四件套同名列同目录：`index.wxml` / `index.wxss` / `index.js` / `index.json`（json 必须存在）；页面在 `app.json` `pages` 中注册且**首页排第一**
- 全局配置 `app.json`（window / tabBar / 分包 / 最低基础库 `setting.minifyWXML` 等）；全局样式 `app.wxss`
- 目录按特性域组织：`pages/models/`、`components/`、`services/`（网络与业务封装）、`utils/`
- 组件在页面 json 的 `usingComponents` 声明（路径相对）；公共组件入 `components/`

## 二、Page 与 Component

- **可复用 UI 与复杂页面用 `Component` 构造器**：behaviors 复用、定义段清晰（`properties` / `data` / `methods` / `lifetimes` / `pageLifetimes`）
- 纯展示页可用 `Page` 构造器，保持轻量

```js
// ✅ Component 构造器
Component({
  options: { pureDataPattern: /^_/ },        // _ 开头为纯数据字段
  properties: {
    model: { type: Object, value: null },
  },
  data: { expanded: false },
  lifetimes: {
    detached() { this.clearTimer(); },       // 清理定时器/监听
  },
  methods: {
    toggle() { this.setData({ expanded: !this.data.expanded }); },
    _refreshInternal() { … },                // 纯数据字段不触发渲染
  },
});
```

- properties 带完整类型定义（`type` + 可选 `observer` 慎用——优先 observers 定义段）

## 三、setData 与性能

- **只传变化的数据**：禁一次性 setData 整个 data 对象
- **路径更新**数组元素与深层字段：

```js
// ❌ 全量回写
this.setData({ list: this.data.list });

// ✅ 路径更新
this.setData({ [`list[${index}].status`]: 'deployed' });
```

- 高频场景（滚动联动、输入联想）：节流（`throttle`）后再 setData
- 纯数据字段（`pureDataPattern`）：不影响模板的数据走 this.data 直接读，不经渲染层
- 后台页面禁 setData（不可见时无意义且耗性能）
- 单次 setData 数据量控制（过长字符串/大列表考虑分页与局部更新）

## 四、WXML 指令与模板

- `wx:for` 必带 `wx:key`（稳定 id，禁 index 用于会重排的列表）
- 频繁切换显示用 `hidden`（保留节点）；条件渲染用 `wx:if`（惰性创建）
- 复杂计算放 `wxs` 模块（视图层执行，避免逻辑层通信开销）或直接预处理进 data
- 模板内禁长表达式与函数调用链——预处理进 data / wxs
- 组件占位与插槽：`slot` 声明具名插槽，`multipleSlots` 按需开启

## 五、WXSS 样式

- 尺寸单位 **rpx**（750 设计稿基准）；字号可用 rpx 或按可读性固定 px
- 类名 BEM（`.model-card__title--warning`），与组件封装配合
- 组件样式默认隔离：`styleIsolation: 'isolated'`；需要页面级覆盖用 `apply-shared` / `shared` 按需选择
- 全局样式 `app.wxss` 只放 reset、令牌变量、通用工具类
- 禁标签选择器滥用（组件隔离下不可预期）

## 六、生命周期与资源清理

| 场景 | 钩子 | 清理 |
| --- | --- | --- |
| 页面 | `onLoad`（参数解析）/ `onShow` / `onReady` | `onUnload` 清定时器/监听 |
| 组件 | `attached` / `ready` | `detached` 清理 |
| 页面级组件 | `pageLifetimes.show` / `hide` | hide 时暂停轮询等 |

- 定时器、`wx.onXxx` 监听、事件总线回调必须配对清理

## 七、网络请求封装

- 统一封装 `wx.request`：拦截 token、统一错误码处理、超时与重试策略、登录态失效统一跳转
- 域名必须在小程序后台配置**合法域名**（HTTPS）；禁上线后 `urlCheck: false` 蒙混
- 分页、防重复提交（loading 锁）、错误 toast 统一在封装层

```js
// services/request.js（示意）
export function request(options) {
  const token = getToken();
  return new Promise((resolve, reject) => {
    wx.request({
      …options,
      url: BASE_URL + options.url,
      header: { Authorization: token ? `Bearer ${token}` : '', …options.header },
      success: (res) => (res.statusCode === 200 ? resolve(res.data) : reject(res)),
      fail: reject,
    });
  });
}
```

## 八、分包与启动性能

- 主包 ≤ 2M（整包 ≤ 30M）：按 tab/特性拆 `subPackages`；独立分包仅放不依赖主包的工具页
- `preloadRule` 声明分包预下载（进入页面预下载下一跳）
- 启动优化：精简 `app.json` 首屏逻辑、`lazyCodeLoading: 'requiredComponents'`、初始渲染缓存、图片压缩与 CDN
- `wx:if` 惰性渲染首屏非关键块；骨架屏提升感知

## 九、缓存与存储

- `wx.setStorageSync` / `getStorageSync`：**同步接口阻塞**，大数据用异步版
- 本地缓存上限 10MB：只存令牌、配置、小型字典；列表数据走网络
- 敏感信息不落明文本地（如用户手机号）；令牌按安全策略存储与刷新

## 十、安全与合规

- `AppSecret` 严禁出现在小程序代码与本地存储（code2Session 在服务端完成）
- 用户信息：`getUserProfile` 授权弹窗 + 隐私协议（`__usePrivacyCheck__` / 隐私接口声明），不缓存超范围数据
- 用户生成内容过内容安全接口（`msgSecCheck` / `imgSecCheck`）
- web-view 仅加载已配置的业务域名；开放数据用开放数据组件而非爬取
