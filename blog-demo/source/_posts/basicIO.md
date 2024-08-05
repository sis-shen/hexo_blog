---
title: 基础IO
date: 2024-07-10 15:24:34
tags: Linux
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-25_12-52-35.png
---
# 共识原理
+ 文件 = 内容 + 属性
+ 被打开的文件需要加载到内存中
+ **内存中**的文件需要被操作系统**管理**

# 用户级文件接口
[详见C++文件操作](https://www.supdriver.top/2024/05/14/cpp-file-op/)

[详见C语言文件操作](https://www.supdriver.top/2024/07/15/cfileop/)

# Linux系统调用接口

## fd 文件描述符 与访问文件的本质

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-24_17-10-26.png)

`fd`(*file descriptor*),即文件描述符,下文的系统调用接口经常以`fd`命名变量，`fd`是整形变量，作为数组下标，用于管理**打开的文件**

可以看到,一个进程通过`struct files _struct`里的指针数组，管理多个同时打开的文件

且每个进程启动时，会默认打开三个文件,且默认`fd`固定

1. stdout 

## read 
**所需头文件**
`#include <unistd.h>`
**声明**
`ssize_t read(int fd, void *buf, size_t count);`

**参数**
+ `fd`即为目标文件的文件描述符
+ `buf`为要从文件读取字节到的内存地址
+ `count`为最大读取字节数

**返回值**
+ 若成功，返回读取文件的字节数,类型为`ssize_t`,是层层封装的`long int`
+ 若失败，返回`-1`,并设置`errno`的值

## write
**所需头文件**
`#include <unistd.h>`
**声明**
`ssize_t write(int fd, const void *buf, size_t count);`

**参数**
+ `fd` 为目标文件的文件描述符
+ `buf`为要写入文件的`源内存地址`,输入字节数量取决于`count`形参
+ `count`为要输入的字节数量,若要输入为字符串，且要输入字符串的全部内容，建议使用`strlen(buf)`，防止输入`\0`,因为对于**文件**来说，`\0`是**非法字符**

**返回值**
+ 若成功，返回写入文件的字节数,类型为`ssize_t`,是层层封装的`long int`
+ 若失败，返回`-1`,并设置`errno`的值

**特别的**
`read`函数从文件中读取的是`字节`内容，不把读取的内容看作字符串，因此，**不会自动**添加`\0`在写入`buf`内容的结尾

*用法见后文对`open`的介绍*


## open
**所需头文件**
```C
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
```
**声明**
`int open(const char *pathname, int flags);`
`int open(const char *pathname, int flags，mode_t mode);`

**参数**:

+ `pathname`为文件路径，若只有文件名，则默认在当前工作路径搜索
+ `flag`则是一个`位图`,而**不应**看作整型参数，传参时可用`|`位运算传递多个参数到位图中,例如`O_CREAT | O_WRONLY`
+ `mode`则是在**创建**文件时,传入权限信息,这里使用`八进制表示法`，例如传入`0666`
**返回值**:
+ 若成功，返回打开文件的`fd`值
+ 若失败，则返回`-1`

### 写入操作

相关的`flags`

+ `O_WRONLY` 仅写入
+ `O_CREAT` 如果文件不存在，就创建，**注**新文件的权限由`open`函数传入的`mode`参数决定
+ `O_TRUNC` 如果文件已存在且是`常规文件`,并且打开的模式组合**支持写入操作**(`O_RDWR`或`O_WRONLY`),该文件内容将会被清除。但如果该文件是`FIFO`(命名管道)文件或`终端设备文件`，则`O_TRUNC`将会被**忽略**
+ `O_EXCL` 保证此次`open`操作打开新文件。**必须**和`O_CREAT`联合使用，否则打开**失败**。若`pathname`存在，即该路径的文件存在时，也会打开**失败**

#### 打开已有文件并写入
```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    //前提是log.txt已存在
    int fd = open("log.txt",O_WRONLY,0666);//只写模式打开文件
    
    char msg[] = "this is a msg";//准备字符串
    write(fd,msg,strlen(msg));//写入
    close(fd);//关闭文件
    return 0;
}
```

当原本`log.txt`为空文件时
```log.txt
this is a msg
```
当原`log.txt`不为空且内容长度大于程序输入的`msg`时，发生`部分覆写`

例如原内容为`0000111100001111`时，执行后为
```log.txt
this is a msg111
```
可以看到有一部分没有被覆盖

#### 打开空文件 或 创建空文件
```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    //唯一的区别是这里的参数
    //O_CREAT可以创建空文件
    //O_TRUNC保证打开已有文件时，清空原文内容
    int fd = open("log.txt",O_CREAT|O_WRONLY|O_TRUNC,0666);
    
    char msg[] = "this is a msg";
    write(fd,msg,strlen(msg));
    close(fd);
    return 0;
}
```
#### 追加写入
追加写入只需把`O_TRUNC`改成`O_APPEND`即可

```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    int fd = open("log.txt",O_CREAT|O_WRONLY|O_APPEND,0666);//追加模式打开文化
    
    char msg[] = "this is a msg";
    write(fd,msg,strlen(msg));
    close(fd);
    return 0;
}
```

这里我们事先删除`log.txt`文件，然后运行两次编译出的程序,可以获得如下内容
```log.txt
this is a msgthis is a msg
```
可以看到内容追加了两次

### 读取操作
相关的`flags`

+ `O_RDONLY`只读模式打开文件
+ `O_RDWR` 读写模式打开文件

#### 只读模式读取内容
```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    int fd = open("log.txt",O_RDONLY);
    char* buf[1024];
    ssize_t n = read(fd,buf,sizeof(buf)-1);//这里要储存字符串，所以要留一位给\0
    if(n<0) perror("read");//打开失败，输出错误信息
    else buf[n] = '\0';//添加结尾
    printf("%s\n",buf);//打印读取到的内容
    return 0;
}
```
实现准备内容为`123456`的`log.txt`文件

然后运行`./mycmd`

得到输出和文件内容
```SHELL
456
```
```log.txt
zzz456
```
关于`read`没读取到前面新写入的`zzz`,是因为`wtrite`和`read`操作都是从文件的`同一处继续`操作的，并不会发生回退
## close
`int close(int fd);`

用于冲刷缓冲区，并**关闭**一个文件描述符

## dup2 文件重定向
`int dup2(int oldfd, int newfd);`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-24_19-55-30.png)

如图所示，`dup2`能将`oldfd`对应的数组元素`覆盖`到`newfd`对应的数组元素处，完成对`newfd`对应文件的重定向

图中就是完成了对`标准输出`的重定向,像`printf`之类的函数会直接输出内容到文件中,而不是显示器

```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    int fd = open("log.txt",O_RDWR|O_CREAT|O_TRUNC,0666);//打开一个新的空文件
    dup2(fd,1);//标准输出重定向

    printf("output1\n");//输出
    printf("output2\n");//输出

    return 0;
}
```
运行代码后，可以看到`终端`**没有输出**

而打开`log.txt`
```log.txt
output1
output2
```
 
# 子进程 与 父进程的文件关系

## 子进程对父进程的拷贝
先运行一段代码测试 子进程是否**继承**父进程的打开文件

```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/wait.h>

int main()
{
    int fd = open("log.txt",O_RDWR|O_CREAT|O_TRUNC|O_APPEND);//追加模式打开一个新文件
    dup2(fd,1);//在fork前就打开文件

    pid_t id = fork();//创建子进程
    if(id == 0)
    {
        printf("child output\n");//子进程输出到文件
        exit(0);
    }
    else 
    {
        waitpid(id,0,0);//阻塞等待子进程
        printf("parent output\n");//父进程输出
    }

    return 0;
}
```
这段代码中，我们在`fork`**之前**完成了对标准输出的**重定向**,然后`fork`之后令父进程和子进程进行不同的标准输出

运行结果为父进程和子进程的`标准输出`都**重定向**到了文件

```log.txt
child output
parent output
```

## 子进程和父进程的 独立性
接下来一段代码测试 父进程 和 子进程 的打开文件是否独立

```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/wait.h>

int main()
{
    int fd = open("log.txt",O_RDWR|O_CREAT|O_TRUNC|O_APPEND);//追加模式打开一个新文件

    pid_t id = fork();//创建子进程
    if(id == 0)
    {
        dup2(fd,1);//只有子进程重定向了标准输出
        printf("child output\n");//子进程输出到文件
        exit(0);
    }
    else 
    {
        waitpid(id,0,0);//阻塞等待子进程
        printf("parent output\n");//父进程输出
    }

    return 0;
}
```

这里我们在`fork`之前都不进行重定向，`fork`后仅对子进程进行了标准输出重定向，而父进程不作任何重定向

在运行后发现子进程的输出重定向不会影响父进程,二者有`独立性`

```SHELL
parent output
```
```log.txt
child output
```

# 进程替换
先在同级文件夹准备一个待替换的程序

> execute.c
```C
#include <stdio.h>

int main()
{
    printf("exe output\n");
}
```
然后运行`gcc -o execute execute.c`编译获得一个程序

然后准备主程序

> mycmd.c
```C
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/wait.h>

int main()
{
    int fd = open("log.txt",O_RDWR|O_CREAT|O_TRUNC|O_APPEND);//追加模式打开一个新文件

    pid_t id = fork();//创建子进程
    if(id == 0)
    {
        dup2(fd,1);//只有子进程重定向了标准输出
        execvp("./execute",NULL);//进程替换
        exit(0);
    }
    else 
    {
        waitpid(id,0,0);//阻塞等待子进程
        printf("parent wait success\n");//父进程输出
    }

    return 0;
}
```
这里我们使子进程**先**标准输出重定向， **再**进行进程替换，发现替换后的进程，也是标准输出重定向的状态

```log.txt
exe output
```

## 结论
进程替换**不会**改变原进程的文件打开状态和重定向关系