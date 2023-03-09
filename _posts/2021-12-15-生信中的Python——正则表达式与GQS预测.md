---
layout: post
title: 生信中的Python——正则表达式与GQS预测
categories:
 - 生物信息
---

（这个博客没写完，SOR）正则表达式算是我最常用的代码技巧吧，因为经常需要文本的模式匹配（就是查找某种字符串）。从我的经验，正则表达式很强大，效率也很高。最近有一个查找转录组中的GQS的项目，正好整理一下正则表达式。
>* 基础匹配
>* m
>* 用

***

写在前面：

我准备用Python来实现，其实各个语言的正则表达式（Regular expression）语法差异都不大。我不准备写太多基础的东西，就像文档教程那么难看，我就写一些实用的东西。

## 基础匹配

GQS，G-quadruplex 是一种富含G的碱基片段，现在发现有一大堆不啦不啦的功能。所以第一步就是把他找出来。

比较经典的GQS模式是G3N1-7,用正则表达式这样的：

```python
GGG [ATCG]{1,7} GGG [ATCG]{1,7} GGG [ATCG]{1,7} GGG

e.g.,

GGG AT GGG CTA GGG AGTT GGG

其中：
[ATCG]是这4个字符的一个，{1,7}是1-7个

所以上面这个正则表达式简写一下就是：
(G{3}[ATCG]{1,7}){3}G{3} 
```
![picture 1](https://pic.atlasbioinfo.com/pic_1639566409748_1639566417010.png)  

这个例子是3个G，也叫G3。当然植物中还有2个G的情况。不过按照这个正则表达式，是不是就能找到全部的GQS了呢？让我们试试

```python

```




![博主简介](https://pic.atlasbioinfo.com/logo.png)