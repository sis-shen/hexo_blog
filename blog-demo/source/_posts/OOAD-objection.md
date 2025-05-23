---
title: 面向对象分析与设计(2)--对象模型
date: 2025-01-29 18:37:46
tags:
---

# 对象模型引入
面向对象要素建立在很好的**工程基础**之上。它的要素统称为`“开发对象模型”`，或者简称为`“对象模型”`。拆开来讲，它包含如下原则：
`抽象`  `封装`  `模块化`  `层次结构`  `类型`  `并发`  `持久`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503041440868.webp)

就它们本身来说，没有一个原则是创新的。但重要的是，**在对象技术中**，它将这些要素以一种**相互配合**的方式结合起来了。`面向对象分析和设计`在本质上与`传统的结构化设计`方法提出了一种不同的方式来解决软件问题。

# 对象模型的演进
了解对象模型是如何诞生和一步步走向成熟的，有利于我们理解现代对象模型是如何工作的。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503041444045.webp)

1. 第一代语言主要用于科学和工程应用，因此以数学表达式为主。但它依然标志着高级程序设计语言向着问题空间**靠近**了一步，向底层计算机**远离**了一步
2. 第二代语言中，重点是**算法抽象**。
3. 第三代语言则是演进到了支持**数据抽象**
4. 后来发展到面向对象语言的兴盛。
5. 之后出现的许多语言活动，新版本和标准化工作，导致了程序设计框架的出现


# 程序设计语言的拓扑结构演进
对于一门语言的`拓扑结构`，是指这种语言的`基本物理构成单元`，以及这些部分是如何**连接**的。比起其它方面，拓扑着重强调**连接是如何组织的**

## 第一代和第二代早期
此时用这些语言编写的应用展现出相对较平的物理结构，即只包含**全局数据**和**子程序**。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503041519805.webp)

没有足够的封装性和隔离性使之注定难以编写更大的程序，就算实现了也难以维护。

+ *这些语言编写的程序常常会包含子程序间的大量交叉耦合、对数据含义的假定及复杂的控制流，从而对整个系统的可靠性造成威胁，降低解决方的整体清晰性。*

## 第二代后期和第三代早期
子程序的抽象性得到重视，开始出现参数传递机制以及子程序的嵌套

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503041537699.webp)

## 第三代后期
此时为了解决编写大型软件的问题，开始出现了能够独立编译的模块。然而在实践中相比于作为抽象机制，模块化更多地用于给子程序编组。

在这一代语言中虽然很多都支持模块化结构，但对模块间接口的语义的一致性要求不是很严格。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503050815290.webp)

## 基于对象和面向对象的程序设计语言的结构
**数据抽象**对于掌握复杂性是十分重要的。先前的语言更多地是针对`过程抽象`，但在实际问题中，要操作的`数据对象`的复杂性在很大程度上**决定**了问题的复杂性。为此出现了`数据驱动`的方法。相应的，程序的拓扑结构也由基于`子程序`变成了基于`数据对象`。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503050835412.webp)

这类语言的构建块是模块，它表现为逻辑上的一组`类或对象`，而不像早期语言那样是子程序。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503050900092.webp)

但如果我们要超越**大型编程**`programming-in-the-large`,面对**巨型编程**`programming-in-the-colossal`问题，目前由`类`、`对象`和`模块`所提供的基本抽象手段还不够。所幸对象模型在规模上可以扩展。对于更大规模的系统，使用`层次化`的手段组织各组抽象，可以进一步提高对复杂性的描述性能。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503050921870.webp)

# 对象模型基础

## 面向对象编程
> `面向对象编程的定义`：面向对象编程时一种实现的方法，在这种方法中，程序被组织成许多组相互协作的对象，每个对象代表某个类的一个实例，而类属于一个通过继承关系形成的层次结构

+ 这种定义有三个要点:
  + 利用`对象`作为面向对象编程的`基本逻辑构建块`，而不是算法
  + 每个`对象`都是某个`类`的一个`实例`
  + 类与类之间可以通过`继承关系`联系在一起

这三点**都满足**才能称之为面向对象的编程。例如没有继承的编程显然不是面向对象的，它只是利用抽象数据类型在编程。

+ `基于对象`：如果一种语言不提供对`继承`的直接支持，那么它只能称为**基于对象**`object-based`的
+ `面向对象`：一种语言要能够支持继承，表达类型之间的"是一种"关系

## 面向对象设计 
设计是无关语言特性的。设计方法的重点是正确有效地构造出复杂系统的结构。

+ 它有如下两个要点:
  + `面向对象设计`导致了`面向对象分解`
  + 面向对象设计使用了不同的表示法来表达系统**逻辑设计**(类和对象结构)和**物理设计**(模块和处理架构)的不同模型，以及系统的静态和动态特征

## 面向对象分析 
+ 传统的结构化分析技术关注系统中的数据流
+ 面向对象分析的重点在于构建真实世界的模型，利用面向对象的观点来看世界

# 对象模型要素
对于当前存在的五种编程风格，它们和它们使用的抽象如下:

1. 面向过程 算法
2. 面向对象 类和对象
3. 面向逻辑 目标，通常以谓词演算的方式表示
4. 面向规则 如果----那么规则
5. 面向约束 不变的关系

每一种编程风格都是基于它自己的概念框架，而对于所有**面向对象**的东西,概念框架就是对象模型。这个模型有`4`个主要要素,已经在上面的演进中提到了：
1. 抽象
2. 封装
3. 模块化
4. 层次结构

所谓的主要就是指这些要素必须全部具备才是面向对象的。此外还有三个次要要素

1. 类型
2. 并发
3. 持久

接下来我们逐个探究这些要素

## 抽象的意义
> “抽象描述了一个对象的基本特征，可以将这个对象与所有其他类型的对象区分开来，因此提供了清晰定义的概念边界，**它与观察者的视角有关**

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503051125158.webp)

在上一篇博客中，我们已经讨论过了怎么利用`抽象`来更好地管理`复杂性`。

抽象也分类型和等级，从最有用到最没用的次序，这些抽象是:
+ **实体抽象** 一个对象，代表了`问题域`或`解决方案域`实体的一个有用的模型
+ **动作抽象** 一个对象，提供了一组通用的`操作`,所有这些操作都执行同类的功能
+ **虚拟机抽象** 一个对象，集中了某种高层控制要用到的所有操作，或者这些操作将利用某种更底层的操作机
+ **偶然抽象** 一个对象，封装了一组相互没有关系的操作

在实际的抽象中，没有对象是孤立的，每个对象都与其他对象协作，实现某些行为。这些对象之间如何协作的设计决策，定义了每种抽象的边界，从而也定义了每个对象的责任和协议。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503051135309.webp)

## 封装的意义


