---
layout:     post
title:      移除 Windows 上 Linux 子系统中的发行版
date:       2022-08-26
catalog: true
tags: [Linux]
---
# 移除 Windows 上 Linux 子系统中的发行版

- 如果电脑系统比较老，可以采用以下命令移除已安装的子系统

  ```bash
  lxrun /uninstall /y
  ```

- 如果系统比较新，运行以上命令则会提示以下错误

  ```bash
  lxrun : 无法将“lxrun”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼写，如果包括路径，请确保路径正
  确，然后再试一次。
  ```

  则需要先使用以下命令查看当前在子系统中已安装的发行版

  ```bash
  wslconfig /l
  ```

  ```bash
  适用于 Linux 的 Windows 子系统分发版:
  Ubuntu-22.04 (默认)
  ```

  接下来运行以下命令后则可以移除已安装的发行版

  ```bash
  wslconfig /u Ubuntu-22.04
  ```

  ```bash
  正在注销...
  ```

  