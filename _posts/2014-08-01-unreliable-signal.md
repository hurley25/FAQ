---
layout: post
category: question
title: 答题赢公仔系列——关于Unix信号
tagline: by 浅奕
tags: [network, Linux]
---

## 问题描述

Unix信号都是不可靠的吗？（不排队，可能丢失……）在服务端编程中，需要处理/忽略哪些信号呢？

意义：服务端编程必须考虑到信号对程序的影响，另外nginx也是依靠信号实现配置文件更新后热替换老进程的。