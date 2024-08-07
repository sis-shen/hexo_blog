---
title: =C入门=深入研究 字符串与字符数组
date: 2023-11-08 21:11:51
tags: C语言 字符串 数组
---
# 什么是字符串 #
## 初见字符串 #
我们最先遇到的字符串,一般是`hello_world`程序中用到的`"hello world"`,也就是**两个双引号括起来的一串字符**,输出时的占位符是`%s`,可以直接拿去传值，代码如下
```C
printf("%s","hello world");
```
## 声明字符串变量 #
有时我们想要先把字符串存起来，再进行操作，那么就使用**字符数组**，并在**初始化**的时候把字符串传给它,这样在**创建数组**时会编译器会自动分配内存给它，代码如下

```C
char str[] = "abcdef";
```

此时我们也可以开启VS的**调试**，并打开**内存**和**监视**窗口观察字符串是如何在内存中储存的,如下图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-09_10-06-06.jpg)

通过观察可以发现，C语⾔字符串的字符串有个`规定`(特点)，就是以字符`\0`结尾，无论是初始化数组时，还是在分配内存时，都有`\0`的位置。

### strlen()函数 #
依据以`\0`为字符串结尾的规则，`strlen`函数就可以计算字符串的长度，它会从字符串的第一个字符向后扫描，直到遇到`\0`结束,且`\0`不进入计数，最后返回字符串的长度,代码如下
```C
#include <string.h> //需要引对应的头文件

int len = strlen("abcdef");//len的值为6
int sz = sizeof("abcdef");//sz的大小为7(\0被计入总数)
```

### 验证字符串的结尾 #

#### 正向验证 #
我们做在字符数组里插入一个`\0`,来看看函数`printf`和`strlen`找到的结尾在哪里,如下图

```C
char str[] = {'a','b','c','\0','d','e','f','\0'};
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-09_11-01-35.jpg)

可以看到字符数组似乎被“截断”了，`printf`只输出了`\0`前面的内容, `strlen`算出来的长度也只有`3`,可见插入的`\0`被作为了**字符串**的结尾，字符串提前中指,而没到达**字符数组**的结尾



#### 反向验证 #

我们来**反向**验证一下，`\0`是字符串结尾的标志,如下图，我们声明一个**没有**`\0`结尾的字符数组，看看函数`printf`和`strlen`还找不找得到我们“认为”的结尾
```C
char str[] = {'a','b','c','d','e','f'};
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-09_14-02-07.jpg)

可以看到函数对字符串的判断出现了严重**失误**，所以**字符数组**里没有`\0`标记结尾是非常严重的问题，不光是找不到字符串的结尾，而且会**越界访问**！危险操作，写代码的时候一定要注意

---

## 从字符串到字符数组 #
虽然上面已经用到了字符数组，但主要还是为了方便讨论**字符串**,接下来着重研究字符数组。

### 先整清楚几个概念 #
`什么是数组`：数组是⼀组相同类型元素的集合,会在内存中开辟一段连续的空间，将元素储存在那段内存中

`什么是数组元素`：存放在数组的值被称为数组的元素，数组在创建的时候可以指定数组的⼤⼩和数组的元素类型。

所以`字符数组`是一组`字符`的集合，字符数组里的`元素`都是`字符`!,访问到的字符数组里的**元素**都是**字符**，像`'a'`,`'b'`,`'c'`这种的单个的字符,别和`字符串`混为一谈！

```C
char str = "abc";
int sz = sizeof(str);//这里str代表了整个数组,所以包括\0
,sz的值为4

```

### 字符数组的声明 #
字符数组的声明和其他类型的数组差不多，有**初始化**，**不完全初始化**，**声明长度**，**不声明长度**

*正确的声明代码如下*
```C
char str1[] = "abc";//初始化,不声明长度

char str2[] = {'a','b','c','\0'};//这也是初始化，且不声明长度

char str3[10] = { 0 };//初始化，用值0(等价于`\0`)填充

char str4[10] = "abc";//不完全初始化，存入字符串"abc",后面都用'\0'填充

char str5[10] = {'a','b','c','d'};//不完全初始化,从下标为0开始，依次往后填充字符 a,b,c,\0

```
*错误的声明代码*
```C
char str1[3] = "abc";//数组声明短了，放不下结尾的\0,编译过不了

char str2[3] = {'a','b','c','d'};//同上,放不下

char str3[] = { 0 };//能声明，但是字符数组长度为1，这个数组大概率是用不了的/会被拿去错误使用的

```
### 来看看这些声明方式在内存中的表现 #

#### 不初始化的声明(极度不推荐) #
```C
char str[];//这个不加长度，直接编译失败（如下图）
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_11-39-10.jpg)

```C
char str[10];//语法没有问题，来看看此时数组里存了什么
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_11-46-49.jpg)
可以看到全都存了`-52`,对应的中文字符是`烫`，这样**不好**，请在声明字符数组的时候**初始化数组**
#### 不声明长度的数组声明
```C
char str1[] = "abc";

char str2[] = {'a','b','c','\0'};
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-09_14-59-34.jpg)

如上图，不声明长度时，编译器自动给字符数组分配内存，既不给多，也不给少，初始化给的`字符串`或者`{...}`多长，创建的数组就多长。

**注意红框**，再强调一遍，字符串以`\0`结尾,看到双引号括起来的字符串，要记得最后隐藏了一个`\0`,用字符数组储存的时候一定要留足空间

#### 声明长度的数组声明 #
```C
char str3[10] = { 0 };

char str4[10] = "abc";

char str5[10] = {'a','b','c','d'};
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-09_15-25-54.jpg)

可以看到，在声明的长度**足够长**时，你初始化的时候给它多少字符，它就从`下标0`处开始**按顺序**存进去多少,剩下的部分**自动**用`'\0'`填充,

所以实际上上面代码中的`str5`因为长度`10`>初始化给的`4`个字符，后面六个元素用`\0`填充了，所以`str5`里存了有**结尾**的完整字符串

#### 错误声明 #
```C
char str1[3] = "abc";
char str2[3] = { 'a','b','c','d' };
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_07-44-57.jpg)

可以看到上面两种错误的声明方式，甚至直接**编译失败**,所以声明字符数组的时候一定要**留足空间**

```C
char str3[] = { 0 };
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_07-51-55.jpg)

如上图，可以看到这样写还是编译成功了，但是在监视查看**数组长度**的时候，发现长度为`1`,里面存了一个`\0`,这么**短**的数组能用吗？只能**用一点点**，甚至还不如直接声明一个`char`类型的字符变量

## 当字符数组加上const #
```C
const char str[5] = "abc";
```
### 一些性质 #

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_18-55-52.jpg)

可以看到，声明时加了`const`之后，字符数组`str`在声明时的**初始化**之后便不可更改了，只能**访问**其元素,而**不能通过访问元素来改变数组内容**

那么`scanf`还能写入内容吗？答案是**可以**!*(如下图)*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_19-10-43.jpg)

那它能拿来初始化别的数组吗？很遗憾，**不能**
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_19-05-15.jpg)

### 对应的指针类型 #
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_19-15-32.jpg)

可以看到，这里得用`const char*`来储存字符数组的地址，而使用`char*`就会报错

那么用`双引号括起来的字符串`，是否也有地址，能用指针储存它的地址呢？

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-10_19-21-10.jpg)

如图，可以看到,字符串`"abc"`是属于`const char`类型的数组，对应的指针是`const char*`,不能通过访问元素来改变内部的值，也不能用`char*`来储存地址

#### 在内存中的表现 #
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/const_str.gif)

可以看到哪怕是字符串`"abc"`,也是在内存中开辟了一段空间，并把字符**储存在内存中**了的

但是，**不要**试图用`scanf`去改变字符串的值
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/const_strrrr.gif)

---

## 如何向字符数组里添加内容 #
添加的方法多种多样，搞不好可能还会出错，所以把字符数组**学明白**很重要！

以下使用的数组样例声明如下
```C
char str[10] = { 0 };
```

### 初始化 #
在初始化的时候就把值传进去，有哪些初始化方式**上面**已经介绍过了，这里不多赘述

### 访问数组元素 #
通过`[]`可以访问数组元素，并对没有`const`修饰的数组，修改其元素,例如:

```C
str[0] = 'A';//将数组的第一个元素改成字符A
```

我们也可以通过循环的方式，将数组的所有元素填充为某个字符

```C
char place_holder = 'A';
for(int i = 0; i < 10 ; i++)//这里使用左闭右开区间，10为数组的大小
{
    str[i] = place_holder;
}
```

### 使用`scanf`函数 #
由上面的探究已知：对于已声明的字符数组，**无论**有没有`const`修饰，都可已用`scanf`修改内容,那么`scanf`怎么用，又具体怎么工作的，我们接着往下探究

#### 使用示例 #
```C
char str[10] = { 0 };
scanf("%s",str);//占位符是 %s ,右边的参数是 str ,也就是数组名
//或者 scanf("%s",&str)
```
**注意**！这边的数组名`str`储存的是**数组首元素的地址**，而`&str`储存的是`整个数组的地址`，值是一样的，两者皆可用于传参，但**指针类型不一样**，要做好区分

#### `scanf`都做了什么 #
先来看看它分别对用`{ 0 }`**初始化**和**不初始化**的数组做了什么
```C
char str1[10] = { 0 };
scanf("%s",str1);
char str2[10];
scanf("%s",str2);
```
*两个数组的输入均为abc*
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-11_19-09-50.jpg)

可以看到，对`str1`,字符非常正常地填充进去了，因为整个数组原本是用`\0`填充的,看不出什么端倪

而对于`str2`,观察发现，除了输入进去的字符`a`,`b`,`c`,它还自动在结尾补了一个`\0`,使`str2`里储存了一个完整的字符串。**但是**，剩下的部分还是用**值**`-52`填充，即未初始化的状态，所以依然**不提倡**声明的时候没有初始化

然后是在字符数组内已有内容的情况下，再次使用`scanf`的情况
```C
char str1[5] = "abc";
char str2[5] = "abc";

scanf("%s %s",str1,str2);
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-12_09-33-02.jpg)

如图，`scanf`做的是把输入的字符串`覆盖`式存入字符数组，比原来长，就完全覆盖，比原来短，就部分覆盖，未覆盖的部分无改动

#### 关于`scanf`的危险操作 #
由于`scanf`无法预测**字符数组**能否存下输入的**字符串**，如果**字符数组**声明的长度不够，就可能出现**越界访问**,随之而来的便是奇奇怪怪的`bug`

```C
char str[4] = { 0 };//先声明一个长度为4的数组
scanf("%s",str);//这次输入abcdef试试
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-12_09-47-00.jpg)

可以看到，确实**越界访问**了，所以声明字符数组的时候，建议比预计最大输入，在多些长度，防止越界访问。















