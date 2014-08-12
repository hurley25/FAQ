---
layout: post
category: question
title: Linux如何打印函数调用栈
tagline: by 浅奕
tags: [network, Linux]
---

## 问题描述

大家知道Java抛出异常的时候能打印出函数调用栈，那我们的C/C++程序想实现的话怎么做呢？gdb能做到，那我们自己的代码如何实现呢？

提示：有个系统调用叫 backtrace（如果你自己实现了这个函数，那就碉堡了）

意义：理解ELF文件格式，文件操作等等。