---
title: GET和POST的区别
order: 1
nav:
  order: 1
  title: 面试
toc: menu

---

# GET和POST的区别

`GET和POST方法没有实质区别`，只是报文格式不同。

- GET和POST只是HTTP协议中两种请求方式，而HTTP协议是基于TCP/IP的应用层协议，无论GET还是POST，用的都是同一个传输层协议，`所以在传输上没有区别`。
- 报文格式上
  - 不带参数时，区别只是第一行方法名不同
  - 带参数时，在约定中，GET方法的参数应该放在url中，POST方法参数应该放在body中。



