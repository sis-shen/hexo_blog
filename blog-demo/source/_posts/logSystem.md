---
title: Linux日志系统
date: 2024-08-13 18:17:50
tags: Linux C++ 日志
---
# 引入
首先看看AI对日志系统的重要性是怎么解释的

>日志系统在软件开发和运维中扮演着至关重要的角色，主要体现在以下几个方面：
>
>`问题排查和调试`：日志记录了系统运行过程中的重要信息，包括错误信息、异常堆栈、接口调用信息等。开发人员可以通过分析日志快速定位问题，减少排查时间，提高开发效率。
>
>`性能监控`：通过记录应用程序的性能指标（如响应时间、请求吞吐量等），开发团队可以监控系统的健康状态，发现性能瓶颈并进行优化。
>
>`安全审计`：日志可以记录用户的操作和系统事件，为后期的安全审计提供依据。一旦发生安全事件，通过分析日志可以追踪到攻击来源和攻击方式。
>
>`用户行为分析`：通过对日志进行分析，开发团队可以了解用户的使用习惯和需求，这有助于改进产品功能和用户体验。
>
>`合规要求`：在许多行业中，日志记录是合规的要求之一。维护完整的日志记录可以帮助企业满足法律法规和行业标准。
>
>`系统健康监控`：通过实时监控日志，可以及时发现系统异常，进行预警和自动化处理，从而提高系统的可用性和稳定性。
>
>`故障恢复`：在系统出现故障时，日志记录可以帮助开发和运维人员还原问题发生前的状态，从而有助于快速恢复系统。

总之，良好的日志系统能够帮助开发和运维团队提高效率、强化安全、提升用户体验，并确保系统的稳定性和可靠性。因此，在系统设计时，合理规划和实施日志记录机制是非常必要的。

因此出于学习实践的目的，和方便日后自己打印日志，我们可以先自己用C++封装一个简单的日志系统

# 基本功能要求
+ 打印日志等级
+ 打印日志日期和时间
+ 格式化打印日志消息
+ 可选择输出到显示器，还是单个文件，还是按等级分文件

功能缺陷：没有完善清理日志文件的功能，因此不能投入到真正的项目中，否则时间长了可能造成日志文件过大的问题

# 头文件 和 宏定义
本次要用到的头文件依然较多,而用到的宏定义有文件名和缓冲区大小，因为有时日志要打印到文件中，所以要准备一个文件名
```C++
#include <iostream>
#include <stdarg.h>
#include <time.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define LOG_FILE "log.txt"//日志文件名

#define SIZE 1024  //缓冲区大小
```

# 枚举
日志系统中要用到较多选项，这里使用枚举最为直观和简洁

```C++
enum OUT_MODE//输出模式
{
    Screen,  //输出到显示器
    OneFile, //输出到一个文件
    MultiFile//按等级输出到多个文件
};

enum LEVEL //日志等级
{
    Info,    //一般信息
    Debug,   //debug日志
    Warning, //警告
    Error,   //错误
    Fatal    //失败
};
```
# 类的基本构成
这里我们封装一个`Log`类,
+ 储存一个私有成员变量`_om`(output mode) 
+ 重载`operator()()`使其有仿函数的功能
+ 声明`enable`函数用于切换输出模式
+ 声明`levelToString`将`LEVEL`的枚举类型转换为`string`类
+ 声明`printLog`用于调用不同接口打印日志
+ 声明`printMultiFile`用于向指定`文件名`打印日志
+ 声明`printMultiFile`用于向指定`日志等级`的文件打印日志

# 功能突破
有个别功能由于平时不常用/没见过，需要特别的突破一下

## 打印日期和时间
我们都知道在`<time.h>`中提供了`time()`函数获取时间戳，但如何方便地获取年月日和时分秒呢？

这里要引出里面的另一个接口`localtime()`了

`struct tm *localtime(const time_t *timep);`

可以看到，给它一个时间戳的指针，它就会返回一个`struct tm`结构体的指针，我们再来看看结构体里有什么

```C
struct tm {
    int tm_sec;         /* seconds */           //秒
    int tm_min;         /* minutes */           //分钟
    int tm_hour;        /* hours */             //小时
    int tm_mday;        /* day of the month */  //一个月的中的第几天
    int tm_mon;         /* month */             //月份(从0月开始)
    int tm_year;        /* year */              //年份,从1900年为0年开始
    int tm_wday;        /* day of the week */   //一周的第几天
    int tm_yday;        /* day in the year */   //一年中的第几天
    int tm_isdst;       /* daylight saving time */
};
```
可以看到，借助`localtime()`函数可以方便地获取详细时间,简化了代码

```C++
char leftBuffer[SIZE] = {0};
time_t t = time(nullptr);
struct tm *ctime = localtime(&t);//获取时间

snprintf(leftBuffer,sizeof(leftBuffer),"[%s][%d-%d-%d %d:%d:%d]",levelToString(level).c_str(),1900+ctime->tm_year,1+ctime->tm_mon,ctime->tm_mday,
    ctime->tm_hour,ctime->tm_min,ctime->tm_sec);
```
*levelToString()函数后文再介绍*

## 可变参数列表 与 格式化字符串
为了能让函数能够接受`格式化字符串`来方便打印日志，重载`operator()()`时要仿照`printf()`的函数声明

`void operator()(LEVEL level,const char *format,...)`

如上，最后一个参数用`...`

那么如何取出来呢？这里就要用到头文件`stdarg.h`的接口了

首先用`va_list`声明一个变量`s`，

然后用`va_start()`初始化`s`,因为函数的形参是从右往左实例化的，，所以还要传`...`左边第一个参数给它定位,例如`va_start(s,format)`

使用`va_start()`初始化后便可以调用其它函数了

而在使用完的最后，还要调用`va_end()`来结束对变量`s`的使用,例如`va_end(s)`

### va_arg()
`type va_arg(va_list ap, type);`

这个`宏函数`可以传类型作为参数，并从参数列表取出对应的参数

以写一个n个数相加的函数为例
```C++
#include <stdarg.h>
#include <iostream>

using namespace std;

int nsum(int n,...)//参数n表示n个数相加
{
    va_list ap;
    va_start(ap,n);

    int sum = 0;
    while(n--)
    {
        int num = va_arg(ap,int);//每次获取一个int 参数
        sum+=num;
    }
    va_end(ap);
    return sum;
}

int main()
{
    cout<<nsum(5,1,2,3,4,5)<<endl;//输出5个数相加
    return 0;
}
```
可以看到我们用`va_arg`实现了任意参数数量的整型相加

### vsnprintf()
`int vsnprintf(char *str, size_t size, const char *format, va_list ap);`

+ `str`:存放字符串的缓冲区指针
+ `size`:缓冲区大小
+ `format`原格式字符串
+ `ap`: va_list类型的变量

有了这个函数就可以方便地把可变参数列表打印到字符串里了

例如在本文的`Log`类中,

```C++
//将内容输出到缓冲区中
va_list s;
va_start(s,format);
char rightBuffer[SIZE];
vsnprintf(rightBuffer,sizeof(rightBuffer),format,s);
va_end(s);
```

# 完整代码
剩下的部分就比较简单了，也就不分开讲解了，直接放出完整代码

```C++
#pragma once

#include <iostream>
#include <stdarg.h>
#include <time.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define LOG_FILE "log.txt"//日志文件名

#define SIZE 1024  //缓冲区大小

enum OUT_MODE//输出模式
{
    Screen,  //输出到显示器
    OneFile, //输出到一个文件
    MultiFile//按等级输出到多个文件
};

enum LEVEL //日志等级
{
    Info,    //一般信息
    Debug,   //debug日志
    Warning, //警告
    Error,   //错误
    Fatal    //失败
};

class Log
{
public:
    Log(OUT_MODE om = Screen):_om(om)//默认构造函数
    {}
    ~Log(){}//没有特殊用处
    void enable(OUT_MODE om)//启用指定的输出模式
    {
        _om = om;
    }
    std::string levelToString(LEVEL level)
    {
        switch (level)
        {
            case Info:return "Info";
            case Debug:return "Debug";
            case Warning:return "Warning";
            case Error:return "Error";
            case Fatal:return "Fatal";
        default: return "None";
        }
    }


    void operator()(LEVEL level,const char *format,...)
    {
        char leftBuffer[SIZE] = {0};
        time_t t = time(nullptr);
        struct tm *ctime = localtime(&t);//获取时间

        snprintf(leftBuffer,sizeof(leftBuffer),"[%s][%d-%d-%d %d:%d:%d]",levelToString(level).c_str(),1900+ctime->tm_year,1+ctime->tm_mon,ctime->tm_mday,
            ctime->tm_hour,ctime->tm_min,ctime->tm_sec);

        va_list s;
        va_start(s,format);
        char rightBuffer[SIZE];
        vsnprintf(rightBuffer,sizeof(rightBuffer),format,s);
        va_end(s);

        char logtxt[SIZE*2];
        snprintf(logtxt,sizeof(logtxt),"%s %s",leftBuffer,rightBuffer);
        printLog(level,logtxt);;//打印日志
    }

    void printLog(LEVEL level, const std::string& logtxt)
    {
        switch(_om)
        {
            case Screen:std::cout<<logtxt<<std::endl;break;
            case OneFile:printOneFile(LOG_FILE,logtxt);break;
            case MultiFile:printMultiFile(level,logtxt);break;
            default: break;
        }
    }

    void printOneFile(const std::string& logname,const std::string& logtxt)//传两个参数用于代码复用
    {
        int fd = open(logname.c_str(), O_WRONLY|O_CREAT|O_APPEND,0666);
        if(fd<0) return;
        write(fd,logtxt.c_str(),logtxt.size());
        close(fd);
    }

    void printMultiFile(LEVEL level,const std::string logtxt)
    {
        std::string filename = LOG_FILE;//构建文件名
        filename+= ".";
        filename += levelToString(level);
        printOneFile(filename,logtxt);//复用代码
    }
private:
    OUT_MODE _om = Screen;
};
```
