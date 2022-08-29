---
layout:     post
title:      VS Code报错：Build failed, do you want to continue? 解决方案
date:       2022-08-29
catalog: true
tags: [Java]
---

# VS Code报错：Build failed, do you want to continue? 解决方案

今天开学第一天，开始刷力扣了，用 `VSCode` 写 `Java` 时，编译运行后，右下角报错

```bash
Build failed, do you want to continue?
```

点击 `continue` 后，运行结果是正确的，但为什么会报这个错呢？

解决方案是，点击 `Fix` 按钮，打开日志文件，在右边缩略图中找到比较密集的部分，点击之后发现如下报错

```bash
Java Model Exception: Error in Java Model (code -1): '2.两数相加' is not a valid Java identifier
...
	at org.eclipse.jdt.internal.core.Openable.generateInfos(Openable.java:254)
	at org.eclipse.jdt.internal.core.JavaElement.openWhenClosed(JavaElement.java:597)
...
```

原来是同一个文件夹，也就是同一个 `work space` 里另外一个文件内有错误，导致我这个也会出现问题

在另一个文件中解决错误后，不再报错
