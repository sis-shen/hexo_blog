---
title: C语言文件操作
date: 2024-07-15 20:32:12
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-25_12-56-36.png
---

# 用户级文件操作
`C语言`的文件操作也是用户级的文件操作，通过`FILE`对象来**管理**每一个`被打开的文件`，以及提供了**用户级**文件缓冲区，因此还涉及到冲刷缓冲区等问题

## `FILE` 类
`FILE`类描述了一个文件流。里面存储了**文件控制**所需的信息:

+ 指向自身缓冲区的的`指针`
+ 位置指示器
+ 状态指示器

所以`C语言`中对文件的管理就是对`FILE`对象的管理
# 基础操作 - 针对一般文件

## 基础示例
```C
#include <stdio.h>

int main()
{
	FILE* pfile = fopen("file.txt", "w");//"w"模式打开文件file.txt
	int code = 1;
	const char* msg = "this is a msg";
	fprintf(pfile, "get msg : %s code:%d", msg, code);//格式化输出字符串
	fclose(pfile);//冲刷缓冲区并关闭文件
	return 0;
}
```
以上代码创建了一个`file.txt`文件，输入`格式化字符串`(就和使用printf打印一样)。然后用`flcose`关闭文件流

## fopen 打开文件
`fopen`能够打开以各种`模式`磁盘上的文件

`FILE* fopen( const char * filename, const char * mode );`

**返回值**:
+ 成功时，返回一个不为空的`FILE*`指针，用于控制该文件
+ 失败时，返回`NULL`空指针并设置了全局变量`errno`

### 常见模式
| 模式 | 简述 |
| === | === |
| `"w"` | 创建一个**新的空文件**用于输出操作。如果已存在`同名文件`,清除原文件并当作新文件处理 |
| `"r"` | 只读模式打开文件。且该文件必须存在 |
| `"a"` | 打开已有文件时，仅用于在文件末尾`追加`新的内容。并且重定位函数`(fseek,fsetpos,reweind)`会被忽略，即使成功调用，也没有效果；当文件不存在时，会创建一个新的空文件 |
| "`r+`" | 读写模式打开已有文件，**不会清除**原文件内容,并且读写时均从文件开头开始。打开后第一次操作为写入时，从文件头部开始逐字符覆盖原文件。**注**读写模式同时只能`读`或`写`的一种，第一次取决于先进行哪种操作，可以用`fseek`函数转换读写模式 |
| "`w+`" | 读写模式打开新文件，若存在，则清除原文件内容;读写模式的切换和`"r+"`模式相同，唯一的区别就是打开时是否清除原文件内容 |
| "`a+`" | 从文件末尾打开读写模式，**不会清除原文件内容**，若打开后第一次操作为写，则从文件末尾开始；若第一次操作为读，则从头开始；读写模式的切换同上 |

### 二进制模式
如果要以二进制模式打开文件，只需要在上面的模式末尾加上字符`b`

若有`+`,则`b`既可以放在末尾也可以放在中间

+ `r+b` `w+b` `a+b`
+ `rb+` `wb+` `ab+`

### 强制新建文件
新的C语言标准,`C2011`(不是C++11),添加了一种新的说明符`w`,可以被添加在任意`"w"`后面

+ `"wx"` `"wbx"` `"w+x"`  `"w+bx"`/`"wb+x"`

当文件**存在**时,`w`会**强制**`fopen`函数失败,返回一个`NULL`空指针

## freopen 重定向文件流
`FILE* freopen ( const char *filename, const char *mode, FILE * pFile );`

+ 如果传入了新的文件名(与`pFile`控制的文件相比),该函数会**关闭**`pFile`原本指向的文件流，并取消关联。然后**不论**是否成功关闭，`freopen`会用和`fopen`同样的方式打开该文件
+ 如果文件名还是原文件，则只会改变打开模式

**返回值**:
+ 成功时，返回`pFile`内储存的地址
+ 失败时，返回`NULL`

### 特别的
`freopen`用于进程的输入输出重定向会特别有用

```C
freopen ("outfile.txt","w",stdout);//标准输出重定向到文件

freopen ("readfile.txt","w",stdin);//标准输入重定向到文件

freopen ("errdfile.txt","w",stderr);//标准错误输出重定向到文件
```

## 重定位 文件流位置指示器(stream position indicator)

### 文件的抽象内存结构
首先我们要明确一下文件的内存结构，如下图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-20_21-23-32.png)

这里及下文用图中的`ptr`代指标题的中的 `文件流位置指示器`,这个`ptr`决定了每一次对文件的`读/写`操作的**起点**,同时每一次`读/写`操作都会使`ptr`自动往后走，因此要显示控制`ptr`，就得使用`fseek,fsetpos`等接口

## fseek 重定位
`int fseek ( FILE *pFile, long int offset, int origin );`

`fseek`能过直接重定位`ptr`所指的

### 参数
+ `pFile`：用于控制文件的`FILE*`类型指针
+ `offset`：则是**偏移量**，长整型，表示偏移多少字节
+ `origin`：该形参标注了**偏移量**相对于哪个位置计算**实际位置**

`origin`有三个宏可以选
| 宏 | 实际位置 |
| `SEEK_SET` | 偏移量从`文件头`开始算 |
| `SEEK_CUR` | 偏移量从`当前文件指针ptr(上文介绍的)所在位置`开始算 |
| `SEEK_END` | 偏移量从`文件尾`开始算 |

### 返回值

+ 成功时,返回`0`
+ 失败时，返回`非零值`，同时，**这条语句失效**,上文说的`ptr`没有改变

## fgetpos 和 fsetpos 设置 ptr
可以用`fgetpos`获取`ptr`的当前位置，并使用`输出型参数`输出一个`fpos_t`类型的变量，而`fsetpos`可以用`fpos_t`类型的形参设置`ptr`的当前位置

就好比`ptr`是当前坐标，每次`fgetpos`得到一个传送点信息，而`fsetpos`就可以用这个传送点信息传送`ptr`过去

*示例如下*
```C
#include <stdio.h>

int main()
{
	//准备一个文件
	FILE* pfile = fopen("file.txt", "w");//"w"模式打开文件file.txt
	int code = 1;
	const char* msg = "this is a msg";
	fprintf(pfile, "get msg : %s code:%d", msg, code);
	fclose(pfile);
	//===========

	fpos_t pos1,pos2;
	pfile = fopen("file.txt", "r");

	fgetpos(pfile, &pos1);
	fgetc(pfile);
	fgetpos(pfile, &pos2);
	for (int i = 0; i < 3; ++i)
	{
		fsetpos(pfile, &pos2);//循环令ptr指向第二个字符
		printf("第2个字符为: %c\n", fgetc(pfile));
	}                                                     
	fsetpos(pfile, &pos1);//令ptr指向第一个字符
	printf("第1个字符为: %c\n", fgetc(pfile));

	fclose(pfile);

	return 0;
}
```
```输出
第2个字符为: e
第2个字符为: e
第2个字符为: e
第1个字符为: g
```

## fclose 关闭文件流
可以用`fclose`显式地关闭文件流

用法为`fclose(pFile);`

进程正常退出时，也会自动关闭文件流

## fprintf 格式化输出字符串
`fprintf`能格式化输出字符串到指定文件流，除了要**指定**文件流，格式化字符串的方式和`printf`一样

+ 且`fprintf(stdout,format,...)`和`printf(format,...)`效果一样

## fputs 输出字符串
`int fputs ( const char * str, FILE * stream );`

`fputs`能将`C语言`的字符串输入到指定文件流中

## fwrite 输出内存数据块
`size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);`

+ `ptr`是指向`内存数据块`的指针
+ `size`是每个`数组元素`的大小
+ `nmemb`是元素数量
+ `stream`是文件流

fwrite可以向指定文件流输入特定大小的内存数据块

```C
int main()
{
	FILE* pfile = fopen("file.txt", "w");//"w"模式打开文件file.txt
	char msg[] = "this is a msg";
	fwrite(msg, sizeof(char), strlen(msg), pfile);

	fclose(pfile);

	return 0;
}
```

## fscanf 格式化输入
`fscanf`能像`scanf`读取标准输入流一样,读取指定文件流

+ 且`scanf(stdin,format,...)`和`printf(format,...)`效果一样

## fgets 获取一行
`char * fgets (char *str, int num, FILE *stream );`
### 行为
`fgets`会一直读取直到`换行符`或`EOF文件结尾`结束读取，但`换行符`作为**非法**字符不会被拷贝到形参`str`中

+ 字符串结尾的`\0`会被自动添加,且**计算**在读入的**最大字符数**
+ `fgets`和`gets`有很大差别，它需要**指定**最大的读入字符数

### 形参
+ `str`为传入的字符数组作为缓冲区
+ `num`为拷贝的最大字符数，**包括**自动添加的结尾`\0`
+ `stream`为指定的文件流

### 返回值
+ 成功时，返回`str`的值
+ 失败时，返回`NULL`

### 示例
```C
#include <stdio.h>

int main()
{
	//准备一个文件
	FILE* pfile = fopen("file.txt", "w");//"w"模式打开文件file.txt
	int code = 1;
	const char* msg = "this is a msg";
	fprintf(pfile, "get msg : %s code:%d", msg, code);
	fclose(pfile);
	//=====

	pfile = fopen("file.txt", "r");
	char str[5] = { 0 };
	fgets(str, sizeof(str), pfile);//除去自动添加的\0,最多从文件里读入4个字符
	printf(str);
	printf("|");
	return 0;
}
```
```输出
get |
```
