---
name: "js-coding-standard"
description: "JavaScript 语言级编码规范，按 ES 版本（ES5 遗留 / ES2015 / ES2016~2019 / ES2020~2022 / ES2023~2025）组织最佳实践：变量声明、字符串、解构、对象、数组、函数、相等比较与类型判断、迭代、异步、Class、模块、命名、注释、安全禁区与语言版本适配。当编写、重构或审查 JavaScript 代码、按项目语言版本选择写法时使用。凡涉及 JavaScript 代码编写与修改的任务均应主动使用。"
---

# JavaScript 编码规范

本 skill 按 ECMAScript 版本组织 JavaScript 编码最佳实践，融合 Airbnb、Google、StandardJS、clean-code-javascript 等主流规范，适用于任意 JavaScript 代码，不限框架与运行时。

> 结构说明：本文件为精要版与索引；各版本完整规则与代码示例见 `references/`，按项目语言版本与主题按需加载。

## 何时使用

- 确定项目语言版本后，选择该版本可用的最佳实践写法
- 编写、重构或审查任意 JavaScript 代码（相等比较、异步、迭代、命名、注释、模块等）
- 维护 ES5 遗留代码，或将旧代码升级到更高版本写法
- 代码审查时作为编码规范检查依据

## 适用范围

### 适用对象（本规则适用于）

- 任意 JavaScript 代码（浏览器脚本、Node.js 脚本、模块等），不限框架与运行时
- ES5（遗留）至 ES2025 全部语言版本，按项目实际版本选用（见“三、各版本最佳实践速查”）

### 不适用（边界）

- TypeScript 类型层规范（本 skill 仅约束运行时写法，类型写法遵循项目约定）
- 框架级规范（React/Vue 组件结构、状态管理等，参见 `vue-coding-standard` / `react-coding-standard` skill）
- 分号、缩进、换行等纯排版格式，遵循项目工具链（ESLint/Prettier）配置，本 skill 不作规定

## 一、版本判定与总则

1. **以项目声明的语言版本为准**（browserslist / tsconfig `target` / 构建配置 / 运行时环境）
2. **禁止越级**：只使用该版本及以下支持的语法特性，禁止使用更高版本语法（否则编译失败或运行时报错）
3. **用足当前版本**：在该版本约束内，主动采用该版本可用的最佳实践写法，不写低版本的冗长等价物
4. 新项目无特殊约束时，建议以 ES2020+ 为基线

**正反例（假设项目构建目标为 ES2017）：**

```js
// ❌ 项目已支持 ES2017，仍使用低版本的 Promise 链写法
getData()
  .then((data) => saveData(data))
  .catch((error) => console.error(error));

// ✅ 使用 ES2017 async/await
async function load() {
  try {
    const data = await getData();
    await saveData(data);
  } catch (error) {
    console.error(error);
  }
}
```

**正反例（假设项目构建目标为 ES5）：**

```js
// ❌ 使用了高于项目版本的语法，运行直接报错
const nodes = [...foo];
class Queue {
}

// ✅ ES5 语法
var nodes = Array.prototype.slice.call(foo);
function Queue() {
}
```

## 二、通用规则精要（各版本通用）

以下为跨版本的质量底线，完整规则与正反例见 `references/general-quality.md`：

- 相等比较一律 `===` / `!==`；布尔用短路判断，数字与字符串用显式比较；默认值在 ES2020+ 用 `??`，更低版本用显式判断
- 数组迭代优先高阶函数（`map` / `filter` / `reduce` / `find` / `some` / `every`），回调必须有返回值
- 异步不吞错误：`catch` 与 `.catch()` 必须处理错误（记录、上报或通知），不忽略 rejected promise
- 函数参数卫生：不修改、不重赋值参数；配置项集中为最后一个对象参数，布尔不作裸参数
- 命名：变量/函数/实例 camelCase，类/构造器 PascalCase，不用前后下划线表示私有，缩写全大写或全小写
- 注释：多行用 JSDoc（`/** */`），单行独立成行、禁止尾随注释，TODO/FIXME 注明标记人与时间
- 安全禁区：禁 `eval()` / `new Function()`、禁扩展原生对象原型、禁 `debugger`、不用 stage 3 以下 TC39 提案
- 不直接调用 `Object.prototype` 方法：ES2022+ 用 `Object.hasOwn`，更低版本用 `Object.prototype.hasOwnProperty.call`

## 三、各版本最佳实践速查

高版本特性覆盖低版本：项目为 ES2020 时，同时适用此前所有版本的规范。

| 版本 | 解锁的关键特性与最佳实践 | 详细规范 |
| --- | --- | --- |
| ES5（遗留） | `var` / `function` / `prototype` / `require`（仅维护遗留代码时使用） | `references/es-legacy.md` |
| ES2015（ES6） | `let`/`const`、箭头函数、模板字符串、解构、默认/rest 参数、数组展开、`class`/`extends`、`import`/`export`、`Promise` | `references/es2015.md` |
| ES2016~2019 | `**` 指数、`async`/`await`、对象展开 `{ ...a, x: 3 }`、`flat`/`flatMap`、`Object.fromEntries`、可选 catch 绑定 | `references/es2016-2019.md` |
| ES2020~2022 | `?.` 可选链、`??` 空值合并、`Object.hasOwn`、`#field` 私有字段、顶层 `await`、`.at()`、`replaceAll`、逻辑赋值 | `references/es2020-2022.md` |
| ES2023~2025 | findLast、toSorted/toReversed/with、Promise.withResolvers、Object.groupBy、Set 组合方法、Iterator 辅助、Import attributes | `references/es2023-2025.md` |

## References（完整规范与代码示例）

- `references/general-quality.md` — 通用质量规范：相等比较与类型判断、迭代与高阶函数、异步错误处理、函数参数卫生、命名、注释、安全禁区
- `references/style-examples.md` — 编码风格完整示例与市面常见正反例速查（变量、字符串、解构、对象、数组、函数、相等比较、迭代、异步、模块、命名注释、安全禁区）
- `references/es-legacy.md` — ES5 遗留环境：Google 遗留例外、完整写法对照表、循环闭包陷阱、最小侵入原则
- `references/es2015.md` — ES2015 基线：块级作用域与变量声明、字符串、解构、对象、数组、函数、Map、Class、模块
- `references/es2016-2019.md` — ES2016~ES2019 增量：指数运算符、async/await、对象展开与剩余属性、数组与对象新方法、可选 catch 绑定
- `references/es2020-2022.md` — ES2020~ES2022 增量：可选链、空值合并、allSettled/any、replaceAll、逻辑赋值、hasOwn、私有字段、顶层 await、at
- `references/es2023-2025.md` — ES2023~ES2025 版本特性：不可变数组方法、findLast、Promise.withResolvers、groupBy、Set 组合方法、Iterator 辅助、Import attributes
