---
title: =Linux=一步步自己写一个shell程序
date: 2024-06-04 00:13:55
tags: Linux C
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/craftTerminal.png
---

***
系统：阿里云服务器Linux CentOs 7

编辑器: vim 

编译器: gcc (支持C99)
***

# 文件
本次写的程序较为简单，所以只使用一个源文件

所以在shell中`touch`一个`makefile`和一个`myshell.c`
> shell
```Linux
touch makefile
touch myshell.c
```

然后编辑`makefile`文件
> makefile

```makefile
1 myshell:myshell.c                         
  gcc -o $@ $^ -std=c99

.PHONY:clean
clean:
   rm -f myshell
```
# 头文件
本程序因函数较杂，会`include`较多头文件

> myshell.c
```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>
```

# 宏定义
为了统一修改部分参数，以及使参数更易读，这里使用部分宏定义
> myshell.c
```C
#define LEFT "["
#define RIGHT "]"
#define LABEL "# "//注意有个空格

#define LINE_SIZE 1024
#define ARGC_SIZE 32
#define DLIM " \t" //可能有多个分隔符
#define EXIT_CODE 446 //特殊的退出码，表示程序未正常退出，具体数字目前没有约定
```

# 全局变量
我们需要用全局的变量来存储`命令行`(command line)和`参数包`

> myshell.c
```C
char cline[LINE_SIZE];
char* arg[ARGC_SIZE];
int last_code = 0;
```

# 用`Interact`函数实现交互功能

## 打印命令行头部
为了打印命令行头部，我们需要知道三样东西：用户，主机，工作路径，这里包装了三个函数来分别调用`getenv`函数

> myshell.c
```C
const char* getusername()//获取用户名
{
    return getenv("USER");
}

const char* gethostname()//获取主机名
{
    return getenv("HOSTNAME");
}

const char* getpwd()//获取工作路径
{
    return getenv("PWD");
}

```

因此打印的代码为
```C
printf(LEFT "%s@%s %s" RIGHT LABLE,getusername(),gethostname(),getpwd());
```

## 获取命令行
使用Linux的终端时，我们会打`命令+空格+参数...`,因此我们的`myshell`程序也要支持连空格一起读入,读入一整行命令

所以`scanf`并不适合用来读入命令，这次我们使用`fgets`函数，这个函数可以从`文件流`中整行读入，而正好在终端输入的字符都储存在`标准输入流`,即`stdin`中,因此可以用一行代码获取`命令行`

为安全考虑，这里使用一个临时变量`s`来接受`fgets`的返回值并用`assert`判空,但在`release`版本中`assert`不被编译，导致变量`s`未被调用，而报警告（甚至报错），所以还要再加一句`(void) s`,只为了调用一下`s`,没有更多用处

之后便完成了文件流的读取

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-06-04_16-25-34.png)

但此时获得的命令行在`\0`前以`\n`结尾，所以要把`\n`替换为`\0`

```C
char *s = fgets(cline,size,stdin);
assert(s);//s为空时报错
(void) s;//防止因未调用s而报警告

cline[strlen(cline) - 1] = '\0';
```

## 整个函数体
> myshell.c
```C
void Interact(char* cline,int size)
{
    printf(LEFT "%s@%s %s" RIGHT LABLE,getusername(),gethostname(),getpwd());//打印头部
    fgets(cline,size,stdin);//获取命令行
    assert(s);//s为空时报错
    (void) s;//防止因未调用s而报警告
    cline[strlen(cline) - 1] = '\0';

    printf("echo: %s\n");//写一段测一段的测试代码，输出获取的命令行,测完可删
}
```

## 测试
先在`main`函数里调用一次`Interact`函数测试一下

我的测试结果如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-06-04_16-42-55.png)

可以看到达到了预期效果，但是`工作路径`太长了，还是学一学Linux的展示方式吧，我们来把`getpwd`函数重写一下

## 重写`getpwd()`
```C
const char* getpwd()
{
    const char* pwd = getenv("PWD");
    int n = strlen(pwd);
    while(n)
    {
        if(pwd[n] == '/') break;
        n--;
    }
    return (pwd+n+1);
}
```
这样打印出的工作路径仅为当前文件夹，可以缩短很多长度

# 分割命令行
现在的`cline`中的命令行还是完整的一串，需要分割出命令和参数包，因此我们也封装一个函数`Splitcline`

这里使用的是`string.h`中的`strtok`函数，可以用特定的单个或多个字符将字符串分割

> myshell.c

```C
int Splitcline(char*cline,char** argv)
{
    memset(argv,0,sizeof(char*) * ARGV_SIZE);
    int i = 1;
    argv[0] = strtok(cline,DLIM);
    if(argv[0])
        while(argv[i++] == strtok(NULL,DLIM));

    *argv_n = i-1;//输出型参数
}

```

再写一段测试代码

> myshell.c

```C
int main()
{
    Interact(cline,sizeof(cline));

    Splitcline(cline,argv);
    for(int i = 0;argv[i];i++)//逐行打印输出argv的内容
    {
        printf("%s\n",argv[i]);
    }
    return 0;
}
```

# 执行外部命令
通过`fork`函数创建子进程，然后用`execvp`替换子进程，通过环境变量`PATH`找到外部命令并替换到子进程执行，同时父进程`myshell`调用`waitpid`函数等待子进程结束，保证`myshell`程序正常运行

> myshell.c

```C
void ExternalCommand()
{
    pid_t id = fork();
    if(id <0)
    {
        perror("fork");
        return;
    }
    else if(id == 0)//child
    {
        execvp(argv[0],argv+1);//+1之后才是参数列表
        exit(EXIT_CODE);
    }
    else
    {
        int status = 0;
        pid_t rid = waitpid(id,&status,0);
        if(rid == id)
        {
            last_code = WEXITSTATUS(status);
        }
        return;
    }
}
```

# 执行内建命令
`shell`中并不是所有的命令都由子进程完成的，比如用`cd`命令改变工作路径，就不能让子进程去执行(~~否则只是改了子进程的路径~~),因此我们还需要加一个内建命令接口

> myshell.c

```C
int BuildCommand(char* _argv[],int _argv_n)//处理内建命令
{
  if(_argv_n == 2 && strcmp(_argv[0],"cd")== 0)//特殊处理的命令1
  {
    chdir(argv[1]);
    getpwd();
    sprintf(getenv("PWD"),"%s",pwd);
    return 1;//完成执行返回1
  }
  //还可以继续else uf 加特殊处理的命令2,3,4,,,n
  return 0;//未执行内建命令。返回0
}

```

# 完成框架

至此，把`main`函数组织好后，一个简单的`shell`代码框架就搭好了，可以根据需要继续扩展`内建命令`的内容，比如导出环境变量，实现`echo`指令等（略写）。

> myshell.c

```C
int main()
{
   int quit = 0;
   while(!quit)
   {
     Interact(cline,sizeof(cline));
     int argv_n;
     Splitcline(cline,argv,&argv_n);
 
     if(argv_n == 0 )continue;
     
     int flag = BuildCommand(argv,argv_n);
     if(!flag) ExeternalCommand();
   }
 
    return 0;                            
}          
```

# 拓展
这里的命令行处理并没有考虑`输入/输出重定向`,所以仍有较大的需要完善的地方

## 源代码

[点我去往github仓库](https://github.com/sis-shen/Linux_Code)。

