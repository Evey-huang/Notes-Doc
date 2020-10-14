---
title: script/srcipt async/script defer的区别
order: 1
nav:
  order: 1
  title: 面试
toc: menu
---

# 请解释`<script>`、`<srcipt async>`和`<script defer>`的区别

- script：被异步加载后并不会立即执行，而是等待文档被解析完毕后执行
- defer：浏览器指示脚本在文档被解析后执行
- async：同样是异步加载脚本，区别是脚本加载完毕后立即执行，这导致async属性下的脚本是乱序的。`对于script有先后依赖关系的情况下，并不适用。`