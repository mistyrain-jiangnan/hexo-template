---
title: JavaScript 的 8 种数据类型以及它们的底层数据结构
urlname: vprpcx
date: '2021-09-06 15:59:45 +0800'
tags: []
categories: []
---

基本类型：String、Boolean、Number、Undefined、Null、Symbol、BigInt
引用类型：Object
下面我会详细说一下其中七种数据类型（不包括 BigInt）中可能你不太知道的一些细节。

## 1. String

### **储存结构**

计算机是以二进制存储以及发送接收数据的。二进制的 1 位，也叫做 1bit 。它是计算机内部存储的最基本的单位。
计算机只能处理数字，如果想要处理文本，必须将文本转换为数字才能处理，在计算机中 1bit 能表示 2 个状态（0 或 1），1Byte 等于 8bit，所以 1Byte 能表示 2^8 - 1 个整数，也就是 255 个。如果想表示更大的数字，就需要更多的字节数，比如 2Byte 能表示 2^16 - 1 ，也就是 65535 个整数。最早只有 127 个字符被编码到计算机里，也就是大小写英文字母、数字和一些其他字符，这个编码表就是 ASCII 表。
但如果要表示中文，那么 1Byte 显然是不够的，至少需要 2Byte ，所以中国制定了 GB2312 编码，但每个国家如果都制定一个自己的编码表，那么就没办法正确显示多语言混合的文本。为了解决这个问题，Unicode 编码应运而生。它把所有语言统一到一个编码里，采用 2Byte 表示一个字符，即最多可以表示 2^16 - 1 ，也就是 65535 个字符。这样基本上可以覆盖世界上常用的文字，如果要表示更多的文字，也可以采用 4Byte 进行编码，这是一种通用的编码规范 。
JS 中的字符也采用 Unicode 编码，也就是 js 中的中英文字符都占用 2Byte（16bit）大小。
在 JS 中的二进制数据储存中，二进制前三位为 `100` 代表字符串

### **基本包装类型**

在 js 中，只有引用类型才有属性或者方法，基本类型理论上没有属性或方法，而字符串又属于基本类型，但为什么字符串能调用一些属性和方法呢？
原因是 js 为了方便对字符串进行操作，ECMA 提供了一个基本包装类型 String 对象。它是一种特殊的引用类型， 当 js 引擎需要读取或操作一个字符串时，他会在内部创建一个 String 类型的包装对象实例，然后调用这个实例的属性或方法并返回结果后，再立即清除这个实例。
这也是为什么 字符串明明是一个基本类型 却能调用属性和方法的原因。

### **几个 Unicode 问题总结**

#### 基本概念

Unicode 是目前最常见的字符编码，它用一个码位映射一个字符。在 js 中，Unicode 码位范围为 `'\u{0000}'` ~ `'\u{10ffff}'` ，可以表示超过 110 万个字符。格式为 `'\u{十六进制数字}'`

```
console.log('\u{0041}') // 'A'
console.log('\u{0061}') // 'a'
console.log('I \u{2661} Hagan') // 'I ♡ Hagan'
console.log('\u{20bb7}') // '𠮷'
```

Unicode 最前面的 65536 个字符位称为 基本多文种平面，它的码位范围为 `'\u{0000}'` ~ `'\u{ffff}'` ，最常见的字符都放在这个平面上。
剩下的字符都放在 辅助平面 上，码位范围为 `'\u{010000}'` ~ `'\u{10ffff}'`
判断是否为辅助平面的方法为十六进制数字的位数是否超过 4 位。

#### 字符串长度问题

1. 解决代理对长度问题

在内部，JavaScript 将辅助平面内的字符表示为代理对，并将单独的代理对分开为单独的 “字符”，所以代理对的 length 属性可能与我们的预期不一致，比如：

```
const str = '\u{20BB7}'
console.log(str) // '𠮷'
console.log(str.length) // 2
```

而我们想获取的长度应该为 1，这里可以使用以下方法正确获取长度

```
const str = '\u{20BB7}'
console.log(Array.from(str).length) // 1
```

2. 解决组合标记长度问题

`\u{0307}` 表示 q̣̇ 上面的点， `\u{0323}` 表示 q̣̇ 下面的点，这三个字符共同组成了一个 q̣̇ ，如下代码

```
const str = 'q\u{0307}\u{0323}'
console.log(str) // `q̣̇`
console.log(str.length) // 3
```

我们期待拿到的长度应该为 1，可实际拿到的 length 为 3，这里可以使用以下方法获取长度

```
const str = 'q\u{0307}\u{0323}'
const regex = /(\P{Mark})(\p{Mark}+)/gu
const trim = str.replace(regex, ($0, $1, $2) => $1)
console.log(Array.from(str).length) // 1
```

3. 将以上代码封装起来，可封装成以下方法

_注意：此方法无法处理表情字符序列组合_ '👨‍👩‍👧‍👦'

```
const getStringLength = function (string) {
  const regex = /(\P{Mark})(\p{Mark}+)/gu
  const str = string.replace(regex, ($0, $1, $2) => $1)
  return Array.from(str).length
}
export default getStringLength
```

#### 字符串反转

_注意：此方法无法正确处理组合标记_

```
const getReverseString = function (string) {
  return Array.from(string).reverse().join('')
}
export default getReverseString
```

一位名叫 Missy Elliot 的聪明的计算机科学家提出了一个防弹算法来解决组合标记问题

#### 根据码位获取字符串

_注意：此方法无法正确处理组合标记_

```
String.fromCodePoint(0x20bb7) // '𠮷'
```

#### 根据字符串获取码位

_注意：此方法无法正确处理组合标记_

```
'𠮷'.codePointAt().toString(16) // 20bb7
```

#### 遍历字符串

_注意：此方法无法正确处理组合标记_

```
for (const item of '𠮷') {
  console.log(item) // '𠮷'
}
```

## Number

### **储存结构**

js 采用 IEEE754 标准中的 双精度浮点数来表示一个数字，标准规定双精度浮点数采用 64 位存储，即 8 个字节表示一个浮点数。存储结构如下图：
![](https://cdn.nlark.com/yuque/0/2021/webp/12838787/1630915226481-43e8fc2c-b848-434d-8c84-9c3acd0a497c.webp#align=left&display=inline&height=218&margin=%5Bobject%20Object%5D&originHeight=218&originWidth=786&size=0&status=done&style=none&width=786)
在双精度浮点数中，第一位的 1bit 符号位 决定了这个数的正负，指数部分的 11bit 决定数值大小，小数部分的 52bit 决定数值精度。
在 JS 中的二进制数据储存中，二进制前三位为 `010` 代表双精度数字

### **数值范围**

指数部分为 11bit 也就是 2^11 - 1 = 2047，取中间值进行偏移得到 [-1023, 1024]，因此这种存储结构能够表示的数值范围为 2^-1023 至 2^1024，超出这个范围无法表示。转换为科学计数法为：

> 2^-1023 = 5 × 10^-324
> 2^1024 = 1.7976931348623157 × 10^308

```
Number.MAX_VALUE // 1.7976931348623157e+308
Number.MIN_VALUE // 5e-324
```

### **安全整数**

IEEE754 规定，有效数字第一位默认总是 1，但它不保存在 64 位浮点数之中。所以有效数字为 52bit + 1 = 53bit。
这意味着 js 能表示并进行精确算术运算的安全整数范围为 [-2^53 -1, 2^53 - 1] 即 -9007199254740991 到最大值 9007199254740991 之间的范围。

```
Math.pow(2, 53) - 1 // 9007199254740991
-Math.pow(2, 53) - 1 // -9007199254740991
```

可以通过 Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER 来分别获取安全整数最大值和最小值。

```
console.log(Number.MAX_SAFE_INTEGER) // 9007199254740991
console.log(Number.MIN_SAFE_INTEGER) // -9007199254740991
```

对于超过这个安全整数的运算，需要使用 BigInt 来计算。

```
console.log(9007199254740991 + 2) // 9007199254740992
console.log(BigInt(9007199254740991) + BigInt(2)) // 9007199254740993n
```

### **精度丢失**

计算机中的数字都是用二进制储存的，如果要计算 0.1 + 0.2 那么计算机会分别把 0.1 和 0.2 转成二进制，然后相加，最后把相加的结果转为 10 进制。
但有一些浮点数转化为二进制时会出现无限循环，比如 0.1

> 0.0001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 10001 无限循环

而上面我们说过计算机能储存的小数位最多只有 53 位，为了尽可能的接近目标值，所以采用类似十进制四舍五入的方法，在二进制中 0 舍 1 入，最终 0.1 储存到计算机中成为以下数值。

> 0.0001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 101

**0.1** 转换成科学技术法

> (−1)^0 × 2^(-4) × (1.1001100110011001100110011001100110011001100110011010)2

**0.2** 科学技术法表示为

> (−1)^0 × 2^(-3) × (1.1001100110011001100110011001100110011001100110011010)2

在浮点数做加法时要先进行对位操作，将较小的指数转化为较大的指数，并将小数部分右移

> (−1)^0 × 2^(-3) × (0.111001100110011001100110011001100110011001100110011010)2
> (−1)^0 × 2^(-3) × (1.1001100110011001100110011001100110011001100110011010)2

最终 0.1 + 0.2 在计算机中的计算过程如下
![](https://cdn.nlark.com/yuque/0/2021/webp/12838787/1630915226886-12be20de-4f2d-4202-acfa-11f57cfea89e.webp#align=left&display=inline&height=350&margin=%5Bobject%20Object%5D&originHeight=350&originWidth=772&size=0&status=done&style=none&width=772)
计算后 0.1 + 0.2 的结果为

> (−1)^0 × 2^(−2) × (1.0011001100110011001100110011001100110011001100110100)2

然后通过 js 将二进制转为 10 进制

```
(-1)**0 * 2**-2 * (0b10011001100110011001100110011001100110011001100110100 * 2**-52) === 0.30000000000000004 // true
console.log(0.1 + 0.2) ; // 0.30000000000000004
```

这就是经典的 0.30000000000000004 问题，0.1 和 0.2 在转换为二进制时由于做了 0 舍 1 入 ，发生了一次精度丢失，而对于计算后的二进制又 做了一次 0 舍 1 入 发生一次精度丢失，因此得到的结果是不准确的。
**解决办法：**
精度丢失解决办法就是先将小数转换成整数，然后用整数进行计算得到结果后再转换为小数，用 js 封装方法如下
math.js

```
// 判断number是否为一个整数
const isInteger = function (number) {
  return Math.floor(number) === number
}
// 四舍五入
const toFixed = function (number, decimalLength = 0) {
  var times = Math.pow(10, decimalLength)
  var fixed = number * times + 0.5
  return parseInt(fixed) / times
}
// 将一个浮点数转成整数，返回整数和倍数
const toInteger = function (floatNumber) {
  const numberInfo = { times: 1, number: 0 }
  const isNegative = floatNumber < 0
  if (isInteger(floatNumber)) {
    numberInfo.number = floatNumber
    return numberInfo
  }
  const stringFloatNumber = String(floatNumber)
  const dotPosition = stringFloatNumber.indexOf('.')
  const length = stringFloatNumber.substr(dotPosition + 1).length
  numberInfo.times = Math.pow(10, length)
  numberInfo.number = toFixed(Math.abs(floatNumber) * numberInfo.times)
  if (isNegative) numberInfo.number = -numberInfo.number
  return numberInfo
}
// 加
export const add = function (number1, number2, decimalLength = 0) {
  const { number: num1, times: times1 } = toInteger(number1)
  const { number: num2, times: times2 } = toInteger(number2)
  const maxTimes = Math.max(times1, times2)
  let result
  if (times1 === times2) result = (num1 + num2) / maxTimes
  if (times1 > times2) result = (num1 + num2 * (times1 / times2)) / maxTimes
  if (times1 < times2) result = (num1 * (times2 / times1) + num2) / maxTimes
  return toFixed(result, decimalLength)
}
```

```
import { add } from './math.js'
console.log(add(0.1, 0.2, 1)) // 0.3
```

### **特殊数值变量**

JavaScript 提供了几个特殊数值，用于判断数字的边界和其他特性

```
Number.MAX_VALUE // JavaScript 中的最大值
Number.MIN_VALUE // JavaScript 中的最小值
Number.MAX_SAFE_INTEGER // 最大安全整数，为 2^53 - 1
Number.MIN_SAFE_INTEGER // 最小安全整数，为 -(2^53 - 1)
Number.POSITIVE_INFINITY // 对应 Infinity，代表正无穷
Number.NEGATIVE_INFINITY // 对应 -Infinity，代表负无穷
Number.EPSILON // 是一个极小的值，用于检测计算结果是否在误差范围内
Number.NaN // 表示非数字，NaN与任何值都不相等，包括NaN本身
Infinity // 表示无穷大，分 正无穷 Infinity 和 负无穷 -Infinity
```

### **四舍五入**

有时我们需要对一些数字进行四舍五入，而这些数字可能包含小数
Math.round(number) 方法无法对小数进行计算
Number.toFixed() 方法实际上采用四舍六入五成双的规则实现，存在一些缺陷，具体可看一下这篇文章
以上两个方法有时候无法满足我们的需求，所以封装以下方法
math.js

```
// 四舍五入
export const toFixed = function (number, decimalLength = 0) {
  var times = Math.pow(10, decimalLength)
  var fixed = number * times + 0.5
  return parseInt(fixed) / times
}
```

```
import { toFixed } from './math.js'
toFixed(0.2286298683746, 3) // 0.229
```

## Boolean

### **储存结构**

在 JS 中的二进制数据储存中，二进制前三位为 `110` 代表布尔值

### **基本概念**

Boolean 只有两个类型，true、false。在 js 中所有类型的值都能转换成 Boolean 值。如下代码

```
Boolean('') // false // 除了空字符串意外，其他字符串都为true
Boolean(0 || NaN) // false // 除了0与NaN，其他数字都为true
Boolean(undefined) // false
Boolean(Symbol()) // true
Boolean(null) // false
Boolean({} && []) // true // 所有引用类型都为true
```

### **隐式类型转换**

当使用以下操作符获取 Boolean 结果时，在过程中 js 内部会先进行隐式类型转换，再使用转换后的结果进行对比，最终确定为 true、 false。

> `>` `>=` `<` `<=` `==` `!=` `if else` `while`

```
4 > 3 // 4 > 3 // true
'4' > 3 // Number('4') > 3 // 4 > 3 // true
'a' > 'b' // 'a'.codePointAt() > 'b'.codePointAt() // 97 > 98 // false
true > 2 // Number(true) > 2 // 1 > 2 // false
undefined > 0 // Number(undefined) > 0 // NaN > 0 // false
null > 0 // Number(null) > 0 // NaN > 0 // false
new Date() > 100 // new Date().valueOf() > 100 // 1587608237665 > 100 // true
if (1) { } // Boolean(1) // true
if (!(1 > '10')) { } // !(1 > Number('10')) // !(1 > 10) //!false // true
```

### **显式类型转换**

与隐式类型转换相对应，当使用 `!` 操作符时，会将变量强制转换为 Boolean 值，并进行取反操作。

```
!1 // false
!!1 // true
!undefined // true
!!undefined // false
```

### **全等操作符**

> `===` `!==`

全等操作符在进行对比出 Boolean 结果时不会进行隐式类型转换，为了避免因隐式类型转换带来的预期之外的情况，推荐在实际项目中使用全等操作符来进行对比。

```
'1' == 1 // Number('1') == 1 // 1 == 1 // true
'1' === 1 // false
true != 1 // Number(true) != 1 // 1 != 1 // false
true !== 1 // true
```

### **逻辑操作符**

以下两个操作符为逻辑操作符

> `&&` `||`

```
1 && 2 // 2 // 取最后一个为 true 的值
1 || 2 // 1 // 取第一个为 true 的值
```

## Symbol

### **储存结构**

这是 ES6 新增的一种数据类型，它的字面意思为，符号，标记。代表独一无二的值。

### **基本概念**

Symbol 类型可以作为对象的 key 值。Symbol 最常用的方式就是作为唯一 id。

```
class Person {
  constructor (name, age) {
    this.name = name
    this.age = age
    this.id = Symbol('身份证号')
  }
}
const hagan1 = new Person('hagan', 25)
hagan1.id // Symbol(身份证号)
const hagan2 = new Person('hagan', 24)
hagan2.id // Symbol(身份证号)
hagan1.id === hagan2.id // false // 身份证号永远唯一
```

Symbol 也可以当成私有变量来使用，但它并不是真的为私有。
Girl.js

```
const _age = Symbol('女生的年龄')
export default class {
  constructor (name, age) {
    this.name = name
    this[_age] = age // 女生实际年龄
  }
  getAge () {
    return 18 // 女生告诉你的年龄
  }
}
```

rita.js

```
import 'Girl' from './Girl.js'
const rita = new Girl('rita', 28)
// 女生实际年龄只有她自己知道，而你毫无办法，因为女生永远 18 岁
rita.age // undefined
rita[Symbol('女生的年龄')] // undefined
rita.getAge() // 18
// 除非你开挂
const [ _age ] = Object.getOwnPropertySymbols(rita)
rita[_age] // 36
```

Symbol 作为对象的属性，不会被 `for in` `for of` 循环到，也不会被 `Object.keys()` `Object.getOwnPropertyNames()` `JSON.stringify()` 返回，但是它也不是任何办法都无法访问， `Object.getOwnPropertySymbols(object)` 方法能够获取到对象的所有 Symbol 类型的属性名。

### **Symbol.for()**

`Symbol.for()` 也可以生成 Symbol 值，他与直接调用 `Symbol()` 唯一的区别就是，`Symbol.for()` 生成的值不能作为唯一 id

```
const hagan1 = Symbol.for('hagan')
const hagan2 = Symbol.for('hagan')
hagan1 === hagan2 // true
```

### **Symbol.keyFor()**

返回通过 `Symbol.for()` 方法创建的 Symbol 类型的 key 值

```
const hagan = Symbol.for('hagan')
Symbol.keyFor(hagan) // 'hagan'
```

## Undefined

### **储存结构**

基本类型之一的 Undefined 只拥有一个值 `undefined` ，代表未定义的值。

```
let name
console.log(name) // undefined
const hagan = { }
console.log(hagan.job) // undefined
const arr = [ ]
console.log(arr[0]) // undefined
(function (a, b) {
  console.log(a) // 1
  console.log(b) // undefined
})(1, 2)
```

凡是未被定义和赋值的变量、属性或参数，都默认为 `undefined`

## Null

### **储存结构**

在 JS 中的二进制数据储存中，Null 的全部数位都为 0

### **基本概念**

基本类型之一的 Null 只拥有一个值 `null`，代表空值。表示一个变量被人为重置为空对象，在内存中的表示就是栈中的变量即不是其他 5 中基本类型，也没有引用类型中指向堆中的指针。当一个引用类型变量被赋值为 `null` 时，原来的引用类型对象在堆中处于游离状态，GC 会择机回收该对象并释放内存。因此想要回收哪个变量。就将它设为 `null` 就好了

### **为什么 typeof null 会被判断为 object**

在 JS 中。数据在底层都是以二进制储存，引用类型的二进制前三位为 `0`，`typeof` 是根据这个特性来进行判断类型的工作，可这里有一个问题就是，`null` 类型所有位数都为`0`，所以它的前三位也为 `0` ，所以 `null` 会被判断为 `object`

## Object

### **基本概念**

Object 类型 也叫引用类型，是一组没有特定顺序的值的集合。

### **储存结构**

在 JS 中，基本类型的实际值储存在 **栈** 中，而引用类型的实际值储存在 **堆** 中。栈中储存的只有指向到堆中的 **指针**，这也是 Object 类型也被称为 **引用类型** 的原因。
在 JS 中的二进制数据储存中，二进制前三位为 `000` 代表引用类型

### **JS 中的堆内存与栈内存**

请看下面的代码

```
const num = 1
console.log(num) // 1
num = 2 // 报错
const hagan = { name: 'hagan' }
console.log(hagan) // { name: 'hagan' }
hagan.name = 'han'
console.log(hagan) // { name: 'han' }
```

const 不是定义常量么？为什么还能改？这时候就要涉及到 JS 中的堆内存与栈内存了
在 js 引擎中对变量的存储主要有两种位置，**堆内存和栈内存**。
**栈内存**主要用于存储各种**基本类型的**变量，包括 Boolean、Number、String、Undefined、Null，以及对象变量的指针。
而**堆内存**主要负责像对象 Object 这种变量类型的存储，如下图
![](https://cdn.nlark.com/yuque/0/2021/webp/12838787/1630915226568-ad23f5cc-7e68-4c6f-899c-aaadd33ea7b8.webp#align=left&display=inline&height=482&margin=%5Bobject%20Object%5D&originHeight=482&originWidth=674&size=0&status=done&style=none&width=674)
引用类型的数据的地址指针是存储于栈中的，当我们想要访问引用类型的值的时候，需要先从栈中获得对象的地址指针，然后在通过地址指针找到堆中的所需要的数据。
因此当我们定一个 `const` 常量时，不可改变的只是 **栈内存** 中的数据，但 **堆内存** 中的数据还是可以通过变量的引用进行改变的。
总结：**栈内存** 用来储存基本类型，以及引用类型的指针，**堆内存** 用来储存引用类型数据。

### **对象拷贝**

#### 深拷贝

```
const hagan = { name: 'hagan', age: 25 }
const haganDeepClone = JSON.parse(JSON.stringify(hagan))
console.log(haganDeepClone === hagan) // false
```

#### 浅拷贝

```
const hagan = { name: 'hagan', age: 25 }
const haganClone = Object.assign({}, hagan)
console.log(haganClone === hagan) // false
```

### **数据属性与访问器属性**

#### 数据属性

```
const hagan = { age: 22 }
Object.defineProperty(hagan, 'name', {
  configurable: false, // 能否通过delete删除，调用defineProperty前默认为true，调用后默认为false
  enumerable: false, // for in 能否循环到，调用defineProperty前默认为true，调用后默认为false
  writable: false, // 是否可修改，调用defineProperty前默认为true，调用后默认为false
  value: 'hagan' // 属性值，默认为undefined
})
delete hagan.name
console.log(hagan.name) // 'hagan'
for (let attr in hagan) {
    console.log(hagan[attr]) // 22
}
hagan.name = 'rita'
console.log(hagan.name) // 'hagan'
```

#### 访问器属性

```
const hagan = { age: 25 }
Object.defineProperty(hagan, 'name', {
    get () {
        return this._name
    },
    set (value) {
        this._name = value
    }
})
hagan.name = 'hagan'
console.log(hagan) // { age: 25, _name: "hagan" }
console.log(hagan.name) // 'hagan'
```

### **一些 Object 方法整理**

#### Object.create()

以第一个参数为原型，创建新对象。可用于原型继承

```
function Animal (type) {
  this.type = type
}
Animal.prototype.getType = function () {
  return this.type
}
function People (name) {
  Animal.call(this, 'people') // 继承属性
  this.name = name
}
People.prototype = Object.create(Animal.prototype)
People.prototype.getName = function () {
  return this.name
}
const hagan = new People('hagan')
hagan.getName() // 'hagan'
hagan.getType() // 'people'
```

#### Object.defineProperty()

用于定义对象的数据属性和访问器属性

```
const hagan = { age: 25 }
Object.defineProperty(hagan, 'name', {
    get () {
        return this._name
    },
    set (value) {
        this._name = value
    }
})
hagan.name = 'hagan'
console.log(hagan) // { age: 25, _name: "hagan" }
console.log(hagan.name) // 'hagan'
```

#### Object.defineProperties()

用于定义对象的数据属性和访问器属性

```
const hagan = { age: 25 }
Object.defineProperties(hagan, {
  name: {
    value: 'hagan'
  }
})
hagan.name = 'hagan'
console.log(hagan) // { age: 25, _name: "hagan" }
console.log(hagan.name) // 'hagan'
```

#### Object.getOwnPropertyDescriptor()

获取对象某属性的数据属性或访问器属性

```
const hagan = { age: 22 }
Object.defineProperty(hagan, 'name', {
    value: 'hagan'
})
Object.getOwnPropertyDescriptor(hagan, 'name') // {value: "hagan", writable: false, enumerable: false, configurable: false}
```

#### Object.getOwnPropertyNames()

获取所有的属性名并返回一个数组，不包含原型链

```
const hagan = { age: 22 }
Object.defineProperty(hagan, 'name', {
  configurable: false, // 能否通过delete删除，调用defineProperty前默认为true，调用后默认为false
  enumerable: false, // for in 能否循环到，调用defineProperty前默认为true，调用后默认为false
  writable: false, // 是否可修改，调用defineProperty前默认为true，调用后默认为false
  value: 'hagan' // 属性值，默认为undefined
})
Object.getOwnPropertyNames(hagan) // ["age", "name"]
```

#### Object.keys()

获取所有的可枚举属性名并返回一个数组，不包含原型链

```
const hagan = { age: 22 }
Object.defineProperty(hagan, 'name', {
  configurable: false, // 能否通过delete删除，调用defineProperty前默认为true，调用后默认为false
  enumerable: false, // for in 能否循环到，调用defineProperty前默认为true，调用后默认为false
  writable: false, // 是否可修改，调用defineProperty前默认为true，调用后默认为false
  value: 'hagan' // 属性值，默认为undefined
})
Object.keys(hagan) // ["age"]
```

#### Object.preventExtensions()

使对象不能添加属性，但属性的值可以删除和修改

```
const hagan = { age: 22 }
Object.preventExtensions(hagan)
hagan.name = 'hagan'
console.log(hagan.name) // undefined
```

#### Object.seal()

使对象不能添加属性，也不可以删除，但可以修改

#### Object.freeze()

使对象不能添加属性，也不可以删除和修改
