---
title: 说一下Web Worker
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---

# 说一下Web Worker

Web Worker的作用，`就是为JavaScript创造多线程环境，允许主线程创建Worker线程，将一些任务分配给Worker运行`。在主线程运行的同时，Worker线程在后台运行，两者互不干扰。等到Worker线程完成计算任务，再把结果返回给主线程。这样的好处是，`一些计算密集型或高延迟的任务，被Worker线程负担了，主线程（通常负责UI交互）就会很流畅，不会被阻塞或拖慢`

