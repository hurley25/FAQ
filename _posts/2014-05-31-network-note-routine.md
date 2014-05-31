---
layout: post
category: note
title: 网络编程模式
tagline: by 浅墨
tags: [network, routine]
---

### 其实网络编程中有很多是事务性（routine）的工作，可以提取为公共的框架或库，而用户只要填上关键的业务逻辑代码，并将回调注册到框架中，就可以实现完整的网络服务。
