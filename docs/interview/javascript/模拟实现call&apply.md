---
title: 模拟实现call和apply
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# 模拟实现call和apply

在JavaScript中，call和apply都是为了`改变某个函数运行时的上下文(context)而存在的`，换句话说，`就是为了改变函数体内部this的指向`。

对于call和apply二者而言，作用完全一样，只是接受的参数不同。call需要把参数`按顺序传递`进去，而apply则是`把参数放在数组里`。如：

```js
var func = function(arg1, arg2) {};

func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2]);
```

JavaScript中，函数参数数量固定用call，不固定用apply，把参数push进数组传递进去。当参数数量不确定时，函数内部也可以通过arguments这个伪数组来遍历所有的参数。

#### 常见用法

##### 数组之间追加

```js
var array1 = [12, "foo", {name "Joe"}, -2458];
var array2 = ["Doe", 555, 100];
Array.prototype.push.applay(array1, array2);
console.log(array1);// [12, "foo", {name "Joe"}, -2458, "Doe", 555, 100]
```

##### 获取数组中的最大值和最小值

```js
var numbers = [5, 458, 120, -215];
// apply
var maxInNumber = Math.max.apply(Math, numbers); 
// call
var maxInNumber = Math.max.call(Math, 5, 458, 120, -215);
console.log(maxInNumber) // 458
```

##### 验证是否是数组（前提是toString()方法没有被重写）

```js
functionisArray(obj) {
  return Object.prototype.toString.call(obj) === '[object Array]'
}
```



#### 模拟实现call

1. 判断当前`this`是否为函数，防止`Function.prototype.myCall()`直接调用
2. `context`为可选参数，如果不传的话默认上下文为`window`
3. 为`context`创建一个`Symbol`属性，将当前函数赋值给这个属性
4. 处理参数，传入第一个参数后的其余参数
5. 调用函数后即删除该`Symbol`属性

```js
Function.prototype.myCall = function (context = window, ...args) {
  // 判断当前this是否为函数
  if (this === Function.prototype) {
    return undefined;
  }
  // context为可选参数，如果不传的话默认上下文为window
  context = context || window;
  // 为context创建一个Symbol属性，将当前函数赋值给这个属性
  const fn = Symbol();
  context[fn] = this;
  // 处理参数，传入第一个参数后的其余参数
  const result = context[fn](...args);
  // 调用函数后即删除该Symbol属性
  delete context[fn];
  return result;
}
```

#### 模拟实现apply

```js
Function.prototype.myApply = function (context = window, args) {
  if (this === Function.prototype) {
    return undefined;
  }
  const fn = Symbol();
  context[fn] = this;
  let result;
  if (Array.isArray(args)) {
    result = context[fn](...args);
  } else {
    result = context[fn]();
  }
  delte context[fn];
  return result;
}
```

