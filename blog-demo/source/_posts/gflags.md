---
title: gflags命令行参数解析库使用探究
date: 2025-03-06 09:10:27
tags:
---
# gflags库介绍
`gflags`是`Google`开发的一个开源库，用于`C++`应用程序中**命令行参数**的声明、定义和解析。`gflags`库提供了一种简单的方式来`添加`、`解析`和`文档化`命令行标志（`flags`），使得程序可以根据不同的运行时配置进行调整。

## 为什么用gflags
它有如下优点:
+ **易于使用**: `flags`提供了一套简单直观的`API`来定义和解析命令行标志，使得开发者可以轻松地为应用程序添加新的参数。
+ **自动帮助和文档**：`gflags`可以**自动生成**每个标志的帮助信息和文档(内容由开发者填充)，这有助于用户理解如何使用程序及其参数。
+ **类型安全**：`gflags`支持多种数据类型的标志，包括`布尔值`、`整数`、`字符串`等，并且提供了**类型检查和转换**。
+ **多平台支持**：`gflags` 可以在多种操作系统上使用，包括`Windows`、`Linux`和`macOS`
+ **可扩展性**：`gflags`允许开发者**自定义**标志的注册和解析逻辑，提供了强大的扩展性。

**官方文档**: [https://gflags.github.io/gflags/](https://gflags.github.io/gflags/)
**代码仓库**: [https://github.com/gflags/gflags.git ](https://github.com/gflags/gflags.git)

## gflags安装
目前只介绍Ubuntu环境下的安装

### apt安装

```shell
sudo apt update
sudo apt install -y libgflags-dev
```

## 源码安装

```shell
# 下载源码 
git clone https://github.com/gflags/gflags.git 
# 切换目录 
cd gflags/
mkdir build 
cd build/ 
# 生成Makefile 
cmake .. 
# 编译代码 
make 
# 安装 
make install 
```

# gflags使用
目前用到的函数来自于下面的头文件
```C++
#include <gflags/gflags.h>
```

**编译时要添加 -lgflags 链接库**
## 声明参数
`gflags`提供形如`DEFINE_*`来声明各种类型的命令行参数，其参数如下:
+ `name`: 参数名称，就和声明变量一样，不需要加引号
+ `val`: 默认值，当不导入配置文件或者通过修改命令行参数时，所使用的默认值
+ `txt`: 字符串，用于填写该**参数的帮助信息**
+ `返回值`: 不关心

各种支持声明的类型

```C++
DEFINE_bool
DEFINE_int32
DEFINE_int64
DEFINE_uint64
DEFINE_double
DEFINE_string
```
## 访问参数
出于突出显示命令行参数的目的，同时也可以避免冲突，所有在`DEFINE_*`中声明的变量名（例如）`name`，在其作用域内都被统一加上了`FLAGS_`前缀，变成了`FLAGS_name`。

代码示例如下

```C++
#include <iostream>
#include <gflags/gflags.h>

//声明参数
DEFINE_string(ip, "127.0.0.1", "这是服务器的监听IP地址，格式：127.0.0.1");
DEFINE_int32(port, 8080, "这是服务器的监听端口, 格式: 8080");
DEFINE_bool(debug_enable, true, "是否启用调试模式, 格式：true/false");

int main(int argc, char *argv[])
{
    google::ParseCommandLineFlags(&argc, &argv, true);//解析命令行参数

    std::cout << FLAGS_ip << std::endl;//访问参数时要加上FLAGS_前缀
    std::cout << FLAGS_port << std::endl;
    std::cout << FLAGS_debug_enable << std::endl;
    return 0;
}
```
输出结果如下图:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503061410745.webp)

### 不同文件访问参数
`glfags库`提供了`DECLEAR_*`系列函数来实现类似`extern`声明的功能，使之支持声明和实现分离，或者通过链接.o文件的方式访问其它源文件内的全局变量,例子如下

```C++
//////////////////////////
//main.cpp 
//////////////////////////

#include <iostream>
#include <gflags/gflags.h>
//只声明
DECLARE_string(ip);
DECLARE_int32(port);
DECLARE_bool(debug_enable);


int main(int argc, char *argv[])
{
    google::ParseCommandLineFlags(&argc, &argv, true);//解析命令行参数

    std::cout << FLAGS_ip << std::endl;//访问参数时要加上FLAGS_前缀
    std::cout << FLAGS_port << std::endl;
    std::cout << FLAGS_debug_enable << std::endl;
    return 0;
}

//////////////////////////////
/// header.cpp
//////////////////////////////
#include <gflags/gflags.h>
//声明参数
DEFINE_string(ip, "127.0.0.1", "这是服务器的监听IP地址，格式：127.0.0.1");
DEFINE_int32(port, 8080, "这是服务器的监听端口, 格式: 8080");
DEFINE_bool(debug_enable, true, "是否启用调试模式, 格式：true/false");
```

## 解析命令行参数
为了使`FLAGS_*`变量能够成功地被命令行参数赋值，需要在`main`函数中调用解析函数

```C++
uint32 ParseCommandLineFlags(int *argc, char*** argv, bool remove_flags);
```

+ `arc`和`argc`: `main`函数的入口参数的地址，传入时加个`&`即可
+ `remove_flags`
  + `true`: 会从 `argv` 中移除标识和它们的参数，相应减少 `argc` 的值。
  + `false`:会保留argc 不变，但将会重新调整它们的顺序，使得标识再前面

## 命令行参数设置
`gflags`提供了多种在命令行中配置参数的方式,以编译获得的可执行程序`cmd`为例

+ `string`和`int`的设置方式
```shell
cmd --ip="0.0.0.0" 
cmd -ip="0.0.0.0" 
cmd --ip "0.0.0.0" 
cmd -ip "0.0.0.0" 
```
+ `bool`的设置方式
```shell
cmd --debug_enable 
cmd --nodebug_enable
cmd --debug_enable=true 
cmd --debug_enable=false
```

## 配置文件设置
可以通过`配置文件`对参数设置进行文件持久化。配置文件的编写方式十分简单，形式就是`-key=value`的样子

使用方法则是在执行时添加参数`--flagfile=filepath`

示例文件 myconf.conf
```conf
-ip=0.0.0.0
-port=8888
-debug_enable=flase
```
输出结果如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503061514561.webp)

## 特殊参数标识
`gflags`默认提供了一些特殊的参数标识

```shell
--help  # 显示文件中所有标识的帮助信息 
--helpfull  # 和-help 一样, 帮助信息更全面一些 
--helpshort  # 只显示当前执行文件里的标志 
--helpxml  # 以 xml 方式打印，方便处理 
--version  # 打印版本信息，由 google::SetVersionString()设定 
--flagfile  #从指定文件中读取命令行参数 
```

# 代码示例

> main.cpp
```C++
#include <iostream>
#include <gflags/gflags.h>

//声明参数
DEFINE_string(ip, "127.0.0.1", "这是服务器的监听IP地址，格式：127.0.0.1");
DEFINE_int32(port, 8080, "这是服务器的监听端口, 格式: 8080");
DEFINE_bool(debug_enable, true, "是否启用调试模式, 格式：true/false");


int main(int argc, char *argv[])
{
    google::ParseCommandLineFlags(&argc, &argv, true);//解析命令行参数

    std::cout << FLAGS_ip << std::endl;//访问参数时要加上FLAGS_前缀
    std::cout << FLAGS_port << std::endl;
    std::cout << FLAGS_debug_enable << std::endl;
    return 0;
}
```

> myconf.conf
```conf
-ip=0.0.0.0
-port=8888
-debug_enable=false
```

> makefile
```makefile
cmd:main.cpp
	g++ -o $@ $^ -lgflags

.PHONY:clean
clean:
	rm -f cmd
```

## 各种运行结果

### 默认
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503061523361.webp)

### 指定参数
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503061523797.webp)

### 指定配置文件
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503061524772.webp)