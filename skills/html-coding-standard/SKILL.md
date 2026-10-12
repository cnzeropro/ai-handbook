---
name: "html-coding-standard"
description: "HTML 编码规范：文档结构、语义化标签、属性规范、可访问性（a11y）、表单、媒体与 SEO 元信息。当编写、重构或审查 HTML 代码、编写 Vue/React 模板中的标记结构时使用。凡涉及 HTML 代码编写与修改的任务均应主动使用。"
---

# HTML 编码规范

本 skill 提供 HTML 语言层面的编码规范，融合 HTML Living Standard、MDN、WAI-ARIA APG（可访问性应用实践）与 Google HTML/CSS Style Guide，适用于任意 HTML 代码——包括 Vue/React 等框架模板中的标记结构。

> 结构说明：本文件为精要版与索引；各主题完整规则与代码示例见 `references/`，按需加载。
>
> 与其他 skill 的关系：样式写法见 `css-coding-standard`；脚本行为见 `js-coding-standard`；框架层组件设计见 `vue-coding-standard` / `react-coding-standard`。

## 何时使用

- 编写、重构或审查任意 HTML（静态页面、邮件模板、框架模板中的标记结构）
- 需要确定语义化标签选择、属性写法、可访问性（a11y）合规时
- 代码审查时作为标记规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 HTML：静态页面、邮件模板、框架模板中的标记结构（框架指令语法除外）
- 不限运行环境（浏览器、SSR 输出、构建产物模板）

### 不适用（边界）

- CSS 样式写法（参见 `css-coding-standard` skill）
- JS 行为与交互逻辑（参见 `js-coding-standard` skill）
- 框架层组件设计与状态（参见 `vue-coding-standard` / `react-coding-standard` skill）
- 缩进、换行等纯排版格式，遵循项目工具链（Prettier 等）配置

## 一、通用规则精要

完整规则与理由见 `references/general-quality.md`，完整正反例见 `references/style-examples.md`：

- 文档骨架四件套必备：`<!DOCTYPE html>`、`<html lang>`、`<meta charset="utf-8">`、viewport meta；`<title>` 唯一且有意义
- 语义优先：按含义选标签（`header` / `nav` / `main` / `article` / `section` / `aside` / `footer`），禁用无语义 `div` / `span` 堆砌；`main` 每页唯一；`section` 应配标题
- 属性规范：双引号、小写属性名；布尔属性简写；属性顺序 id → class → 普通属性 → data-* / aria-*
- 可访问性底线：图片必带 `alt`（装饰图 `alt=""`）；表单控件必关联 `label`；标题层级不跳级、每页一个 `h1`；交互用原生元素（`button` 而非 `div` + click）；`aria-*` 仅在原生语义不足时补充
- 表单：`label` 必须关联控件、`placeholder` 不得替代 `label`、`button` 显式声明 `type`、合理使用 `autocomplete` 与精确 `input type`
- 媒体与资源：`img` 声明宽高防布局偏移、首屏外图片 `loading="lazy"`；脚本默认 `defer`；样式表置顶
- SEO 元信息：`title` / `meta description` / Open Graph / `rel="canonical"` 完整
- 遗留写法一律禁用：`font` / `center` / `marquee` 等已废弃标签与 table 布局——HTML 为 Living Standard 连续演进、无版本阶梯，一律按现代写法（遗留页面按最小侵入原则维护）

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：文档结构、语义化、属性、可访问性、表单、媒体与资源、SEO 各章完整规则与理由
- `references/style-examples.md` — 编码风格完整示例与市面常见正反例速查（页面骨架、div 汤、图片与表单 a11y、head 模板等）
