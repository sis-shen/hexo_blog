---
title: Linux原生线程 与 互斥锁
date: 2024-08-14 14:55:39
tags:
---

# 什么是线程

在一个程序（进程）里的一条执行流就叫做`线程`(thread),也就是说有多线程功能的进程内，可以有多个线程同时执行

所以我们可以认为:
+ 一个进程至少有一个执行进程
+ 线程在进程内部运行，本质是在进程提供的地址空间内运行

而对于`Linux`实现的线程，本质上是轻量化的进程，还是用的`task_struct`去维护的每一个**线程**

![Linux进程结构](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread.png)

**关于线程间内存共享**
如上图所示，线程之间只有`栈区`是相互**独立**的， 像是`全局变量`，`堆区数据`都是共享的

## 线程的优点
+ 创建一个新线程的`代价`要比创建一个新进程小得多
+ 与进程之间的`切换`相比，线程之间的切换需要操作系统做的`工作`要少很多
+ 线程占用的`资源`要比进程少很多
+ 能充分利用多处理器的可`并行`数量
+ 在等待慢速`I/O`操作结束的同时，程序可执行其他的计算任务
+ 计算密集型应用，为了能在多处理器系统上运行，将`计算分解`到多个线程中实现
+ `I/O密集型应用`，为了提高性能，将I/O操作重叠。线程可以同时等待不同的I/O操作。

## 线程的缺点

### 性能损失
*一个很少被外部事件阻塞的计算密集型线程往往无法与共它线程共享同一个处理器。如果计算密集型线程的数量比可用的处理器多，那么可能会有较大的性能损失*，这里的`性能损失`指的是增加了额外的
`同步`和`调度`开销，而**可用的资源不变。**

### 健壮性降低
编写多线程需要更全面更深入的考虑，在一个多线程程序里，因时间分配上的细微偏差或者因共享了不该共享的变量而造成不良影响的可能性是很大的，换句话说线程之间是缺乏保护的。

### 缺乏访问控制
`进程`是访问控制的`基本粒度`，在一个线程中调用某些OS函数会对整个进程造成影响。

### 编程难度提高
编写与调试一个多线程程序比单线程程序困难得多

## 线程异常
所有线程好比铁索连环，如果单个线程出现诸如 `除零`，`野指针`等异常问题导致线程崩溃,整个`进程`，包括所有线程，都会崩溃退出

## 线程的用途
+ 合理的使用多线程，能提高CPU密集型程序的执行效率
+ 合理的使用多线程，能提高IO密集型程序的用户体验

# 线程与进程辨析

## 进程
资源分配的基本单位,即每个进程都会分配一套独立的进程地址空间/虚拟地址，并且供内部的所有线程共享

## 线程
线程是调度的基本单位，线程共享进程数据，但也拥有自己的一部分数据<span id = "qianwen"></span>

+ `线程ID`
+ 一组寄存器
+ 栈
+ errno
+ 信号屏蔽字
+ 调度优先级

进程的多个`线程共享` 同一地址空间,因此`Text Segment`、`Data Segment`都是共享的,如果定义一个函数,在各线程
中都可以调用,如果定义一个全局变量,在各线程中都可以访问到,除此之外,各线程还共享以下进程资源和环境

+ 文教描述符表
+ 每种信号的处理方式(SIG_IGN、SIG_DFL或自定义的信号处理函数)
+ 当前工作目录
+ 用户id和组id

# Linux线程控制
这里介绍POSIX线程库，这是一个第三方线程库，且是个动态库,有以下特点

+ 与线程有关的函数构成了个完整的系列，绝大多数函数的名字都是以`pthread_`打头的
+ 要使用这些函数库，要通过引入头文`<pthread.h>`
+ 链接这些线程函数库时要使用**编译器**命令的`-lpthread`选项  *很容易忘*

## 创建线程
```C++
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);
```
使用`pthread_create`函数创建新线程

+ 参数
  + thread:输出型参数，返回新线程的`线程id`
  + attr:设置线程属性，为`nullptr`时使用默认属性，一般为`nullptr`
  + start_routine:函数地址，即新线程**启动时调用**的函数
  + arg:传给`start_routine`指向函数的参数
+ 返回值:成功返回`0`，失败返回错误码

关于错误码
> + 统的一些函数是，成功返回0，失败返回-1，并且对全局变量errno赋值以指示错误。
> + threads函数出错时不会设置全局变量errno（而大部分其他POSIX函数会这样做）。而是将错误代码通过返回值返回
> + hreads同样也提供了线程内的errno变量，以支持其它使用errno的代码。对于pthreads函数的错误,建议通过返回值业判定，因为读取返回值要比读取线程内的errno变量的开销更小

> 使用示例
```C++
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#include <string.h>
#include <pthread.h>
void *rout(void *arg)
{
    int i;
    for (;;)
    {
        printf("I'am thread 1,got arg:%d\n",*(int*)arg);
        sleep(1);
    }
}
int main(void)
{
    pthread_t tid;//储存tid的变量
    int ret;
    int arg = 114514;//准备参数
    //创建新线程
    if ((ret = pthread_create(&tid, NULL, rout, &arg)) != 0)
    {
        //创建失败
        fprintf(stderr, "pthread_create : %s\n", strerror(ret));
        exit(EXIT_FAILURE);
    }
    int i;
    for (;;)
    {
        //主线程输出
        printf("I'am main thread\n");
        sleep(1);
    }
}
```

## 线程id及进程地址空间布局
+ hread_create函数会产生一个线程id，存放在第一个参数指向的地址中。该线程id和前面说的线程ID(本文用大小写区分)不是一回事。
+ [前文](#qianwen)的`线程ID`于进程调度的范畴。因为线程是轻量级进程，是操作系统调度器的最小单位，所以需要一个数值来唯一表示该线程。
+ hread_create函数第一个参数指向一个虚拟内存单元，该内存单元的地址即为新创建线程的线程ID,属于NPTL线程库的范畴。线程库的后续操作，就是根据该线程ID来操作线程的。
+ 程库NPTL提供了pthread_self函数，可以获得线程自身的ID:

`pthread_t` 到底是什么类型呢？取决于实现。对于Linux目前实现的NPTL实现而言，pthread_t类型的线程ID，本质就是一个进程地址空间上的一个地址。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409091528968.png)

## 线程终止
如果需要只终止某个线程而不终止整个进程,可以有三种方法:

+ 从线程函数`return`。这种方法对主线程**不适用**,从main函数return相当于调用exit。
+ 线程可以调用`pthread_exit`终止自己。
+  一个线程可以调用`pthread_cancel`终止同一进程中的另一个线程。

### pthread_exit函数
`void pthread_exit(void *retval);`

**由子线程本身执行**

`retval`为输出型参数，而函数调用后，该线程退出，栈帧销毁，所以,需要注意,pthread_exit或者return返回的指针所指向的内存单元必须是`全局`的或者是用`malloc`分配的,不能在线程函数的栈上分配,因为当其它线程得到这个返回指针时线程函数已经退出了。

### pthread_cancel函数
`int pthread_cancel(pthread_t thread);`

**由主线程向目标线程使用**

用于取消一个正在执行中的线程。

## 线程等待

为什么需要线程等待？*这一点可以类比多进程中的进程等待*

+ 已经退出的线程，其空间没有被释放，仍然在进程的地址空间内。
+ 创建新的线程不会复用刚才退出线程的地址空间。
+ 进程等待可以回收已退出线程的资源

### pthread_join函数
`int pthread_join(pthread_t thread, void **retval);`

+ `thread`:线程id
+ `retval`:输出型参数，随着线程的结束方式的不同而改变
+ `返回值`: 成功返回`0`，失败返回错误码

调用该函数的线程将`挂起等待`,直到目标`thread`线程终止。

**`retval`详解**
thread线程以不同的方法终止,通过pthread_join得到的终止状态是不同的，总结如下:

1. `return返回`:如果thread线程通过return返回,value_ ptr所指向的单元里存放的是thread线程函数的`返回值`。
2. `pthread_cancel`:如果thread线程被别的线程调用`pthread_ cancel异常终止`,value_ ptr所指向的单元里存放的是常数`PTHREAD_ CANCELED`。
3. `pthread_exit`:如果thread线程是自己调用pthread_exit终止的,value_ptr所指向的单元存放的是传给pthread_exit的参数。
4. `不感兴趣`:如果对thread线程的终止状态不感兴趣,可以传`NULL`给value_ ptr参数。

## 分离线程
默认情况下，新创建的线程是`joinable`的，即线程退出后，需要对其进行`pthread_join`操作，否则有内存泄漏风险

但如果不关心线程的返回值，此时`join`是一种负担，这时我们可以通过接口告诉系统，当线程退出时，自动释放线程资源，这个调用接口的操作就是`分离线程`

### pthread_detach函数
`int pthread_detach(pthread_t thread);`

该函数可以使目标线程分离，配合`pthread_self()`函数可以**使自己分离**

特别的,`joinable`和分离是冲突的，分离之后就不能再执行`join`操作了

## 线程互斥
**背景概念补充**
+ `临界资源`：多线程执行流共享的资源,这些资源在**某一时刻只能被一个线程访问**,这样的资源就叫做`临界资源`
+ `临界区`：每个线程内部，访问临界资源的代码，就叫做`临界区`
+ `互斥`：任何时刻，互斥保证有且只有一个执行流进入临界区，访问临界资源，通常对临界资源起保护作用
+ `原子性`（后面讨论如何实现）：不会被任何调度机制打断的操作，该操作只有两态，要么完成，要么未完成

### 互斥量mutex
大部分情况，线程使用的数据都是局部变量，变量的地址空间在线程栈空间内，这种情况，变量归属单个线程，其他线程无法获得这种变量

但有时候，很多变量都需要在线程间共享，这样的变量称为共享变量，可以通过数据的共享，完成线程之间的交互。

多个线程并发的操作共享变量，会带来一些问题。下面有一段代码展示

```C++
// 操作共享变量会有问题的售票系统代码
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
int ticket = 100;//抢票程序，预留100张票
void *route(void *arg)
{
    char *id = (char *)arg;//从参数里获取线程名
    while (1)
    {
        if (ticket > 0)
        {
            usleep(1000);//降低每次抢票的速度
            printf("%s sells ticket:%d\n", id, ticket);
            ticket--;
        }
        else
        {
            break;
        }
    }
}
int main(void)
{
    pthread_t t1, t2, t3, t4;//准备四个线程id
    char name1[] = "thread 1";//准备四个线程名
    char name2[] = "thread 2";
    char name3[] = "thread 3";
    char name4[] = "thread 4";

    pthread_create(&t1, NULL, route, name1);//创建四个线程
    pthread_create(&t2, NULL, route, name2);
    pthread_create(&t3, NULL, route, name3);
    pthread_create(&t4, NULL, route, name4);
    pthread_join(t1, NULL);//进程等待
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);
    pthread_join(t4, NULL);
}
```
某一次的执行结果

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread10086.png)

可以看到代码的执行与预期的行为发生了严重误差

+ 有的票被重复卖出
+ 有负数的票被卖出

究其原因，就是全局变量`ticket`被所有线程共享，对其的访问控制确得不到限制，同时对其进行非原子性的操作时，就会出现`数据不一致问题`，进而引发一系列问题

例如对这段代码有如下分析
+ `if` 语句判断条件为真以后，代码可以并发的切换到其他线程
+ `usleep` 这个模拟漫长业务的过程，在这个`漫长的业务过程`中，可能有很多个线程会进入该代码段
+ `--ticket` 操作本身就不是一个`原子操作`

```Assembly
取出ticket--部分的汇编代码
objdump -d a.out > test.objdump
152 40064b: 8b 05 e3 04 20 00 mov 0x2004e3(%rip),%eax # 600b34 <ticket>
153 400651: 83 e8 01 sub $0x1,%eax
154 400654: 89 05 da 04 20 00 mov %eax,0x2004da(%rip) # 600b34 <ticket>
```

显然`--`操作并不是原子操作，而是对应三条汇编指令

+ `load`: 将共享变量`ticket`从内存加载到寄存器中
+ `update`: 更新寄存器里面的值，执行`-1`操作
+ `store`: 将新值，从寄存器写回共享变量`ticket`的内存地址

**将新值，从寄存器写回共享变量ticket的内存地址**
+ 代码必须要有互斥行为：当代码进入临界区执行时，不允许其他线程进入该临界区。
+ 如果多个线程同时要求执行临界区的代码，并且临界区没有线程在执行，那么只能允许一个线程进入该临界区。
+ 如果线程不在临界区中执行，那么该线程不能阻止其他线程进入临界区。

要做到这三点，本质上就是需要一把锁。Linux上提供的这把锁叫互斥量。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread_mutex09191134202.png)

### 互斥量的接口

#### 初始化互斥量
初始化互斥量主要有两种方法

1. 静态分配
`pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER`
2. 动态分配
`int pthread_mutex_init(pthread_mutex_t *restrict_mutex, const pthread_mutexattr_t *restrict_attr);`
+ `mutex`为要初始化的的互斥量的变量地址
+ `attr`为可选参数，传`nullptr`即可

#### 销毁互斥量
特别的，**静态分配**的，即使用`PTHREAD_MUTEX_INITIALIZER`初始化的互斥量不需要手动销毁

而对于动态分配的互斥量
+ 不要销毁一个已经加锁的互斥量
+ 已经销毁的互斥量，要确保后面不会有线程再尝试加锁

使用的如下函数

`int pthread_mutex_destroy(pthread_mutex_t *mutex)；`

#### 互斥量加锁和解锁
```C++
int pthread_mutex_lock(pthread_mutex_t *mutex);

int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

调用 pthread_ lock 时，可能会遇到以下情况:

+ 互斥量处于**未锁状态**，该函数会将互斥量锁定，同时返回**成功**
+ 发起函数调用时，其他线程已经锁定互斥量，或者存在其他线程同时申请互斥量，但没有竞争到互斥量，那么`pthread_ lock`调用会陷入`阻塞`(执行流被挂起)，**等待**互斥量解锁。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread_mutex09191134202.png)


#### 现学现用
我们来利用锁来改进上面的选票程序

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

int ticket = 100;//抢票程序，预留100张票
pthread_mutex_t mutex;//准备一个互斥量

void *route(void *arg)
{
    char *id = (char *)arg;//从参数里获取线程名
    while (1)
    {
        pthread_mutex_lock(&mutex);//进入if判断前就得加锁，因为判断时ticket也是共享的
        if (ticket > 0)
        {
            printf("%s sells ticket:%d\n", id, ticket);
            ticket--;
            pthread_mutex_unlock(&mutex);//完成对共享资源的访问，可以释放互斥量
            //usleep不访问共享资源，放在锁外面，以提高多线程并行效率
            usleep(1000);//降低每次抢票的速度
        }
        else
        {
            pthread_mutex_unlock(&mutex);//原理同上
            break;
        }
    }
    return nullptr;
}
int main(void)
{
    pthread_mutex_init(&mutex,nullptr);//初始化互斥量
    pthread_t t1, t2, t3, t4;//准备四个线程id
    char name1[] = "thread 1";//准备四个线程名
    char name2[] = "thread 2";
    char name3[] = "thread 3";
    char name4[] = "thread 4";

    pthread_create(&t1, NULL, route, name1);//创建四个线程
    pthread_create(&t2, NULL, route, name2);
    pthread_create(&t3, NULL, route, name3);
    pthread_create(&t4, NULL, route, name4);
    pthread_join(t1, NULL);//进程等待
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);
    pthread_join(t4, NULL);

    pthread_mutex_destroy(&mutex);//销毁互斥量

    return 0;
}
```

可以看到，在代码中我们用互斥量将共享资源`ticket`保护了起来，使其变成了临界资源，相关的代码区变成了临界区，特别的，临界区的代码越少越好，将例如`usleep`这样耗时长，又与共享资源无关的代码移出临界区，这样才能最大化多线程并行的效率。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread_mutex109191850805.png)

## 可重入VS线程安全

### 概念
+ `线程安全`：多个线程并发`同一段代码`时，不会出现不同的结果。常见对全局变量或者静态变量进行操作，并且没有锁保护的情况下，会出现该问题。
+ `重入`：同一个`函数`被不同的执行流调用，当前一个流程还没有执行完，就有其他的执行流再次进入，我们称之为重入。一个函数在重入的情况下，运行结果不会出现任何不同或者任何问题，则该函数被称为可重入函数，否则，是不可重入函数。

## 常见线程不安全的情况
+ 不保护`共享变量`的函数
+ 函数状态随着被调用，**状态发生变化**的函数
+ 返回指向`静态变量指针`的函数
+ 调用**线程不安全函数**的函数

## 常见线程安全的情况
+ 每个线程对全局变量或者静态变量只有**读取的权限**，而没有写入的权限，一般来说这些线程是安全的
+ 类或者接口对于线程来说都是原子操作
+ 多个线程之间的切换不会导致该接口的执行结果存在二义性

## 常见不可重入的情况
+ 调用了`malloc/free`函数，因为`malloc`函数是用全局链表来管理堆的
+ 调用了`标准I/O库函数`，标准I/O库的很多实现都以不可重入的方式使用全局数据结构
+ 可重入函数体内使用了**静态的数据结构**

## 常见的可重入的情况
+ 不使用`全局变量`或`静态变量`
+ 不使用用`malloc`或者`new`开辟出的空间
+ **不调用**不可重入函数
+ **不返回静态或全局数据**，所有数据都有函数的调用者提供
+ 使用本地数据，或者通过制作全局数据的`本地拷贝`来保护全局数据

## 可重入与线程安全的联系
+ 函数是可重入的，那就是线程安全的   `可重入`->`线程安全`
+ 函数是不可重入的，那就不能由多个线程使用，`有可能`引发线程安全问题
+ 如果一个函数中有`全局变量`，那么这个函数既**不是**线程安全也**不是**可重入的。

## 可重入与线程安全的区别
+ 可重入函数是线程安全函数的一种
+ 线程安全不一定是可重入的，而可重入函数则一定是线程安全的。
+ 如果将对临界资源的访问**加上锁**，则这个函数是线程安全的，但如果这个重入函数若锁还未释放则会产生`死锁`，因此是不可重入的。

## 常见锁概念

### 死锁
死锁是指在一组进程中的各个进程均**占有不会释放的资源**，但因互相申请被其他进程所站用不会释放的资源而处于的**一种永久等待状态。**

#### 死锁四个必要条件
1. `互斥条件`：一个资源每次只能被**一个执行流**使用
2. `请求与保持条件`：一个执行流因请求资源而阻塞时，对已获得的资源**保持不放**
3. `不剥夺条件`:一个执行流已获得的资源，在末使用完之前，**不能强行剥夺**
4. `循环等待条件`:若干执行流之间形成一种头尾相接的**循环等待资源**的关系

#### 避免死锁
+ 破坏死锁的四个必要条件
+ 加锁顺序一致
+ 避免锁未释放的场景
+ 资源一次性分配

#### 避免死锁的算法
+ 避免死锁算法
+ 银行家算法

## Linux线程同步

### 条件变量
当`一个线程`互斥的访问某个变量时，它可能发现在`其它线程`改变状态之前,它什么也做不了。

**例如**，一个线程访问队列时，发现队列为空，就只能等待，直到其它线程将一个节点添加到队列中，这种情况就要用到条件变量

### 同步概念与竞态条件
`同步`：在保证数据安全的前提下，让线程能够按照**某种特定的顺序**访问临界资源，从而**有效避免饥饿问题**，叫做`同步`

`竞态条件`：因为**时序问题**，而导致程序异常，我们称之为竞态条件。在线程场景下，这种问题也不难理解

### 条件变量函数 

*初始化*
```C++
int pthread_cond_init(pthread_cond_t *restrict_cond,const pthread_condattr_t *restrict_attr);

    restrict_cond:要初始化的条件变量的地址
    restrict_attr:可选参数，传nullptr即可
```

*销毁*
```C++
int pthread_cond_destroy(pthread_cond_t *cond)
```

*等待条件满足(互斥地)*
```C++
int pthread_cond_wait(pthread_cond_t *restrict_cond,pthread_mutex_t *restrict_mutex);

    restrict_cond:目标条件变量的地址
    restrict_mutex:互斥量，用于保护条件变量
```
*唤醒等待*
```C++
int pthread_cond_broadcast(pthread_cond_t *cond);
int pthread_cond_signal(pthread_cond_t *cond);
```

#### 简单示例
```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

pthread_cond_t cond;   // 创建信号量
pthread_mutex_t mutex; // 创建互斥量

void *chicken(void *arg)
{
    while (1)
    {
        pthread_cond_wait(&cond, &mutex);
        printf("鸡哥获得了一个篮球\n");
    }
}

void *kunkun(void *arg)
{
    for (int i = 0; i < 5; ++i)
    {
        printf("坤坤扔出了一个篮球[%d]\n",i);
        pthread_cond_signal(&cond);
        sleep(1);
    }
}

int main()
{
    pthread_t t1, t2;
    pthread_cond_init(&cond, NULL);
    pthread_mutex_init(&mutex, NULL);
    pthread_create(&t1, NULL, chicken, NULL);
    pthread_create(&t2, NULL, kunkun, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);

    return 0;
}
```

在如上代码中,`chicken`线程负责等待条件变量(这里具象化为获取篮球),`kunkun`线程负责唤醒信号量（这里具象化为扔出篮球）

特别的，本段代码通过`for`循环控制`kunkun`线程只唤醒五次信号量，而`chicken`线程工作在死循环中，我们看一下代码的输出

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread_signal9192008788.png)

可以看到，`kunkun`线程按照预期执行了5次循环，而`chicken`却也因为条件变量的控制，只执行了`5次`循环

### 为什么`pthread_cond_wait`需要互斥量?
+ `条件等待`是线程间同步的一种手段，如果只有一个线程，条件不满足，一直等下去都不会满足，所以必须要有**另一个线程**通过某些操作，改变共享变量，使原先不满足的条件变得满足，并且友好的**通知等待**在条件变量上的线程。
+ 条件不会无缘无故的突然变得满足了，必然会牵扯到**共享数据的变化**。所以一定要用**互斥锁来保护**。没有互斥锁就无法安全的获取和修改共享数据。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thraed_sig_mutex09192026148.png)

按照上面的说法，我们设计出如下的错误代码：先上锁，发现条件不满足，解锁，然后等待在条件变量上不就行了，如下代码:

```C++
// 错误的设计
pthread_mutex_lock(&mutex);
while (condition_is_false)
{
    pthread_mutex_unlock(&mutex);
    // 解锁之后，等待之前，条件可能已经满足，信号已经发出，但是该信号可能被错过
    pthread_cond_wait(&cond);
    pthread_mutex_lock(&mutex);
}
pthread_mutex_unlock(&mutex);
```

+ 由于解锁和等待不是原子操作。调用解锁之后,`pthread_cond_wait`之前，如果已经有其他线程获取到互斥量，摒弃条件满足，发送了信号，那么 `pthread_cond_wait`将错过这个信号，可能会导致线程永远阻塞在这个`pthread_cond_wait`。所以解锁和等待必须是一个原子操作。
+ `int pthread_cond_wait(pthread_cond_ t *cond,pthread_mutex_ t * mutex);` 进入该函数后，会去看条件量等于`0`不？等于，就把互斥量变成`1`，直到`cond_ wait`返回，把条件量改成`1`，把互斥量恢复成原样。


### 条件变量使用规范
*等待条件代码*
```C++伪代码
pthread_mutex_lock(&mutex);
while (条件为假）
    pthread_cond_wait(cond, mutex);
修改条件
pthread_mutex_unlock(&mutex);
```
*给条件发送信号代码*
```C++伪代码
pthread_mutex_lock(&mutex);
设置条件为真
pthread_cond_signal(cond);
pthread_mutex_unlock(&mutex);
```
## 生产者消费者模型

生产者消费者模式就是通过一个**容器**来解决生产者和消费者的**强耦合**问题。生产者和消费者彼此之间不直接通讯，而通过**阻塞队列来进行通讯**，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于**一个缓冲区**，平衡了生产者和消费者的处理能力。这个阻塞队列就是用来给生产者和消费者**解耦**的。

### 生产者消费者模型的有点
+ 解耦
+ 支持并发
+ 支持忙闲不均

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/thread_PCM09192237411.png)

### 基于BlockingQueue的生产者消费者模型
在多线程编程中阻塞队列(`Blocking Queue`)是一种常用于实现生产者和消费者模型的数据结构。其与普通的队列区别在于，当队列**为空**时，从队列获取元素的操作将会被**阻塞**，直到队列中被放入了元素；当队列**满时**，往队列里存放元素的操作也会被**阻塞**，直到有元素被从队列中取出(以上的操作都是基于不同的线程来说的，线程在对阻塞队列进程操作时会被阻塞)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/THread_BlockQ09192244045.png)


## POSIX信号量
POSIX信号量和SystemV信号量作用相同，都是用于同步操作，达到无冲突的访问共享资源目的。 但POSIX可以用于线程间同步。

*初始化信号量*
```C++
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
参数：
pshared:0表示线程间共享，非零表示进程间共享
value：信号量初始值
```

*销毁信号量*
```C++
int sem_destroy(sem_t *sem);
```

*等待信号量*
```C++
功能：等待信号量，会将信号量的值减1
int sem_wait(sem_t *sem); //P()
```

发布信号量
```C++
功能：发布信号量，表示资源使用完毕，可以归还资源了。将信号量值加1。
int sem_post(sem_t *sem);//V()
```

上一节生产者-消费者的例子是基于queue的,其空间可以动态分配,现在基于固定大小的环形队列重写这个程序
（POSIX信号量）

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409222124291.png)

## 线程池

### 概念
一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着
监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利
用，还能防止过分调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。

### 线程池的应用场景：
1. 需要大量的线程来完成任务，且完成任务的**时间比较短**。 WEB服务器完成网页请求这样的任务，使用线程池技术是非常合适的。因**为单个任务小，而任务数量巨大**，你可以想象一个热门网站的点击次数。 但对于长时间的任务，比如一个Telnet连接请求，线程池的优点就不明显了。因为Telnet会话时间比线程的创建时间大多了。
2. 对性能要求苛刻的应用，比如要求**服务器迅速响应客户请求**。
3. 接受突发性的大量请求，但不至于使服务器因此产生大量线程的应用。突发性大量客户请求，在没有线程池情况下，将产生大量线程，虽然理论上大部分操作系统线程数目最大值不是问题，短时间内产生大量线程可能使内存到达极限，出现错误.


### 代码示例
```C++
#pragma once
#include <iostream>
#include <vector>
#include <pthread.h>
#include <string>
#include <queue>
#include <unistd.h>
struct ThreadInfo//储存线程信息
{
    pthread_t tid;
    std::string name;
};

static const int defaultnum = 5;//默认5个线程

template<class Task>
class ThreadPool
{
public:
    void Lock()
    {
        pthread_mutex_lock(&_mutex);
    }

    void Unlock()
    {
        pthread_mutex_unlock(&_mutex);
    }

    void Wakeup()
    {
        pthread_cond_signal(&_cond);
    }

    void ThreadSleep()
    {
        pthread_cond_wait(&_cond,&_mutex);
    }

    bool IsQueueEmpty()
    {
        return _tasks.empty();//封装一下接口
    }

    std::string GetThreadName(pthread_t tid)
    {
        for(const auto &ti:_threads)
        {
            if(tid == ti.tid) return ti.name;
        }
        return "None";
    }

    Task Pop()
    {
        //确认只在加锁的情况下调用
        Task t = _tasks.front();
        _tasks.pop();
        return t;
    }

public:
    ThreadPool(int num = defaultnum):_threads(num)
    {
        pthread_mutex_init(&_mutex,nullptr);
        pthread_cond_init(&_cond,nullptr);
    }

    ~ThreadPool()
    {
        pthread_mutex_destroy(&_mutex);
        pthread_cond_destroy(&_cond);
    }

    static void *HandlerTask(void* args)
    {
        ThreadPool<Task> *tp = static_cast<ThreadPool<Task> *>(args);
        std::string name = tp->GetThreadName(pthread_self());

        // std::cout<<"get this success"<<std::endl;//DEBUG

        while(true)
        {
            tp->Lock();
            while(tp->IsQueueEmpty())//访问临界资源
            {
                tp->ThreadSleep();
            }
            // std::cout<<"task ready"<<std::endl; //DEBUG
            Task t = tp->Pop();
            // std::cout<<"Pop success"<<std::endl; //DEBUG
            tp->Unlock();

            t();//处理线程私有的任务，不需要加锁,可以并发
            std::cout<<name<<" run :result :" << t._result<<std::endl;
            sleep(4);
        }

    }
    void Start()
    {
        int num = _threads.size();
        for(int i = 0;i<num;++i)
        {
            _threads[i].name = "thread-"+std::to_string(i);
            pthread_create(&(_threads[i].tid),nullptr,HandlerTask,this);
            std::cout<<_threads[i].name<<" create successfully"<<std::endl;
        }
    }

    void Push(const Task& t)
    {
        Lock();
        _tasks.push(t);//推送任务
        Wakeup();
        Unlock();
    }

private:
    std::vector<ThreadInfo> _threads;//线程池
    std::queue<Task> _tasks;//任务队列

    pthread_mutex_t _mutex;//锁
    pthread_cond_t _cond;//条件队列
};
```
## 线程安全的单例模式

单例模式是一种"经典的，常用的，常考的"**设计模式**

某些类, 只应该具有一个对象(实例), 就称之为单例

在很多服务器开发场景中, 经常需要让服务器加载很多的数据 (上百G) 到内存中. 此时往往要用一个单例的类来**管理**这些数据

### 饿汉实现方式和懒汉实现方式
单例的实现主要有两种方式。

*[洗完的例子]*
```
吃完饭, 立刻洗碗, 这种就是饿汉方式. 因为下一顿吃的时候可以立刻拿着碗就能吃饭.
吃完饭, 先把碗放下, 然后下一顿饭用到这个碗了再洗碗, 就是懒汉方式.
```

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

## STL,智能指针和线程安全

### STL库是不是线程安全的
原因是, STL 的设计初衷是将性能挖掘到极致, **而一旦涉及到加锁保证线程安全, 会对性能造成巨大的影响**.而且对于不同的容器, 加锁方式的不同, 性能可能也不同(例如hash表的锁表和锁桶).因此 STL **默认不是线程安全**. 如果需要在多线程环境下使用, 往往需要调用者自行保证线程安全.

### 智能指针是线程安全的
+ `unique_ptr`, 由于只是在`当前代码块范围内`生效, 因此不涉及线程安全问

+ `shared_ptr`, 多个对象需要共用一个引用计数变量, 所以会存在线程安全问题. 但是标准库实现的时候考虑到了这
个问题, 基于原子操作(CAS)的方式保证 shared_ptr 能够高效, 原子的操作引用计数.