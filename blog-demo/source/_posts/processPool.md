---
title: 【Linux】简易进程池
date: 2024-08-02 20:02:48
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409230843742.png
---

# 原理
`匿名管道`可用于父子进程间的通讯，于是可以有父进程创建多个子进程形成`进程池`，并通过`匿名管道`文件向各个子进程`派发任务`

[戳我去管道原理🔗](https://www.supdriver.top/2024/08/02/pipe/)

这里使用C++编写进程池代码，在程序中创建多个进程，并在父进程中使用自定义类`channel`用于描述和管理子进程,然后在`task.hpp`中模拟一些任务给主程序随机派发
# 代码

## 代码规范
这次对形参有了新的规范,这里用`variable`代指形参名

+ 输入: `const &variable`
+ 输出: `*variable`
+ 输入输出: `&variable`

## 准备makefile文件
这里使用`g++`编译，规定语法标准为`C++11`
```makefile
processPool:processPool.cc
	g++ -o $@ $^ -std=c++11

.PHONY:clean
clean:
	rm -rf processPool
```

## 准备任务文件

首先要准备前后需要用到的头文件,然后构建模拟任务，并提供加载任务列表的函数
> `task.hpp`
```C++
#pragma once

#include <vector>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <cassert>
#include <iostream>
#include <string>


typedef void (*task_t)();//定义函数指针

void task1()
{
    std::cout << "PVZ 刷新日志" << std::endl;
}
void task2()
{
    std::cout<< "PVZ 生成阳光"<<std::endl;
}
void task3()
{
    std::cout<<"PVZ 检测更新" <<std::endl;
}
void task4()
{
    std::cout<<"PVZ 使用能量豆"<<std::endl;
}

void LoadTask(std::vector<task_t>*tasks)//载入任务列表的接口
{
    tasks->push_back(task1);
    tasks->push_back(task2);
    tasks->push_back(task3);
    tasks->push_back(task4);
}
```

## 编写主程序
主程序文件名为`processPool.cc`,我们按类和接口从上往下编程

### 描述和组织

我们把头文件准备好,然后定义最大进程数和创建任务列表,接着创建`channel`类来描述每个进程池中的子进程
```C++
#include "task.hpp"
#include <ctime>//获取时间戳
#include <sys/stat.h>
#include <sys/wait.h>


const int processNum = 5;//最大进程数
std::vector<task_t> tasks;//承载任务的映射列表

//先描述
class channel
{
public:
    channel(int cmdfd,pid_t slaverid,std::string& name)
    :_cmdfd(cmdfd),_slaverid(slaverid),_processname(name)
    {}

public:
    int _cmdfd;                    //发送任务的文件描述符
    pid_t _slaverid;               //子进程的id 
    std::string _processname;             //子进程的名字  --方便打印日志
};
```

### slaver函数
创建子进程后，子进程都进入`slaver`函数等待获取任务

这里统一采用`输入重定向`，在创建子进程后，进入`slaver()`前先把标准输入重定向到管道文件,然后从标准输入读取`任务码`

```C++
void slaver()
{
    while(true)
    {
        int cmdcode = 0;
        int n = read(0,&cmdcode,sizeof(int));//读取任务码
        if(n == sizeof(int))//成功获取任务码
        {
            //std::cout<<"child say@ "<<"get cmdcode: "<<cmdcode<<std::endl;//DEBUG
            if(cmdcode >=0 && cmdcode < tasks.size())//输入合法的输入码
            {
                tasks[cmdcode]();
            }
            else //输出错误信息
            {
                std::cout<<"wrong cmdcode: "<<cmdcode 
                    <<" max size: "<<tasks.size()<<std::endl;
            }
        }
    }
}
```

### InnitChannels
该函数用鱼创建子进程,创建子进程，并封装进频道类

```C++
void InitChannels(std::vector<channel>* channels)
{
    for(int i = 0;i<processNum;i++)
    {
        int pipefd[2];
        int n = pipe(pipefd);
        assert(!n);
        (void)n;//调用一下n

        pid_t id = fork();
        if(id == 0)//child
        {
            close(pipefd[1]);
            dup2(pipefd[0],0);//子进程输入重定向
            slaver();
            std::cout<<"proccess "<< getpid() << " quit"<<std::endl;
            exit(0);
        } 
        //father
        close(pipefd[0]);
        std::string name = "process-"+std::to_string(i);
        channels->push_back(channel(pipefd[1],id,name));
    }
}
```

### channelDEBUG
这里封装一个用于测试创建频道的`DEBUG`函数

```C++
void Debug(const std::vector<channel> channels)
{
    for(auto &c:channels)
    {
        std::cout<<c._cmdfd << " " << c._slaverid<< " " << c._processname << std::endl;
    }
}
```

### ctrlSlaver
这里封装一个`ctrlSlaver`用于控制子进程，也就是用于派发任务的函数

```C++
void ctrlSlaver(const std::vector<channel> &channels)
{
    for(int i = 0;i<10;i++)//随机派发10个任务
    {
        int cmdcode = rand() % tasks.size();//获取随机任务码
        int select_num = rand()%channels.size();//获取选择码
        std::cout<<"father say# "<<"taskcode: "<<cmdcode<<" send to "<<channels[select_num]._processname
            << std::endl;//输出日志
        write(channels[select_num]._cmdfd,&cmdcode,sizeof(cmdcode));//写入任务码，派发任务
        sleep(1);
    }
}
```

### QuitProcess
最后当然要有父进程控制关闭子进程，并回收僵尸进程

因为`InitChannels`中管道文件的处理比较粗糙，会发生如下图的关系，所以关闭时两步操作一定要放在两个循环中

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-13_11-25-01.png)

```C++
void QuitProcess(const std::vector<channel>& channels)
{
    //因为创建子进程时管道文件的连接没有进一步处理，关系复杂，所以下面两个循环一定要分开来
    for(const auto &c:channels)close(c._cmdfd);//关闭主进程管道文件的写端，使子进程的read函数读取失败，返回0
    for(const auto &c:channels)waitpid(c._slaverid,nullptr,0);//阻塞等待子进程，回收僵尸进程
}
```

### main函数
经过上文一系列封装,`main`函数就可以简洁明了地描述子进程的运行过程了

```C++
int main()
{
    srand(time(nullptr) ^ getpid()^1023);//种随机数种子
    std::vector<channel> channels;
    //1.初始化
    LoadTask(&tasks);//必须先载入任务再产生子进程，否则子进程看不到任务列表
    InitChannels(&channels);

    //测试函数
    //Debug(channels);

    //2.控制子进程
    ctrlSlaver(channels);

    //3.关闭进程池
    QuitProcess(channels);

    return 0;
}
```

# 源文件
[戳我去github仓库🔗](https://github.com/sis-shen/Linux_Code/tree/main/pipe_use)