---
title: Ssystem V 共享内存
date: 2024-08-14 14:55:06
tags:
---

共享内存区是最快的IPC形式。一旦这样的内存映射到共享它的进程的地址空间，这些进程间数据传递**不再涉及**到
`内核`，换句话说是进程不再通过执行进入内核的系统调用来传递彼此的数据,而是直接使用内存中的共享区。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/shared_mm.png)

我们接下来认识一下常用的接口

# 接口

## shmget 创建共享内存
需要同时引入`<sys/ipc.h>` `<sys/shm.h>`

`int shmget(key_t key, size_t size, int shmflg);`

+ `key` 是生成`共享内存标示符`的 关键字,唯一的`key`值能返回唯一的`共享内存标示符`,这是获得同一个共享内存的关键参数
+ `size`是指共享内存的大小,按字节算
+ `shmflg`是一个位图,控制创建时的行为和 共享内存文件的`权限`(缺省时为`0`),常见选项如下
  + `IPC_CREAT`:单独一个时，如果申请的共享内存不存在，就创建，然后返回；若存在，则获取并返回
  + `IPC_CREAT | IPC_EXCL`: 如果申请的共享内存不存在，则创建；若存在，则出错并返回`-1`
  + `IPC_EXCL`:不能单独使用
  + `IPC_CREAT | 0666`:创建一个权限为`0666`的共享内存文件,注：`0666`可改成其它权限
+ `返回值`：若成功，则返回`共享内存标示符` ;若失败，则返回`-1`

由上可知，保证`key`的唯一性是获得同一个共享内存的关键步骤，那么如何获得唯一的`key_t`类型的呢？

这里使用新的接口`ftok`,（需同时引入`<sys/types.h>` 和 `<sys/ipc.h>`）

`key_t ftok(const char *pathname, int proj_id);`

+ `pathname`:必须指向一个存在的目录，或者有权限的文件
+ `proj_id`: 该参数必须是**非零**的且至少有八位有效位的整型,可传`0x8888`这样的大整型
+ `返回值`：成功时生成`key`,失败时返回`-1`


![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/keytoshmid.png)

经过一系列操作便可以创建共享内存了

### 查看共享内存
在`shell`命令里输入命令`ipcs -m`就可以看到的共享内存列表

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202408142027880.png)

可以看到我创建了一个大小为`1145`比特的共享内存

然而共享内存的声明周期与内核相同，必须要**手动**删除，所以在命令行上还有指令`ipcrm -m`指令删除共享内存

那么应该用表里的`key`还是`shmid`呢？结论是在用户层，统一使用`shmid`管理共享内存（毕竟全称就是 `共享内存描述符`）

例如上图就需要输入`ipcrm -m 0`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202408142026217.png)

可以看到共享内存被删掉了

### 共享内存的权限
`ipcs -m`列表中的`perms`列就是共享内存文件的权限，没错，共享内存也是文件

当创建时没指定权限时，则默认为`全0`，若要指定权限,需要在`shmflg`处在`|`上权限，例如`0666`

### 共享内存的挂接数
`ipcs -m`列表中的`nattch`下标注了共享内存的挂接数

## shmat 挂接共享内存
`void *shmat(int shmid, const void *shmaddr, int shmflg);`

+ `shmid`:前面获取的`共享内存描述符`
+ `shmadder`:一般传`nullptr`让系统自动选择挂接用的共享内存段地址
+ `shmflg`:位图
  + `0`:传`0`时默认以`读写模式`挂接
  + `SHM_RDONLY`: 以`只读模式`挂接
  + **没有**只写模式挂接
+ `返回值`: 若成功，返回共享内存段地址；若失败，返回`(void *) -1`

### shmdt 取消挂接
虽然进程退出时会自动取消挂接，但如果要在进程内取消挂接，就要用`shmdt`函数取消挂接

`int shmdt(const void *shmaddr);`

把`shmat`返回的指针传进去即可

### 读写共享区内存
挂接上的进程是真正意义上的看到同一块内存，而且完全可以像`malloc`申请出的一段内存一样操作,比如把共享区的内存当成字符串的缓冲区,直接把标准输入用`fgets`拷贝到共享内存中

不过默认的共享内存并没有同步互斥行为,需要额外控制,比如使用`FIFO命名管道`来完成同步操作

## shmctl 控制共享内存
`int shmctl(int shmid, int cmd, struct shmid_ds *buf);`

该函数内容较多，下面只列举常见用法，更多信息请查阅`man`手册

+ `shmid`:共享内存描述符
+ `cmd`:控制指令，有很多种
  + `IPC_RMID`:由共享内存创建者调用，声明该段共享内存已经取消挂接，传该参数时，后面的`buf`可传`nullptr`
  + `IPC_STAT`:获取共享内存的状态并拷贝到`buf`指向的内存中
+ `buf`:输出型参数,指向输出的内存缓冲区

`shmid_ds`的部分声明如下
```C
struct shmid_ds {
    struct ipc_perm shm_perm;    /* Ownership and permissions */
    size_t          shm_segsz;   /* Size of segment (bytes) */
    time_t          shm_atime;   /* Last attach time */
    time_t          shm_dtime;   /* Last detach time */
    time_t          shm_ctime;   /* Last change time */
    pid_t           shm_cpid;    /* PID of creator */
    pid_t           shm_lpid;    /* PID of last shmat(2)/shmdt(2) */
    shmatt_t        shm_nattch;  /* No. of current attaches */
    ...
};
```

## 进程互斥
由于各进程要求共享资源，而且有些资源需要互斥使用，因此各进程间**竞争使用**这些资源，进程的这种关系为进程的互斥。

+ `临街资源`:系统中某些资源一次只允许一个进程使用，称这样的资源为`临界资源`或`互斥资源`。*就好比一个萝卜一个坑*
+ `临界区`: 在进程中涉及到互斥资源的**程序段**叫`临界区`
+ `同步`: 内存共享中的同步，主要指使`写入`和`读取`操作`互斥`，使二者有明确的先后顺序，能够在共享内存中一次写入或读取完整报文

在共享内存中，使用`FIFO命名管道`建立另一条进程间通信，就能较为简单地完成同步功能：

1. 读端等待命名管道的信息
2. 写端完成写入后利用命名管道，写入完成写入的信息，并等待读端的应答
3. 读端接收到命名管道的信息后，才开始读取共享内存的内容
4. 读端完成任务后向写端发送应答，写端返回第`1`步
5. 写端开始继续写入

# 附加：System V 消息队列 和 信号量
System V 还提供了消息队列和信号量用于进程间通信

## 消息队列
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/sharedmsg.png)

消息队列是由内核维护的一种数据结构,用法和普通的队列一样，可以`push`和`pop`数据块，用于进程间通信

## 接口

### 头文件
```C
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
```

### msgget
`int msgget(key_t key, int msgflg);`

用于申请消息队列并获得`消息队列id`

+ `key`:用法和获取与上一致，这里不赘述了
+ `msgflg`:创建消息队列的选项，**基本和上文`shmget`相同**,内容较多，完整内容需翻阅`man`手册
  + `IPC_CREAT`:单独一个时，如果申请的共享内存不存在，就创建，然后返回；若存在，则获取并返回
  + `IPC_CREAT | IPC_EXCL`: 如果申请的共享内存不存在，则创建；若存在，则出错并返回`-1`
  + `IPC_EXCL`:不能单独使用
  + `IPC_CREAT | 0666`:创建一个权限为`0666`的共享内存文件,注：`0666`可改成其它权限
+ `返回值`：若成功，则返回`消息队列id` ;若失败，则返回`-1`

#### 查看消息队列
`ipcs -q`可以查看所有可用的消息队列，剩余操作与上文相同

### msgctl
`int msgctl(int msqid, int cmd, struct msqid_ds *buf);`

`msgctl`可用于控制消息队列

该函数内容较多，下面只列举常见用法，更多信息请查阅`man`手册

+ `msqid`:共享内存描述符
+ `cmd`:控制指令，有很多种
  + `IPC_RMID`:由共享内存创建者调用，声明该段共享内存已经取消挂接，传该参数时，后面的`buf`可传`nullptr`
  + `IPC_STAT`:获取共享内存的状态并拷贝到`buf`指向的内存中
+ `buf`:输出型参数,指向输出的内存缓冲区

```C
struct msqid_ds {
    struct ipc_perm msg_perm;   /* Ownership and permissions */
    time_t          msg_stime;  /* Time of last msgsnd(2) */
    time_t          msg_rtime;  /* Time of last msgrcv(2) */
    time_t          msg_ctime;  /* Time of creation or last
                                  modification by msgctl() */
    unsigned long   msg_cbytes; /* # of bytes in queue */
    msgqnum_t       msg_qnum;   /* # number of messages in queue */
    msglen_t        msg_qbytes; /* Maximum # of bytes in queue */
    pid_t           msg_lspid;  /* PID of last msgsnd(2) */
    pid_t           msg_lrpid;  /* PID of last msgrcv(2) */
};
```

### 向消息队列收发消息

#### msgsnd
`int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);`

`msgsnd`用于向消息队列发送消息

+ `msqid`:消息队列的`id`
+ `msgp`:是指向结构体数据块(消息)的指针
+ `msgsz`:数据块的大小，按字节算
+ `msgflg`: 位图,具体选项详见`man`手册，一般使用可以为`0`
+ 返回值：失败时返回`-1`,成功时返回`0`

其中,`msgp`要遵循如下格式

```C
struct msgbuf {
    long mtype;       /* 消息的类型,必须> 0 */
    char mtext[1];    /* 消息的数据，长度可比1大 */
};
```

其中第一个成员必须是`long mtype`,且大于`0`。
而第二个成员是字符数组用于储存字节数据，长度没有限制，因此函数传参也是`void*`,适配各种长度的结构体指针


#### msgrcv
`ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp,int msgflg);`

`msgrcv`用于从消息队列中接收数据（消息）

+ `msqid`:消息队列的`id`
+ `msgp`:是指向数据块(消息)的指针
+ `msgsz`:数据块的大小，按字节算
+ `msgtyp`:消息的类型
+ `msgflg`: 位图,具体选项详见`man`手册，一般使用可以为`0`
+ 返回值：失败时返回`-1`,成功时返回读取的字节数

## 信号量
信号量的本质是是一种计数器，但它特殊在只能`互斥访问`,它本身就是一种互斥资源,但也可用于描述另一种临界资源的多少

信号量的工作原理:
1. 申请计数器成功，就表示我有访问资源的**权限**了
2. 申请了计数器资源，不代表当前我要访问资源了。当前只是预定了资源
3. 计数器可以有效保证进入共享资源的执行流的量

## 特殊情况
当信号量只能为`0`或`1`时,即`二元信号量`，该信号量就可以作为一把`互斥锁`来使用，访问前申请信号量，访问完毕再释放信号量即可

### 系统调用接口
这里不作详细介绍，具体用法请翻阅`man`手册

+ `semget`用于获取一个或多个信号量
+ `semctl`用于控制信号量，可以`初始化`，`删除`，`获取状态参数`等
+ `semop`用于获取信号量或释放信号量



