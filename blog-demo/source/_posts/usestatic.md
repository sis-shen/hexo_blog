---
title: C/C++ static关键字的使用
date: 2024-06-30 18:21:50
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/static.png
---

因为`static`的用法又多又杂，值得单出一篇博客用以汇总

# C/C++ 通用用法

## 局部变量->全局属性
当对原本声明在函数栈帧里的`变量`使用`static`修饰时,该`变量`的存储空间会改变到`静态区`，不会随着函数栈帧的销毁而销毁。

初始化：初次调用声明语句时会执行声明操作，而之后再执行到该语句处时会自动跳过。


![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-03_10-51-24.png)

作用范围：与不加`static`时的作用范围相同，还是局部可用

销毁：和全局变量一样在`main`函数的栈帧销毁时一并销毁

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-03_11-09-04.png)

## 全局变量->限制访问
原本同级文件夹下的源文件可以用`extern`关键字互相获取全局变量,但如果用`static`修饰本地全局变量，那么这个全局变量只能在本文件调用，而其它文件看不到它

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-03_11-56-27.png)

## 全局函数->限制访问
原本同级文件夹下的源文件可以用`extern`关键字声明函数，然后去其它源文件的全局函数中寻找`实现方式`,但如果用`static`修饰本地全局函数，那么这个全局函数的实现只能在本文件调用，而其它文件看不到它

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-03_12-02-16.png)

# C++类和对象

## 成员变量->静态成员 (全局变量)
原本声明的成员变量在实例化后，属于由`类`实例化出来的`对象`，生命周期与所属对象相同，但在加了`static`后，该变量属于该类域中的全局变量，不再属于某个**具体**的`对象`

**初始化**: 因为已经不属于某一个对象，初始化也不能在类的接口中完成了，因此在类中声明`static`成员变量后，**必须**在全局区初始化`类域`中的成员变量。  #*在main函数等非全局域中无法初始化*

常见应用：引用计数

```C++
class A
{
public:
	A()
	{
		cnt++;
	}

private:
	static int cnt;
};

int A::cnt = 0;
```

## 成员函数->静态成员函数 (不含this指针)
一般声明在类中的成员函数的参数列表隐藏`this`指针，要调用函数时得用类实例化出的`对象`来调用，由这个对象提供`this`指针

而使用`static`修饰后的静态成员函数不含`this`指针，该函数属于整个类域，调用时使用类域调用

```C++
class A
{
public:
	A()
	{
		cnt++;
	}

static int getCNT()
{
    return cnt;
}

private:
	static int cnt;
};

int A::cnt = 0;

int main()
{
    A a,b,c;
    cout<<A::getCNT();//此时输出3
    return 0;
}
``` 