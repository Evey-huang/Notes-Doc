---
title: Fragments
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---

# Fragments

React中的一个常见模式是`一个组件返回多个元素`。Fragments允许你将子列表分组，而无需向DOM添加额外节点。

```js
render() {
  return (
  	<React.Fragment>
    	<ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  )
}
```

## 短语法

可以使用一种新的且更简短的语法来声明Fragments，它看起来像`空标签`，你可以像使用任何其他元素一样使用`<></>`，除了它不支持`key`属性：

```js
class Columns extends React.Component {
  render() {
    return (
    	<>
        <td>Hello</td>
        <td>World</td>
      </>
    )
  };
}
```

