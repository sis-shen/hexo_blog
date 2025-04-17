---
title: redis-plus-plus适用于C++的Redis API库使用探究
date: 2025-03-09 12:43:27
tags:
hidden: false
---

# Redis简介
Redis是一款开源的**内存型键值数据库**，支持多种数据结构（字符串、哈希、列表、集合、有序集合等），提供持久化、原子操作及高达10万+/秒的读写性能。主要应用于缓存、消息队列、会话存储、实时排行榜等场景。采用单线程模型保障操作原子性，同时支持主从复制、集群部署和高可用方案。

为了能在程序中调用Redis服务，我们使用`redis-plus-plus`库编写Redis的C++客户端
## redis-plus-plus安装
这里仅介绍Ubuntu安装

```shell
sudo apt update
# 安装前置依赖（C语言的库
sudo apt install -y libhiredis-dev 

# 安装redis-plus-plus只能源码编译了
git clone https://github.com/sewenew/redis-plus-plus.git

cd ./redis-plus-plus/
mkdir build
cd ./build/
cmake ..
make 
sudo make install
ldconfig
```
特别的，最后要再执行一下`ldconfig`指令防止程序运行时，找不到对应的动态库。我在使用`docker`容器时遇到了这样的问题。QWQ
## 编译链接选项
若在代码中使用了`redis-plus-plus`库，则需要在编译时同时假如连接选项`-lredis++`和`-lhiredis`

```shell
g++ main.cpp -lredis++ -lhiredis
```

# 简单使用
## 关于命名空间
该库的大部分代码位于命名空间`sw::redis::`中，其中`sw`是作者名字的缩写

## 创建客户端对象
我们所有的`Redis`操作都是通过`Redis客户端`的接口向服务端发起网络请求来完成的，所以我们要先创建客户端对象才能进行后序的操作

我们的步骤如下
1. 创建`连接选项对象`，填入相关参数
2. 使用`连接选项对象`创建客户端对象

```C++
sw::redis::ConnectionOptions opts;
opts.host = ip;
opts.port = port;
opts.db = db;
opts.keep_alive = keep_alive;

sw::redis::Redis redis(opts);
```
## 使用Redis指令
该库中大部分的接口都是和`redis-cli`命令行客户端的指令是**同名的**。

例如我们使用字符串指令
```C++
redis.set("mykey","myvalue");

sw::redis::OptionalString res = _redis->get("mykey");
if(!res.has_value())
{
    std::cout<<"key不存在！"<<std::endl;
}
else
{
    std::string value = res.value();
}
```

## OptionalString
`redis-plus-plus库`提供了`OptionalString`类型，专门用来处理这种返回值可能不存在的情况。

我们先看一下这个`OptionlaString`底层用啥实现的:
```C++

template <typename T>
using Optional = std::optional<T>;

using OptionalString = Optional<std::string>;
```
可以看到，其实它就是特化了的`std库`中的`std::optional<T>`，模板参数固定为`std::string`类型

为什么要有Optional类型？因为对于一些结果不存在的情况，可以选择抛异常，可是对于即使结果不存在也不影响程序执行的情况，抛异常过于**小题大做**了，因此我们选择使用更温柔的处理方式：返回`Optional`类型。这样在处理结果时，先判断结果是否存在(调用has_value()接口),如果不存在，进入一个分支，如果存在，则调用`value()`接口进入另一个执行分支。

## 批量执行

该库也提供了接口将`STL容器`的内容批量插入到Redis服务中，或者从Redis服务中批量获取数据并存储到STL容器中。

## 批量插入
这类接口用起来比较简单，传入一个`key`和`开始接迭代器`和`结束迭代器`即可

```C++
std::unordered_map<string,string> m;
redis->hmset("hash_key",m.begin(),m.end())
```

## 批量插入
这里要用到`STL库`提供的`插入器类`,即`std::inserter`

```C++
std::unordered_map<string,string> m;
redis->hgetall("hash_key",std::inserter(m,m.begin()));
```



