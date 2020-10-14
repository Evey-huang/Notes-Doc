---
title: 模拟实现bind
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# 模拟实现bind

`bind()`方法会创建一个新函数，称为`绑定函数`，当调用这个绑定函数时，绑定函数会以创建它时传入bind()方法的第一个参数作为`this`，传入bind()方法的第二个以及第二个以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

#### 用例

1. 

```js
var bar = function() {
  console.log(this.x)
}
var foo = {
  x: 3
}
bar(); // undefined
var func = bar.bind(foo);
func(); // 3
```

这里创建了一个新的函数func，当使用bind()创建一个绑定函数之后，它被执行的时候，它的this会被设置成foo，而不是像我们调用bar()时的全局作用域。

2. 如果连续bind()两次或两次以上，第二次后面的bind()不生效

```js
var bar = function() {
  console.log(this.x)
}
var foo = {
  x: 3
}
var sed = {
  x: 4
}

var func = bar.bind(foo).bind(sed);
func(); // 3

var fiv = {
  x: 5
}
var func = bar.bind(foo).bind(sed).bind(fiv);
func(); // 3
```

在JavaScript中，多次bind()无效。`因为bind()的实现，相当于使用函数在内部包了一个call/apply，第二次bind()相当于再包住第一次bind()，故第二次以后的bind()是无法生效的。`

#### 模拟实现bind

1. 处理参数，返回一个闭包
2. 判断是否为构造函数调用，如果是则使用new调用当前函数
3. 如果不是，使用apply，将context和处理好的参数传入

```js
Function.prototype.myBind = function (context, ...args1) {
  // 判断当前this是否为函数，防止Function.prototype.myBind()直接调用
  if (this === Function.prototype) {
    throw new TypeError('Error');
  }
  const _this = this;
  return function F(...args2) {
    // 判断是否为构造函数
    if (this instanceof F) {
      return new _this(...args1, ...args2)
    }
    return _this.apply(context, args1.concat(args2))
  }
}
```

