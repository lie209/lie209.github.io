---
layout: post
title: LeetCode 刷题笔记
date: 2022-9-27
catalog: true
tags: [Java]
subtitle: 持续更新...
---

# `LeetCode` 刷题笔记

这篇笔记主要记录学习`labuladong`算法秘籍的一些心得体会

## 前缀和数组

以第[`303`](https://leetcode.cn/problems/range-sum-query-immutable/)题为例，一开始尝试以`前缀和`的思想减少时间开销，代码如下

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

## 差分数组

### 差分数组的构造

所谓的差分数组就是构造一个数组，其中第0个元素不定，可以为0或者为原数组第0个元素，剩余元素为原数组对应位置减去前一个位置的差

![image-20220923101642435](https://s2.loli.net/2023/03/10/6UstIOj3nR2MNJh.png)

### 差分数组的使用

- 利用差分数组可以很方便的在一个循环里对一个数组做**多个范围内的加减**操作

  > 或者乘除操作也可以，只需要在构造差分数组的时候变换一下构造方式

  1. 先建立一个差分数组

     ```java
     //给数组i到j的元素加上val
     public int[] increment(int i,int j,int val,int[] diff)
     {   
         diff[i]=diff[i]+val;
         if(j+1<diff.length)
         {
             diff[j+1]=diff[j+1]-val;
         }
         return diff;
     }
     ```

     以上就是利用差分数组对原数组第`i`到`j`的元素加`val`的操作，只需变换差分数组两个位置的元素即可，后面的`-val`原因是，前面`i`处将`i`以及之后的元素都加上了`val`，在`j`之后的元素加上了多余的`val`，就要减掉

  2. 使用差分数组，在一个循环里对原数组进行操作

     ```java
     public int[] getResult(int[] diff,int n)
     {
         //建立一个长度为n的结果数组
         int[] result = new int[n];
         result[0]=diff[0];
         //数据操作
         for(int i=1;i<n;i++)
         {
             result[i]=result[i-1]+diff[i];
         }
         return result;
     }
     ```

- 以力扣[1094题](https://leetcode.cn/problems/car-pooling)为例

  1. 先建立差分数组

     ```java
     public int[] difference(int[][] trips)
     {
         int[] diff=new int[1001];
         for(int i=0;i<trips.length;i++)
         {
             diff[trips[i][1]]=diff[trips[i][1]]+trips[i][0];
             if(trips[i][2]+1<=1000)
             {
                 diff[trips[i][2]]=diff[trips[i][2]]-trips[i][0];
             }   
         }
         return diff;
     } 
     ```

  2. 利用差分数组进行区间的加减法

     ```java
     public int[] incremnet(int[] diff)
     {
         int[] result=new int[1001];
         result[0]=diff[0];
         for(int i=1;i<result.length;i++)
         {
             result[i]=result[i-1]+diff[i];
         }
         return result;
     }
     ```

     这样就在一个循环里，对一个数组进行了多次的加减法操作，大大降低了时间复杂度

## 使用优先级队列进行排序

> 最小堆指的是根节点的值小于所有其他堆节点的值

> 在`Java`中可以使用`PriorityQueue`来实现和使用最小堆

`Java`中的优先级队列基于堆排序实现，具体底层实现可见[此处](https://mp.weixin.qq.com/s/o7tdyLiYm668dpUWd-x7Lg)

建立最小优先级队列：

```java
//建立一个优先级队列，传入优先级队列长度以及比较的接口
PriorityQueue<ListNode> list=new PriorityQueue<>(length,(a,b)->(a.val-b.val));
```

后面的`(a,b)->(a.val-b.val)`是一个`lambada`表达式形式的接口，通过改变这个可以改变队列的优先级

以`Java`中最小优先级队列为例，每次使用`poll()`方法，可以返回队列中的最小元素：

```java
//获得优先级队列中最小的元素
p.next=list.poll();
```

使用`add()`方法往堆中添加元素，元素会自动在堆中排好序

```java
list.add(node);
```

具体使用实例：力扣[23题](https://leetcode.cn/problems/merge-k-sorted-lists/)

## 使用双指针解决某些问题

比如力扣[26题](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)，题目要求删除有序数组中的重复项

![image-20220929160157458](https://s2.loli.net/2023/03/10/F3txT6RZLaphvij.png)

具体思路是：

- 使用两个指针，一个`p`指向前面，一个`q`指向后面，`p`用于指向最终结果的最后一个，`q`用于向后探测
- 因为是**有序数组**，所以只要`q`指向的元素不等于`p`指向的元素，便可以将这个元素放到`p`位置，实现了在一个循环里删除掉了一个数组中的重复元素

力扣[83题](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)也是一样的操作，实现在**一个循环**里查找和删除重复元素

对于力扣[283题](https://leetcode.cn/problems/move-zeroes/)以及力扣[27题](https://leetcode.cn/problems/remove-element/)，可以使用双指针实现原地删除或移动元素，即不需要使用额外的空间，以`283题`为例，实现原地删除或移动元素的代码模板大致如下：

```java
int p=0,q=0;
//将非零元素移动到前面
while(q<nums.length) 
{
    if(nums[q]!=0)
    {
         nums[p]=nums[q];
         p++;
         q++;
     }
     else
     {
          q++;
     }
}    
```

主要是使用两个指针，一个“守家”，一个“探路”，进而实现要求的操作

