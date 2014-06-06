---
layout: post
category: note
title: WordPress加载缓慢原因分析
tagline: by Jensyn
tags: [web, google]
---
## 问题描述
最近不知怎么回事，搭建在sae上的wordpress加载起来特别慢,本以为是sae服务器的问题，结果测试发现同在sae上的另一个应用却能够正常地访问。当时想着会不会是安装了某个不恰当的插件或是主题造成的结果。于是在小组群里请教，刘欢学长给出指示说可能是google资源被屏蔽造成的。在欢学长的指导下最终解决了问题。

## 解决思路

**分析原因的方法**

方法一：利用chrome或者是firefox自带的开发者工具（针对这两款浏览器，都可以用快捷键F12打开），选择network标签，可以查看当前网页加载的过程：包括哪个时间段加载哪些资源以及加载每个资源所用的时间等等。
针对本人所遇到的情况，结果显示确实是fonts.googleapis.com****之类的资源加载耗费了大多数的时间。
![image](https://github.com/ButBueatiful/dotvim/raw/master/screenshots/vim-screenshot.jpg)

方法二："wp有插件可以统计自己生成一个页面所用的时间，可以细化到php时间和mysql时间，再配合apache或者nginx和系统日志，基本上能解决各端的所有问题"（转自刘欢学长）

**闲话两句**

前端开发人员在设计CSS的时候，引用GOOLE CDN的资源的本意是为网页加速或者是美工的需求，但是因为在天朝,google经常被屏蔽的原因，这样的做法倒极有可能导致整个页面加载失败。关于这一点，这里引用某网友的一段话：

"I can give a good reason why google fonts is a bad idea: some countries are not Google-friendly. Just like trying to deal with Twitter and Facebook apis thrown carelessly all over the web; generally accomplishing nothing and usually called when not needed, google fonts is just another way to bog down the experience.

I can see the benefits of clouding, but not as a starting point for a self-hosted project framework. I find it weird that there's a downloaded font-squirrel font "Genericons" in the theme, but still it seemed necessary to include a call to the google fonts as well.

Anyway, thanks for the "Disable Google Fonts" plugin link Cyril, it also worked perfectly for me.

I added this comment just to point out a reason. I've seen a few others trying to figure this problem out, including adrelanos, but it doesn't seem to be understood why people might not want it. 
"

**具体过程描述**

既然整个过程主要卡在了google资源的加载中，最直接的方法就是替换相关的资源或者弃之不用（当然后者可能是一个不太理想的解决方法）。针对前一中方法，国内有360CDN(内容分发网络)可以代替GOOGLE CDN,所以可以直接把web源码中所有引用到GOOGLE CDN的资源全部替换为360CDN上的资源，简单粗暴的办法就是直接在源码根目录下查找包含"googleapis"的文件（终端下执行 grep -R "googleapis ./"），然后将"googleapis"一一替换为"useso"。

还有一种方法，就是根据网上所说的，把引用到的资源下载到本地，然后修改相关的代码即可。

经过以上的修改，网页基本上可以正常访问了。

## 参考文档

[网站加载速度变慢的原因和解决方法](http://www.zhuji91.com/wangzhanjiazaisudu)
[删除wordpress内置的googleapis在线字体 ](http://blog.motea.org/29.html)
[How to disable fonts.googleapis.com?](http://wordpress.org/support/topic/how-to-disable-fontsgoogleapiscom)
[CDN（内容分发网络）技术原理](http://kb.cnblogs.com/page/121664/)
[谷歌字体API使用教程](http://www.chinaz.com/web/2010/0604/117757.shtml)

