---
layout: post
category: question
title: 答题赢公仔系列——解析出mp3文件的ID3信息
tagline: by 浅奕
tags: [network, Linux]
---

## 问题描述

mp3文件中有一段信息称之为ID3，记录了该mp3的歌手，标题，专辑名称，年代，风格等信息。试着写一个程序，能**递归的**对一个目录中的文件名杂乱的MP3文件根据ID3信息重命名。命名格式为“歌名-歌手.mp3”

涉及知识点：Linux文件和目录操作，MP3文件中ID3的格式。