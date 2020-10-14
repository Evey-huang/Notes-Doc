---
title: rest参数
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---

# rest参数

ES6引入`rest参数(形式为...变量)`，用于获取函数的多余参数，这样就不需要使用arguments对象了。

`rest参数搭配的变量是一个数组，该变量将多余的参数放入数组中。`

例如，如何实现一个求和函数？

- 传统写法：

```js
function addNumbers(a, b, c, d, e) {
  var numbers = [a, b, c, d, e];
  return numbers.reduce((sum, number) => {
    return sum + number;
  }, 0)
}
console.log(addNumbers(1, 2, 3, 4, 5)); // 15
```

- rest写法：

```js
function addNumber(...numbers) {
  return numbers.reduce((sum, number) => {
    return sum + number;
  }, 0)
}
console.log(addNumber(1, 2, 3, 4, 5)); // 15
```

- rest+解构赋值：

```js
var array = [1, 2, 3, 4, 5, 6];
var [a, b, ...c] = array;
console.log(a); // 1
console.log(b); // 2
console.log(c); // [3, 4, 5, 6]
```

- rest+箭头函数：

```js
const numbers = (...nums) => nums;
numbers(1, 2, 3, 4, 5); // [1, 2, 3, 4, 5]
```

##### 注意⚠️

- 每个函数最多只能声明一个rest参数，而且rest参数必须是最后一个参数，否则报错
- rest参数不能用于对象字面量setter之中

```js
let object = {
  set name(...value) { // 报错
    // do something
  }
}
```

