---
title: 初识进程
date: 2024-07-04 23:02:14
tags: 进程 Linux
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_23-41-09.png
---

***
操作系统平台:Linux
服务器系统: CentOS 7
***

# 概念抽象

## 程序

`程序` = `代码` + `数据`

程序是储存在硬盘上的可执行文件

## 进程
将`程序`加载到`内存`后，就在`内存`中程序的就是进程。也就是说一个正在运行的程序就能叫做进程

*结构关系如下*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-06_13-55-08.png)

如图，操作系统为了**管理**内存中的进程,使用了`PCB`结构体来描述进程,通过管理`PCB`来管理进程,依然是*先描述再组织*

`PCB`:进程控制块的数据结构(process control block)

所以实际上:`进程`=`PCB`+`代码和数据`

对于`代码和数据`没什么好说的，接下来主要讨论`PCB`

## task_struct
`Linux`平台下的`PCB`叫做`task_struct`

> `task_struct`内容分类:

+ **标示符**: 描述本进程的唯一标示符，用来区别其他进程。
+ **状态**: 任务状态，退出代码，退出信号等。
+ **优先级**: 相对于其他进程的优先级。
+ 程序计数器: 程序中即将被执行的下一条指令的地址。
+ 内存指针: 包括程序代码和进程相关数据的指针，还有和其他进程共享的内存块的指针
+ 上下文数据: 进程执行时处理器的寄存器中的数据。
+ I／O状态信息: 包括显示的I/O请求,分配给进程的I／O设备和被进程使用的文件列表。
+ 记账信息: 可能包括处理器时间总和，使用的时钟数总和，时间限制，记账号等。
+ 其他信息
  
*加粗部分会详细介绍*

# 查看进程
进程的信息可以通过/proc 系统文件夹查看,其中文件夹的名字就是进程的`PID`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-06_21-53-59.png)

大多数进程信息同样可以使用top和ps这些用户级工具来获取

> 以我自己编写的一个程序为例

*Makefile*
```
mycmd:mycmd.c
	gcc -o $@ $^

.PHONY:clean
clean:
	rm -rf mycmd
```

**注**： 后面的程序都是这三个头文件，仅修改`main()`函数体即可
*mycmd.c*
```C
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>


int main()
{
  while(1) sleep(1);//死循环使该进程常驻内存
  return 0;
}
```
然后编译并运行程序
*命令行*
```
make
./mycmd
ps aux | grep mycmd | grep -v grep
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-06_22-56-05.png)

## 进程标示符(PID PPID)
可以通过系统接口获取进程标示符

+ 进程id (pid)    使用`getpid()`
+ 父进程id (ppid) 使用`getppid()`
  
*来修改一下代码*
*mycmd.c*
```C

int main()
{
  printf("pid: %d\n",getpid());//打印pid (该进程id)
  printf("ppid: %d\n",getppid());//打印ppid (父进程id)
  return 0;
}
```

*命令行*
```
make clean
make
./mycmd
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-06_23-28-35.png)

## 利用`fork()`创建子进程

*mycmd.c*
```C

int main()
{
  pid_t id = fork();
  if(id == 0)//子进程
  {
    printf("child pid: %d\n",getpid());//打印pid (该进程id)
    printf("child ppid: %d\n",getppid());//打印ppid (父进程id)
    sleep(1);
  }
  else//父进程
  {
    printf("father pid: %d\n",getpid());//打印pid (该进程id)
    printf("father ppid: %d\n",getppid());//打印ppid (父进程id)
    sleep(1);
  }
  return 0;
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_10-56-48.png)

如图，`fork()`创建了子进程，且子进程的`PPID`和父进程`PID`相同

### fork()的返回值
父子进程中`fork()`函数的返回值(此处用变量`id`储存)是不同的:

**父进程**里`id`的值为子进程的`PID`,其值`>0`;**子进程**里`id`值固定为`0`

+ `id > 0` 父进程
+ `id == 0` 子进程
+ `id < 0` fork()失败

### 父子进程分流
利用`fork()`返回值不同的特性可以做到分流操作，利用`if...else...`让父子进程执行不同的代码

[戳我去fork的详细介绍](https://www.supdriver.top/2024/07/07/fork/)

# 进程状态 
*状态在kernel源代码里定义*
```C
 static const char * const task_state_array[] = {
 "R (running)", /* 0 */
 "S (sleeping)", /* 1 */
 "D (disk sleep)", /* 2 */
 "T (stopped)", /* 4 */
 "t (tracing stop)", /* 8 */
 "X (dead)", /* 16 */
 "Z (zombie)", /* 32 */
 };
```

## R 运行状态(running)
R状态并不一定正在运行，而是`正在运行`和`处于运行队列`中的一种

```C
int main()
{
  while(1);
  return 0;
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_13-14-49.png)

## S 睡眠状态(sleeping)
S 意味着进程在**等待**运行完成

(*这里的睡眠有时也可叫做可中断睡眠 interruptible sleep*)

*下面展示两种S状态的进程*
```C
int main()
{
  sleep(50);
  return 0;
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_13-45-14.png)

直接使用`sleep()`系列的函数直接使进程休眠

```C
int main()
{
  int n;
  printf("Enter the num: ");
  scanf("%d",&n);
  return 0;
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_13-47-33.png)

像`scanf()`这种需要等待外设(键盘)的接口，在**阻塞等待**资源的过程中会使进程进入`S`状态

## D 磁盘休眠状态（Disk sleep）
有时候也叫不可中断睡眠状态（uninterruptible sleep），在这个状态的进程通常会等待`IO`的结束。

## T 停止状态（stopped）
可以通过(*kill等命令*)发送 `SIGSTOP` 信号给进程来停止（T）进程。这个被暂停的进程可以通过发送 `SIGCONT `信号让进程继续运行。

```C
int main()
{
  while(1) sleep(1);
  return 0;
}
```
运行前先**复制**ssh渠道，其中一个窗口用于执行进程
```
make clean
make
./mycmd
```
另一个进程用于输入命令

先查看该进程的`PID`
```
ps ajx | head -1 && ps ajx | grep mycmd | grep -v grep
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_15-04-51.png)

如图，这次的`PID`是`20275`

然后用`kill`发送`SIGSTOP`,对应参数为`-19`

```
kill -19 20275
ps ajx | head -1 && ps ajx | grep mycmd | grep -v grep
```
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_15-06-38.png)

可以看到它已经由`S`状态改为`T`状态了

接下来发送`SIGCONT`,对应参数`-18`,使进程恢复

```
kill -18 20275
ps ajx | head -1 && ps ajx | grep mycmd | grep -v grep
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_15-08-45.png)

可以看到已经由`T`变为原本的`S`状态了

## X 死亡状态（dead）
这个状态只是一个返回状态，你不会在任务列表里看到这个状态。

## Z 僵尸进程(zombie)
这个详细讨论下

### 产生
当该进程退出后，父进程尚未使用`wait()`之类的接口获取该进程的`退出码`,且父进程**没有结束**时，该进程会变成僵尸进程

*父进程比子进程先退出时，子进程的父进程会改变为PID为1的进程，由新进程托管*

*下面创建一个例子*

```C
int main()
{
  pid_t id = fork();
  if(id == 0)//子进程
  {
    printf("child exit\n");
  }
  else//父进程
  {
    sleep(30);
    printf("father exit\n");
  }
  return 0;
}
```

运行程序后的`30秒`内查看进程状态，可以看到子进程进入了`Z`状态

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_16-27-27.png)

### 行为
僵尸进程会以终止状态保持在进程表中，等待父进程读取退出状态代码

### 危害
+ 父进程一直**未获取**子进程的退出码,僵尸状态就会一直保持
+ 保持`Z`状态的进程的`PCB`仍然要一直维护，占用资源
+ 未退出`Z`状态的子进程可能造成内存泄漏

## 孤儿进程
当父进程比子进程**先**退出后,这个子进程便成了`孤儿进程`

既然原本的父进程没了，谁来托管子进程呢？答案是`PID`为`1`的那个进程

*例子如下*

```C
int main()
{
  pid_t id = fork();
  if(id == 0)//子进程
  {
    sleep(30);
    printf("child exit\n");
  }
  else//父进程
  {
    sleep(1);
    printf("father exit\n");
  }
  return 0;
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_19-05-32.png)

# 进程优先级

## 基础概念
+ cpu资源分配的先后顺序，就是指进程的优先权（priority）。
+ 优先权高的进程有优先执行权利。配置进程优先权对多任务环境的linux很有用，可以改善系统性能。
+ 还可以把进程运行到指定的CPU上，这样一来，把不重要的进程安排到某个CPU，可以大大改善系统整体性能

## 查看优先级
*首先写一个常驻进程*
```C
int main()
{
  while(1)sleep(1);
  return 0;
}
```

然后使用`ps -la`查看进程信息
```
./mycmd
ps -la
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_19-59-12.png)

其中的`PRI`和`NI`与进程优先级有关,`PRI`就是进程的优先级，跟排队摇号一样，此值`越小`，被执行的优先级越高,而`NI`就是nice值，用于修正原`PRI`值

### PRI值的计算
首先在看到的`PRI`值之外，还有个隐藏的基准值，本文用`PRI0`指代，这个`PRI0`是固定的，当`NI`值为`0`时，`PRI == PRI0`,而
无论怎么修改多少次`NI`,`PRI`的值减去`NI`值都相等，所以大可以推断在本系统(Linux)中,`PRI`值有如下计算公式

`PRI = PRI0 + NI`

### 修改NI值
因为修改`NI`值要管理员权限，所以要么`root`用户用`top`，要么信任用户用`sudo top`打开界面，然后按`r`,输入待修改进程的`PID`,按下回车后再输入新的`NI`值(有效范围`-20~19`)

*此处可以用`ps -la`查看进程的PID,或调用`getpid()`*

*再写一个例子*
```C
int main()
{
  while(1)
  {
    printf("mypid: %d\n",getpid());
    sleep(1);
  }
  return 0;
}
```
`sudo top`然后按`r`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_21-14-38.png)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_21-15-50.png)

`ps -la`可以看到被修改后的进程优先级

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-07_21-17-02.png)

# 其它概念

+ 竞争性: 系统进程数目众多，而CPU资源只有少量，甚至1个，所以进程之间是具有竞争属性的。为了高效完成任务，更合理竞争相关资源，便具有了优先级
+ 独立性: 多进程运行，需要独享各种资源，多进程运行期间**互不干扰**
+ 并行: 多个进程在**多个CPU**下分别，同时进行运行，这称之为并行
+ 并发: 多个进程在**一个CPU**下采用进程切换的方式，在一段时间之内，让多个进程都得以推进，称之为并发

*下一章*[环境变量](https://www.supdriver.top/2024/07/06/evn/)
