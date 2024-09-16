---
title: 初识动静态库
date: 2024-07-08 09:10:13
tags: 动态库 静态库
---

# 静态库(.a)
程序在**编译链接**的时候把库的代码链接到可执行文件中。编译出的程序**运行时不再需要**静态库

## 生成和使用静态库
这里使用`gcc`编译获得`.o`链接文件，再用`gnu`归档工具中的`ar`指令配合`-rc`选项(replace and create)封装库文件,最后借助`makefile`简化文件目录的封装操作

使用第三方库时,还可以使用`ar -tv`查看库中的目录列表
+ t:列出静态库中的文件
+ v:verbose 详细信息

使用静态库：

`gcc`编译时使用第三方静态库必须包含以下选项：
+ `-I`指定头文件目录
+ `-L`指定库路径
+ `-l`指定库名（不加空格）,库名要把`文件名`前面去掉`lib`,后面去掉`.a`才是真正的库名

否则编译器不知道头文件在哪，或者不知道链接哪个库

### 库搜索路径
库的搜索会按照一定的顺序

1. 从左到右搜索-L指定的目录。
2. 由环境变量指定的目录 （`LIBRARY_PATH`）
3. 由系统指定的目录
   + `/usr/lib`
   + `/usr/local/lib`


### 样例
测试程序（共`5`个文件）
```C
/////////////add.h/////////////////
 #ifndef __ADD_H__
 #define __ADD_H__ 
 int add(int a, int b); 
 #endif // __ADD_H__
 /////////////add.c/////////////////
 #include "add.h"
 int add(int a, int b)
 {
 return a + b;
 }
 /////////////sub.h/////////////////
 #ifndef __SUB_H__
 #define __SUB_H__ 
 int sub(int a, int b); 
 #endif // __SUB_H__
 /////////////add.c/////////////////
 #include "add.h"
 int sub(int a, int b)
 {
 return a - b;
 }
 ///////////main.c////////////////
 #include <stdio.h>
 #include "add.h"
 #include "sub.h"
 
 int main( void )
 {
 int a = 10;
 int b = 20;
 printf("add(10, 20)=%d\n", a, b, add(a, b));
 a = 100;
 b = 20;
 printf("sub(%d,%d)=%d\n", a, b, sub(a, b));
 }

```
然后我们编写`makefile`文件
```makefile
lib = libmymath.a

$(lib):add.o sub.o 
	ar -rc $@ add.o sub.o 

add.o:add.c 
	gcc -c add.c -o add.o

sub.o:sub.c
	gcc -c sub.c -o sub.o

.PHONY:clean
clean:
	rm -f *.o *.a 

.PHONY:output 
output:
	mkdir -p lib/include
	mkdir -p lib/mymathlib
	cp *.h lib/include
	cp *.a lib/mymathlib
```
`lib`作为`makefile`中的变量，储存了`库名`，默认的`make`指令用于打包库文件,其依赖的`add.o`和`sub.o`会自动调用后两句编译产生。

`make output`指令自动准备好目录并拷贝头文件和静态库

然后编译产生可执行程序

```SHELL
gcc main.c -o mycmd -I ./lib/include -L ./lib/mymathlib -lmymath
```
注意,使用第三方库时`-I`,`-L`和`-l`缺一不可

但如果安装了第三方库时，即将`库文件/库文件的软链接`拷贝至`/lib64/`，将`头文件/头文件的软链接`拷贝至`/usr/inlude/`，使`gcc`能够自动搜索到路径，则编译时只需用`-l`指定`库名`即可


# 动态库(.so)
动态库的作用是使程序在**运行时**才去链接动态库的代码，多个程序共享使用库的代码，从而使编译出的可执行文件更小，节省磁盘空间。使用动态库时，动态库也要**加载到内存中**

## 使用动态库底层的过程
1. 使用动态库编译可执行程序：一个与动态库链接的可执行文件仅仅包含它用到的函数入口地址的一个表，而不是外部函数所在目标文件的整个机器码
2. 动态链接: 在可执行文件开始运行以前，外部函数的机器码由操作系统从磁盘上的该动态库中复制(加载)到内存中，这个过程称为`动态链接`（dynamic linking）
3. 运行时调用动态库：调用来自动态库的函数时，实现函数功能的机器码将由动态库提供

## 生成和使用动态库
这里使用`gcc`加`-fPIC`(产生位置无关码position independent code)和`-c`编译获得`.o`链接文件，再用`gcc`指令配合封装库文件,最后借助`makefile`简化文件目录的封装操作

使用动态库：

`gcc`编译时使用第三方静态库必须包含以下选项：
+ `-I`指定头文件目录
+ `-L`指定库路径
+ `-l`指定库名（不加空格）,库名要把`文件名`前面去掉`lib`,后面去掉`.a`才是真正的库名

否则编译器不知道头文件在哪，或者不知道链接哪个库

特别的，当动静态库同时存在时，`gcc`默认优先使用动态库，除非加`-static`参数

**但是**，仅仅编译出可执行文件还不够，因为可执行文件还**找不到**动态库,所以我们还得告诉系统中的`加载器`，我们的动态库在哪,所以我们要把动态库安装到`/lib64/`,拷贝文件或建立软连接都可以（需要`sudo`或`root`权限）

亦或者修改环境变量`LD_LIBRARY_PATH`

### 示例
文件代码同上，这里展示`makefile`文件

```makefile
lib = mymathlib.a

$(lib):add.o sub.o 
	gcc -shared -o libmymath.so *.o 

add.o:add.c 
	gcc -fPIC -c add.c -o add.o 

sub.o:sub.c
	gcc -fPIC -c sub.c -o sub.o

.PHONY:clean
clean:
	rm -f *.o *.a 

.PHONY:output 
output:
	mkdir -p lib/include
	mkdir -p lib/mymathlib
	cp *.h lib/include
	cp *.so lib/mymathlib
```

### 库搜索路径
库的搜索会按照一定的顺序

1. 从左到右搜索-L指定的目录。
2. 由环境变量指定的目录 （`LIBRARY_PATH`）
3. 由系统指定的目录
   + `/usr/lib`
   + `/usr/local/lib`

# 小结
库文件命名规则`libxxx.a`/`libxxx.so`

库文件名称->库名：去掉前缀`lib`,去掉后缀`.so`/`.a`

使用动态库的可执行程序需要`安装动态库`或修改环境变量`LD_LIBRARY_PATH`
