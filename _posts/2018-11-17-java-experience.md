---
layout: post
title: 关于Number类的一个注意事项（from书上的一个题目）
date:  2018-11-17
categories: blog
tags: [Java]
description: 
---

从书上一个题目说起。

![Cache_649ec4f1cab57c90.](https://s2.loli.net/2023/03/10/t57yo48fh1CmnGs.jpg)

具体代码如下：

```java
public class Test
{
     public static void main(String[] args)
     {
          Number x=new Integer(3);
          System.out.println(x.intValue());
          System.out.println((x.compareTo(new Integer(4)));
     }
}
```


当然这是错误的，并且有些使用是过时的，很明显这只是书上为了出题而刻意写出来的

**_可是具体为什么错呢？_**

百度了好久，终于找到了。

看代码理解：

```java
package java.lang.Number  
  public abstract class Number implements java.io.Serializable  
```

```java
package java.lang.Integer  
  public final class Integer extends Number implements Comparable<Integer>  
```

终于知道错的原因了。

>Number没有实现Comparable接口！！！！！


具体为什么要这么做，[点这里](https://stackoverflow.com/questions/480632/why-doesnt-java-lang-number-implement-comparable)

正确代码如下：

```java
public class Test
{
     public static void main(String[] args)
     {
          Number x=new Integer(3);
          System.out.println(x.intValue());
          System.out.println(((Integer) x).compareTo(new Integer(4)));
     }
}
```

当然，这个代码也是不推荐的，因为其中有些是过时的。