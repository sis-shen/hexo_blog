---
title: 设计模式的C++实现(1)——单例模式
date: 2024-09-24 17:24:16
tags: 设计模式
---
# 单例模式介绍
> **模式名称**: 单例模式-Singleton
> **类型**: 创建型
> **问题-使用场景**: 希望一个类在全局只实例化出一个对象
> **解决方案**： 使用静态成员，将构造函数私有化，删除拷贝构造和`opetator=()`函数
> **效果**： 略微增加代码复杂性，使用了静态成员变量/函数。但是能够保证该类只会有一个实例。

**⼀个类只能创建⼀个对象，即单例模式**，该设计模式可以保证系统中该类只有⼀个实例，并提供⼀个访问它的全局访问点，该实例被所有程序模块共享。⽐如在某个服务器程序中，该服务器的配置信息存放在⼀个⽂件中，这些配置数据由⼀个单例对象统⼀读取，然后服务进程中的其他对象再通过这个单例对象获取这些配置信息，这种⽅式**简化了在复杂环境下的配置管理**。

单例模式有两种实现模式：`饿汉模式`和`懒汉模式`

## 饿汉模式
饿汉模式中，**程序启动时**就会创建⼀个唯⼀的实例对象。 因为单例对象已经确定， 所以⽐较适⽤于**多线程环境中**， 多线程获取单例对象不需要加锁， 可以有效的避免资源竞争,⾼性能。

```C++
template<typename T>
class Singleton {
private:
	static T _eton;
private:
	Singleton() {}//私有化构造函数
public:
	~Singleton() {}
	Singleton(const Singleton&) = delete;//删除拷贝构造
	Singleton& operator=(const Singleton&) = delete;//删除赋值运算重载
	static T& getInstance()
	{
		return _eton;
	}
};

template<typename T>
T Singleton<T>::_eton = T();
```

## 懒汉模式
懒汉模式:第⼀次使⽤要使⽤单例对象的时候创建实例对象。如果单例对象构造特别**耗时**或者**耗费济源**(加载插件、加载⽹络资源等)， 可以选择懒汉模式， 在第⼀次使⽤的时候才创建对象。

```C++
 //懒汉模式
template <typename T>
class Singleton {
private:
	Singleton() {}
	~Singleton() {}
public:
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;
	static T& getInstance()
	{
		static T _eton=T();
		return _eton;
	}
};
```

# 具体应用

+ [C++基于多设计模式下的同步&异步⽇志系统🔗](https://www.supdriver.top/2024/09/24/muiltiDesignPatternsLogSystem/)中对全局建造者使用单例模式