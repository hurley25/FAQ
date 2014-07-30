---
layout: post
category: question
title: 答题赢公仔系列——实现CountDownLatch数据结构
tagline: by 浅奕
tags: [network, Linux]
---

## 问题描述

有这样一个需求，M个线程要等待若N个线程完成任务后才能继续（M、N不定），那么请设计一种数据结构，使用mutex，condtion等机制实现它，使得这些线程持有这个数据结构很容易实现这种同步。（在自己动脑想之前，请不要百度CountDownLatch）
