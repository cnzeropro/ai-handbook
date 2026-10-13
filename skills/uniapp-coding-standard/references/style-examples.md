# uni-app 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：uni-app 官方文档与多端项目审查常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 页面骨架（Vue 3 setup + 页面生命周期）
- 2. 条件编译集中隔离
- 3. uni.request 封装
- 4. pages.json 与分包
- 5. 样式与安全区
- 6. easycom 组件
- 7. 常见正反例速查

## 1. 页面骨架（Vue 3 setup + 页面生命周期）

```vue
<!-- pages/models/list.vue -->
<script setup lang="ts">
import { onLoad, onShow, onUnload } from '@dcloudio/uni-app';
import { ref } from 'vue';

const models = ref<ModelInfo[]>([]);
const keyword = ref('');

// ✅ 参数解析在 onLoad
onLoad((options) => {
  keyword.value = decodeURIComponent(options?.q ?? '');
});

// ✅ 每次进入页面刷新（onShow）
onShow(() => {
  fetchModels();
});

let timer: ReturnType<typeof setInterval> | undefined;
onLoad(() => {
  timer = setInterval(refreshBadge, 30000);
});
onUnload(() => {
  if (timer) clearInterval(timer);       // ✅ 配对清理
});

async function fetchModels() {
  models.value = await request<ModelInfo[]>({ url: '/models', data: { q: keyword.value } });
}
</script>

<template>
  <view class="models">
    <input v-model="keyword" class="models__search" placeholder="搜索模型" />
    <model-card
      v-for="model in models"
      :key="model.id"
      :model="model"
      @click="openDetail(model.id)"
    />
  </view>
</template>

<style>
.models { padding: 24rpx; padding-bottom: calc(24rpx + env(safe-area-inset-bottom)); }
</style>
```

## 2. 条件编译集中隔离

```ts
// services/location.ts —— 统一签名，平台差异收口在适配层
export interface LocationResult { latitude: number; longitude: number; }

export function getLocation(): Promise<LocationResult> {
  // #ifdef H5
  return h5Locate();                       // H5：浏览器 geolocation
  // #endif

  // #ifdef MP-WEIXIN
  return new Promise((resolve, reject) => {
    uni.getLocation({
      type: 'gcj02',
      isHighAccuracy: true,
      success: resolve,
      fail: reject,
    });
  });
  // #endif

  // #ifdef APP-PLUS
  return appLocateByVendor();              // App：厂商定位 SDK
  // #endif
}
```

```ts
// ❌ 业务组件内散落条件编译（漂移源头）
function openDetail(id: number) {
  // #ifdef MP-WEIXIN
  wx.navigateTo({ url: `/pkg-models/detail/detail?id=${id}` });
  // #endif
  // #ifdef H5
  router.push(`/models/detail/${id}`);
  // #endif
  // #ifdef APP-PLUS
  plus.webview.create(…);
  // #endif
}

// ✅ 路由跳转统一封装，平台差异收口
router.push(`/models/detail/${id}`);
```

文件级条件编译用于整平台差异实现（构建期按平台选择文件）。

## 3. uni.request 封装

```ts
// utils/request.ts
const BASE_URL = 'https://api.example.com';

export function request<T>(options: UniApp.RequestOptions): Promise<T> {
  return new Promise((resolve, reject) => {
    uni.request({
      ...options,
      url: BASE_URL + options.url,
      header: { Authorization: `Bearer ${getToken()}`, ...options.header },
      success: (res) => {
        if (res.statusCode === 401) {
          clearToken();
          uni.reLaunch({ url: '/pages/login/index' });
          return reject(res);
        }
        res.statusCode === 200 ? resolve(res.data as T) : handleBizError(res, reject);
      },
      fail: (e) => {
        uni.showToast({ title: '网络异常', icon: 'none' });
        reject(e);
      },
    });
  });
}
```

❌ 业务页面散调 `uni.request`（无 token、无统一错误处理）；❌ 各端域名白名单未配置。

## 4. pages.json 与分包

```json
{
  "pages": [
    { "path": "pages/index/index", "style": { "navigationBarTitleText": "首页" } },
    { "path": "pages/login/index", "style": { "navigationBarTitleText": "登录" } }
  ],
  "tabBar": {
    "list": [
      { "pagePath": "pages/index/index", "text": "首页" },
      { "pagePath": "pages/models/list", "text": "模型" }
    ]
  },
  "subPackages": [
    { "root": "pkg-models", "pages": [ { "path": "detail/detail" } ] }
  ],
  "preloadRule": {
    "pages/models/list": { "network": "all", "packages": ["pkg-models"] }
  },
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarTitleText": "示例系统"
  }
}
```

- tabBar 页面切换用 `uni.switchTab`；普通页面 `uni.navigateTo`；关闭当前页 `uni.redirectTo`
- 小程序端分包生效；H5/App 自动合并

## 5. 样式与安全区

```vue
<style>
/* ❌ 裸底部：全面屏被手势条遮挡 */
.submit-bar { position: fixed; bottom: 0; }

/* ✅ 安全区 + 令牌 */
.submit-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding-bottom: calc(env(safe-area-inset-bottom) + 16rpx);
  /* #ifdef MP-WEIXIN */
  padding-bottom: calc(constant(safe-area-inset-bottom) + 16rpx);
  /* #endif */
}
</style>
```

rpx 尺寸（750 基准）、主题色 css 变量令牌、差异行才用条件编译样式。

## 6. easycom 组件

```
components/
└── model-card/
    └── model-card.vue        # 自动注册：<model-card />
```

```vue
<template>
  <model-card :model="model" @open="openDetail" />
  <!-- ❌ 无需 import + components 注册（easycom 规则内） -->
</template>
```

- easycom 不满足命名时显式 import；uni-ui 组件（`uni-icons` 等）同样 easycom 引入
- 组件通信 Vue 3 标准：props 下行、emits 上行；`uni.$emit` / `uni.$on` 仅跨页面松耦合通知并配对 `$off`

## 7. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 平台差异散落 | 组件内到处 `#ifdef` | 适配层统一签名 |
| 整文件按平台复制 | `list.wx.vue` + `list.h5.vue` 双维护 | 条件编译差异行/差异文件 |
| 散调 uni.request | 页面内直调 | 统一封装拦截 token/错误 |
| 忘记清理 | `uni.$on` 不 `$off` | `onUnload` 配对 `$off` |
| 页面参数未解码 | `options.id` 含中文 | `decodeURIComponent` |
| 裸底部 | 固定 bottom: 0 | `env(safe-area-inset-bottom)` |
| tab 页用 navigateTo | `uni.navigateTo` 到 tab 页 | `uni.switchTab` |
| upx 单位 | `width: 100upx` | `width: 100rpx`（upx 已废弃） |
| 主包超限 | 图片/大资源进主包 | 分包 + CDN |
| 跨端登录散写 | 各端 login 逻辑复制 | 适配层统一 login() |
