---
title: This
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# **this**

`this`的值是在执行的时候才能确认，定义的时候不能确认，因为`this`是执行上下文环境的一部分，而执行上下文需要在代码执行之前确定，而不是定义的时候。

- 情况1

```js
function foo() {
  console.log(this.a)
}
var a = 1;
foo(); // 1
```

- 情况2

```js
function fn() {
  console.log(this)
}
var obj = {fn: fn};
obj.fn(); //this -> obj
```

- 情况3

```js
function CreateJsPerson(name, age) {
  // this是当前类的一个实例p1
  this.name = name; // p1.name = name
  this.age = age; // p1.age = age
}
var p1 = new CreateJsPerson('Evey', 24)
```

- 情况4

```js
function add(c, d) {
  return this.a + this.b + c + d
}
var o = {a: 1, b: 3};
add.call(o, 5, 7); // 1+3+5+7=16
add.apply(o, [10, 20]); // 1+3+10+20=34
```

- 情况5

```js
<button id="btn1">箭头函数this</button>
<script type="text/javascript">
  let btn1 = document.getElementById('btn1');
  let obj = {
    name: 'Evey',
    age: 24,
    getName: function() {
      btn1.onclick = () => {
        console.log(this); // obj
      }
    }
  }
  obj.getName();
</script>
```

###### 解析：

- 对于直接调用foo来说，不管foo函数被放在哪里，this一定是window
- 对于obj.foo()来说，谁调用了函数，谁就是this，所以在这个场景下foo函数中的this就是obj对象
- 在构造函数模式中，类中(函数体中)出现的this.xxx = xxx中的this，是当前类的一个实例
- 在call、apply和bind中，this是第一个参数
- 箭头函数this指向：箭头函数没有自己的this，看其外层是否有函数，如果有，外层函数的this就是内部箭头函数的this，如果没有，则this就是window