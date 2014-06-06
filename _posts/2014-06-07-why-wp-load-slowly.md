---
layout: post
category: note
title: WordPress加载缓慢原因分析
tagline: by Jensyn
tags: [web, google]
---
## 问题描述
最近不知怎么回事，搭建在sae上的wordpress加载起来特别慢,本以为是sae服务器的问题，结果测试发现同在sae上的另一个应用却能够正常地访问。当时想着会不会是安装了某个不恰当的插件或是主题造成的结果。于是在小组群里请教，刘欢学长给出指示说可能是google资源被屏蔽造成的。在欢学长的指导下最终解决了问题。

## 解决方案

**原因分析**
方法一：利用chrome或者是firefox自带的开发者工具（针对这两款浏览器，都可以用快捷键F12打开），选择network标签，可以查看当前网页加载的过程：包括哪个时间段加载哪些资源以及加载每个资源所用的时间等等。
针对本人所遇到的情况，结果显示确实是fonts.googleapis.com****之类的资源加载耗费了大多数的时间。
![image](https://github.com/ButBueatiful/dotvim/raw/master/screenshots/vim-screenshot.jpg)

方法二："wp有插件可以统计自己生成一个页面所用的时间，可以细化到php时间和mysql时间，再配合apache或者nginx和系统日志，基本上能解决各端的所有问题"（转自刘欢学长）

**解决思路**
既然整个过程主要卡在了google资源的加载中，最直接的方法就是替换相关的资源或者弃之不用（当然后者可能是一个不太理想的解决方法）。针对前一中方法，国内有360CDN(内容分发网络)可以代替GOOGLE CDN,所以可以直接把web源码中所有引用到GOOGLE CDN的资源全部替换为360CDN上的资源，简单粗暴的办法就是直接在源码根目录下查找包含"googleapis"的文件（终端下执行 grep -R "googleapis ./"），然后将"googleapis"一一替换为"useso"。

还有一种方法，就是根据网上所说的，把引用到的资源下载到本地，然后修改相关的代码即可。

经过以上的修改，网页基本上可以正常访问了。

