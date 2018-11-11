---
layout: post
title: Deepin Linux 上双显卡电脑如何关闭独显，使用核显
date: 2018-11-10
categories: blog
tags: [Linux]
description: linux摸索中的经验。
---

>参考：http://wiki.deepin.org/wiki/%E6%98%BE%E5%8D%A1


一直以来，因为我的电脑是双显卡的（核显+1050ti)，由于英伟达的原因导致linux上问题不断。
当然，有折腾才有进步嘛。

Ubuntu上可以直接进入英伟达的显卡设置切换为核显。Deepin目前还不行。因为我的电脑无法在深度显卡驱动管理器里切换到大黄蜂模式，试了好多次都不行。

因为在网上看到的针对这个问题解决的方案不是很多，所以我就打算自己写一篇博客了，这也是我的第一篇博客。

无需多言，进入正题。

关闭独显的目的是**解决屏幕的撕裂感以及发热耗电的问题**（我个人认为打游戏用Win不就好了嘛，Linux本来就不是用来打游戏的）其实解决很简单，不需要更改什么文件什么的，我以前也试过一个[通过更改显卡的什么文件的那个方法](https://bbs.deepin.org/forum.php?mod=viewthread&tid=167512&extra=)来解决，不过效果我感觉不好，比如拖动鼠标时后面跟的尾巴太长了，强迫症不喜欢。

终于我找到了官方的解决方案：

1.自己进入BIOS关闭独显
我感觉这个方法对装双系统不太好，本人学生党，去Win打游戏的时候总不能每次都要进BIOS开启独显吧，有点麻烦。

**2.最佳解决方案。**
安装第三方的Bumblebee程序，可实现NVIDIA Optimus，终端中执行：

<kbd>针对闭源驱动</kbd>：安装Bumblebee-nvidia,*其中primus可选,用于提升性能*
```
sudo apt-get install bumblebee-nvidia primus 
```
<kbd>针对开源驱动</kbd>：

```
sudo apt-get install bumblebee primus
```
**注意 16年以后的新笔记本可能存在重新启用独显以后，造成系统冻结。**(这个方法我没试过，因为我并不打算在linux上用独显）

解决方法是在grub中给内核添加参数来防止系统出现冻结。

```
acpi_osi=!
```
安装完毕后重启电脑使Bumblebee生效，但是Bumblebee并不能做到集显（Intel显卡）和独显（NVIDIA显卡）之间的自动切换。系统运行时会默认使用集成显卡（Intel显卡），当你需要使用独显（NVIDIA显卡）运行某个程序或游戏时候，终端执行如下命令即可：

```
optirun command #使用独显运行command程序

optirun -b primus command #使用独显运行command程序,提升性能
```
例如运行0 A.D.游戏：

```
optirun 0ad #使用独显（NVIDIA显卡）运行0 A.D.游戏
```
当使用optirun命令的程序关闭后，系统将会自动关闭独显（NVIDIA显卡）。

推荐安装bbswitch，并修改bumblebee的配置，使用bbswitch。

编辑/etc/bumblebee/bumblebee.conf，修改文件最后的nvidia和nouveau节点中，PMMethod=auto 修改为 PMMethod=bbswitch。












