---
title: React16之前不好的生命周期如何过渡
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# react16 之前的那些不好的生命周期怎么过度到 react16 的新生命周期

##### React16被废弃生命周期

- componentWillMount
- componentWillReceiveProps
- componentWillUpdate

##### 答

- componentWillMount -> state = {...}
- componentWillReceiveProps -> getDriverStateFromProps
- componentWillUpdate -> getSnapshotBeforeUpdate



