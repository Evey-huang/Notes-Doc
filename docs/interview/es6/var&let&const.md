---
title: var/let/const 区别
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# **var、let和const 区别**

## 简介

`let`用法类似于`var`，但是`let`只在所在的代码块内执行有效，所以我们一般使用let替代`var`，而`const`用来声明常量。

## 变量提升

`变量提升`就是变量可在声明之前使用。

###### var

```js
console.log(a); // 正常运行，控制台输出undefined
var a = 1;
```

###### let

```js
console.log(b); // 报错 Uncaught ReferenceError: b is not defined
let b = 1;
```

###### const

```js
console.log(c); // 报错 Uncaught ReferenceError: Cannot access 'c' before initialization
const c = 1;
```

`var`命令经常会发生`变量提升`现象，`ES6规定let和const不发生变量提升`，使用let和const命令声明变量之前，该变量都是不可用的。主要是为了减少运行时错误，防止变量声明前就使用这个变量，从而导致意料之外的行为。

## 暂时性死区

`暂时性死区`就是如果代码块中存在`let`或`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域，凡是在声明之前就使用这些变量，就会报错。

```js
var tmp = 123;
if (true) {
  tmp = 'abc'; // 报错？本地console并没有报错 输出的是abc
  let temp;
}
```

这段代码的愿意是在if内定义外部的`tmp`为abc，但现实是，存在全局变量`tmp`，但是块级作用域内`let`又声明了一个`tmp`变量，导致后者被绑定在这个块级作用域中，所以`let`声明变量前，对`tmp`赋值就报错。

## 重复声明

`重复声明`指的是在相同作用域内，重复声明同一个变量。`let`和`const`命令声明的变量不允许重复声明。

```js
function func() {
  let a = 10;
  const PI = 3.1415;
  
  var a = 1; // 报错，Uncaught SyntaxError: Identifier 'a' has already been declared
  var PI = 3; // 报错，Uncaught SyntaxError: Identifier 'PI' has already been declared
}
```

`var`允许重复定义，但是会产生麻烦

```js
var i = 10;
for (var i = 0;i < 5; i++) {
  console.log(i);
}
console.log(i); // 5
```

如果这段代码替换成其他静态语言，输出结果应该是10，但是JavaScript输出5，因为`var`命令没有块级作用域，所有for循环括号内的变量`i`会覆盖外层i，而且`var`允许重复声明，这段代码中的`i`被声明了2次，`i`最终结果就被for循环的i给覆盖了。

## 初始化值

由于`const`声明的是只读的常量，一旦声明，就必须立即初始化，声明之后值不能改变。

```js
const PI = 3.1415;
PI = 3; // 报错，Uncaught TypeError: Assignment to constant variable.
```

## 作用域

ES5中只有全局作用域和函数作用域，没有块级作用域。这带来很多不合理的场景。

- 内层变量可能会覆盖外层变量

```js
var tmp = new Date();
function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world'
  }
}
f(); // undefined
```

上述代码原意是，`if`代码块的外部使用外层的`tmp`变量，内部使用内层的`tmp`变量。

然而现实中是，function内部的两个`tmp`变量处于同一函数作用域，由于`变量提升`导致函数作用域中的`tmp`覆盖全局作用域中的`tmp`，所以f()输出结果为undefined

- 用来计数的循环变量会泄露为全局变量

```js
var i = 10;
for(var i = 0; i< 5; i++) {
  console.log(i);
}
console.log(i); // 5
```

