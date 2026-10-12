# 通用质量规范（各版本通用）

> 本文件约束与语言版本无关的代码质量底线，适用于 ES5 至 ES2022+ 全部版本（个别条目对低版本的替代写法有标注）。版本相关的语法特性见各版本分册。

## 一、相等比较与类型判断

### 1. 一律使用 === 和 !==

`==` / `!=` 会做隐式类型转换（如 `'1' == 1` 为 `true`），是常见 bug 来源（Standard 强制、Airbnb 15.1）：

```js
// ❌
if (name == 'test') {
}

// ✅
if (name === 'test') {
}
```

### 2. 布尔用短路判断，字符串与数字用显式比较

```js
// ❌
if (isValid === true) {
}
if (collection.length) {
}

// ✅
if (isValid) {
}
if (collection.length > 0) {
}
```

### 3. 默认值优先用 ??，而非 ||（ES2020+）

`||` 会把 `0`、`''`、`false` 一并当作“空”；`??` 仅在 `null` / `undefined` 时取右值（Airbnb 15.9，示例见 `es2020-2022.md`）。低于 ES2020 的环境用显式判断表达同样语义。

### 4. 避免嵌套三元

```js
// ❌
const foo = a > b ? 'x' : c > d ? 'y' : null;

// ✅ 拆分为独立变量或 if 语句
const fallback = c > d ? 'y' : null;
const foo = a > b ? 'x' : fallback;
```

### 5. 数字判断用 Number 系静态方法

全局 `isNaN` / `isFinite` 会先转数字再判断，产生误判（Airbnb 29.1/29.2）：

```js
// ❌
isNaN('1.2.3'); // true

// ✅
Number.isNaN(Number('1.2.3')); // true
```

### 6. 混合逻辑运算符加括号

`&&` / `||` 与比较混合时加括号明确意图（`+`、`-`、`**` 等算术优先级清晰的可省略）（Airbnb 15.8）：

```js
// ❌
if (a || b && c) {
}

// ✅
if (a || (b && c)) {
}
```

## 二、迭代与高阶函数

### 1. 数组优先用高阶函数而非循环

`map` / `filter` / `reduce` / `find` / `findIndex` / `some` / `every` / `forEach` 优先于 `for` / `for-in` / `for-of`——返回值的纯函数比带副作用的循环更易推理（Airbnb 11.1；ES5 环境可用 `forEach` / `filter` 等同样适用）：

```js
const numbers = [1, 2, 3, 4, 5];

// ❌
let sum = 0;
for (let num of numbers) {
  sum += num;
}

// ✅
const sum = numbers.reduce((total, num) => total + num, 0);

// ❌
const increasedByOne = [];
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1);
}

// ✅
const increasedByOne = numbers.map((num) => num + 1);
```

### 2. 对象迭代用 Object 静态方法

```js
// ✅
Object.keys(obj).forEach((key) => {
  // ...
});

const pairs = Object.entries(obj).map(([key, value]) => [key, value * 2]);
```

### 3. 数组方法回调必须有返回值

```js
// ❌ 回调无返回值，reduce 从第二次迭代起 acc 为 undefined
[[0, 1], [2, 3]].reduce((acc, item) => {
  acc.concat(item);
});

// ✅
[[0, 1], [2, 3]].reduce((acc, item) => acc.concat(item), []);
```

## 三、异步错误处理原则

`catch` 与 `.catch()` 捕获的错误必须处理（记录、上报或通知用户），不能静默吞掉；不忽略 rejected promise（clean-code Error Handling）。async/await 与 Promise 链的写法选择见 `es2016-2019.md`：

```js
// ❌ 仅 console.log 甚至什么都不做
try {
  await fetchData();
} catch (error) {
  console.log(error);
}

// ✅
try {
  await fetchData();
} catch (error) {
  console.error(error);
  reportError(error);
}
```

## 四、函数参数卫生

- 不在函数体内修改参数对象（会污染调用方）（Airbnb 7.12）
- 不对参数重新赋值，需要兜底值时用默认参数或新变量（Airbnb 7.13）
- 配置项集中在一个对象、放在最后一个参数；布尔值不可直接作为参数（clean-code：flag 参数使函数同时承担两种职责）

```js
// ❌
function f1(obj) {
  obj.key = 1;
}

// ❌
function f2(a) {
  if (!a) { a = 1; }
}

// ❌
function divide(a, b, option = false) {
}

// ✅
function f4(a = 1) {
}

function divide(a, b, { option = false } = {}) {
}
```

> 默认参数语法为 ES2015+；ES5 环境的等价写法见 `es-legacy.md` 对照表。默认参数放最后、避免副作用（Airbnb 7.8/7.9）。

## 五、命名规范

### 1. 变量、函数、实例用 camelCase（Airbnb 23.2、Standard）

```js
// ❌
const this_is_my_object = {};
function c() {
}

// ✅
const thisIsMyObject = {};
function thisIsMyFunction() {
}
```

### 2. 类、构造器用 PascalCase（Airbnb 23.3）

```js
// ✅
class User {
  constructor(options) {
    this.name = options.name;
  }
}

const user = new User({ name: 'yup' });
```

### 3. 不用前后下划线表示私有

JS 属性没有真正的私有，下划线会让使用者误以为非公开 API（Airbnb 23.4）；真私有使用 `#` 私有字段（ES2022+，见 `es2020-2022.md`）或闭包：

```js
// ❌
this._firstName = 'Panda';

// ✅
this.firstName = 'Panda';
```

### 4. 缩写全大写或全小写，不混用（Airbnb 23.9）

```js
// ❌
import SmsContainer from './containers/SmsContainer';
const HttpRequests = [];

// ✅
import SMSContainer from './containers/SMSContainer';
const HTTPRequests = [];
const httpRequests = [];
```

### 5. 避免单字母命名；仅导出的可信常量用全大写

全大写 `CONSTANT` 仅用于：导出的、`const` 声明的、可确信不再变化的值；文件内普通常量无需大写（Airbnb 23.1/23.10）：

```js
// ✅ 导出常量
export const API_KEY = 'SOMEKEY';

// ❌ 文件内私有变量不必大写
const PRIVATE_VARIABLE = 'should not be unnecessarily uppercased';
```

## 六、注释规范

### 1. 多行注释用 JSDoc 风格 /** */

```js
// ❌ 用行注释拼文档
// make() returns a new element
// based on the passed-in tag name

// ✅
/**
 * make() returns a new element based on the passed-in tag name.
 */
function make(tag) {
  // ...
}
```

### 2. 单行注释独立成行，禁止尾随注释

```js
// ❌
const active = true;  // is current tab

// ✅
// is current tab
const active = true;
```

### 3. TODO / FIXME 须注明标记人与时间，并定期清理

```js
// TODO(张三 2025-10-12): total should be configurable by an options param
// FIXME(李四 2025-10-12): shouldn't use a global here
```

## 七、安全与禁区

- 禁止 `eval()` 与 `new Function()`：等同执行任意字符串，有注入风险（Airbnb 6.4/7.10、Standard）
- 禁止修改内建对象原型（如 `Array.prototype.myMethod = ...`）（Standard）
- 禁止 `debugger` 语句（Standard；ESLint `no-debugger`）
- 不使用未到达 stage 3 的 TC39 提案语法（Airbnb 28.2）

```js
// ❌
eval('console.log("dangerous")');
const add = new Function('a', 'b', 'return a + b');

// ✅ 使用正规函数
const add = (a, b) => a + b;
```
