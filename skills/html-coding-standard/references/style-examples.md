# HTML 编码风格完整示例

> 收录各主题完整示例与市面高频正反例（来源：HTML Living Standard、MDN、WAI-ARIA APG、web.dev 常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 页面骨架完整示例
- 2. 语义化标签（div 汤 vs 语义化）
- 3. 图片可访问性与性能
- 4. 表单可访问性
- 5. 标题层级
- 6. 交互元素（button vs div）
- 7. head 模板（SEO 完整示例）
- 8. 常见正反例速查

## 1. 页面骨架完整示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>用户中心 - 示例系统</title>
  <meta name="description" content="管理您的订单、账户与偏好设置。">
  <link rel="icon" href="/favicon.ico">
  <link rel="stylesheet" href="/main.css">
  <script src="/app.js" defer></script>
</head>
<body>
  <header>
    <nav aria-label="主导航">…</nav>
  </header>
  <main>
    <h1>用户中心</h1>
    …
  </main>
  <footer>…</footer>
</body>
</html>
```

常见骨架缺失（❌ 逐一对照）：

```html
<!-- ❌ 缺 doctype（触发怪异模式）、缺 lang、缺 charset/viewport、title 无意义 -->
<html>
<head>
  <title>Untitled</title>
</head>
…
```

## 2. 语义化标签（div 汤 vs 语义化）

```html
<!-- ❌ div 汤：读屏无地标、SEO 无权重、后人难读 -->
<div class="top">
  <div class="menu">
    <div class="item" onclick="go('/a')">A</div>
  </div>
</div>
<div class="content">
  <div class="title">文章标题</div>
  <div class="text">…</div>
</div>

<!-- ✅ -->
<header>
  <nav aria-label="主导航">
    <a href="/a">A</a>
  </nav>
</header>
<main>
  <article>
    <h1>文章标题</h1>
    <p>…</p>
  </article>
</main>
```

article vs section 的取舍：

```html
<!-- ✅ 可独立成篇的内容单元（评论、卡片）用 article -->
<section>
  <h2>最新评论</h2>
  <article>
    <h3>张三</h3>
    <p>…</p>
  </article>
</section>

<!-- ❌ 无标题的 section（应配标题或改用 div） -->
<section>
  <p>一些内容</p>
</section>
```

## 3. 图片可访问性与性能

```html
<!-- ❌ 无 alt、无尺寸（加载时布局跳动） -->
<img src="/charts/q3.png">

<!-- ❌ 装饰图 alt 复读文件名 -->
<img src="/divider.png" alt="divider.png">

<!-- ✅ 信息图：描述内容 + 防布局偏移 + 懒加载（首屏外） -->
<img src="/charts/q3.png" alt="第三季度销售额同比增长 12%" width="640" height="360" loading="lazy">

<!-- ✅ 装饰图：空 alt（读屏跳过） -->
<img src="/divider.png" alt="" width="100%" height="8">

<!-- ✅ 现代格式回退 -->
<picture>
  <source srcset="/hero.avif" type="image/avif">
  <source srcset="/hero.webp" type="image/webp">
  <img src="/hero.jpg" alt="产品全景图" width="1200" height="600">
</picture>
```

功能图标 alt 描述动作而非外观：

```html
<!-- ❌ -->
<button><img src="/search-icon.png" alt="放大镜图标"></button>

<!-- ✅ -->
<button><img src="/search-icon.png" alt="搜索"></button>
```

## 4. 表单可访问性

```html
<!-- ❌ placeholder 充当 label；无关联；无类型 -->
<form>
  <input placeholder="请输入邮箱">
  <input placeholder="密码">
  <button>提交</button>
</form>

<!-- ✅ label 关联 + 精确类型 + autocomplete + 显式 type -->
<form action="/login" method="post">
  <label for="email">邮箱</label>
  <input type="email" id="email" name="email" autocomplete="email" required>

  <label for="password">密码</label>
  <input type="password" id="password" name="password" autocomplete="current-password" required>

  <button type="submit">登录</button>
</form>
```

成组选项用 fieldset + legend：

```html
<fieldset>
  <legend>通知方式</legend>
  <label><input type="checkbox" name="notify" value="email"> 邮件</label>
  <label><input type="checkbox" name="notify" value="sms"> 短信</label>
</fieldset>
```

## 5. 标题层级

```html
<!-- ❌ 跳级 + 多个 h1 + 按字号选标签 -->
<h1 style="font-size:12px">小字用了 h1</h1>
<h3>直接跳到 h3</h3>
<h1>第二个 h1</h1>

<!-- ✅ 层级递进；字号交给 CSS -->
<h1>页面主标题</h1>
<h2>章节</h2>
<h3>小节</h3>
```

## 6. 交互元素（button vs div）

```html
<!-- ❌ div 假按钮：无键盘支持、无角色、读屏识别为普通文本 -->
<div class="btn" onclick="submit()">提交</div>
<div onclick="location.href='/a'">前往 A</div>

<!-- ✅ 原生交互元素：自带键盘与语义 -->
<button type="submit" class="btn">提交</button>
<a href="/a">前往 A</a>

<!-- ✅ 原生语义不足时才补 ARIA，且必须配行为 -->
<div role="tab" tabindex="0" aria-selected="true" @keydown.enter="select">Tab 1</div>
```

链接文本有意义：

```html
<!-- ❌ 读屏用户按链接列表导航时无法区分 -->
<a href="/help">点击这里</a>
<a href="/help">更多</a>

<!-- ✅ -->
<a href="/help">查看帮助文档</a>
```

## 7. head 模板（SEO 完整示例）

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>用户中心 - 示例系统</title>
  <meta name="description" content="管理您的订单、账户与偏好设置。">
  <link rel="canonical" href="https://example.com/account">
  <link rel="icon" href="/favicon.ico">
  <meta property="og:title" content="用户中心 - 示例系统">
  <meta property="og:description" content="管理您的订单、账户与偏好设置。">
  <meta property="og:image" content="https://example.com/og.png">
</head>
```

## 8. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 布局偏移 | `img` 无宽高 | 声明 `width`/`height` |
| 首屏变慢 | 所有图片加 `loading="lazy"` | 仅首屏外图片懒加载 |
| 阻塞渲染 | `<script src>` 无 defer | `defer`（或置于 body 末尾） |
| 误提交 | 表单内 `<button>` 漏 type | 显式 `type="button"/"submit"` |
| 假按钮 | `div` + onclick | 原生 `button` / `a` |
| 装饰图噪音 | `alt="image.png"` | 装饰图 `alt=""` |
| 自造属性 | `<input myattr="1">` | `data-*` 前缀 |
| 实体遗漏 | 文本裸写 `&` / `<` | `&amp;` / `&lt;` |
| 单引号/无引号 | `name=value` | `name="value"` |
| tabindex 乱序 | `tabindex="3"` | 仅 `0` 与 `-1` |
