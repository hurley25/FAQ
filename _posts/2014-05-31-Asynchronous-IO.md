---
layout: post
category: note
title: 异步I/O
tagline: by 浅墨
tags: [network, c++, linux]
---

## 异步I/O

### POSIX.1对同步I/O和异步I/O的定义：

- 同步I/O操作导致发出请求的进程被阻塞直到I/O操作完成。

- 异步I/O操作在I/O操作期间不导致发出请求的进程被阻塞。

**根据这个定义，阻塞，非阻塞，多路复用，信号驱动均属于同步I/O(尽管信号驱动由于习惯原因，在以前被成为异步I/O)**
