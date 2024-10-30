---
title: fork子进程,进程退出与等待
date: 2024-07-07 12:57:17
tags: fork Linux
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-25_22-26-59.png
---
# 认识`fork()`
头文件`<unistd.h>`提供的`fork()`函数用于从已有的`原进程`创建一个新的`子进程`，而原进程在关系式称为`父进程`

## fork()的返回值
```C
#include <unistd.h>
pid_t id = fork();
```

父子进程中`fork()`函数的返回值(此处用变量`id`储存)是不同的:

**父进程**里`id`的值为子进程的`PID`,其值`>0`;**子进程**里`id`值固定为`0`

+ `id > 0` 父进程
+ `id == 0` 子进程
+ `id < 0` fork()失败

## 分流
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-09_13-00-38.png)

利用父子进程中`fork()`返回值的不同，可以用`if...else...`进行分流，让父子进程执行不同的代码

## fork()的过程
进程调用`fork`，当控制转移到**内核**中的fork代码后，**内核**做

+ 分配**新的**内存块和内核数据结构给子进程
+ 将父进程部分数据结构内容**拷贝**至子进程
+ 添加子进程到**系统进程列表**当中
+ `fork`返回，开始调度器调度

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-09_18-54-41.png)

当一个进程调用fork之后，就有一对二进制**代码相同**的父子进程。而且它们都运行到相同的地方。但每个进程都可以**独立**地继续运行代码,并按代码分流至不同的代码段

但这两个进程谁先执行**完全**由调度器决定，而谁先结束，则由实际执行情况和调度器决定

## 拷贝的过程 与 写时拷贝
`fork`时，子进程会将父进程虚拟内存的内容都复制一份在自己的虚拟内存中,但通过页表，二者映射到了同一块物理内存的区域，这样在二者都**没有写入行为**时，减少了物理内存中**冗余**的拷贝行为，有效提高了运行效率

但当父子进程中有一方发生写入行为时，就会触发`写时拷贝`，此时物理内存中发生拷贝行为，但只拷贝进程映射的`数据段`，而由于**不发生进程替换时**父子进程的代码段一定相同，物理内存中,`代码段`映射部分并不发生拷贝

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-09_19-38-58.png)


# 进程退出

## 进程退出的场景
+ 代码运行完毕，结果正确
+ 代码运行完毕，结果不正确
+ 代码异常终止

## 进程常见退出方法
+ `main`函数返回
+ `exit`退出
+ `_exit`退出

**异常退出**: `ctrl` + `c`

## 查看退出码
在终端使用命令`echo $?`可以查看退出码

`注`尽管子进程返回的是`int`,父进程只取退出码的最低`8位`,所以以下三种情况

+ `main`函数返回`-1`
+ `exit(-1)`
+ `_exit(-1)`

在终端输`echo $?`可得退出码`255`

## `exit` 和 `_eixt` 辨析
`main`函数返回就不惜说了，来辨析一下头文件`<stdlib.h>`提供的`exit`和`_exit`函数

### `_exit()`
`_exit`会**直接终止**进程并返回状态码,而**不会**执行用户定义的清理函数，**也不会**清理缓冲区

### `exit()`
`exit`实际上最后也会调用`_exit`,但它会先执行一系列善后工作,顺序如下:

1. 执行用户通过 atexit或on_exit定义的**清理函数**。
2. 关闭所有打开的流，所有的**缓存数据**均被写入
3. 调用`_exit`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-09_22-10-26.png)

`return`退出： 执行`return n`等同于执行`exit(n)`

# 进程等待

## 进程等待的必要性
+ 子进程退出后需要父进程来**回收**僵尸进程,防止产生其引发内存泄漏等问题
+ 僵尸进程难以处理,`kill -9`也清理不掉
+ 父进程通过进程等待的方式，**回收**子进程资源，获取子进程**退出信息**

## 进程等待的方法

### `wait`方法
> #include<sys/types.h>
> #include<sys/wait.h>
>
> pid_t wait(int*status);
>
> `参数`：显然`status`是输出型参数,获取子进程的`退出状态`,若不关心，可传参`NULL`
>
> `返回值`: 若**成功**，则返回被等待进程的`pid`,若**失败**，则返回`-1`

### `waitpid`方法
> 返回值：
>    当正常返回的时候waitpid返回收集到的子进程的进程ID；
>    如果设置了选项WNOHANG,而调用中waitpid发现没有已退出的子进程可收集,则返回0；
>    如果调用中出错,则返回-1,这时errno会被设置成相应的值以指示错误所在；
> 
> 参数：
>    `pid`：
>        `pid == -1`,等待任一个子进程。与wait等效。
>        `pid > 0`.等待其进程ID与pid相等的子进程。
>    `status`:
>        输出型参数传参`&status`: 将子进程的`状态码`存入`status`
>        以下两个宏函数用于处理`状态码`:
>        `WIFEXITED(status)`: 若为**正常终止**子进程返回的状态，则为真。（查看进程是否是**正常退出**）
>        `WEXITSTATUS(status)`: 若WIFEXITED非零，提取子进程`退出码`。（查看进程的`退出码`）
>    `options`:
>        `WNOHANG`: 若pid指定的子进程没有结束，则waitpid()函数返回0，不予以等待。若正常结束，则返回该子进程的ID。
>        `0`: 阻塞等待指定`pid`的进程
>

#### 阻塞等待
`option` == `0`时`waitpid`采用阻塞等待，父进程会阻塞等待到子进程退出

*示例代码*
```C
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main()
{
    int status;
    pid_t id = fork();
    if(id == 0)
    {
        for(int i = 1;i<=5;i++)
        {
            printf("child running [%d]s\n",i);
            sleep(1);
        }
        printf("child exited\n");
        exit(0);
    }
    else
    {
        waitpid(id,&status,0);
        if(WIFEXITED(status))
            printf("father wait success, exit code: %d\n",WEXITSTATUS(status));
        else
            printf("child failed to exit\n");
    }
    return 1;
}
```

#### 非阻塞轮询
`option` == `WNOHANG`时,`waitpid`采用非阻塞等待,若等不到子进程退出，就会继续执行后面的代码，所以一般加上`while`等循环用于轮询，二者共同构成`非阻塞轮询`
```C
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

//非阻塞轮询
int main()
{
    int status;
    pid_t id = fork();
    if(id == 0)
    {
        sleep(5);
        printf("child exited\n");
        exit(0);
    }
    else
    {
        while(1)//轮询
        {
            pid_t rid = waitpid(id,&status,WNOHANG);
            if(rid > 0)
            {
                if(WIFEXITED(status))
                    printf("father wait success, exit code: %d\n",WEXITSTATUS(status));
                else
                    printf("child failed to exit\n");

                break;
            }
            else if(rid == 0)
            {
                printf("等待子进程中...\n");
                sleep(1);
            }
            else//rid<0
            {
                printf("wait failed\n");
                break;
            }
        }
    }
    return 1;
}
```