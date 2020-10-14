---
title: React16 新生命周期
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# **react最新生命周期是怎样的**

React16之后有三个生命周期被废弃(`未被删除，计划17完全删除`)

- componentWillMount
- componentWillReceiveProps
- componentWillUpdate

目前React16.8+的生命周期分为三个阶段，分别是挂载阶段、更新阶段、卸载阶段

#### 挂载阶段

- constructor：构造函数，最先被执行，我们通常在构造函数里初始化`state对象`或者给自定义方法绑定`this`
- getDerivedStateFromProps：`static getDerivedStateFromProps(nextProps, prevState)`，这是个静态方法，当我们接收到新的属性想去修改我们state，可以使用getDerivedStateFromProps
- render：render函数是纯函数，只返回需要渲染的东西，不应该包含其他的业务逻辑，可以返回原生的DOM、React组件、Fragment、Portals、字符串和数字、Boolean和null等内容。
- componentDidMount：组件装载之后调用，此时我们可以获取到DOM节点并操作，比如对canvas、svg的操作，服务器请求、订阅都可以写在这个里面，但是记得在`componentWillUnmount`中取消订阅

#### 更新阶段

- getDerivedStateFromProps：此方法在更新和挂载阶段都可能会调用
- render：更新阶段也会触发此生命周期
- getSnapshotBeforeUpdate：`getSnapshotBeforeUpdate(prevProps, prevState)`，这个方法在render之后，componentDidUpdate之前调用，有两个参数prevProps和prevState，表示之前的属性和之前的state，这个函数有一个返回值，会作为第三个参数传给componentDidUpdate，如果你不想要返回值，可以返回null，此生命周期必须与componentDidUpdate搭配使用
- componentDidUpdate：`componentDidUpdate(preProps, prevState, snapshot)`，如果触发某些回调函数时需要用到DOM元素的状态，则将对比或计算的过程迁移至getSnapshotBeforeUpdate，然后在componentDidUpdate中统一触发回调或更新状态

#### 卸载阶段

- componentWillUnmount：当我们的组件被卸载或者销毁了就会调用，我们可以在这个函数里去清除一些定时器，取消网络请求，清理无效的DOM元素等垃圾清理工作