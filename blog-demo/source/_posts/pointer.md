---
title: 指针详解
date: 2023-11-23 07:35:39
tags:
---

# 内存和地址 #
在了解指针之前，先讲讲内存是如何管理的

首先因为内存很大（一般有几个G）,所以为了高效管理，有了`内存单元`的概念。而这个单元的大小，正好是一个字节。

因为一个`比特位`就是一个二进制位，太小了，超过一个字节，在处理`char`这样一个字节长的变量很麻烦。

定下长度后，就可以给内存单元编号，而每个内存单元获得的独一无二的编号，便是它的**地址**,以声明了一个变量a为例,示意图如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-23_09-59-30.png)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-23_10-21-48.png)

## 变量的地址

上图中`a`占4个字节，每个字节都有自己的地址，但要找到`a`其实只需要找到第一个地址就行了，实际上在`C语言`中也是如此,`a`的地址就是`首字节地址`,即图中的`0x000000AF88DFF6A4`

## 关于几个名词 #
在`C语言`中称`地址`为`指针`,**储存**地址的变量叫`指针变量`,平时也**简称**`指针`,此时强调的是`指针变量`里储存的地址，而不是这个变量。

## 指针变量的组成 #
指针变量也要**拆成两部分**来看

一个是变量的`值`,在同一个程序中，所有指针变量的值的`长度`都是一样的，都指向了某**一**个内存中的`字节`, 至于具体多长，取决于环境:`32位程序是4个字节`,`64位程序是8个字节`

另一个是变量的`类型`,类型决定编译器从`值`所指向的字节，向后总共读**几个**字节，以及用**什么方式**读取内存里的内容。以下图的代码为例

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-23_11-36-47.png)
