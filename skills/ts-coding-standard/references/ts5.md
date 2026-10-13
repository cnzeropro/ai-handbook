# TS 5.x 规范（当前主版本）

> 本文件为 **TS 5.x** 版本增量与约定（含 4.9 时代延续特性），须与版本通用的 `general-quality.md` 配合使用。TS 4.x 及以下见 `ts-legacy.md`；TS 7（原生编译器）见文末“TS 6/7 展望”。

## 目录

- 一、5.x 时代特性阶梯 —— satisfies、const 类型参数、标准化 decorators、bundler 解析、using 声明、推断类型谓词、erasableSyntaxOnly
- 二、TS 6/7 展望 —— 原生编译器与 erasable-only 方向

## 一、5.x 时代特性阶梯

### 1. satisfies 操作符（4.9+）

校验类型匹配但**保留推导出的字面量类型**，替代 `as` 断言的常见场景：

```ts
// ❌ as 断言：丢失字面量类型
const config = {
  host: 'localhost',
  port: 8080,
} as Config;                     // config.port 变成 number 而非 8080

// ✅ satisfies：检查通过，推导保留
const config = {
  host: 'localhost',
  port: 8080,
} satisfies Config;
config.port.toFixed(2);          // 仍知道是 8080（number 字面量）
```

### 2. const 类型参数（5.0）

泛型推导默认保留字面量/只读性，替代调用处 `as const`：

```ts
// ❌ 推导为 string[]
function defineRoute<T extends string>(path: T) { … }
defineRoute('/models/:id');      // path: string

// ✅
function defineRoute<const T extends string>(path: T) { … }
defineRoute('/models/:id');      // path: '/models/:id'
```

### 3. 标准化 decorators（5.0）

不再需要 `experimentalDecorators` 编译项；新项目一律用标准 decorators 语法，遗留实验性写法在升级时整体迁移。

### 4. moduleResolution: bundler（5.0）

打包器项目（Vite/esbuild 等）使用 `bundler` 解析策略，与 `node` 系解析区分；跟随项目构建链选择。

### 5. using 声明与显式资源管理（5.2）

实现 `[Symbol.dispose]` 的对象用 `using` 自动清理，替代手工 try/finally：

```ts
{
  using handle = acquireFileLock(path);   // 作用域结束自动 dispose
  // ...
}
```

### 6. 推断类型谓词（5.5）

过滤函数自动推断 `T` 收窄，无需手写类型谓词注解：

```ts
// 5.4 及以前：需要显式谓词
const models = rows.filter((r): r is ModelRow => isModelRow(r));

// ✅ 5.5+：isModelRow 形如 (x: unknown) => boolean 时自动推断
const models = rows.filter(isModelRow);
```

### 7. erasableSyntaxOnly（5.8）

编译项 `--erasableSyntaxOnly` 禁用一切“需生成运行时代码”的语法（enum、namespace、参数属性）——与 `general-quality.md` 的禁 enum/禁 namespace 规则同向，新项目建议开启。

## 二、TS 6/7 展望 —— 原生编译器与 erasable-only 方向

- **TS 7** 为 Go 原生编译器（项目 Corsa，包 `@typescript/native-preview`，命令 `tsgo`），编译提速约 10 倍；**TS 6.0** 为 JS 侧过渡版本（对齐默认值、清理废弃项）
- **编码层影响**：erasable-only 方向成为主线——enum、namespace（含运行时代码的）、参数属性（`constructor(private x: string)`）、`import x = require(...)` 等非可擦除语法将不受支持；本 skill 的禁 enum / 禁 namespace / 禁参数属性倾向与此完全一致
- **现状**：7 处于预览阶段，生产项目仍以 5.x 稳定版为准；预览可用 `@typescript/native-preview` 试用评估
