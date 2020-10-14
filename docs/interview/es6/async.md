---
title: 异步解决方案-Await
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---

# Async/Await简介与用法

## 简介

- async/await
  - 写异步代码的新方式，优于回调函数和Promise
  - 基于Promise实现的，不能用于普通的回调函数
  - 非阻塞的
  - 使得异步代码看起来像同步代码，但是改变不了JS单线程、异步的本质

## 用法

- 使用await，函数必须用async标识
- await后面跟的是一个Promise实例
- 需要安装babel-polyfill，安装后记得引入`npm i --save-dev babel-polyfill`

```js
function loadImg(src) {
  const promise = new Promise(function(resolve, reject) {
    var img = document.createElement('img');
    img.onload = function() {
      resolve(img);
    }
    img.onerror = function() {
      reject();
    }
    img.src = src;
  })
  return promise; // 返回一个promise实例
}
const src1 = "http://www.imooc.com/static/img/index/logo_new.png";
const src2 = "https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg";
const load = async function() {
  const result1 = await loadImg(src1);
  console.log(result1);
  const result2 = await loadImg(src2);
  console.log(result2)
}
load();
```

`当函数执行的时候，一旦遇到await就会先返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。`

## 错误处理

await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。

```js
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch(err) {
    console.log(err)
  }
}
```

## Async/Await 处理异步优势

- 代码简洁

使用async/await明显节约很多代码，不需要写.then，不需要写匿名函数处理Promise的resolve值，也不需要定义多余的data变量，还避免了嵌套代码。

- 处理中间值更简单直观

比如有个场景，调用promise1，使用promise1返回的结果去调用promise2，然后使用两者的结果去调用promise3，代码如下：

```js
const makeRequest = () => {
  return promise1().then(value1 => {
    return promise2(value1).then(value2 => {
      return promise3(value1, value2)
    })
  })
}
```

使用async/await，代码如下：

```js
const makeRequest = async() => {
  const value1 = await promise1();
  const value2 = await promise2();
  return promise3(value1, value2);
}
```



- 提高条件语句的可读性

如下示例：需要获取数据，然后跟胡返回数据决定是直接返回还是继续获取更多的数据

```js
const makeRequest = () => {
  return getJSON().then(data => {
    if (data.needsAnotherRequest) {
      return makeAnotherRequest(data).then(moreData => {
        console.log(moreData);
        return moreData;
      })
    } else {
      console.log(data);
      return data;
    }
  })
}
```

async/await写法：

```js
const makeRequest = async() => {
  const data = await getJSON();
  if (data.needsAnotherRequest) {
    const moreData = await makeAnotherRequest(data);
    console.log(moreData);
    return moreData;
  } else {
    console.log(data);
    return data;
  }
}
```

