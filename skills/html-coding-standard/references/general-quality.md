# HTML 通用质量规范

> 本文件为 html-coding-standard 的通用质量规范，适用于任意 HTML（静态页面、邮件模板、框架模板中的标记结构）。规则来源：HTML Living Standard、MDN、WAI-ARIA APG、Google HTML/CSS Style Guide。完整正反例见 `style-examples.md`（同目录）。

## 一、文档结构

- `<!DOCTYPE html>` 必须位于首行，触发标准模式渲染
- `<html>` 必须带 `lang` 属性（如 `lang="zh-CN"`），影响屏幕阅读器发音与翻译工具
- `<meta charset="utf-8">` 与 viewport meta 必备，且应位于 `<head>` 前部
- `<title>` 唯一、有意义（页面用途 + 站点名），禁用 `Untitled` / `New Page 1`
- `<head>` 内顺序建议：charset → viewport → title → 样式表 → 脚本

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>用户中心 - 示例系统</title>
  <link rel="stylesheet" href="/main.css">
  <script src="/app.js" defer></script>
</head>
```

## 二、语义化标签

- 按含义选标签，禁用无语义 `div` / `span` 堆砌（"div 汤"）：`header`（页头/区块头）、`nav`（主导航块）、`main`（主体内容，**每页唯一**）、`article`（可独立成篇的内容）、`section`（按主题分组的区块，**应配标题**）、`aside`（侧边/附属内容）、`footer`（页脚/区块脚）
- 语义化收益：屏幕阅读器地标导航、SEO 权重、代码可读性——三重回报
- 仅在"纯布局容器、无任何语义可选"时使用 `div` / `span`
- 标题层级：每页一个 `h1` 作为起点，层级递进不跳级（`h2` 之后才是 `h3`）；不因字号选标签（字号交给 CSS）
- `article` 与 `section` 的取舍：内容独立分发成篇（文章、评论、卡片）用 `article`；仅是主题分组用 `section`
- 文本语义标签：强调用 `em` / `strong`（而非 `i` / `b`）；引用用 `blockquote` / `q`；代码用 `code` / `pre`

```html
<!-- ❌ div 汤：无任何语义 -->
<div class="header">
  <div class="nav">…</div>
</div>
<div class="main">
  <div class="post">…</div>
</div>

<!-- ✅ -->
<header>
  <nav>…</nav>
</header>
<main>
  <article>
    <h1>…</h1>
  </article>
</main>
```

## 三、属性规范

- 属性值一律双引号（Google Style Guide），属性名小写
- 布尔属性使用简写：`disabled`、`required`、`checked`，不写 `disabled="disabled"`
- 属性顺序建议：`id` → `class` → 普通属性（`src` / `href` / `type` / `name` / `value` / `placeholder` 等）→ `data-*` / `aria-*`——高频在前，可访问性与扩展属性殿后
- 自定义数据用 `data-*` 前缀，禁自造无前缀属性
- HTML 实体：`&`、`<`、`>` 在文本中出现时使用实体转义（`&amp;` 等），避免解析歧义

```html
<!-- ❌ -->
<input NAME=keywords DISABLED='true' my-attr=1>

<!-- ✅ -->
<input name="keywords" id="keywords" class="input" disabled data-role="search">
```

## 四、可访问性（a11y）

> 来源：WAI-ARIA APG、MDN。a11y 不是可选项：受益者不止残障用户，还包括键盘党、弱网、搜索引擎。

### 1. 图片

- `img` 必带 `alt`：信息图写描述；纯装饰图 `alt=""`（读屏跳过）
- 功能性图片（图标按钮）`alt` 描述**动作**而非外观（"搜索"而非"放大镜图标"）

### 2. 表单与标签

- 每个表单控件必须关联 `label`（`for` + `id`，或包裹式）
- `placeholder` 只是提示示例，**不得替代** `label`（输入后即消失，读屏支持差）

### 3. 结构与导航

- 标题层级不跳级、每页一个 `h1`
- 链接文本有意义，禁"点击这里"/"更多"裸文本（读屏用户按链接列表导航）
- 跳转主内容提供 skip link（大型页面）

### 4. 交互与 ARIA

- 交互元素用原生标签：按钮用 `button` 而非 `div` + click（原生自带键盘焦点、回车空格触发、角色声明）
- `aria-*` 仅在原生 HTML 语义不足时补充（First Rule of ARIA：能用原生就不加 ARIA）；加 `aria-*` 必须配行为
- 焦点可见：不裸删 `outline`；自定义焦点样式需保证对比清晰
- `tabindex` 仅用 `0`（加入焦点序）与 `-1`（编程聚焦），禁 `tabindex="1+"` 打乱自然顺序

```html
<!-- ❌ div 假按钮：无键盘、无角色、无语义 -->
<div class="btn" onclick="submit()">提交</div>

<!-- ✅ -->
<button type="submit" class="btn">提交</button>
```

## 五、表单

- 每个控件有 `label`；成组选项用 `fieldset` + `legend`
- `input type` 按数据选精确类型（`email` / `tel` / `url` / `number` / `date`），获得正确的键盘与校验
- 关键字段声明 `autocomplete`（`name` / `email` / `new-password` 等），提升自动填充体验
- `button` 显式声明 `type`（`submit` / `button` / `reset`）——表单内默认值是 `submit`，漏写易误提交
- 必填与校验用原生属性优先：`required`、`min` / `max`、`pattern`；错误提示与控件关联（`aria-describedby`）

```html
<form action="/login" method="post">
  <label for="email">邮箱</label>
  <input type="email" id="email" name="email" autocomplete="email" required>
  <button type="submit">登录</button>
</form>
```

## 六、媒体与资源加载

- `img` 声明 `width` / `height`（或 `aspect-ratio`），防止加载时布局偏移（CLS）
- 首屏外图片加 `loading="lazy"`；首屏关键图不加（会延迟 LCP）
- 现代格式优先：`<picture>` + WebP/AVIF 回退
- 脚本默认 `defer`（保持顺序、不阻塞解析）；仅独立埋点类脚本用 `async`
- 样式表置顶、脚本置尾或 `defer`——避免阻塞渲染
- 外链资源声明 `rel="preconnect"` / `dns-prefetch`（跨域关键资源）

## 七、SEO 与元信息

- `<title>`（50~60 字符为宜）与 `<meta name="description">` 每页唯一
- 社交分享：Open Graph 标签（`og:title` / `og:description` / `og:image`）
- 多 URL 同内容声明 `rel="canonical"`
- favicon：`<link rel="icon" href="/favicon.ico">`
- 爬虫控制：`<meta name="robots" content="...">` 按需声明

```html
<head>
  <title>用户中心 - 示例系统</title>
  <meta name="description" content="管理您的订单、账户与偏好设置。">
  <link rel="canonical" href="https://example.com/account">
  <meta property="og:title" content="用户中心 - 示例系统">
  <meta property="og:image" content="https://example.com/og.png">
</head>
```
