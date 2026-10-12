# ES5 及以下（遗留环境）规范

> 适用：构建目标为 ES5 及以下、且无 Babel / TypeScript 转译链的遗留项目。仅在维护遗留代码或项目明确要求时使用；新代码应使用更高版本（见 `es2015.md` 及后续版本规范）。

## 一、Google 遗留平台例外与底线

Google 风格指南"遗留平台例外"（9.4）：

- 允许：使用 `var`、使用 `arguments`、参数省略默认值
- 仍然禁止：在块内写函数声明（ES5 严格模式不允许，各引擎行为不一致），用 `var foo = function () {}` 代替
- `var` 声明尽量靠近首次使用处，便于阅读；变量被块外引用时不要声明在块内
- 需要常量语义时，用 `/** @const */` 注解 `var` 声明

## 二、ES6+ → ES5 写法对照表

| ES6+ 写法 | ES5 等价写法 |
| --- | --- |
| `let` / `const` | `var`（注意函数作用域与循环闭包陷阱） |
| 箭头函数 | `function` + `var self = this` 或 `bind(this)` |
| 模板字符串 | 字符串拼接 `'a' + b + 'c'` |
| 解构赋值 | 逐个赋值 |
| 参数默认值 | 函数体内 `opts = opts \|\| {}` |
| rest 运算符 | `arguments` + `Array.prototype.slice.call` |
| 展开 / 拷贝 | 循环拷贝 / `concat` / `Object.assign`（或自实现浅拷贝） |
| `class` / `extends` | 构造函数 + `prototype` + 继承辅助函数 |
| `import` / `export` | `require` / `module.exports`（或项目既有模块方案） |
| `Promise` / `async-await` | 回调（除非项目引入 polyfill） |

## 三、循环闭包陷阱

`var` 是函数作用域，循环内闭包共享同一个变量：

```js
// ❌ 输出 3, 3, 3 而非 0, 1, 2
for (var i = 0; i < 3; i += 1) {
  setTimeout(function () { console.log(i); }, i * 1000);
}

// ✅ 用 IIFE 捕获每次迭代的值
for (var i = 0; i < 3; i += 1) {
  (function (n) {
    setTimeout(function () { console.log(n); }, n * 1000);
  })(i);
}
```

## 四、遗留代码最小侵入

- 在 ES5 代码库中修改代码时与周边风格保持一致，不顺手做语法现代化，避免无关 diff
- 确需整体升级到更高版本，应作为独立任务执行，不与功能变更混杂
- 遗留库中的新文件：若项目转译链允许，仍应使用高版本写法；否则与所在模块风格一致
