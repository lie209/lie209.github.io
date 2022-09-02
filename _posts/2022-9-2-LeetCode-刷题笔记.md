---
layout: post
title: LeetCode 刷题笔记
date: 2022-9-2
catalog: true
tags: [论文]
description: 持续更新...
---

# `LeetCode` 刷题笔记

## 多复用前面的处理结果，减少时间开销

以第`303`题为例，一开始尝试以`前缀和`的思想减少时间开销，代码如下

```java
public NumArray(int[] nums) 
    {
        this.preNums = new int[nums.length+1];
        this.preNums[0]=nums[0];
        for(int i=1;i<nums.length;i++)
        {
            int tem=0;
            for(int j=0;j<=i;j++)
            {
                tem=tem + nums[j];
            }
            this.preNums[i] = tem;
        }
    }
```

最终运行结果及时间为

```bash
Accepted
15/15 cases passed (52 ms)
Your runtime beats 8.77 % of java submissions
Your memory usage beats 10.82 % of java submissions (44.6 MB)
```

然后发现初始化`前缀和`的时候仍然有优化空间，修改代码如下

```java
public NumArray(int[] nums) 
    {
        this.preNums = new int[nums.length+1];
        this.preNums[0]=nums[0];
        for(int i=1;i<nums.length;i++)
        {
            this.preNums[i] = preNums[i-1]+nums[i];
        }
    }
```

运行结果及时间为

```bash
Accepted
15/15 cases passed (7 ms)
Your runtime beats 100 % of java submissions
Your memory usage beats 44.73 % of java submissions (44.3 MB)
```

优化效果很明显！