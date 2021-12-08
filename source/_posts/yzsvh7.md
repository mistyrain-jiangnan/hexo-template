---
title: 数组 Array方法
urlname: yzsvh7
date: '2021-09-03 15:05:24 +0800'
tags: []
categories: []
---

## 数组去重

```javascript
function noRepeat(arr) {
  return [...new Set(arr)];
}
```

## 查找数组最大

```javascript
function arrayMax(arr) {
  return Math.max(...arr);
}
```

## 查找数组最小

```javascript
function arrayMin(arr) {
  return Math.min(...arr);
}
```

## 返回已 size 为长度的数组分割的原数组

```javascript
function chunk(arr, size = 1) {
  return Array.from(
    {
      length: Math.ceil(arr.length / size),
    },
    (v, i) => arr.slice(i * size, i * size + size)
  );
}
```

## 检查数组中某元素出现的次数

```javascript
function countOccurrences(arr, value) {
  return arr.reduce((a, v) => (v === value ? a + 1 : a + 0), 0);
}
```

## 扁平化数组

- 默认 depth 全部展开

```javascript
function flatten(arr, depth = -1) {
  if (depth === -1) {
    return [].concat(
      ...arr.map((v) => (Array.isArray(v) ? this.flatten(v) : v))
    );
  }
  if (depth === 1) {
    return arr.reduce((a, v) => a.concat(v), []);
  }
  return arr.reduce(
    (a, v) => a.concat(Array.isArray(v) ? this.flatten(v, depth - 1) : v),
    []
  );
}
```

## 对比两个数组并且返回其中不同的元素

```javascript
function diffrence(arrA, arrB) {
  return arrA.filter((v) => !arrB.includes(v));
}
```

## 返回两个数组中相同的元素

```javascript
function intersection(arr1, arr2) {
  return arr2.filter((v) => arr1.includes(v));
}
```

## 从右删除 n 个元素

```javascript
function dropRight(arr, n = 0) {
  return n < arr.length ? arr.slice(0, arr.length - n) : [];
}
```

## 截取第一个符合条件的元素及其以后的元素

```javascript
function dropElements(arr, fn) {
  while (arr.length && !fn(arr[0])) arr = arr.slice(1);
  return arr;
}
```

## 返回数组中下标间隔 nth 的元素

```javascript
function everyNth(arr, nth) {
  return arr.filter((v, i) => i % nth === nth - 1);
}
```

## 返回数组中第 n 个元素

- 支持负数

```javascript
function nthElement(arr, n = 0) {
  return (n >= 0 ? arr.slice(n, n + 1) : arr.slice(n))[0];
}
```

## 数组乱排

```javascript
function shuffle(arr) {
  let array = arr;
  let index = array.length;
  while (index) {
    index -= 1;
    let randomInedx = Math.floor(Math.random() * index);
    let middleware = array[index];
    array[index] = array[randomInedx];
    array[randomInedx] = middleware;
  }
  return array;
}
```

## 数组去重

```javascript
const numbers = [2, 3, 4, 4, 2];
console.log([...new Set(numbers)]); // [2, 3, 4]
```
