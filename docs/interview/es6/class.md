---
title: Class和传统构造函数区别
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---

# Class和传统构造函数的区别

ES6提供了更接近传统语言的写法，引入了Class(类)这个概念，作为对象的模板，通过class关键字，可以定义类。但是类只是基于原型的面向对象模式的语法糖。

## 实现类

- 传统构造函数

```js
function MathHandle(x, y) {
  this.x = x;
  this.y = y;
}
MathHandle.prototype.add = function() {
  return this.x + this.y;
}
var m = new MathHandle(1, 2);
console.log(m.add()); // 3
```



- ES6

```js
class MathHandle {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  add() {
    return this.x + this.y;
  }
}
const m = new MathHandle(1, 2);
console.log(m.add()); // 3
```



## 实现继承

- 传统实现

```js
function Animal() {
  this.eat = function() {
    alert('Animal eat')
  }
}
function Dog() {
  this.bark = function() {
    alert('Dog bark')
  }
}
Dog.prototype = new Animal(); // 绑定原型，实现继承
var husky = new Dog();
husky.bark(); // Dog bark
husky.eat(); // Animal eat
```



- ES6

```js
class Animal {
  constructor(name) {
    this.name = name
  }
  eat() {
    alert(`${this.name} + eat`)
  }
}
class Dog extends Animal {
  constructor(name) {
    super(name); // 有extends必须要有super,它代表父类的构造函数，即Animal中的constructor
    this.name = name;
  }
  say() {
    alert(this.name + 'say')
  }
}
const dog = new Dog('哈士奇');
dog.say(); // 哈士奇 say
do g.eat(); // 哈士奇 eat
```



## 区别

- Class在语法上更贴合面向对象的写法
- Class实现继承更加易读、易理解，对初学者更加友好
- Class本质还是语法糖，使用prototype