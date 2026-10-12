# JavaScript 编码风格完整示例

> 收录各主题完整示例与市面高频正反例（源自 Airbnb、StandardJS、clean-code-javascript 等主流规范的经典案例）。规则正文见 `SKILL.md` 与各版本分册；本文件供代码审查对照与深入学习。


## 目录

- 1. 变量声明 —— var 闭包陷阱、链式赋值、未声明赋值
- 2. 字符串 —— 引号与模板、长字符串拼接
- 3. 解构赋值 —— 对象/嵌套/函数签名解构
- 4. 对象 —— 字面量简写、展开 vs Object.assign、hasOwn
- 5. 数组 —— 类数组转换、push 与 map
- 6. 函数 —— 默认参数、参数卫生、布尔裸参数
- 7. 相等比较与类型判断 —— == 陷阱、truthy、浮点、parseInt、??、嵌套三元
- 8. 迭代与高阶函数 —— filter/map 链、reduce 漏 return、entries
- 9. 异步 —— 回调地狱、forEach + async、不吞错误
- 10. 模块 —— 通配符、可变导出、import 位置
- 11. 命名与注释 —— 大小写约定、注释位置与 JSDoc
- 12. 安全禁区 —— eval、new Function、原型扩展、debugger
## 1. 变量声明

var 的函数作用域与循环闭包陷阱：

```js
// ❌ var 函数作用域，闭包共享同一变量
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // 3, 3, 3
}

// ✅
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // 0, 1, 2
}
```

链式赋值产生隐式全局变量（Airbnb 13.5）：

```js
// ❌ b、c 未声明，成为全局变量
(function example() {
  let a = b = c = 1;
}());
console.log(b); // 1

// ✅
let a = 1;
let b = 1;
let c = 1;
```

未声明赋值：

```js
// ❌ 意外污染全局
function render() {
  total = 100;
}

// ✅
function render() {
  const total = 100;
}
```

## 2. 字符串

引号与模板字符串：

```js
// ❌ 双引号 + 字符串拼接
const name = "Bob";
const greet = 'Hello, ' + name + '!';

// ✅
const name = 'Bob';
const greet = `Hello, ${name}!`;
```

超长字符串不做折行拼接（Airbnb 6.2——难搜索、难维护）：

```js
// ❌
const msg = 'This is a long error message ' +
  'split across lines';

// ✅ 超长也保持单行；确需换行时用模板字符串
const msg = 'This is a long error message kept on a single line';
```

## 3. 解构赋值

对象与嵌套解构：

```js
// ❌ 逐个取值
const firstName = user.firstName;
const lastName = user.lastName;

// ✅
const { firstName, lastName } = user;

// ✅ 嵌套 + 默认值
const { address: { city = 'unknown' } = {} } = user;
```

函数签名解构（Airbnb 5.1“best”形态）：

```js
// ❌
function createUser(options) {
  const name = options.name;
  const role = options.role || 'member';
}

// ✅
function createUser({ name, role = 'member' }) {
}
```

## 4. 对象

字面量、简写与计算属性：

```js
// ❌
const item = new Object();
item.visible = true;

// ✅
const visible = true;
const item = {
  visible,
  [getKey('enabled')]: false,
};
```

展开替代 Object.assign（不修改原对象，Airbnb 3.8）：

```js
// ❌ defaults 被修改
const defaults = { timeout: 3000 };
const config = Object.assign(defaults, options);

// ✅
const config = { ...defaults, ...options };
```

不直接调用 Object.prototype 方法：

```js
// ❌ 可被自身属性遮蔽（{ hasOwnProperty: false }）
if (obj.hasOwnProperty('key')) {}

// ✅
if (Object.hasOwn(obj, 'key')) {}       // ES2022+
if (Object.prototype.hasOwnProperty.call(obj, 'key')) {}  // 旧环境
```

## 5. 数组

类数组转换与拷贝：

```js
// ❌
const args = Array.prototype.slice.call(arguments);
const copy = [];
for (let i = 0; i < items.length; i++) {
  copy[i] = items[i];
}

// ✅
const args = Array.from(arguments);   // 类数组用 from
const copy = [...items];              // 可迭代用展开
```

索引赋值改 push / map：

```js
// ❌
const doubled = [];
for (let i = 0; i < items.length; i++) {
  doubled[i] = items[i] * 2;
}

// ✅
const doubled = items.map((x) => x * 2);
```

## 6. 函数

默认参数与参数卫生：

```js
// ❌ 参数重赋值 + opts || {} 写法
function handleThings(opts) {
  opts = opts || {};
}

// ✅
function handleThings(opts = {}) {
}
```

不修改入参对象（clean-code：避免副作用）：

```js
// ❌ 污染调用方对象
function normalize(user) {
  user.name = user.name.trim();
}

// ✅ 返回新对象
function normalize(user) {
  return { ...user, name: user.name.trim() };
}
```

布尔裸参数改配置对象（clean-code：flag 参数）：

```js
// ❌ 调用处 true 含义不明
render(el, true);

// ✅
function render({ parent = document.body, animate = false } = {}) {
}
render({ parent: el, animate: true });
```

## 7. 相等比较与类型判断

== 隐式转换的经典陷阱：

```js
// ❌ 结果反直觉
'' == 0;            // true
'1' == 1;           // true
null == undefined;  // true

// ✅
'' === 0;           // false
'1' === 1;          // false
null === undefined; // false
```

truthy 判断陷阱：

```js
// ❌ 空数组是对象，恒为真
if ([]) { console.log('runs!'); }
if (collection.length) {}   // 意图不明确

// ✅
if (collection.length > 0) {}
```

浮点数比较：

```js
// ❌ 浮点精度
0.1 + 0.2 === 0.3;  // false

// ✅
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON;
```

parseInt 缺省进制：

```js
// ❌ 旧引擎下 '08' 可能按八进制解析
parseInt('08');

// ✅
parseInt('08', 10);
```

?? 与 || 的区别（ES2020+）：

```js
// ❌ count 为 0 时错误取到默认值
const count = input.count || 10;

// ✅
const count = input.count ?? 10;
```

嵌套三元拆分：

```js
// ❌
const label = a > b ? 'high' : c > d ? 'mid' : 'low';

// ✅
const fallback = c > d ? 'mid' : 'low';
const label = a > b ? 'high' : fallback;
```

## 8. 迭代与高阶函数

命令式循环改 filter/map 链：

```js
// ❌
const activeNames = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].active) {
    activeNames.push(users[i].name);
  }
}

// ✅
const activeNames = users
  .filter((user) => user.active)
  .map((user) => user.name);
```

reduce 回调漏 return（高频 bug）：

```js
// ❌ acc 从第二次迭代起为 undefined
const total = orders.reduce((acc, order) => {
  acc + order.amount;
});

// ✅
const total = orders.reduce((acc, order) => acc + order.amount, 0);
```

对象迭代用 entries：

```js
// ❌ for-in 连原型链属性一起遍历
for (const key in obj) {
  console.log(key, obj[key]);
}

// ✅
Object.entries(obj).forEach(([key, value]) => {
  console.log(key, value);
});
```

## 9. 异步

回调地狱 → Promise → async/await（clean-code Concurrency）：

```js
// ❌ 回调嵌套
getData((err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  saveData(data, (saveErr) => {
    if (saveErr) {
      console.error(saveErr);
    }
  });
});

// ✅
async function load() {
  try {
    const data = await getData();
    await saveData(data);
  } catch (error) {
    console.error(error);
  }
}
```

forEach 不等待 async（高频错误）：

```js
// ❌ forEach 不会等待，完成时机与顺序失控
models.forEach(async (model) => {
  await save(model);
});

// ✅ 串行
for (const model of models) {
  await save(model);
}

// ✅ 并行
await Promise.all(models.map((model) => save(model)));
```

不吞错误（clean-code Error Handling）：

```js
// ❌ 什么都不做
try {
  await run();
} catch (error) {}

// ✅ 记录 + 上报
try {
  await run();
} catch (error) {
  console.error(error);
  reportError(error);
}
```

## 10. 模块

通配符导入与可变导出：

```js
// ❌
import * as utils from './utils';
let version = '1.0';
export { version };

// ✅
import { formatDate, formatMoney } from './utils';
const version = '1.0';
export { version };
```

import 置顶（import 会提升，置顶避免意外行为）：

```js
// ❌
import fmt from './fmt';
init();
import conf from './conf';

// ✅
import fmt from './fmt';
import conf from './conf';

init();
```

## 11. 命名与注释

大小写约定：

```js
// ❌
const HttpRequests = [];
function Get_User() {}
class httpSender {}

// ✅
const httpRequests = [];        // 缩写全小写
function getUser() {}           // camelCase
class HttpRequestSender {}      // PascalCase
```

注释位置与 JSDoc：

```js
// ❌ 尾随注释
const active = true;  // is current tab

// ✅ 独立成行
// is current tab
const active = true;

// ✅ 多行用 JSDoc
/**
 * Parses a raw config string into a structured object.
 * @param {string} raw
 * @returns {{retries: number, timeout: number}}
 */
function parseConfig(raw) {}
```

## 12. 安全禁区

eval / new Function / 原型扩展 / debugger：

```js
// ❌
eval('console.log("boom")');
const sum = new Function('a', 'b', 'return a + b');
Array.prototype.first = function () { return this[0]; };
debugger;

// ✅ 正规函数 + 独立工具函数
const sum = (a, b) => a + b;
const first = (arr) => arr[0];
```
