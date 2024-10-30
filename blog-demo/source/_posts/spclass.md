---
title: C++特殊类的设计
date: 2024-08-12 10:55:17
tags:
---
接下来我们设计一些C++的常用特殊类，同时加深对C++类和对象的理解

# 设计一个不能被拷贝的类

拷贝只会放生在两个场景中：`拷贝构造函数`以及`赋值运算符重载`，因此想要让一个类禁止拷贝，只需**让该类不能调用拷贝构造函数以及赋值运算符重载即可。**

*C++98*

将拷贝构造和赋值重载只`私有声明`且**不定义**即可

**分析**：
1. 设为私有：防止类外调用，还能**防止类外定义**重新实现拷贝功能
2. 只声明不定义：防止类内成员函数对其调用
```C++
//C++98
class NoCopy
{
private://设置成私有
	NoCopy(const NoCopy&);
	NoCopy& operator=(const NoCopy&);//只声明不定义
};
```

*C++11*

C++11扩展了`delete`关键字的用法，可用于删除成员函数，尤其是**删除默认成员函数**

```C++
//C++11
class CopyBan
{
	CopyBan(const CopyBan&) = delete;
	CopyBan& operator=(const CopyBan&) = delete;
};
```

# 设计一个类，只能在堆上创建对象
既然涉及到类实例化成对象（如下图），势必绕不开构造函数,但实际上对类的设计者来说，通过控制构造函数来控制用户的行为难以实现。所以我们选择把`拷贝构造函数`禁掉，把`赋值运算重载`禁掉,把`默认构造函数`私有化，然后单独提供静态成员函数

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409231355404.png)

```C++
class HeapOnly
{
public:
	static HeapOnly* GenerateObj()//提供静态成员函数
	{
		return new HeapOnly();
	}

private:
	HeapOnly();//私有化默认构造函数
	HeapOnly(const HeapOnly&) = delete;//禁掉拷贝构造
	HeapOnly& operator=(const HeapOnly&) = delete;//删除赋值运算重载
};
```

# 设计一个类，只能在栈上创建对象
除了要把构造函数私有化，还要把`operator new`和`operator delete`重载禁掉，然后提供在返回栈上创建的对象的静态成员函数

*赋值运算符重载函数只返回栈上的对象，所以不用管它*

```C++
class StackOnly
{
public:
	static StackOnly CreatObj()
	{
		return StackOnly();//返回临时对象的拷贝
	}

private:
	StackOnly() {}

	StackOnly(const StackOnly&) = delete;

	void* operator new(size_t size) = delete;
	void operator delete(void* ptr) = delete;
};
```

# 设计一个类，不能被继承
*C++98*

将构造函数私有化，派生类中调用不到基类的构造函数，就会继承失败
```C++
class NoInherit
{
public:
	static NoInherit GenerateInstance()
	{
		return NoInherit();
	}
private:
	NoInherit() {}
};
```

*C++11*

C++11提供了`final`关键字，被`final`修饰的类不能被继承
```C++
class InheritBan final
{
    //...
};
```

# 设计一个类，只能创建一个对象(单例模式)
单例模式：
一个类只能创建一个对象，即单例模式，该模式可以保证系统中该类只有一个实例，并提供一个
访问它的全局访问点，该实例被所有程序模块共享。比如在某个服务器程序中，该服务器的配置
信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再
通过这个单例对象获取这些配置信息，这种方式简化了在复杂环境下的配置管理。

懒汉方式最核心的思想是 "延时加载". 从而能够优化服务器的启动速度.

### 饿汉方式实现
```C++
template <typename T>
class Singleton {
    static T data;
public:
    static T* GetInstance() {
        return &data;
    }
};
```
只要通过 `Singleton` 这个包装类来使用 T 对象, 则一个进程中只有一个`static T` 对象的实例.

### 懒汉方式实现
```C++
template <typename T>
class Singleton {
    static T* inst;//声明指针
public:
    static T* GetInstance() {
        if (inst == NULL) {
            inst = new T();
        }
        return inst;
    }
};
```
**存在一个严重的问题, 线程不安全**

因为`inst`是多个进程共享的资源，对其进行判空操作，也要用锁保护起来

**线程安全的版本实现**
```C++
// 懒汉模式, 线程安全
template <typename T>
class Singleton
{
    volatile static T *inst; // 需要设置 volatile 关键字, 否则可能被编译器优化->被放到寄存器中.
    static std::mutex lock;

public:
    static T *GetInstance()
    {
        if (inst == NULL)
        {                // 双重判定空指针, 降低锁冲突的概率, 提高性能.
            lock.lock(); // 使用互斥锁, 保证多线程情况下也只调用一次 new.
            if (inst == NULL)
            {
                inst = new T();
            }
            lock.unlock();
        }
        return inst;
    }
};

```

**特别注意**
1. 加锁解锁的位置
2. 双重if判定，避免不必要的锁竞争
3. `volatile`关键字防止过度优化







