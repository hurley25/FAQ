---
layout: post
category: question
title: 采用非阻塞套接字加IO复用的方式实现一个服务器小程序
tagline: by 浅奕
tags: [network, Linux]
---

## 问题描述

自己定义一种网络消息格式，采用非阻塞套接字加IO复用的方式实现一个服务器小程序。客户端模拟各种情况发给服务端（半个半个发，中间sleep一下，两个拼一起发，甚至一个字节一个字节发），要求服务端不出错的解析出所有包。 
