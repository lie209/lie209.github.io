---
layout:     post
title:      MasterFace 算法实现
date:       2022-08-12
catalog: true
tags: [论文]
---
# MasterFace算法实现

## LVE算法

将生成的图片与人脸识别系统中注册的每张图片计算相似度

![算法实现](../img/2022-8-12-MasterFace/算法实现.png)


## 一些要点

- 输入网络的随机向量的维度`G.z_dim`为512

- 使用`Bob`工具箱实现图片比较

  `*abstract compare(enroll_templates,** probe_templates)`

  ```python
  Computes the similarity score between all enrollment and probe templates.
  ```

  [相关链接](https://www.idiap.ch/software/bob/docs/bob/docs/stable/bob/bob.bio.base/doc/py_api.html)

- 