---
layout: post
title: 在Linux上提高音效
date: 2018-12-23
categories: blog
catalog: true
tags: [Linux]
description: 
---



>参考：https://bbs.deepin.org/forum.php?mod=viewthread&tid=137002

### 缘由
在linux上听音乐，插上耳机总感觉电脑把耳机当成音响来看待了，感觉音效没有在Win上好。感觉低音什么的插上耳机感受不出来。

### 解决方案

在网上找到了解决方案，是在深度社区找到的，说句题外话，深度社区绝对是国内最活跃的Linux社区。比那一些什么麒麟的什么操作系统的那些所谓的论坛，不知道高到哪里去了。

因为Linux上的驱动大部分都是集成在内核里的，不用装驱动，直接就能用。Linux自带的ALSA驱动一般保证声卡能响就行了。

那么，如何提高音效呢？

终端执行以下代码：

```linux
sudo apt install jackd
sudo apt install pulseaudio-module-jack
sudo apt install caps
```
好了，安装好后去试试音效吧。

### 其他
在Linux上听音乐有什么好的软件呢？
我推荐`CoCo Music`，

作者也是一个挺有趣的一个人。

官网[在这里](http://装逼.xyz)

这篇博文我是在Deepin上完成的。Deepin这个发行版真心好用。

![Deepin](https://i.loli.net/2019/01/07/5c331f5434953.png  "Deepin")

