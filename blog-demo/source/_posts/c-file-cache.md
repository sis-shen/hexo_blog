---
title: 文件缓冲区
date: 2024-07-26 10:16:19
tags:
---
前置博客 [基础IO](https://www.supdriver.top/2024/07/10/basicIO/)

# 为什么有缓冲
因为`磁盘的读写`与`内存的读写`操作速度相比，磁盘的读写是相差数量级的慢，所以为了提高内存**多次**，**频繁**读写磁盘文件的效率，`缓冲区`被投入使用。尤其是内存内容**写入**磁盘时，常常先写入`内存级缓冲区`，再在特定规则下一次性将`缓冲区`的内容写入磁盘

**本文以`C语言`提供的用户级缓冲区为例介绍缓冲区

# 缓冲区的刷新规则
首先当一个进程**正常退出**时，会先刷新缓冲区再关闭文件,此时必定有一次刷新

而当进程**运行时**缓冲区的刷新策略主要有以下三种

+ `无缓冲` 内容直接写入文件
+ `行缓冲` 输入一般内容不刷新，遇到`\n`时刷新一次缓冲区
+ `全缓冲` 缓冲区有容量限制，**满了**之后就刷新

# 认识一下C语言的缓冲区
*这里的系统环境是Linux*


## 刷新规则
运行如下代码
```C
#include <stdio.h>
#include <unistd.h>

int main()
{
    FILE* pfile = fopen("file.txt","w");//打开空文件

    fprintf(stdout,"stdout");//向标准输出输出
    fprintf(stderr,"strerr");//向标准错误输出输出
    fprintf(pfile,"file");//向文件输出
    _exit(0);//不刷新缓冲区，直接退出
    return ;
}
```

终端和文件的内容为:
```SHELL
stderr
```
```file.txt

```
可以看到只有`标准错误输出`有实际的输出,而`标准输出`和`文件输出`都没有输出

目前可以得出：
+ `标准错误输出`是`无缓冲`的刷新规则

因此我们再运行如下代码，再输出内容后面加上`\n`换行

```C
#include <stdio.h>
#include <unistd.h>

int main()
{
    FILE* pfile = fopen("file.txt","w");//打开空文件

    fprintf(stdout,"stdout\n");//向标准输出输出
    fprintf(pfile,"file\n");//向文件输出
    _exit(0);
    return ;
}
```
终端输出内容为
```SHELL
stdout

```
**而文件依然为空**

由此可得:
+ `标准输出`遵循`行缓冲`的刷新规则
+ `文件输出`遵循`全缓冲`的刷新规则

## 缓冲区在fork中的行为
```C
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main()
{
    printf("hello1 ");//父进程向标准输出打印一句话
    fprintf(stdout,"hello2 ");//父进程向标准输出打印一句话

    fork();
    return 0;
}
```
上段代码的输出内容为
```SHELL
hello1 hello2 hello1 hello2 
```
可见`fork`前的缓冲区内容被打印了两次（父子进程各一次），所以`fork`也会复制`缓冲区`的内容

实际上`缓冲区`属于进程的一部分，且`fork`时遵循`写时拷贝`

# 模拟一下C语言的文件接口（包括缓冲区）

## 主要目标
采用`Mystdio.h`声明，`Mystdio.c`实现的方式，封装`read`,`write`,`close`系统调用接口。并提供用户级缓冲区和缓冲区的刷新等功能

## 声明结构体和接口
我们先把主要的接口和主要的内容做出来看看封装效果
```C
#ifndef __MYSTDIO_H__ //利用预编译防止头文件被重复编译
#define __MYSTDIO_H__

#include <string.h>

//声明文件结构体
typedef struct IO_FILE{
    int fileno;
}_FILE;

_FILE * _fopen(const char *filename,const char *flag);
int _fwrite(_FILE* fp,const char*s, int len);
void _fclose(_FILE* fp);

#endif 
```

## 部分实现接口
实现的部分由`Mystdio.c`完成
### 头文件
这里的头文件要能够提供使用系统调用接口,以及调用堆区的接口,所以 头文件如下:

```C
#include "Mystdio.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
```

### _fopen函数
我们先模拟实现`fopen`函数的主要功能，主要实现`"w"``"a"``"r"`的打开模式

```C
#define FILE_MODE 0666 //设置默认的文件权限

_FILE * _fopen(const char *filename,const char *flag)
{
    int f = 0;//准备空位图
    int fd = -1;
    if(strcmp(flag,"w") == 0)
    {
        f = (O_CREAT|O_WRONLY|O_TRUNC);
        fd = open(filename,f,FILE_MODE);//打开文件
    }
    else if(strcmp(flag,"a") == 0)
    {
        f = (O_CREAT|O_WRONLY|O_APPEND);
        fd = open(filename,f,FILE_MODE);//打开文件
    }
    else if(strcmp(flag,"w") == 0)
    {
        f = O_RDONLY;
        fd = open(filename,f);//打开文件
    }
    else return NULL;//非法的打开模式

    if(fd == -1) return NULL;//打开失败
    
    _FILE *fp = (_FILE*)malloc(sizeof(_FILE));//创建_FILE结构体
    fp->fileno = fp;//设置_FILE结构体
    return fp;
}
```