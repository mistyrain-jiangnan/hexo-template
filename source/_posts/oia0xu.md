---
title: 实用的基础工具函数
urlname: oia0xu
date: '2021-08-06 16:48:55 +0800'
tags: []
categories: []
---

### remove 移除数组的一项

```javascript
const remove = (arr, el) => {
  const i = arr.indexOf(el);
  if (i > -1) {
    arr.splice(i, 1);
  }
};

// 例子：
const arr = [1, 2, 3];
remove(arr, 3);
console.log(arr); // [1, 2]
```

### isPromise 判断是不是 Promise

```javascript
const isPromise = (val) => {
  return isObject(val) && isFunction(val.then) && isFunction(val.catch);
};
// 判断是不是Promise对象
const p1 = new Promise(function (resolve, reject) {
  resolve("若川");
});
isPromise(p1); // true
```

### isMap 判断是不是 Map 对象

```javascript
const isMap = (val) => toTypeString(val) === "[object Map]";
// 例子：
const map = new Map();
const o = { p: "Hello World" };
map.set(o, "content");
map.get(o); // 'content'
isMap(map); // true
```

> ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。

### isSet 判断是不是 Set 对象

```javascript
const isSet = (val) => toTypeString(val) === "[object Set]";
// 例子：
const set = new Set();
isSet(set); // true
```

> `ES6` 提供了新的数据结构 `Set`。它类似于数组，但是成员的值都是唯一的，没有重复的值。

`Set`本身是一个构造函数，用来生成 `Set` 数据结构。

### isDate 判断是不是 Date 对象

```javascript
const isDate = (val) => val instanceof Date;
// 例子：
isDate(new Date()); // true
// `instanceof` 操作符左边是右边的实例。但不是很准，但一般够用了。原理是根据原型链向上查找的。
isDate({__proto__ : new Date()); // true
// 实际上是应该是 Object 才对。
// 所以用 instanceof 判断数组也不准确。
// 再比如
({__proto__: [] }) instanceof Array; // true
// 实际上是对象。
// 所以用 数组本身提供的方法 Array.isArray 是比较准确的。
```

### isFunction 判断是不是函数

```javascript
const isFunction = (val) => typeof val === "function";
// 判断数组有多种方法，但这个是比较常用也相对兼容性好的。
```

### isString 判断是不是字符串

```javascript
const isString = (val) => typeof val === "string";
// 例子：
isString(""); // true
```

### isSymbol 判断是不是 Symbol

```javascript
const isSymbol = (val) => typeof val === "symbol";
// 例子：
let s = Symbol();
typeof s;
// "symbol"
// Symbol 是函数，不需要用 new 调用。
```

> `ES6` 引入了一种新的原始数据类型`Symbol`，表示独一无二的值。

### isObject 判断是不是对象

```javascript
const isObject = (val) => val !== null && typeof val === "object";
// 例子：
isObject(null); // false
isObject({ name: "若川" }); // true
// 判断不为 null 的原因是 typeof null 其实 是 object
```

### 常用正则

```javascript
// \w 是 0-9a-zA-Z_ 数字 大小写字母和下划线组成
// () 小括号是 分组捕获
const camelizeRE = /-(\w)/g;
/**
 * @private
 */
// 连字符 - 转驼峰  on-click => onClick
const camelize = cacheStringFunction((str) => {
  return str.replace(camelizeRE, (_, c) => (c ? c.toUpperCase() : ""));
});
// \B 是指 非 \b 单词边界。
const hyphenateRE = /\B([A-Z])/g;
/**
 * @private
 */
const hyphenate = cacheStringFunction((str) =>
  str.replace(hyphenateRE, "-$1").toLowerCase()
);
// 举例：onClick => on-click
const hyphenateResult = hyphenate("onClick");
console.log("hyphenateResult", hyphenateResult); // 'on-click'
/**
 * @private
 */
// 首字母转大写
const capitalize = cacheStringFunction(
  (str) => str.charAt(0).toUpperCase() + str.slice(1)
);
/**
 * @private
 */
// click => onClick
const toHandlerKey = cacheStringFunction((str) =>
  str ? `on${capitalize(str)}` : ``
);
const result = toHandlerKey("click");
console.log(result, "result"); // 'onClick'
```

## 时间戳转换为时间

- isMs 为时间戳是否为毫秒

```javascript
function timestampToTime(timestamp = Date.parse(new Date()), isMs = true) {
  const date = new Date(timestamp * (isMs ? 1 : 1000));
  return `${date.getFullYear()}-${
    date.getMonth() + 1 < 10 ? "0" + (date.getMonth() + 1) : date.getMonth() + 1
  }-${date.getDate()} ${date.getHours()}:${date.getMinutes()}:${date.getSeconds()}`;
}
```

## 数字千分位分割

```javascript
function commafy(num) {
  return num.toString().indexOf(".") !== -1
    ? num.toLocaleString()
    : num.toString().replace(/(\d)(?=(?:\d{3})+$)/g, "$1,");
}
```