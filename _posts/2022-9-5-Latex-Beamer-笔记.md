---
layout: post
title: Latex Beamer 笔记
date: 2022-9-5
catalog: true
tags: [论文]
---

# Latex Beamer 笔记

## 前置知识

- 如何使用`Beamer`制作中文的PPT

  插入以下代码即可

  ```latex
  \documentclass{ctexbeamer}
  ```

- 推荐的[参考示例文档](https://cn.overleaf.com/learn/latex/Beamer)

- 使用`Beamer`制作的的PPT，其中的页面放在`frame`环境中，以`frame`为单位，插入新的页面，只需要加上如下代码

  ```latex
  \begin{frame}
  \frametitle{Sample frame title}
  This is some text in the first frame. This is some text in the first frame. This is some text in the first frame.
  \end{frame}
  ```

## `Beamer`主要功能

### 标题页

![image-20220904190905276](https://lie209blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220904190905276.png)

此页面的相关代码如下

```latex
\title 
{About the Beamer class in presentation making}

\subtitle{A short story}

\author 
{A.~B.~Arthur \and J.~Doe}

\date
{Very Large Conference, April 2021}
```

- `\title`定义主标题
- `\subtitle`定义子标题
- `\author`定义作者，中间不同作者用`\and`连接
- `\date`定义日期

还可以用`\institude`定义机构名称，`\inst`定义作者所属机构编号

```latex
\author 
{A.~B.~Arthur\inst{1} \and J.~Doe{2}}

\institute
{
  \inst{1}
  Faculty of Physics\\
  Very Famous University
  \and
  \inst{2}
  Faculty of Chemistry\\
  Very Famous University
}
```

此时显示效果如下

![image-20220904191745422](https://lie209blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220904191745422.png)

需要注意：在序言处定义完封面之后，需要在接下来`document`环境中加上以下语句，才能让标题页正常显示，意思是在文档中插入这个`frame`

```latex
\frame{\titlepage}
```



### 目录页

可以选择将目录放在每个部分的开头，高亮显示当前节的标题，只需在`tex`文档的序言处定义如下语句

```latex
\AtBeginSection[]
{
  \begin{frame}
    \frametitle{Table of Contents}
    \tableofcontents[currentsection]
  \end{frame}
}
```

这样在每一部分的开头处都会显示如下页面

![image-20220904192341438](https://lie209blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220904192341438.png)

---

序言结束后便开始正文部分了，正文部分以`\begin{document}`表示

在正文的前面，也就是`document`环境标题`frame`之后，可以添加目录，添加如下代码即可，目录具体项会自动生成

```latex
\begin{frame}
\frametitle{Table of Contents}
\tableofcontents
\end{frame}
```

![image-20220904192657926](https://lie209blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220904192657926.png)

### 正文页

如前置知识所言，正文的每一个页都放在一个`frame`中，添加新页面只需添加`frame`即可，如下

```latex
\begin{frame}
正文
\end{frame}
```

### 为演示文稿添加效果

