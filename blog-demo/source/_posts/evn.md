---
title: 环境变量
date: 2024-07-06 21:36:21
tags: 环境变量
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-25_22-38-03.png
---

# 基本概念
+ 环境变量(environment variables)一般是指在操作系统中用来指定操作系统运行环境的一些**参数**
+ 环境变量通常具有某些特殊用途，还有在系统当中通常具有**全局特性**

**构成**：环境变量是一系列`字符串`的统称,所以一个环境变量由`变量名`和`值`构成

这么说还是太抽象了，我们接下来会举几个具体样例，体会`环境变量`在获取系统全局的变量，系统指令路径等方面的作用

## 常见环境变量

| 变量名 | 功能 |
| ------- | --------|
| `PATH` | 指定命令的搜索路径 |
| `HOME` | 指定用户的主工作目录(即用户登陆到Linux系统中时,默认的目录) |
| `USER` | 当前用户名 |
| `SHELL` | 当前Shell,它的值通常是`/bin/bash` |
| `PWD` | 当前工作目录

# 操作系统变量

## 查看环境变量
`echo $NAME` *NAME为变量名*

以`PATH`为例,查看`PATH`的值的指令为

```SHELL
echo $PATH
```

https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-08_12-46-35.png

可以看到`PATH`的内容为多个文件路径，互相以`:`分隔

而若要查看当前的全部环境变量,可以使用`env`指令,将当前所有环境变量打印在终端上

也可以通过`管道`和`grep`将`env`的输出内容过滤

*查看PATH*
```SHELL
env | grep PATH
```

同时还有个`set` 指令可以显示本地定义的shell变量和环境变量环境变量的组织方式 


## 获取进程的环境变量
+ 在程序中，可以使用`getenv()`接口获取对应`环境变量名`的对应环境变量值

*`getenv`在`<stdlib.h>`中*
```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("PATH: %s\n",getenv("PATH"));//打印PATH的值
    return 0;
}
```

## 环境变量的组织方式
每个`进程`都有自己的`环境表`,所谓`环境表`就是一个**字符**`指针数组`,每个不为`NULL`的指针指向`环境字符串`

https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-08_20-52-19.png

+ 因此也可以使用`main`函数传参来打印所有环境变量的内容
```C
int main(int argc,char* argv[],con char* env[])
{
    char* str;
    int i = 0;
    while(str =env[i++]) printf("%s\n",str);
    return 0;
}
```

## 导入环境变量
头文件`unistd.h>`提供了一个全局变量`environ`,因此可以用`extern`导入环境变量

```C
#include <stdio.h>
#include <unistd.h>

extern char **environ;//导入换进变量

int main()
{
    const char* str;
    int i = 0;
    while(str =environ[i++]) printf("%s\n",str);
    return 0;
}
```

## 添加新的环境变量
使用`export`命令可以添加新的环境变量

使用方法:`export``=``(变量值)`

例如添加一个值为`supdriver666`,名为`MY_VALUE`的环境变量,最后查看该变量

```SHELL
export MY_VALUE=supdriver666

env | grep MY_VALUE
```

## 程序内设置换进变量
使用`<stdlib.h>`提供的`putenv()`接口可以设置环境变量，用法与`export`指令相同

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    putenv("MY_VALUE=666");
    printf("MY_VALUE = %s",getenv("MY_VALUE"));
    return 0;
}
```


## 删除环境变量
使用`unset`可以删除环境变量

*删除上文的`MY_VALUE`*
```SHELL
unset MY_VALUE

env | grep MY_VALUE
```
可以看到没有输出了 

## 添加本地shell变量 和 查看本地变量与环境变量
添加本地变量: 直接输入`(变量名)``=``(值)`  //*不加空格*/

查看变量: 使用`set`命令查看本地变量与环境变量，但是内容非常多，建议搭配`grep`等使用

*例*
```SHELL
my_value=2024

set

set | grep my_value
```

## 本地变量与环境变量
二者最大的差别是`环境变量`可以被子进程**继承**,而`本地变量`只在本BASH内部有效，**不会**被继承


