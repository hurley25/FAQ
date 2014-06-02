---
layout: post
category: note
title: 如何设计应用层协议格式
tagline: by 浅墨
tags: [network, protocol]
---

**一般而言，应用层协议设计有四种常见方法：

- 包长度固定
- 每行采取特殊结束标记（例如HTTP使用的\r\n）
- 包前添加长度信息（所谓的TLV模式，type、length、value）
- 利用包本身的格式（如XML、JSON等）

**Ps. 还有诸如google的protobuf等跨语言的协议打包格式。**
