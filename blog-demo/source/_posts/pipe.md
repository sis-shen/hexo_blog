---
title: 进程间通信--匿名管道与命名管道
date: 2024-08-02 21:16:14
tags: Linux
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409230859852.png
---

# 什么是管道文件
首先，管道是Unix中最古老的进程间通信的形式。它用于进程间的**单向**通信

那么具体是怎样实现的呢？从标题里就可以发现，是基于`文件`

既然一个文件可以被多个进程打开，那么不妨将文件作为两个进程通信的**媒介**。但是一般位于磁盘上的文件，IO效率相比于`CPU`，`内存`之类的读写速度慢了几个数量级，但文件是可以被加载到内存中的，而专门建立在**内存中**,而没有磁盘文件，专门用于进程间通信的**内存级**文件，我们就叫它`管道文件`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-09_21-19-22.png)

*管道文件由内核维护*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-10_17-52-20.png)
*管道文件是单向的，可以是父进程->子进程，也可以子进程->父进程*

## 管道读写规则
+ 写端未关闭，但读端**无数据**可读时
  + (默认)`O_NONBLOCK disable`：`read`调用阻塞，即进程暂停执行，一直等到有数据来到为止。
  + `O_NONBLOCK enable`：`read`调用返回`-1`，`errno`值为`EAGAIN`。
+ 读端未关闭，但写端写入管道已经**写满**时
  + (默认)` O_NONBLOCK disable`： `write`调用阻塞，直到有进程读走数据
  + `O_NONBLOCK enable`：`wrtie`调用返回`-1`，`errno`值为`EAGAIN`
+ 若写端关闭,则`read`返回`0`
+ 若读端关闭,则`write`操作会产生信号`SIGPIPE`,进而可能导致`write`进程**退出**

### 原子性
头文件提供了宏`PIPE_BUF`,规定了保证原子性读写操作的最大字节数

+ 当要写入的数据量不大于`PIPE_BUF`时，linux将保证写入的原子性。
+ 当要写入的数据量大于`PIPE_BUF`时，linux将不再保证写入的原子性。

## 管道特点

+ 管道提供流式服务
+ 一般而言，进程退出，管道释放，所以管道的生命周期随进程
+ 一般而言，内核会对管道操作进行**同步与互斥**
+ 一个管道只有一个通信方向，数据只能向一个方向流动；需要**双方**通信时，需要建立起**两个**管道

# 匿名管道
匿名管道主要用于`父子进程`间的通信

用到的接口是来自`<unistd.h>`的接口`pipe`

`int pipe(int pipefd[2]);`

可以看到有一个输出型参数`pipefd`数组，其中规定`pipefd[0]`储存了管道文件的`读端fd`,`pipefd[1]`储存了管道文件的`写端fd`

要利用管道通信时，必须用`close`一方关闭写端而另一端关闭读端

## 示例 子进程发送报文，父进程接受模型
```C
#include <iostream>
#include <cstdlib>
#include <unistd.h>
#include <string>
#include <cstring>
#include <sys/types.h>
#include <sys/wait.h>

#define N 2
#define NUM 1024

using namespace std;

//child
void Writer(int wfd)
{
    string s = "hello,I am child";//准备子进程的报文
    pid_t self = getpid();
    int number = 5;

    char buffer[NUM] = {0};
    while(number--)
    {
        buffer[0] = 0;//清空字符串
        snprintf(buffer,sizeof(buffer),"%s-%d-%d\n",s.c_str(),self,number);

        //发送/写入报文
        write(wfd,buffer,strlen(buffer));

        sleep(1);
    }
}

//father
void Reader(int rfd)
{
    char buffer[NUM];

    while(true)
    {
        buffer[0] = 0;
        ssize_t n = read(rfd,buffer,sizeof(buffer));
        if(n > 0)
        {
            buffer[n] = 0;//恢复成字符串
            cout<<"father get a message >> "<<buffer<<endl;
        }
    }
}

int main()
{
    int pipefd[N] = {0};
    
    int n = pipe(pipefd);
    if(n<0) return 1;

    pid_t id = fork();
    if(id  < 0) return 2;
    else if(id == 0)
    {
        //子进程
        close(pipefd[0]);//关闭读
        
        Writer(pipefd[1]);

        close(pipefd[1]);
        exit(0);
    }
    else
    {
        close(pipefd[1]);//关闭写

        Reader(pipefd[0]);

        pid_t rid = waitpid(id,nullptr,0);

        if(id <0) return 3;

        close(pipefd[0]);
    }

    return 0;
}
```

## 小小项目--进程池
详见[此博客🔗](https://www.supdriver.top/2024/08/02/processPool/)

# 命名管道
匿名管道无非实现不相关进程（无亲缘关系）的进程间通信，因此要用到命名管道来实现这个功能

命名管道文件,即`FIFO`文件，是一种用于不相关进程间通信的特殊类型的文件

## 命令行上创建
使用`mkfifo`可以在命令行上创建命名管道

例`mkfile fifo_file`

## 程序内创建和删除
使用接口`mkfifo`创建,注：要同时引用头文件`sys/types.h`和`sys/stat.h`

函数声明如下

`int mkfifo(const char *pathname, mode_t mode);`

+ 返回值：成功时返回`0`,失败时返回`-1`,并设置`errno`
+ `pathname`：文件名（当前目录），或者路径+文件名
+ `mode` 新创建的管道文件的权限,一般用`0644`或`0664`

使用`unlink`删除文件,引用自头文件`<unistd.h>`

`int unlink(const char *pathname);`

+ 返回值：成功时返回`0`,失败时返回`-1`,并设置`errno`
+ `pathname`：文件名（当前目录），或者路径+文件名

## 使用
由一个进程`写模式`打开，同时由另一个进程`读模式`打开，便可建立进程间通信，其余操作与匿名管道相同

## 命名管道的打开规则
+ `读模式`打开`FIFO`文件时
  + `O_NONBLOCK disable`：阻塞等待直到有相应进程为写而打开该`FIFO`
  + `O_NONBLOCK enable`：立刻返回成功
+ `写模式`打开`FIFO`文件时
  + `O_NONBLOCK disable`：阻塞直到有相应进程为读而打开该`FIFO`
  + ` O_NONBLOCK enable`：立刻返回失败，错误码为`ENXIO`

## 示例 命名管道实现 客户端 向 服务端通信

### makefile
小技巧：这里在最前面使用伪目标`all`，这样在使用`make`命令时能编译多个目标文件

```makefile
.PHONY:all
all:sever client

sever:sever.cpp
	g++ -o $@ $^ -std=c++11
client:client.cpp
	g++ -o $@ $^ -std=c++11

.PHONY:cl 
cl:
	rm -f sever client
```

### comm.hpp
使用同一个头文件能方便地统一`fifo`文件的路径，以及统一退出码的约定等

```C++
#pragma once

#include <iostream>
#include <errno.h>
#include <cstring>
#include <cstdlib>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define FIFO_NAME "./myfifo" //设置管道文件名
#define MODE 0664 //设置管道文件的权限

enum//枚举错误码
{
    FIFO_CREAT_ERR = 1,
    FIFO_DELETE_ERR,
    FIFO_OPEN_ERR
};
```

### sever.cpp

```C++
#include "comm.hpp"

using namespace std;

int main()
{
    //创建信道
    int n = mkfifo(FIFO_NAME,MODE);
    if(n == -1)//创建失败
    {
        perror("mkfifo");
        exit(FIFO_CREAT_ERR);
    }

    //打开信道
    int fd = open(FIFO_NAME,O_RDONLY);//只读模式打开FIFO
    if(fd == -1)//打开失败
    {
        perror("open");
        exit(FIFO_OPEN_ERR);
    }

    //开始通信
    cout<<"[@]sever start running"<<endl;
    while(true)
    {
        char buffer[1024] = {0};
        int sz = read(fd,buffer,sizeof(buffer)-1);//给结尾\0预留位置
        if(sz > 0)
        {
            buffer[sz] = 0;
            cout<<"client say# "<<buffer<<endl;
        }
        else if(sz == 0)
        {
            //写端关闭
            cout<<"client quit, sever will quie later..." <<endl;
            break;
        }
        else break;
    }

    //关闭信道
    close(fd);
    int m = unlink(FIFO_NAME);
    if(m == -1)
    {
        perror("unlink");
        exit(FIFO_DELETE_ERR);
    }

    return 0;
}
```

### client.cpp

```C++
#include "comm.hpp"

using namespace std;

int main()
{
    int fd = open(FIFO_NAME,O_WRONLY);//只读模式打开
    if(fd <0)
    {
        perror("open");
        exit(FIFO_OPEN_ERR);
    }

    string line;
    while(true)
    {
        cout<<"Please enter@ ";
        getline(cin,line);//获取输入

        write(fd,line.c_str(),line.size());
    }

    close(fd);
    return 0;
}

```
### 实现效果

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-13_16-49-49.png)

