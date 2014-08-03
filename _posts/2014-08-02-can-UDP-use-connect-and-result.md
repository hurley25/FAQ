---
layout: post
category: question
title: 答题赢公仔系列——数据报(UDP)套接字是否能调用connect函数，使用后效果如何？
tagline: by 浅奕
tags: [network, Linux]
---

## 问题描述

数据报(UDP)套接字使用sendto发送数据，不需要调用connect函数先建立连接。那……能使用这个系统调用吗？使用后效果如何？
