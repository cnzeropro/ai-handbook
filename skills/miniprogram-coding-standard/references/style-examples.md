# 微信小程序编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：微信官方文档与小程序审查常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 页面骨架（四件套）
- 2. Component 构造器复用组件
- 3. setData 路径更新
- 4. 列表与条件渲染（wx:for / wx:if / hidden）
- 5. 网络请求封装
- 6. 分包与预下载
- 7. 授权与隐私合规
- 8. 常见正反例速查

## 1. 页面骨架（四件套）

```js
// pages/models/index.js
Page({
  data: {
    models: [],
    loading: true,
  },
  onLoad(options) {
    this.category = options.category;        // 非渲染数据挂 this，不进 data
    this.fetchModels();
  },
  onUnload() {
    clearTimeout(this._retryTimer);          // 清理定时器/监听
  },
  async fetchModels() {
    this.setData({ loading: true });
    try {
      const models = await modelApi.list(this.category);
      this.setData({ models, loading: false });
    } catch (e) {
      this.setData({ loading: false });
      wx.showToast({ title: '加载失败', icon: 'none' });
    }
  },
});
```

```html
<!-- pages/models/index.wxml -->
<view class="models">
  <model-card
    wx:for="{{models}}"
    wx:key="id"
    model="{{item}}"
    bind:open="onOpenModel"
  />
  <view wx:if="{{loading}}" class="skeleton">加载中…</view>
</view>
```

```json
// pages/models/index.json
{ "usingComponents": { "model-card": "/components/model-card/index" } }
```

❌ json 缺失、页面未注册、onLoad 里写长逻辑不拆方法。

## 2. Component 构造器复用组件

```js
// components/model-card/index.js
Component({
  options: { pureDataPattern: /^_/ },
  properties: {
    model: { type: Object, value: null },
    compact: { type: Boolean, value: false },
  },
  data: { expanded: false },
  lifetimes: {
    attached() { /* 初始化 */ },
    detached() { this.clearTimer?.(); },
  },
  methods: {
    onToggle() {
      this.setData({ expanded: !this.data.expanded });
    },
    onOpen() {
      this.triggerEvent('open', { id: this.data.model.id });
    },
  },
});
```

```json
// components/model-card/index.json
{ "component": true }
```

```html
<!-- components/model-card/index.wxml -->
<view class="model-card {{compact ? 'model-card--compact' : ''}}">
  <view class="model-card__title" bindtap="onToggle">{{ model.name }}</view>
  <view wx:if="{{expanded}}" class="model-card__body">
    <button type="button" bindtap="onOpen" size="mini">查看</button>
  </view>
</view>
```

## 3. setData 路径更新

```js
// ❌ 全量回写：传输量大、渲染开销高
this.setData({ list: this.data.list, loading: false });

// ✅ 路径更新单项
this.setData({
  [`list[${index}].status`]: 'deployed',
  loading: false,
});

// ❌ 高频 setData（滚动/输入未节流）
onPageScroll(e) {
  this.setData({ scrollTop: e.scrollTop });
}

// ✅ 节流 + 纯数据字段
onPageScroll: throttle(function (e) {
  this._scrollTop = e.scrollTop;      // 纯数据字段（pureDataPattern）
  this.updateIndicator();
}, 100),
```

## 4. 列表与条件渲染

```html
<!-- ❌ 无 key、index 作 key（重排错乱） -->
<view wx:for="{{models}}">
<!-- ❌ 频繁切换用 wx:if（反复创建销毁） -->
<view wx:if="{{expanded}}">…</view>

<!-- ✅ 稳定 id 作 key -->
<view wx:for="{{models}}" wx:key="id">…</view>

<!-- ✅ 频繁切换用 hidden（保留节点） -->
<view hidden="{{!expanded}}">…</view>

<!-- ✅ 低频条件用 wx:if -->
<view wx:if="{{error}}">{{error}}</view>
```

## 5. 网络请求封装

```js
// services/request.js
const BASE_URL = 'https://api.example.com';

export function request(options) {
  const token = wx.getStorageSync('token');
  return new Promise((resolve, reject) => {
    wx.request({
      url: `${BASE_URL}${options.url}`,
      method: options.method || 'GET',
      data: options.data,
      header: { Authorization: token ? `Bearer ${token}` : '', ...options.header },
      success: (res) => {
        if (res.statusCode === 401) {
          wx.reLaunch({ url: '/pages/login/index' });   // 登录态失效统一跳转
          return reject(res);
        }
        res.statusCode === 200 ? resolve(res.data) : reject(res);
      },
      fail: reject,
    });
  });
}
```

❌ 业务页面散调 `wx.request`（无 token、无统一错误处理）；❌ 域名未配置合法域名上线即失效。

## 6. 分包与预下载

```json
// app.json
{
  "pages": ["pages/index/index", "pages/login/index"],
  "subPackages": [
    { "root": "pkg-models", "pages": ["list/list", "detail/detail"] },
    { "root": "pkg-tools", "pages": ["export/export"], "independent": true }
  ],
  "preloadRule": {
    "pages/index/index": { "network": "all", "packages": ["pkg-models"] }
  },
  "lazyCodeLoading": "requiredComponents"
}
```

- 主包只保留 tab 页与登录页；特性页入分包
- 独立分包不依赖主包（工具页、落地页）；主包资源（图片）迁 CDN

## 7. 授权与隐私合规

```js
// ❌ AppSecret 落前端、wx.getUserInfo（已废弃）
const secret = 'a1b2c3…';                      // 严禁
wx.getUserInfo({ … });                          // 已废弃接口

// ✅ 登录：code2Session 在服务端完成
const { code } = await wx.login();
const session = await request({ url: '/login', data: { code } });

// ✅ 用户信息：getUserProfile 授权 + 隐私协议
const { userInfo } = await wx.getUserProfile({ desc: '用于完善会员资料' });
```

- 隐私接口（如获取位置、相册）按平台要求完成隐私协议声明与弹窗
- 用户生成内容过内容安全接口（`msgSecCheck` / `imgSecCheck`）
- web-view 仅加载已配置业务域名

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| setData 全量回写 | `setData({ list })` | 路径更新 `[`list[${i}].x`]` |
| 高频 setData | 滚动/输入直更 | 节流 + 纯数据字段 |
| wx:for 无 key | `wx:for` 裸用 | `wx:key="id"` |
| 频繁 wx:if | 切换重建节点 | `hidden` |
| 定时器泄漏 | onUnload 不清理 | 配对清理 |
| 散调 wx.request | 页面内直调 | 统一封装拦截 token/错误 |
| 域名未配置 | http / 未备案域名 | 合法域名 HTTPS |
| 主包超限 | 全塞主包 | subPackages + preloadRule |
| AppSecret 落前端 | 常量硬编码 | 服务端 code2Session |
| 缓存超限 | 大列表落 Storage | 网络为主，Storage 存小型字典 |
| 同步存储大对象 | `setStorageSync` 大 JSON | 异步接口 / 拆分 |
| 废弃接口 | `wx.getUserInfo` | `getUserProfile` + 隐私协议 |
