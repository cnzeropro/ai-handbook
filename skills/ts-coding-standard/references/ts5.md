# TS 5.x 写法规约（当前主版本）

> 本文件为 **TS 5.x** 的写法选择规约（推荐写法 vs 旧写法 + 踩坑原因），不是特性介绍——语法细节以官网为准。TS 4.x 及以下见 `ts-legacy.md`。须与 `general-quality.md` 配合使用。

## 目录

- 一、写法规约 —— satisfies 替代 as、const 类型参数、标准 decorators、bundler 解析、using、推断谓词、erasableSyntaxOnly
- 二、TS 6/7 方向规约

## 一、写法规约

### 1. satisfies 替代 as 断言

`as` 会放宽推导并丢失字面量信息，且编译器照单全收错误形状；`satisfies` 校验通过后**推导保留**：

```ts
// ❌ port 放宽为 number，字面量推导丢失
const config = { host: 'localhost', port: 8080 } as Config;
config.port.toFixed(2);

// ✅ 校验通过，且仍是 8080 字面量类型
const config = { host: 'localhost', port: 8080 } satisfies Config;
config.port.toFixed(2);
```

### 2. const 类型参数替代调用处 as const

泛型推导默认放宽为数组/可变类型；`const` 修饰符让调用处免写 `as const`（漏写即失去窄类型）：

```ts
// ❌ 推导为 string[]
function defineRoute<T extends string>(path: T) { … }
const p = defineRoute('/models/:id');       // string

// ✅
function defineRoute<const T extends string>(path: T) { … }
const p = defineRoute('/models/:id');       // '/models/:id'
```

### 3. 标准 decorators 替代实验性 decorators

`experimentalDecorators` 语法与 TC39 标准不兼容（两套语法互不识别）；新项目用标准 decorators，遗留项目升级时**整体迁移**，不混用。

### 4. 打包器项目用 moduleResolution: bundler

Vite/esbuild 等构建链用 `bundler` 解析策略；禁与 `node`/`nodenext` 混搭自造组合（解析结果不一致是"本地能跑、CI 报错"的经典来源）。

### 5. using 声明替代手工 try/finally 清理

需要 dispose 的资源用 `using`（作用域结束自动清理）；手工 try/finally 容易漏清理分支：

```ts
// ❌ 手工清理，异常路径易漏
const handle = acquireLock(path);
try {
  …
} finally {
  handle.release();
}

// ✅ 实现 [Symbol.dispose] 的对象
using handle = acquireLock(path);
```

### 6. 依赖推断类型谓词，禁恒真假手写谓词

手写 `(x): x is T => true` 恒真谓词等于关闭检查；5.5+ 对形如布尔收窄的函数自动推断：

```ts
// ✅ 谓词自动推断，无需手写注解
const models = rows.filter(isModelRow);
```

### 7. 开启 erasableSyntaxOnly（禁非可擦除语法）

`--erasableSyntaxOnly` 禁用 enum / namespace / 参数属性 / `import x = require` 等**需生成运行时代码**的语法——与 TS 7 原生编译器方向对齐，也与本 skill 禁 enum / 禁 namespace 规则一致（新项目建议开启）。

## 二、TS 6/7 方向规约

- **生产项目以 5.x 稳定版为准**；TS 7（Go 原生编译器，tsgo）处于预览阶段，不进生产
- erasable-only 方向：禁 enum / namespace / 参数属性 / `import x = require` 等非可擦除语法（与 `general-quality.md` 既有规约一致）
- 升级属于独立任务：先升 5.x 最新补丁，再评估 6/7（预览期间不引入）
