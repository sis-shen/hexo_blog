---
title: =C++= sizeof关键字详解
date: 2024-07-10 15:30:19
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-11_16-05-28.png
---
## 简介
`sizeof`作为C/C++关键字,基本用法是求`字节大小`，但仅仅这一项用法在细节上就有很多说法了

## 求内置类型变量的大小
有两种写法，以`int a = 0`的变量`a`为例

+ `sizeof a`
+ `sizeof(a)`

都可以求变量`a`的大小，但**注意**，该变量的大小仅与`变量类型`有关，而与值无关

## 求内置类型的大小
求类型大小时必须**加上括号**

例如`sizeof(int)`

## 求数组的大小
+ 当数组声明在`全局`或`sizeof`处于数组声明语句的`局部作用范围`时,能够用`sizeof(<数组名>)`求数组大小
+ 当数组名经过**函数传参**或`加减常量运算`后，退化为`指针变量`,类型大小在`32位机器`中为`4`,`64位机器`中为`8`

```C
#include <iostream>

using namespace std;

int g_arr[10] = { 0 };

void func(int st_arr[])
{
	cout << sizeof(st_arr) << endl;//此处退化为指针变量，输出4或8
}

int main()
{
	int arr[10] = { 0 };
	cout << sizeof(g_arr) << endl;//输出40
	cout << sizeof(arr) << endl; //输出40
	func(arr);//输出4/8
	return 0;
}
```

## 求类/对象的大小

### 一般情况的内存对齐
为了访问效率问题，类的大小遵循`内存对齐`规则，计算理论大小时需考虑`成员变量的大小`和`内存对齐`,而不考虑普通成员函数,这里不详细讨论

## 含有虚函数
C++的编译器一旦发现一个类型中**有虚函数**，就会为该类型生成`虚函数表`，每一个实例化出的`对象`都含有一个`指向虚函数表的指针`。所以`sizeof`求出来的值还要考虑`该指针`以及`内存对齐`等因素

### 没有成员变量的特殊情况

#### 没有成员函数
这样的类型可以称为`空类型`,因为这样的类型实例化后不含任何信息,本来求`sizeof`应该是`O`，但考虑实际使用时，我们需将类**实例化**为对象，它必须在内存中占有一定的空间，否则无法使用或管理这些实例。至于分配内存，由编译器决定。但出于节省不必要的内存占用原则，理应分配最小内存单元，即`1字节`。正好在`VisualStudio`中，每个空类型的实例占用`1字节`的空间。

```C++
class A
{
    //空类型
};
sizeof(A);//visual studio 中值为1
```

#### 只有普通的成员函数
和上一条一样也是`1字节`。因为考虑实例化时，调用这些普通成员函数只需知道`函数地址`即可，而这些函数地址只与用户自定义的`类`有关，而与实例化出来的`对象`无关，所以不会在`对象`中存储相关信息，不会改变其大小。

#### 含有虚函数
实例化出的`对象`含有了指向虚函数的`指针`,所以`sizeof`求出来的大小为一个`指针`的大小，`32位机器`求得`4`字节,`64位机器`求得`8`字节