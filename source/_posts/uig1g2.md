---
title: 一些常用的代码片段
urlname: uig1g2
date: '2021-09-06 14:17:02 +0800'
tags: []
categories: []
---

## 1. 下载一个 excel 文档

同时适用于 word,ppt 等浏览器不会默认执行预览的文档,也可以用于下载后端接口返回的流数据，见`3`

```javascript
//下载一个链接
function download(link, name) {
  if (!name) {
    name = link.slice(link.lastIndexOf("/") + 1);
  }
  let eleLink = document.createElement("a");
  eleLink.download = name;
  eleLink.style.display = "none";
  eleLink.href = link;
  document.body.appendChild(eleLink);
  eleLink.click();
  document.body.removeChild(eleLink);
}
//下载excel
download("http://111.229.14.189/file/1.xlsx");
复制代码;
```

## 2. 在浏览器中自定义下载一些内容

场景：我想下载一些 DOM 内容，我想下载一个 JSON 文件

```javascript
/**
 * 浏览器下载静态文件
 * @param {String} name 文件名
 * @param {String} content 文件内容
 */
function downloadFile(name, content) {
  if (typeof name == "undefined") {
    throw new Error("The first parameter name is a must");
  }
  if (typeof content == "undefined") {
    throw new Error("The second parameter content is a must");
  }
  if (!(content instanceof Blob)) {
    content = new Blob([content]);
  }
  const link = URL.createObjectURL(content);
  download(link, name);
}
//下载一个链接
function download(link, name) {
  if (!name) {
    //如果没有提供名字，从给的Link中截取最后一坨
    name = link.slice(link.lastIndexOf("/") + 1);
  }
  let eleLink = document.createElement("a");
  eleLink.download = name;
  eleLink.style.display = "none";
  eleLink.href = link;
  document.body.appendChild(eleLink);
  eleLink.click();
  document.body.removeChild(eleLink);
}
复制代码;
```

使用方式：

```
downloadFile('1.txt','lalalallalalla')
downloadFile('1.json',JSON.stringify({name:'hahahha'}))
复制代码
```

## 3. 下载后端返回的流

数据是后端以接口的形式返回的,调用`1`中的 download 方法进行下载

```
download('http://111.229.14.189/gk-api/util/download?file=1.jpg')
 download('http://111.229.14.189/gk-api/util/download?file=1.mp4')
复制代码
```

## 4. 提供一个图片链接，点击下载

图片、pdf 等文件，浏览器会默认执行预览，不能调用 download 方法进行下载，需要先把图片、pdf 等文件转成 blob，再调用 download 方法进行下载，转换的方式是使用 axios 请求对应的链接

```javascript
//可以用来下载浏览器会默认预览的文件类型，例如mp4,jpg等
import axios from "axios";
//提供一个link，完成文件下载，link可以是  http://xxx.com/xxx.xls
function downloadByLink(link, fileName) {
  axios
    .request({
      url: link,
      responseType: "blob", //关键代码，让axios把响应改成blob
    })
    .then((res) => {
      const link = URL.createObjectURL(res.data);
      download(link, fileName);
    });
}
复制代码;
```

注意：会有同源策略的限制，需要配置转发

## 5. cleanObject

去除对象中 value 为空(null,undefined,'')的属性,举个栗子：

```javascript
let res = cleanObject({
  name: "",
  pageSize: 10,
  page: 1,
});
console.log("res", res); //输入{page:1,pageSize:10}   name为空字符串，属性删掉
复制代码;
```

使用场景是：后端列表查询接口，某个字段前端不传，后端就不根据那个字段筛选，例如`name`不传的话，就只根据`page`和`pageSize`筛选，但是前端查询参数的时候（vue 或者 react）中，往往会这样定义

```javascript
export default {
  data() {
    return {
      query: {
        name: "",
        pageSize: 10,
        page: 1,
      },
    };
  },
};
const [query, setQuery] = useState({ name: "", page: 1, pageSize: 10 });
复制代码;
```

给后端发送数据的时候，要判断某个属性是不是空字符串，然后给后端拼参数，这块逻辑抽离出来就是`cleanObject`，代码实现如下

```javascript
export const isFalsy = (value) => (value === 0 ? false : !value);
export const isVoid = (value) =>
  value === undefined || value === null || value === "";
export const cleanObject = (object) => {
  // Object.assign({}, object)
  if (!object) {
    return {};
  }
  const result = { ...object };
  Object.keys(result).forEach((key) => {
    const value = result[key];
    if (isVoid(value)) {
      delete result[key];
    }
  });
  return result;
};
复制代码;
```

```javascript
let res = cleanObject({
  name: "",
  pageSize: 10,
  page: 1,
});
console.log("res", res); //输入{page:1,pageSize:10}
复制代码;
```

## 6. 休眠多少毫秒

```javascript
/**
 * 休眠xxxms
 * @param {Number} milliseconds
 */
export function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
//使用方式
const fetchData = async () => {
  await sleep(1000);
};
复制代码;
```

## 7.计算元素出现个数

```javascript
let names = ["张三", "李四", "张三", "王五", "王五", "王五"];
let total = names.reduce((pre, cur) => {
  if (cur in pre) {
    pre[cur]++;
    console.log("判断为真:");
    console.log(pre);
  } else {
    pre[cur] = 1;
    console.log("判断为假:");
    console.log(pre);
  }
  return pre;
}, {});
console.log(total); // => {张三: 2, 李四: 1, 王五: 3}
```

首先先传入一个{}对象，说明初始的 pre 为{}。那么第一轮判断 if 的时候就变成 `'张三' in {}` 很明显此时判断条件是 **false** 。所以就执行 **else** 里面的逻辑后变成:`{'张三':1}`。第二轮时 **李四** 也是如此。当第三轮时再次遇到“张三”,此时对象是 `{'张三':1,'李四':1}` ,所以 if 判断是 **true** ,所以张三直接+1。来看看打印情况:

```javascript
判断为假:
// => {张三: 1}
判断为假:
// => {张三: 1, 李四: 1}
判断为真:
// => {张三: 2, 李四: 1}
判断为假:
// => {张三: 2, 李四: 1, 王五: 1}
判断为真:
// => {张三: 2, 李四: 1, 王五: 2}
判断为真:
// => {张三: 2, 李四: 1, 王五: 3}
```