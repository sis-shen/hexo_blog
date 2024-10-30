---
title: 初识Linux编程&初识makefile
date: 2024-09-30 15:12:06
tags:
---
# 前言
本文将以实用的视角，以快速上手Linux平台的C++编程为目的，介绍:
+ Linux的基本操作 
+ gcc/g++的常用指令
+ makefile的常用编写方式

# 软件准备
*如果选择用虚拟机，那请自己解决*

这里介绍使用`Linux服务器`配合`本地ssh链接软件`的方式在服务器上编写代码

首先是服务器，可以在主流服务器平台(阿里云，腾讯云，华为云)等购买服务器，一般有学生认证会更便宜,服务器配置不限，若要使用Linux,可以选择Ubuntu内核，(CentOs也行，就是太老了，且停止维护了)，然后自行查阅相关教程，设置默认管理员账户和密码，以及配置服务器，**开启ssh远程链接权限**

关于本地的ssh链接软件，这里我只推荐两个我使用过的
+ xshell7
+ vscode+ssh插件

其中`xshell`比较简单，安装好就能用 [戳我去下载免费版xshell🔗](https://www.xshell.com/en/free-for-home-school/#:~:text=We%20believe%20users%20from%20all%20backgrounds%20and%20circumstances)，怎么登录服务器？自行百度~。但缺点是仅支持控制台，所以写代码只能用`vim`等支持控制台的文本编辑器

`vscode`+`ssh`插件会比`xshell`麻烦点，还要修改配置文件，具体教程可参考[b战视频🔗](https://www.bilibili.com/video/BV13Z421T7oz/)

但优点是`vscode`在支持终端(控制台)的同时还支持原本的`文本编辑器界面`,相当于可以用windows的`GUI可视化窗口`写代码了

*下图为xshell(左)和vscode(右)的界面对比*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409302132012.png)

# Linux快速入门
不同于Windows平台的`GUI可视化`界面，在服务器编写代码，由于为了节省服务器资源，是不会安装GUI可视化界面的，所以还需要适应和学习终端的操作

## Linux 常用指令

### ls 
`ls`指令用于查看当前`目录`所含有的文件

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409302202715.png)

但是单纯的ls显示的内容太简略了，所幸它是能加参数的,格式是`-参数字符(串)`,多个参数时直接连在一起，不用间隔

常用参数如下

+ `-l`显示更多的详细信息
+ `-a`显示全部文件，包括隐藏文件(以`.`开头的目录/文件)
+ `-i`显示唯一文件标示符s

但是每次都打参数太麻烦了怎么办？没关系~！还有快捷命令`ll`，相当于`ls -al`

平时最常用的就是`ll`

### cd
那么怎么切换路径呢？使用`cd 路径名称`即可

**关于路径**，又分为`绝对路径`和`相对路径`

在`Linux`中，所有目录的最终根目录都是`/`,没错，单一个`/`

而用户登录后所在的文件夹是用户文件夹，以用户`supdriver`为例,一般路径为`/home/supdriver`,其中这个目录也可以用`~`指代

至于`相对目录`,对于除了根目录的普通目录，都会存在一个`.`目录和`..`目录，其中`.`目录就是指这个目录自己，而`..`则是上级目录

在了解完路径的基本概念后，使用`cd`命令就很简单了

下面举几个简单例子

+ `cd /home`打开家目录，所有的用户文件夹都在这
+ `cd ~`打开当前登录用户的用户文件夹
+ `cd -`打开上次`cd`打开的文件夹
+ `cd /`打开根目录
+ `cd ./tmp`打开当前目录下的`tmp`目录
+ `cd ../`打开上级目录

### pwd
既然已经知道了路径的概念，有时候我们想获取当前所在路径怎么办呢？使用`pwd`指令吧,不需要参数

### mkdir
`mkdir`是专门用于创建普通目录的指令，用法有:

+ `mkdir 目录名`在当前目录创建目录
+ `mkdir 路径/目录名`在指定路径创建目录（如果有权限的话）

## touch
`touch`指令可以用于生成文件,`linux`下的文件都是同样的种类，而且在linux中文件的后缀对操作系统识别文件类型并不重要

`touch file`在当前目录创建一个名为`file`的普通文件,默认权限为`不可执行，可读，可写`，当然touch支持生成不同权限的文件，这里不作讨论，对与Linux权限系统有兴趣的可以自行学习

## rm
`rm`可用于删除文件,同时也有可选参数，不带参数删除文件时，会询问确认

+ `rm <文件名>`删除文件，但会先询问确认
+ `rm -f <文件名>`强制删除文件，不会确认
+ `rm -rf <文件名/目录名>`强制递归删除文件/目录，可用于删除含有多层结构的目录

千万不要执行`rm -rf /`，否则会把系统文件也删了，这个操作系统就废掉了

## Linux编程
不同于Windows上常用的`集成开发环境`，如`Visual Studio`,`devC++`,Linux的开发环境为`文本编辑器`+`gcc/g++`编译器，二者是独立的。

### 文本编辑器

#### Vim

最常用的Linux文本编辑器当属`vim`了，因为基本每台Linux服务器都是自带`vim`的，支持插件扩展和配置文件，这里不多作介绍

`Vim`是一款文本编辑器,下面介绍在vim界面中的常用指令

**三种模式**:`命令模式(Command Mode)` `插入模式（Insert Mode` `命令行模式（Command-Line Mode）`（这里称命令行模式为`底行模式`）

三者关系如下图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2023-12-11_10-39-04.png)

+ 命令模式
`vim`界面中多摁几次`ESC`就能退出其它模式回到`命令模式`，在这个模式下可以使用一系列vim[快捷键](https://linux.cn/article-8144-1.html)

+ 底行模式 
`tips`:不管目前是什么模式,先狂按`ESC`,回到`命令模式`,然后输入`:`进入`底行模式`,准备开始输命令

`命令组成`
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2023-12-11_12-48-54.png)

+ 保存`:w`->强制保存`!w`
+ 退出`:q`->强制退出`:!q`
+ 保存并退出`:wq`-.强制保存并退出`:!wq`
+ 对比`:vs `+`(源文件路径)`


+ 插入模式
在`命令模式`下按键盘`i`进入`插入模式`，执行正常的文本编辑功能

#### vscode
`vscode`本身就是**文本编辑器**,这是各种插件让它的功能多了起来,这里推荐安装`C/C++`相关的插件（其实它自己识别到C++文件就会推荐你安装的）。至于`vscode`怎么用，就跟平时在Windows上用一样，不过同时也可以用终端命令行来辅助操作。


### gcc/g++编译器
**gcc是专门编译C语言文件的，拿去编译C++文件会报错！！**

这里介绍g++的使用

`g++`默认编译终端当前文件夹下的源文件，接下来介绍常见用法

1. 直接编译
`g++ <源文件>`将直接编译目标源文件，并产生`a.out`可执行文件

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410010914401.png)

2. 自定义产生的目标可执行文件
`g++ -o <可执行文件名> <源文件>`/`g++ <源文件> -o <可执行文件名>`,二者皆可

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410010932958.png)

3. 编译产生目标文件(`*.o`)并链接产生可执行文件

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410011007675.png)

如图，使用`-c`参数可以编译产生`*.o`目标文件，最后再链接到同一个文件中。下面举一个实际例子

我们有如下源文件`main.cpp` `add.cpp` `sub.cpp` `add.hpp` `sub.hpp`(其中main.cpp已经#include了头文件)

我们将依次输入如下指令产生`mycmd`可执行文件

+ `g++ -c main.cpp -o main.o`
+ `g++ -c add.cpp -o add.o`
+ `g++ -c sub.cpp -o sub.o`
+ `g++ -o mycmd *.o`  `*.o`表示选中所有以`.o`结尾的文件

通过如上四个指令，就完成了可执行文件的编译，结果如下图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410011018172.png)

## Makefile入门
讲真手动维护这些编译文件太麻烦了，有没有更高效，更自动化一点的方式呢？正好Linux服务器都提供了`makefile`工具用于构建项目

我们接下来学习如何用`makefile`自动编译代码和清理可执行文件

### 创建makefile文件
实际上创建`makefile`和创建`Makefile`文件效果是一样的

### 单源文件的makefile编写

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410140900036.png)

如上图为示例代码，接下来我们逐步讲解

在一段makefile语句中，通常遵循以下格式
```makefile
<目标文件>:<依赖文件> (<依赖文件2> <依赖文件3> ...)
    <命令行命令>
    (<命令行命令2>)
    (<命令行命令3>)
    ...
```

其中的`<目标文件>`就是我们这句`makefile`指令执行完毕后要产生的目标文件,依赖文件则是在执行这句`makefile`指令前，这些依赖文件必须存在，若不存在，则执行失败/向后寻找可产生依赖文件的`makefile`命令并自动执行

至于命令行命令，就是我们在终端中输入的指令

图中还有个特殊的用法，就是`.PHONEY`声明`伪目标对象`,一个`伪`字就说明了它不是真正的文件，而是一个抽象的目标文件，执行该段`makefile`时不会在磁盘上真正创建这样的文件。因此它也有个特性--`每次调用该段语句都会执行命令`。那如果是不使用`.PHONY`的普通`makefile`语句段呢？它只会在依赖文件发生更新后，才会在调用时真正执行命令

**makefile的部分简化写法**

makefile里也是可以声明和调用变量的，这里先不介绍声明变量，主要介绍调用变量，因为它还提供了默认变量用于简化代码

其中调用变量的方法为`$`+`变量名`,常用的默认变量为

+ `$@`调用储存了`依赖文件`的变量
+ `$^`调用储存了`目标文件`的变量

以上图的的makefile为例，我们可以将其改写为:

```makefile
mycmd:main.cpp
	g++ -o $@ $^

.PHONY:clean
clean:
	rm -f mycmd
```

**makefile指令的使用**
如何在终端调用makefile里写好的命令呢？`Linux`提供了一个指令，叫做`make`可用于调用`makefile`内的指令

具体用法为`make <目标文件名>`,比如要生成`clean`伪目标对象，（其实就是执行清理操作，而不是创建操作）,就要在终端输入`make clean`

**特别的**，为了方便使用，往往`makefile`文件的第一个目标文件的构建，可以省略参数，直接输入`make`再回车即可执行命令

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410011920420.png)


#### 综合使用之多文件编译

我们有如下源文件`main.cpp` `add.cpp` `sub.cpp` `add.hpp` `sub.hpp`(其中main.cpp已经#include了头文件)

我们该如何用makefile一键(make)编译出可执行文件呢？这里就要用到上文的知识，组织好文件之间的依赖关系和构建指令

```makefile
mycmd:main.o add.o sub.o
	g++ -o mycmd *.o

main.o:main.cpp
	g++ -c main.cpp -o main.o

add.o:add.cpp
	g++ -c add.cpp -o add.o

sub.o:sub.cpp
	g++ -c sub.cpp -o sub.o

.PHONY:clean
clean:
	rm -f mycmd *.o
```

代码如上，最终的`mycmd`文件依赖`main.o`,`add.o`,`sub.o`，所以后文又提供了这三者的构建方法，最后再提供一个clean方法用于清理文件

# 小结
以上便是Linux平台CPP编程常用到的工具和方法