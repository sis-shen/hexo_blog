---
title: 【保持最新】C++基于多设计模式下的同步&异步⽇志系统
date: 2024-09-24 16:25:24
tags:
---
> **当前版本v1.0**
## 前置知识
+ [Linux日志器（简易版）](https://www.supdriver.top/2024/08/13/logSystem/)
+ [初识Linux线程](https://www.supdriver.top/2024/08/14/thread/)
+ [初识设计模式&单例模式&代理模式--设计模式介绍(1)](https://www.supdriver.top/2024/09/24/DP-singelton-pattern/)
+ [工厂模式与建造者模式--设计模式介绍(2)](https://www.supdriver.top/2024/09/24/DP-factory-pattern/)

# 项目介绍

## 背景引入

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

## 目标功能
相比之前较为简单的**日志类**,我们这次将日志系统其升级为一个项目级的完整系统。

因此，我们要实现更多的功能:
+ ⽀持多级别⽇志消息
+ ⽀持同步⽇志和异步⽇志
+ ⽀持**可靠写⼊**⽇志到控制台、⽂件以及滚动⽂件中
+ ⽀持**多线程程序并发**写⽇志
+ ⽀持扩展不同的⽇志⽬标落地

## 开发环境
+ Linux环境（Ubuntu/CentOS）
+ vscode/vim 文本编辑器
+ g++/gdb 编译器
+ Makefile 

## 核心技术应用
+ 类层次设计（抽象类，继承和多态的应用）
+ C++11 新语法的应用（多线程，右值引用等）
+ 双缓冲区
+ 生产者消费者模型
+ 多线程与线程安全
+ 多设计模式（单例，工厂，建造，代理者等）

## 环境搭建
本项目不依赖其它任何第三方库，准备好Linux环境和文本编辑器即可

## 日志系统再介绍

### 为什么需要日志系统
+ ⽣产环境的产品为了保证其稳定性及安全性是**不允许开发⼈员附加调试器**去排查问题， 可以借助⽇志系统来打印⼀些⽇志帮助开发⼈员解决问题
+ 上线客⼾端的产品**出现bug⽆法复现**并解决， 可以借助⽇志系统打印⽇志并上传到服务端帮助开发⼈员进⾏分析
+ 对于⼀些⾼频操作（如定时器、⼼跳包）在少量调试次数下可能⽆法触发我们想要的⾏为，通过断点的暂停**⽅式，我们不得不重复操作⼏⼗次、上百次甚⾄更多，导致排查问题效率是⾮常低下， 可以借助打印⽇志的⽅式查问题
+ 在**分布式、多线程/多进程代码**中， 出现bug⽐较**难以定位**， 可以借助⽇志系统打印log**帮助定位**bug
+ 帮助 **⾸次接触**项⽬代码的新开发⼈员理解代码的运⾏流程

### 日志系统的技术实现
日志系统的技术实现主要报错三种类型：
+ 1. `控制台输出`:利用printf,std::cout等输出函数将日志信息打印到控制台
+ `文件输出`:对于⼤型商业化项⽬， 为了⽅便排查问题，我们⼀般会将⽇志输出到⽂件或者是数据库系统⽅便查询和分析⽇志， 主要分为同步⽇志和异步⽇志⽅式
   2. 同步写日志
   3. 异步写日志

#### 同步写日志
同步⽇志是指当输出⽇志时，必须等待⽇志输出语句执⾏完毕后，才能执⾏后⾯的业务逻辑语句，⽇志输出语句与程序的业务逻辑语句将在同⼀个线程运⾏。每次调⽤⼀次打印⽇API就对应⼀次系统调⽤write写⽇志⽂件。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409261006084.png)

在⾼并发场景下，随着⽇志数量不断增加，同步⽇志系统容易产⽣系统瓶颈：
+ ⼀⽅⾯，⼤量的⽇志打印陷⼊等量的write系统调⽤，有⼀定系统开销
+ 另⼀⽅⾯，使得打印⽇志的进程附带了⼤量同步的磁盘IO，影响程序性能

#### 异步写日志
异步⽇志是指在进⾏⽇志输出时，⽇志输出语句与业务逻辑语句并不是在同⼀个线程中运⾏，⽽是**有专⻔的线程**⽤于进⾏⽇志输出操作。业务线程只需要将⽇志放到⼀个**内存缓冲区**中不⽤等待即可继续执⾏后续业务逻辑（作为⽇志的⽣产者），⽽⽇志的落地操作交给单独的⽇志线程去完成（作为⽇志的消费者）,这是一个典型的`生产者-消费者模型`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409261023775.png)

这样做的好处是即使⽇志没有真的地完成输出也不会影响程序的主业务，可以提⾼程序的性能：
+ 主线程调⽤⽇志打印接⼝成为⾮阻塞操作
+ 同步的磁盘IO从主线程中剥离出来交给单独的线程完成

# 日志系统框架设计
本项⽬实现的是⼀个**多⽇志器⽇志系统**，主要实现的功能是让程序员能够轻松的将程序运⾏⽇志信息落地到指定的位置，且⽀持同步与异步**两种⽅式**的⽇志落地⽅式。

项⽬的框架设计将项⽬分为以下⼏个模块来实现。

## 模块划分

+ `⽇志等级模块`：对输出⽇志的等级进⾏划分，以便于控制⽇志的输出，并提供等级枚举转字符串功能。
  + `OFF`：关闭
  + `DEBUG`：调试，调试时的关键信息输出。
  + `INFO`：提⽰，普通的提⽰型⽇志信息。
  + `WARN`：警告，不影响运⾏，但是需要注意⼀下的⽇志。
  + `ERROR`：错误，程序运⾏出现错误的⽇志
  + `FATAL`：致命，⼀般是代码异常导致程序⽆法继续推进运⾏的⽇志
+ `⽇志消息模块`：中间存储⽇志输出所需的各项要素信息
  + `时间`：描述本条⽇志的输出时间。
  + `线程ID`：描述本条⽇志是哪个线程输出的。
  + `⽇志等级`：描述本条⽇志的等级。
  + `⽇志数据`：本条⽇志的有效载荷数据。
  + `⽇志⽂件名`：描述本条⽇志在哪个源码⽂件中输出的。
  + `⽇志⾏号`：描述本条⽇志在源码⽂件的哪⼀⾏输出的。
+ `⽇志消息格式化模块`：设置⽇志输出格式，并提供对⽇志消息进⾏格式化功能。
  + `系统的默认⽇志输出格式`： `%d｛%H:%M:%S｝%T[%t]%T[%p]%T[%c]%T%f:%l%T%m%n`
  + `%d｛%H:%M:%S｝`：表⽰⽇期时间，花括号中的内容表⽰⽇期时间的格式。（**注：这里用本应用英文花括号，但是由于网站限制，暂时改成了中文花括号,上，下同，但多行代码框里是对的**）
  + `%T`：表⽰制表符缩进。
  + `%t`：表⽰线程ID
  + `%p`：表⽰⽇志级别
  + `%c`：表⽰⽇志器名称，不同的开发组可以创建⾃⼰的⽇志器进⾏⽇志输出，⼩组之间互不影响。
  + `%f`：表⽰⽇志输出时的源代码⽂件名。
  + `%l`：表⽰⽇志输出时的源代码⾏号。
  + `%m`：表⽰给与的⽇志有效载荷数据
  + `%n`：表⽰换⾏
  + 设计思想：设计不同的⼦类，不同的⼦类从⽇志消息中取出不同的数据进⾏处理。
+ `⽇志消息落地模块`：决定了⽇志的落地⽅向，可以是标准输出，也可以是⽇志⽂件，也可以滚动⽂件输出
  + `标准输出`：表⽰将⽇志进⾏标准输出的打印
  + `⽇志⽂件输出`：表⽰将⽇志写⼊指定的⽂件末尾。
  + `滚动⽂件输出`：当前以⽂件⼤⼩进⾏控制，当⼀个⽇志⽂件⼤⼩达到指定⼤⼩，则切换下⼀个⽂件进⾏输出
  + 后期，也可以扩展远程⽇志输出，创建客⼾端，将⽇志消息发送给远程的⽇志分析服务器。
  + `设计思想`：设计不同的⼦类，不同的⼦类控制不同的⽇志落地⽅向。
+ `⽇志器模块`:
  + 此模块是对以上⼏个模块的整合模块，⽤⼾通过⽇志器进⾏⽇志的输出，有效降低⽤⼾的使⽤难度
  + 包含有：⽇志消息落地模块对象，⽇志消息格式化模块对象，⽇志输出等级
+ `⽇志器管理模块`:
  + `解耦合`：为了降低项⽬开发的⽇志耦合，不同的项⽬组可以有⾃⼰的⽇志器来控制输出格式以及落地⽅向，因此本项⽬是⼀个多⽇志器的⽇志系统。
  + 管理模块就是对创建的所有⽇志器进⾏统⼀管理。并提供⼀个默认⽇志器提供标准输出的⽇志输出。
+ `异步线程模块`:
  + 实现对⽇志的异步输出功能，⽤⼾只需要将输出⽇志任务放⼊任务池，异步线程负责⽇志的落地输出功能，以此提供更加⾼效的⾮阻塞⽇志输出。

---

# 代码设计

## 作用域的设计
为了防止命名冲突，以及提示整个项目的完整性和功能性等，将所有的相关代码封装在`suplog`命名空间中，其中`log`表示功能，`sup`为作者自定义的名称，以提高其唯一性

然后将内部的代码块按功能/所属的业务组封装在子级命名空间中，具体示例见后面的代码封装

## 实用类设计

提前完成一些零碎的功能接口，以便于项目会用到。而由于实现以下功能并不需要成员变量，所以将成员函数设置成**静态的成员函数**,即知识封装在类域中

+ `date`类域
  + 获取系统时间
+ `file`类域
  + 判断文件是否存在
  + 获取文件的所在目录路径
  + 创建目录

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409271457541.png)

> util.hpp

```C++
#pragma once

#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <ctime>
#include <cassert>
#include <sys/stat.h>

namespace suplog{
    namespace util{
        class date
        {
        public:
            static size_t now(){return (size_t)time(nullptr);}
        };

        class file{
        public:
            static bool exists(const std::string&name)
            {
                //struct stat配合stat函数作为输出型参数
                struct stat st;
                //stat函数是系统调用接口
                return stat(name.c_str(),&st) == 0;
            }
            static std::string path(const std::string &name)
            {
                //规定找不到就返回".""
                if(name.empty()) return ".";

                size_t pos = name.find_last_of("/\\");//找到最后一个"/"或"\\"
                return name.substr(0,pos+1);//取得目录的路径
            }
            static void create_directory(const std::string &path)
            {
                if(path.empty()) return;//路径为空
                if(exists(path)) return;//路径不存在-使用系统调用接口
                size_t pos,index = 0;
                while(index<path.size())
                {
                    //找到index及以后的"/"
                    pos = path.find_first_of("/\\",index);
                    if(pos == std::string::npos)
                    {
                        //创建最后一层目录
                        mkdir(path.c_str(),0755);
                        return;
                    }
                    //二者重叠时，跳到下一次循环
                    if(pos ==index) {index = pos+1;continue;}
                    std::string subdir = path.substr(0,pos);
                    //文件夹已存在，不用创建，跳到下一段即可
                    if(subdir == "." || subdir == "..")
                        {index = pos + 1;continue;}
                    if(exists(subdir))//理由同上
                        {index = pos + 1;continue;}
                    //逐级创建目录，权限为0755
                    mkdir(subdir.c_str(),0755);
                    index = pos + 1;
                }
            }
        };
    }
}
```

### 代码测试
写一部分测一部分才是开发的好习惯，以免以后面对满屏报错，找不到bug点

写一段测试代码

> main.cpp
```C++
#include "util.hpp"
#include <iostream>

using namespace std;

int main()
{
    cout<<"测试时间: "<<suplog::util::date::now()<<endl;
    cout<<"测试文件"<<endl;
    cout<<"测试path()，获取到的path为： " << suplog::util::file::path("/home/supdriver/code/file.txt");
    cout<<"测试创建目录:... ";
    suplog::util::file::create_directory("./testdir/dir1/dir2");
    cout<<"创建成功"<<endl;
    cout<<"测试exists(), 测试路径:\"./testdir/dir1/dir2\",返回结果"<<suplog::util::file::exists("./testdir/dir1/dir2")<<endl;
    return 0;
}
```

测试结果:
```shell
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ make clean
rm -rf mycmd testdir
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ make
g++ -o mycmd main.cpp -std=c++11
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ ./mycmd 
测试时间: 1727424996
测试文件
测试path()，获取到的path为： /home/supdriver/code/file.txt测试创建目录:... 创建成功
测试exists(), 测试路径:"./testdir/dir1/dir2",返回结果1
```
可以看到，各个接口工作正常

## 日志等级类设计
日志ID鞥机总共分为7个等级，分别为：

+ `OFF`：关闭所有日志输出
+ `DEBUG`：调试，调试时的关键信息输出。
+ `INFO`：提⽰，普通的提⽰型⽇志信息。
+ `WARN`：警告，不影响运⾏，但是需要注意⼀下的⽇志。
+ `ERROR`：错误，程序运⾏出现错误的⽇志
+ `FATAL`：致命，⼀般是代码异常导致程序⽆法继续推进运⾏的⽇志

其中用到了`TOSTRING`宏函数，利用宏的预编译特性 ，**减少了函数调用**，提高了程序性能

> level.hpp
```C++
#pragma once

namespace suplog{
    class LogLevel
    {
    public:
        enum class Level{
            DEBUG = 1,
            INFO,
            WARN,
            ERROR,
            FATAL,
            OFF
        };

        static const char* toString(LogLevel::Level lv)
        {
            switch(lv)
            {
                //在内部定义一个宏函数,把变量名name变成字符串
                #define TOSTRING(name) #name
                case LogLevel::Level::DEBUG: return TOSTRING(DEBUG);//直接返回，不加用break
                case LogLevel::Level::INFO: return TOSTRING(INFO);
                case LogLevel::Level::WARN: return TOSTRING(WARN);
                case LogLevel::Level::ERROR: return TOSTRING(ERROR);
                case LogLevel::Level::FATAL: return TOSTRING(FATAL);
                case LogLevel::Level::OFF: return TOSTRING(OFF);
                #undef TOSTRING //取消宏函数
                default: return "UNKOWN";

            }

            return "UNKNOWN";//防止编译报错
        }

    };
}
```

### 代码测试

```C++
#include "util.hpp"
#include "level.hpp"
#include <iostream>

using namespace std;

int main()
{
    cout<<"测试字符串转换 FATAL:";
    cout<<suplog::LogLevel::toString(suplog::LogLevel::Level::FATAL)<<endl;
    return 0;
}
```

```shell
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ make clean
rm -rf mycmd testdir
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ make 
g++ -o mycmd main.cpp -std=c++11
.supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ ./mycmd 
测试字符串转换 FATAL:FATAL
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ 
```

可以看到成功转换,该模块正常工作

## 日志消息类设计
⽇志消息类主要是封装⼀条完整的⽇志消息所需的内容，其中包括⽇志等级、对应的`logger name`、打印⽇志源⽂件的位置信息（包括⽂件名和⾏号）、线程ID、时间戳信息、具体的⽇志信息等内容。

> message.hpp
```C++
#pragma once

#include "util.hpp"
#include "level.hpp"
#include <thread>//c++多线程库
#include <memory>
#include <string>

namespace suplog{
    struct LogMsg
    {
        //声明一个全局且唯一的指针
        using ptr = std::shared_ptr<LogMsg>;
        std::string _name;  //日志器名称
        std::string _file;  //文件名
        size_t _line;       //行号
        std::string _payload;//日志消息
        size_t _ctime;      //时间-时间戳
        std::thread::id _tid;//线程id
        LogLevel::Level _level;//日志等级
        //默认构造函数
        LogMsg(){}
        //构造函数
        LogMsg(
            std::string name,
            std::string file,
            size_t line,
            std::string payload,
            LogLevel::Level level
        ):
        _name(name),
        _file(file),
        _payload(payload),
        _level(level),
        _line(line),
        _ctime(util::date::now()),
        _tid(std::this_thread::get_id()){}
    };
}
```

这段代码就不作单独测试了

## 日志输出格式化类设计
⽇志格式化（Formatter）类主要负责格式化⽇志消息。其主要包含以下内容

1. `pattern`成员：保存⽇志输出的格式字符串。
  + `%d` 日期
  + `%T` 缩进
  + `%t` 线程id
  + `%p` 日志级别
  + `%c` 日志器名称'
  + `%f` 文件名
  + `%l` 行号
  + `%m` 日志消息
  + `%n` 换行
2. `std::vector<FormatItem::ptr> items`成员：⽤于按序保存格式化字符串对应的⼦格式化对象。

其中`FormatItem`类主要负责⽇志消息⼦项的获取及格式化。其包含以下`⼦类`

+ `MsgFormatItem` ：表⽰要从`LogMsg`中取出有效⽇志数据
+ `LevelFormatItem` ：表⽰要从`LogMsg`中取出⽇志等级
+ `NameFormatItem` ：表⽰要从`LogMsg`中取出⽇志器名称
+ `ThreadFormatItem` ：表⽰要从`LogMsg`中取出线程ID
+ `TimeFormatItem` ：表⽰要从`LogMsg`中取出时间戳并按照指定格式进⾏格式化
+ `CFileFormatItem` ：表⽰要从`LogMsg`中取出源码所在⽂件名
+ `CLineFormatItem` ：表⽰要从`LogMsg`中取出源码所在⾏号
+ `TabFormatItem` ：表⽰⼀个制表符缩进
+ `NLineFormatItem` ：表⽰⼀个换⾏
+ `OtherFormatItem` ：表⽰⾮格式化的原始字符串

**示例**:将格式化字符串`"[%d｛%H:%M:%S｝] %m%n"`实例化为日志消息字符串

```C++
pattern = "[%d{%H:%M:%S}] %m%n";
items = {
  {OtherFormatItem(), "["},
  {TimeFormatItem(), "%H:%M:%S"},
  {OtherFormatItem(), "]"},
  {MsgFormatItem(), ""},
  {NLineFormatItem(), ""}
};

LogMsg msg = {
  size_t _line = 22;
  size_t _ctime = 12345678;
  std::thread::id _tid = 0x12345678;
  std::string _name = "logger";
  std::string _file = "main.cpp";
  std::string _payload = "创建套接字失败";
  LogLevel::value _level = ERROR;
};

```
最后组织出来的格式化消息为：`"[22:32:54] 创建套接字失败\n"`

> formatter.hpp
> 实现FormatItem及其派生类
```C++
namespace suplog{

    /*=========FormatItem及其派生类===========*/
    class FormatItem
    {
    public:
        using ptr = std::shared_ptr<FormatItem>;
        virtual ~FormatItem(){}
        virtual void format(std::ostream& os,const LogMsg& msg) = 0;
    };

    class MsgFormatItem:public FormatItem
    {
    public:
        MsgFormatItem(const std::string& str = ""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<msg._payload;
        }
    };

    class LevelFormatItem:public FormatItem{
    public:
        LevelFormatItem(const std::string& str = ""){};
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<LogLevel::toString(msg._level);
        }
    };

    class NameFormatItem:public FormatItem{
    public:
        NameFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<msg._name;
        }
    };

    class ThreadFormatItem:public FormatItem{
    public:
        ThreadFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<msg._tid;
        }
    };

    class TimeFormatItem:public FormatItem{
    private:
        std::string _format;
    public:
        TimeFormatItem(const std::string& format="%H:%M:%S"):_format(format){
            if(format.empty()) _format = {"%H:%M:%S"};
        }
        virtual void format(std::ostream& os,const LogMsg& msg){
            time_t t = msg._ctime;
            struct tm lt;
            localtime_r(&t,&lt);//从时间戳t中提取时间信息到结构体lt中
            char tmp[128];
            strftime(tmp,127,_format.c_str(),&lt);//格式化日期信息到字符串
            os<<tmp;
        }
    };

    class CFileFormatItem:public FormatItem{
    public:
        CFileFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<msg._file;
        }
    };

    class CLineFormatItem:public FormatItem{
    public:
        CLineFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<msg._line;
        }
    };

    class TabFormatItem:public FormatItem{
    public:
        TabFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<'\t';
        }
    };        

    class NLineFormatItem:public FormatItem{
    public:
        NLineFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<'\n';
        }
    };

    class OtherFormatItem:public FormatItem{
    private:
        std::string _str;
    public:
        OtherFormatItem(const std::string &str=""):_str(str){}
        virtual void format(std::ostream& os,const LogMsg& msg){
            os<<_str;
        }
    };

    /*===============END==============*/

}
```
可以看到`FormatItem`的派生类非常多，我们用两行注释标注一下这一大块代码的作用和边界，这样有利于后期查阅和维护

同时很明显这里用到了`虚函数`和`继承抽象类`来实现`多态`，保证了后期能用统一的`shared_ptr<FormatItem>`父类指针统一管理所有派生类

> formatter.hpp
> 实现Formatter类
> Formatter类主要用于实现解析 pattern字符串和将LogMsg转化为对应的日志消息
> 又由于有众多种类的`FormatItem`派生类要创建，所以我们还要将创建功能单独封装，因此确定有如下接口
> + `pattern`:返回`const string`,查看对象内储存的`pattern`字符串
> + `create_item`:在堆上创建`Formatitem`派生类并返回子类指针
> + `parsePattern`:**解析格式串**并生成`Formatitem`派生类列表
>
> *对字符串解析的示意图*
> ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409291637114.png)

关于解析格式串,这里使用**缓存的思想尤为重要**

+ `string_row`是对原始字符进行缓存，一次性为多个字符创建一个`MsgFormatItem`,减少内存开支和性能消耗
+ `format_key`,`format_val`是对格式字符和子格式串的缓存
+ `arry`储存分析后的原素材,每一个元素对应一个**待创建**的`FormatItem`派生类

正是有缓存的存在，我们可以将字符串分析，创建`FormatItem`派生类等复杂工作分开处理。当然相应地**维护成本会增加**：**缓存最后都是要在合适的时机清空的**,否则会丢失信息，让工作不完整。

```C++
namespace suplog{
    //......上面的代码略...
    /*===============END==============*/

      class Formatter
    {
    public:
        using ptr = std::shared_ptr<Formatter>;
        /*
            %d ⽇期
            %T 缩进
            %t 线程id
            %p ⽇志级别
            %c ⽇志器名称
            %f ⽂件名
            %l ⾏号
            %m ⽇志消息
            %n 换⾏
            */

        Formatter(const std::string& pattern = "[%d{%H:%M:%S}][%t][%p][%c][%f:%l] %m%n")
        :_pattern(pattern)
        {
            assert(parsePattern());//调用成员函数，后面声明
        }

        //简单的查看pattern的接口
        const std::string pattern() {return _pattern;}
        //最后实现，但是是最常用的接口，故写在前面
        std::string format(const LogMsg& msg)
        {
            std::stringstream ss;
            for(auto& it: _items)
            {
                it->format(ss,msg);
            }
            return ss.str();
        }

        //创建FormatItem(给调用者使用多态)
        FormatItem::ptr createItem(const std::string& fc,const std::string& subfmt)
        {
            if(fc == "m") return FormatItem::ptr(new MsgFormatItem(subfmt));
            if(fc == "p") return FormatItem::ptr(new LevelFormatItem(subfmt));
            if(fc == "c") return FormatItem::ptr(new NameFormatItem(subfmt));
            if(fc == "t") return FormatItem::ptr(new ThreadFormatItem(subfmt));
            if(fc == "n") return FormatItem::ptr(new NLineFormatItem(subfmt));
            if(fc == "d") return FormatItem::ptr(new TimeFormatItem(subfmt));
            if(fc == "f") return FormatItem::ptr(new CFileFormatItem(subfmt));
            if(fc == "l") return FormatItem::ptr(new CLineFormatItem(subfmt));
            if(fc == "T") return FormatItem::ptr(new TabFormatItem(subfmt));
            return FormatItem::ptr();//未知fc
        }
        //pattern解析

        //每个要素分为三部分：
        // 格式化字符 : %d %T %p...
        // 对应的输出⼦格式 ： {%H:%M:%S}
        // 对应数据的类型 ： 0-表⽰原始字符串，也就是⾮格式化字符，1-表⽰格式化数据类型

        //默认格式"[%d{%H:%M:%S}][%t][%p][%c][%f:%l] %m%n"
        bool parsePattern()
        {
            //储存分割好的字符串,是否为原始字符串由int指示
            std::vector<std::tuple<std::string,std::string,int>> arry;
            std::string format_key;//存放%之后的格式化字符
            std::string format_val;//存放格式化字符串后边{}中的子格式化字符串
            std::string string_row;//存放原始的非格式化字符串
            bool sub_format_error = false;
            int pos = 0;
            while(pos < _pattern.size())
            {
                if(_pattern[pos] != '%')
                {
                    string_row.push_back(_pattern[pos++]);
                    continue;
                }
                if(pos+1<_pattern.size() && _pattern[pos+1] == '%')
                {
                    //规定%%格式化为原生的'%'字符
                    string_row.push_back('%');
                    pos+=2;
                    continue;
                }
                if(string_row.empty() == false)//原生字符串不为空
                {
                    arry.push_back(std::make_tuple(string_row,"",0));
                    string_row.clear();
                }

                //当前位置指向%字符
                pos++;//pos指向格式化字符位置
                if(pos<_pattern.size() && isalpha(_pattern[pos]))//判断是不是字符
                {
                    format_key = _pattern[pos];
                }
                else 
                {
                    std::cout<< &_pattern[pos-1]<<"位置附近格式错误！\n";
                    return false;
                }
                //下一步使pos指向格式化字符的下个位置，判断是否包含有子格式,例如在%d{%Y-%m-%d}中
                pos++;
                if(pos+1 < _pattern.size() && _pattern[pos] == '{')
                {
                    sub_format_error = true;
                    pos++;//pos指向花括号下一个字符
                    while (pos<_pattern.size())
                    {
                        if(_pattern[pos] == '}')
                        {
                            //循环出口
                            sub_format_error = false;
                            pos++;
                            break;
                        }
                        //在循环中时
                        format_val.push_back(_pattern[pos++]);
                    }
                    if(sub_format_error)
                    {
                        std::cout<<"{}对应出错\n";
                        return false;
                    }
                }
                arry.push_back(std::make_tuple(format_key,format_val,1));
                format_key.clear();
                format_val.clear();
            }//结束循环

            if(string_row.empty() == false)//原生字符串不为空
                arry.push_back(std::make_tuple(string_row,"",0));
            if(format_key.empty() == false)//格式化字符不为空，注，上下顺序不能换,要和循环内一致
                arry.push_back(std::make_tuple(format_key,format_val,1));

            //开始拼接item列表
            if(_items.empty() == false)_items.clear();//清理_items
            for(auto& it:arry)
            {
                if(std::get<2>(it) == 0)//获取第三个元素
                {
                    FormatItem::ptr fi(new OtherFormatItem(std::get<0>(it)));
                    _items.push_back(fi);
                }
                else 
                {
                    FormatItem::ptr fi = createItem(std::get<0>(it),
                                                    std::get<1>(it));
                    if(fi.get() == nullptr)
                    {
                        std::cout<<"没有对应的格式化字符串： %"
                                 <<std::get<0>(it)
                                 <<std::endl;
                        return false;
                    }
                    _items.push_back(fi);
                }
            }//完成拼接
            return true;
        }

    private:
        std::string _pattern;
        std::vector<FormatItem::ptr> _items;
    };

}
```

接下来我们写一下测试代码，把之前举的例子部分模拟出来

```C++
#include "util.hpp"
#include "level.hpp"
#include "message.hpp"
#include "formatter.hpp"
#include <iostream>

using namespace std;

int main()
{
    suplog::Formatter fmt;
    std::string pattern = "[%d{%H:%M:%S}] %m%n";
    suplog::LogMsg msg={
        "logger",       //名字
        "main.cpp",     //文件名
        22,             //行数
        "创建套接字失败",//正文
        suplog::LogLevel::Level::ERROR   //等级
    };
    cout<<fmt.format(msg);
    return 0;
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409291610709.png)

输出结果如上，显然成功处理出了格式化字符串并输出了预期的日志信息

## 日志落地类(LogSink)设计（简单工厂模式）
日志落地类主要负责落地日志消息到目的地。
它主要包括以下内容：
+ `Formatter`日志格式化器：主要负责格式化日志消息
+ `mutex互斥锁`:保证多线程日志落地过程中的线程安全，避免出现交叉输出的情况。这个类要设计成**支持可扩展**，所以其成员函数`log`设置为**纯虚函数**，当我们需要增加一个`log`输出目标,可以增加一个类继承自该类并重写`log`方法实现具体的落地日志逻辑。（简单工厂模式）

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409251006985.png)

+ 标准输出：`StdoutSink`
+ 固定文件: `FileSink`
+ 滚动文件: `RollSink`
  + 滚动日志文件输出的**必要性**:
    1. 由于**机器磁盘空间有限**， 我们不可能⼀直⽆限地向⼀个⽂件中增加数据
    2. 如果⼀个⽇志 **⽂件体积太⼤**，⼀⽅⾯是不好打开，另⼀⽅⾯是即时打开了由于包含数据巨⼤，也不利于查找我们需要的信息
    3. 所以**实际开发中**会对单个⽇志⽂件的⼤⼩也会做⼀些控制，即当⼤⼩超过某个⼤⼩时（如1GB），我们就重新创建⼀个新的⽇志⽂件来滚动写⽇志。 对于那些过期的⽇志， ⼤部分企业内部都有专⻔的运维⼈员去**定时清理过期的⽇志**，或者设置系统定时任务，定时清理过期⽇志。
  + 日志文件滚动的思想：⽇志⽂件滚动的条件有两个:⽂件⼤⼩ 和 时间。我们可以选择：
    + 日志文件在⼤于 1GB 的时候会更换新的⽂件
    + 每天定点滚动⼀个⽇志⽂件

本项目选择基于 文件大小 的潘墩滚动生成新的文件

> sink.hpp

```C++
namespace suplog{
    class LogSink//抽象类
    {
    public:
        using ptr = std::shared_ptr<LogSink>;
        LogSink(){}
        virtual ~LogSink() =default;
        virtual void log(const char*data,size_t len) = 0;
    };

    class StdoutSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<StdoutSink>;
        StdoutSink()=default;
        
        void log(const char*data,size_t len) override{//加override关键字明示这是虚函数重载
            std::cout.write(data,len);
        }
    };

    class FileSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<FileSink>;
        FileSink(const std::string& filename):_filename(filename){
            //创建目录
            util::file::create_directory(util::file::path(filename));
            _ofs.open(_filename,std::ios::binary|std::ios::app);//二进制方式写入
            assert(_ofs.is_open());
        }

        //提供给外界获取文件名
        const std::string &file(){return _filename;}

        void log(const char*data,size_t len) override {//加override关键字明示这是虚函数重载
            _ofs.write((const char*)data,len);
            if(_ofs.good() == false){
                std::cout<<"日志输出文件失败!\n";
            }
        }
    private:
        std::string _filename;
        std::ofstream _ofs;
    };

    class RollSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<RollSink>;
        RollSink(const std::string& basename,size_t max_size)
        :_basename(basename),_max_fsize(max_size),_cur_fsize(0)
        {
            //创建目录
            util::file::create_directory(util::file::path(_basename));
        }

        void log(const char*data,size_t len) override {//加override关键字明示这是虚函数重载
            initLogFile();
            _ofs.write(data,len);
            if(_ofs.good() == false){
                std::cout<<"日志输出文件失败!\n";
            }
            _cur_fsize += len;
        }

    private:
        //在这里实现对日志文件的管理
        void initLogFile(){
            if(_ofs.is_open() == false || _cur_fsize >=_max_fsize)
            {
                _ofs.close();
                if(_cur_fsize >=_max_fsize) sleep(1);//防止同一秒有过多日志消息要打印，导致打开同一个日志文件
                std::string name = createFilename();
                _ofs.open(name,std::ios::binary | std::ios::app);
                assert(_ofs.is_open());
                _cur_fsize = 0;
                return;
            }
            return;
        }
        //管理文件名的生成
        std::string createFilename()
        {
            time_t t = time(NULL);
            struct tm lt;
            localtime_r(&t,&lt);
            std::stringstream ss;
            //获取详细时间信息
            ss << _basename;
            ss<< lt.tm_year + 1900;
            ss <<lt.tm_mon + 1;
            ss << lt.tm_mday;
            ss << lt.tm_hour;
            ss << lt.tm_min;
            ss << lt.tm_sec;
            ss << ".log";
            return ss.str();
        }
    private:
        std::string _basename;
        std::ofstream _ofs;
        size_t _max_fsize;
        size_t _cur_fsize;
    };

        //创建简单工厂
    class SinkFactory{
    public:
        template<typename SinkType,typename ...Args>
        static LogSink::ptr create(Args&&...args){
            //返回shared_ptr,传的是构造函数参数列表
            return std::make_shared<SinkType>(std::forward<Args>(args)...);
        }
    };
}
```

### 代码测试
```C++
#include "util.hpp"
#include "level.hpp"
#include "message.hpp"
#include "formatter.hpp"
#include "sink.hpp"
#include <iostream>

using namespace std;

int main()
{
    suplog::Formatter fmt;
    std::string pattern = "[%d{%H:%M:%S}] %m%n";
    suplog::LogMsg msg={
        "logger",       //名字
        "main.cpp",     //文件名
        22,             //行数
        "创建套接字失败",//正文
        suplog::LogLevel::Level::ERROR   //等级
    };
    std::string str = fmt.format(msg);

    std::shared_ptr<suplog::LogSink> lsptr;

    lsptr.reset(new suplog::StdoutSink());
    lsptr->log(str.c_str(),str.size());

    lsptr.reset(new suplog::FileSink("./testdir/log.log"));
    lsptr->log(str.c_str(),str.size());

    lsptr.reset(new suplog::RollSink("./testdir/rollsink/log",10));//故意设置小，查看滚动效果
    string msg1=string(str).append("msg1");
    string msg2=string(str).append("msg2");

    lsptr->log(msg1.c_str(),msg1.size());
    lsptr->log(msg2.c_str(),msg2.size());

    return 0;
}
```

为了测试日志落地的功能,我们设计了一段测试代码，测试其多态性的同时，测试了三种落地方式,**特别的**,在`RollSink`中故意把`max_size`设的很小，让它能够在测试中滚动输出日志。

输出结果如下

> 标准输出落地
> ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409301424289.png)

> 文件输出落地
> ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409301426958.png)

> 滚动文件输出落地
> ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409301426958.png)

可以看到各类落地功能都执行得很好

```小插曲
其实之前在suplog::util::file::path函数中
作者不小心写错了一个函数，把name.find_last_of("/\\");中的
find_last_of写成了find_last_not_of,结果前面的测试代码
没有测试出bug,导致目录是创建出来了，但是把文件夹路径最后的
文件名也当成目录给创建出来了，可见每一步都做测试的重要性，因为在
这次的测试中，我发现了文件输出失败，原因是打不开文件。（目录当然打不开了）
经过一通排查发现问题出在了之前写的path函数里

所以一定要多做测试！
```

## 日志类(Logger)主干部分设计（建造者模式）
⽇志器主要是⽤来和**前端交互**， 当我们需要使⽤⽇志系统打印log的时候， 只需要创建`Logger`对象，调⽤该对象`debug`、`info`、`warn`、`error`、`fatal`等 **⽅法**输出⾃⼰想打印的⽇志即可，⽀持**解析可变参数列表和输出格式**， 即可以做到像使⽤printf函数⼀样打印⽇志。

当前⽇志系统计划⽀持`同步⽇志` & `异步⽇志`两种模式，两个不同的⽇志器唯⼀不同的地⽅在于他们在⽇志的落地⽅式上有所不同：

+ `同步⽇志器`：直接对⽇志消息进⾏输出。
+ `异步⽇志器`：将⽇志消息放⼊缓冲区，由**异步线程**进⾏输出。

因此⽇志器类在设计的时候先设计出⼀个`Logger基类`，在Logger基类的基础上，继承出`SyncLogger同步⽇志器`和`AsyncLogger异步⽇志器`。

且因为⽇志器模块是对前边多个模块的整合，想要创建⼀个⽇志器，需要设置⽇志器名称，设置⽇志输出等级，设置⽇志器类型，设置⽇志输出格式，设置落地⽅向，且落地⽅向有可能存在多个，整个⽇志器的**创建过程较为复杂**，为了保持良好的代码⻛格，编写出优雅的代码，因此⽇志器的创建这⾥采⽤了`建造者模式`来进⾏创建

因为同步日志器类比较简单，就直接在这里实现了。

而异步日志器虽然也在这里实现，但是由于比较复杂，在下一步实现

```C++
#include "util.hpp"
#include "level.hpp"
#include "message.hpp"
#include "formatter.hpp"
#include "sink.hpp"
#include "looper.hpp"//不认识？没关系，
                    //这个头文件后面才实现,给异步日志器的实现用的
#include <vector>
#include <list>
#include <atomic>
#include <unordered_map>
#include <cstdarg>
#include <type_traits>

namespace suplog{
    class Logger{
    public:
        enum class Type{
            LOGGER_SYNC = 0,
            LOGGER_ASYNC
        };
        using ptr = std::shared_ptr<Logger>;

        Logger(const std::string& name,
            Formatter::ptr formatter,
            std::vector<LogSink::ptr> &sinks,
            LogLevel::Level level = LogLevel::Level::DEBUG
        ):_name(name),_level(level),_formatter(formatter),
        _sinks(sinks.begin(),sinks.end())
        {}

        std::string loggerName(){return _name;}
        LogLevel::Level loggerLevel(){return _level;}

        //使用C语言风格的不定参数
        //=========start========
        void debug(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::DEBUG) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::DEBUG,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void info(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::INFO) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::INFO,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void warn(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::WARN) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::WARN,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void error(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::ERROR) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::WARN,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void fatal(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::FATAL) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::FATAL,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        //===============end===============

    public:
        //在内部声明一个建造者抽象类
        class Builder
        {
        public:
            using ptr = std::shared_ptr<Builder>;

            Builder():_level(LogLevel::Level::DEBUG),
                _logger_type(Logger::Type::LOGGER_SYNC)
            {}

            void buildLoggerName(const std::string& name){_logger_name = name;}

            void buildLoggerLevel(LogLevel::Level level){_level = level;}

            void buildLoggerType(Logger::Type type){_logger_type = type;}

            void buidFormatter(const Formatter::ptr& formatter){
                _formatter = formatter;
            }

            void buidFormatter(const std::string& formatStr){
                auto formatter = std::make_shared<suplog::Formatter>(formatStr);
                _formatter = formatter;
            }

            //C++风格不定参数
            template<typename SinkType,typename ...Args>
            void buildSink(Args &&...arfs){
                auto sink = SinkFactory::create<SinkType>(std::forward<Args>(args)...);
                _sinks.push_back(sink);
            }

            virtual Logger::ptr build() = 0;
        protected:
            Logger::Type _logger_type;
            std::string _logger_name;
            LogLevel::Level _level;
            Formatter::ptr _formatter;
            std::vector<LogSink::ptr> _sinks;
        };

    protected:
    bool shouldLog(LogLevel::Level level){return level>= _level;}

    void log(LogLevel::Level level,const char*file,
        size_t line,const char*fmt,va_list al)
    {
        char *buf;//可以不初始化
        std::string msg;
        int len = vasprintf(&buf,fmt,al);//自动在堆区申请内存
        if(len < 0)
            msg = "格式化日志消息失败!!";
        else 
        {
            msg.assign(buf,len);
            free(buf);//释放空间
        }

        //LogMsg(name, file, line, payload, level)
        LogMsg logmsg(_name,file,line,std::move(msg),level);
        std::string str;
        str = _formatter->format(logmsg);
        logIt(std::move(str));
    }
    virtual void logIt(const std::string &msg) = 0;
    protected:
        std::mutex _mutex;
        std::string _name;
        Formatter::ptr _formatter;
        std::atomic<LogLevel::Level> _level;
        std::vector<LogSink::ptr> _sinks; 
    };

    //同步日志器比较简单，所以直接实现了
    class SyncLogger:public Logger
    {
    public:
        using ptr = std::shared_ptr<SyncLogger>;

        SyncLogger(const std::string& name,
            Formatter::ptr formatter,
            std::vector<LogSink::ptr>&sinks,
            LogLevel::Level level = LogLevel::Level::DEBUG)
            :Logger(name,formatter,sinks,level){
                std::cout << LogLevel::toString(level)<<"同步日志器创建成功...\n";
            }

        private:
            virtual void logIt(const std::string& msg_str){
                std::unique_lock<std::mutex> lock(_mutex);//函数结束后自动释放锁
                if(_sinks.empty()) {return;}
                for(auto &it:_sinks)
                    it->log(msg_str.c_str(),msg_str.size());
            }
    };

    class LocalLoggerBuilder:public Logger::Builder
    {
    public:
        virtual Logger::ptr build()
        {
            if(_logger_name.empty()){
                std::cout<<"日志器名称不能为空！！";
                abort();
            }
            if(_formatter.get() == nullptr){
                std::cout<<"当前日志器： "<<_logger_name;
                std::cout<<" 未检测到⽇志格式,默认设置为: ";
                std::cout<<" %d{%H:%M:%S}%T%t%T[%p]%T[%c]%T%f:%l%T%m%n\n";
                _formatter = std::make_shared<Formatter>();
            }
            if(_sinks.empty())
            {
                std::cout<<"当前日志器: "<<_logger_name<<"问检测到落地方向，默认为标准输出!\n";
                _sinks.push_back(std::make_shared<StdoutSink>());
            }

            Logger::ptr lp;
            if(_logger_type == Logger::Type::LOGGER_ASYNC)
            {
                lp = std::make_shared<AsyncLogger>(_logger_name,_formatter,_sinks,_level);
            }
            else 
            {
                lp = std::make_shared<SyncLogger>(_logger_name,_formatter,_sinks,_level);
            }
            return;
        }
    };
}
```

## 双缓冲区异步任务处理器（AsyncLooper）设计
设计思想：异步处理线程 + 数据池

使⽤者将需要完成的任务添加到任务池中，由**异步线程来完成任务的实际执⾏操作**。

任务池的设计思想:**双缓冲区阻塞数据池**

优势：缓冲区避免了空间的频繁申请释放，且尽可能的减少了生产者与消费者之间**锁冲突**的概率,提高了任务处理的效率

> 在任务池的设计中，有很多备选⽅案，⽐如循环队列等等，但是不管是哪⼀种都会涉及到**锁冲突**的情况，因为在⽣产者与消费者模型中，任何两个⻆⾊之间都具有互斥关系，因此每⼀次的任务**添加与取出都有可能涉及锁的冲突**，⽽双缓冲区不同，双缓冲区是处理器将⼀个缓冲区中的任务全部处理完毕后，然后交换两个缓冲区，重新对新的缓冲区中的任务进⾏处理，虽然同时多线程写⼊也会冲突，但是冲突并不会像每次只处理⼀条的时候频繁（减少了⽣产者与消费者之间的锁冲突），且不涉及到空间的频繁申请释放所带来的消耗。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410041822328.gif)

> buffer.hpp

```C++
#include <iostream>
#include <string>
#include <vector>
#include <thread>
#include <mutex>
#include <atomic>
#include <condition_variable>
#include <functional>
#include <cassert>

namespace suplog
{

#define BUFFER_DEFUALT_SIZE (1*1024*1024)
#define BUFFER_INCREMENT_SIZE (1*1024*1024)
#define BUFFER_THRESHOLD_SIZE (10*1024*1024)

class Buffer{
public:
    Buffer():_reader_idx(0),_writer_idx(0),_v(BUFFER_DEFUALT_SIZE){}

    bool empty(){return _reader_idx == _writer_idx;}
    size_t readAbleSize(){return _writer_idx - _reader_idx;}
    size_t writeAbleSize(){return _v.size() - _writer_idx;}
    void reset(){_reader_idx = _writer_idx = 0;}
    void swap(Buffer& buf)
    {
        _v.swap(buf._v);
        std::swap(_reader_idx,buf._reader_idx);
        std::swap(_writer_idx,buf._writer_idx);
    }

    void push(const char*data,size_t len)
    {
        ensureEnoughSpace(len);
        assert(len<=writeAbleSize());
        std::copy(data,data+len,&_v[_writer_idx]);
        _writer_idx+=len;
    }

    void pop(size_t len)
    {
        _reader_idx +=len;
        assert(_reader_idx <=_writer_idx);
    }

    const char* begin() {return &_v[_reader_idx];}

protected:
    void ensureEnoughSpace(size_t len)
    {
        if(len <=writeAbleSize())return;

        size_t new_capacity;
        //每次增加1M的大小
        if(_v.size() < BUFFER_THRESHOLD_SIZE)
            new_capacity = _v.size()*2+len;
        else 
            new_capacity = _v.size() + BUFFER_INCREMENT_SIZE + len;

        _v.resize(new_capacity);
    }

private:
    size_t _reader_idx;
    size_t _writer_idx;
    std::vector<char> _v;
};

}
```
> looper.hpp

```C++
#include "util.hpp"
#include "buffer.hpp"
#include <vector>
#include <thread>
#include <mutex>
#include <atomic>
#include <condition_variable>
#include <functional>

namespace suplog
{
    class AsyncLooper
    {
    public:
        using ptr = std::shared_ptr<AsyncLooper>;
        using Functor = std::function<void(Buffer &buffer)>;

        AsyncLooper(const Functor&cb)
        : _running(true),
        _looper_callback(cb),
        _thread(std::thread(&AsyncLooper::worker_loop,this))
        {}

        ~AsyncLooper(){stop();}

        void stop()
        {
            _running =false;
            _pop_cond.notify_all();//取消所有线程的条件变量等待
            _thread.join();
        }

        void push(const std::string&msg)
        {
            if(_running == false) return;

            std::unique_lock<std::mutex> lock(_mutex);
            _push_cond.wait(lock,[&]{
                return _tasks_push.writeAbleSize() >= msg.size();
                });//防止消息过大

            _tasks_push.push(msg.c_str(),msg.size());//完成消息任务推送

            _pop_cond.notify_all();//唤醒所有消费者
        }

    private:
        static void worker_loop(void* arg)
        {
            //多线程执行函数得靠arg传递this指针
            AsyncLooper* al = (AsyncLooper* )arg;
            while(1)
            {
                 // lock 的析构函数在离开作用域时自动释放互斥锁
                std::unique_lock<std::mutex> lock(al->_mutex);
                //线程出口,为空且关闭时退出循环
                if(al->_running == false && al->_tasks_push.empty())
                    return;
                
                //生产者缓冲不为空或者停止运行时才会被唤醒
                al->_pop_cond.wait(lock,[&]{return !al->_tasks_push.empty() || !al->_running;});
                al->_tasks_push.swap(al->_tasks_pop);//交换缓冲区

                al->_push_cond.notify_all();//唤醒所有生产者缓冲区
                al->_looper_callback(al->_tasks_pop);//输出消费者缓冲区
                al->_tasks_pop.reset();
            }
        }
    
    private:
        Functor _looper_callback;
    private:
        std::mutex _mutex;
        std::atomic<bool> _running;
        std::condition_variable _push_cond;
        std::condition_variable _pop_cond;
        Buffer _tasks_push;
        Buffer _tasks_pop;
        std::thread _thread;
    };
}
```

## 异步日志器(AsyncLogger)设计
在实现了`AsyncLooper`后，异步日志器的实现就很简单了，具体的`生产者-消费者`交互工作都交给了`AsyncLooper`，异步日志器主要负责推送日志任务，以及获取日志任务，转发给落地类(`LogSink`)去完成

+ `LogIt`为重写父类函数，专门现将⽇志数据加⼊异步队列缓冲区中
+ `readLog`函数在**异步线程**上执行调用,完成日志的实际落地工作

> logger.hpp

```C++
//class SyncLogger:public Logger
//上面已经实现过了，代码略......

    class AsyncLogger:public Logger
    {
    public:
        using ptr = std::shared_ptr<AsyncLogger>;

        AsyncLogger(const std::string& name,
            Formatter::ptr formatter,
            std::vector<LogSink::ptr>&sinks,
            LogLevel::Level level = LogLevel::Level::DEBUG)
            :Logger(name,formatter,sinks,level)
            ,_looper(std::make_shared<AsyncLooper>(
                std::bind(&AsyncLogger::readLog,this,std::placeholders::_1)))
                //传一个this,使包装器里的函数能够是成员函数,this后面的才是包装器指定的参数
            {
                std::cout << LogLevel::toString(level)<<"异步日志器创建成功...\n";
            }

    private:
        virtual void logIt(const std::string &msg)
        {
            _looper->push(msg);//推送消息
        }

        void readLog(Buffer& msg)
        {
            if(_sinks.empty()){return;}//判空

            for(auto &it:_sinks)
            {
                //调用落地功能
                it->log(msg.begin(),msg.readAbleSize());
            }
        }
    protected:
        AsyncLooper::ptr _looper;
    
    };

//  class LocalLoggerBuilder:public Logger::Builder
//  实现过了，代码略......
```

## 单例日志器管理类设计（单例模式）

日志的输出，我们希望能够在**任意位置都可以进行**，但是当我们创建了一个日志器后，就会受到日志器所在**作用域**的访问属性限制。

因此，为了突破访问区域的限制，我们再封装一个日志器管理类，并采用`单例模式`，安全地在全局的任意位置调用指定的日志器

`全局日志器建造者类`：既然日志器管理类是在全局上工作的，我们决定再封装一个配套的**全局日志器建造者类**,**自动**将新增的日志器添加到单例日志类管理器中。

对特定日志器的访问:这里采用通过名字索引的方式获取指定日志器

> logger.hpp

```C++

//接在LocalLoggerBuilder后面
    class LoggerManager
    {
    public:
        static LoggerManager& getInstance()
        {
            static LoggerManager lm;//饿汉模式创建全局对象
            return lm;
        }

        bool hasLogger(const std::string& name)
        {
            std::unique_lock<std::mutex> lock(_mutex);
            auto it = _loggers.find(name);
            if(it == _loggers.end())
                return false;
            return true;
        }

        void addLogger(const std::string& name,
            const Logger::ptr logger)
        {
            std::unique_lock<std::mutex> lock(_mutex);
            _loggers.insert(std::make_pair(name,logger));
        }

        Logger::ptr getLogger(const std::string &name)
        {
            std::unique_lock<std::mutex> lock(_mutex);
            if(hasLogger(name))
            {
                auto it = _loggers.find(name);
                if(it!= _loggers.end())
                    return it->second;//找到了，返回指针
            }
            //找不到
            return Logger::ptr();
        }

        Logger::ptr rootLogger()
        {
            std::unique_lock<std::mutex> lock(_mutex);
            return _root_logger;
        }

    private:
        LoggerManager()//私有化防止外部调用，破坏单例模式
        {
            //调用本地建造者
            std::unique_ptr<LocalLoggerBuilder> slb(new LocalLoggerBuilder());
            slb->buildLoggerName("root");
            slb->buildLoggerType(Logger::Type::LOGGER_SYNC);
            //格式使用默认
            //采用默认落地方向
            _root_logger = slb->build();
            _loggers.insert(std::make_pair("root",_root_logger));
        }

        LoggerManager(const LoggerManager&) = delete;//删除拷贝构造
        LoggerManager& operator=(const LoggerManager&) = delete;//删除重载
    private:
        std::mutex _mutex;//互斥量
        Logger::ptr _root_logger;//至少有一个日志器
        std::unordered_map<std::string,Logger::ptr> _loggers;
        //使用父类指针统一管理，使用多态
    };

    class GlobalLoggerBuilder:public Logger::Builder
    {
    public:
        virtual Logger::ptr build() override
        {
            if(_logger_name.empty())
            {
                std::cout<<"日志器名称不能为空!!";
                abort();
            }

            //不能有重名的日志器
            assert(LoggerManager::getInstance().hasLogger(_logger_name) == false);

            if(_formatter.get() == nullptr){
                std::cout<<"当前日志器： "<<_logger_name;
                std::cout<<" 未检测到⽇志格式,默认设置为: ";
                std::cout<<" %d{%H:%M:%S}%T%t%T[%p]%T[%c]%T%f:%l%T%m%n\n";
                _formatter = std::make_shared<Formatter>();
            }
            if(_sinks.empty())
            {
                std::cout<<"当前日志器: "<<_logger_name<<"问检测到落地方向，默认为标准输出!\n";
                _sinks.push_back(std::make_shared<StdoutSink>());
            }

            Logger::ptr lp;
            if(_logger_type == Logger::Type::LOGGER_SYNC)
            {
                lp = std::make_shared<SyncLogger>(_logger_name,_formatter,_sinks,_level);
            }
            else 
            {
                lp = std::make_shared<AsyncLogger>(_logger_name,_formatter,_sinks,_level);
            }

            //加入全局日志器管理类中
            LoggerManager::getInstance().addLogger(_logger_name,lp);

            return lp;
        }
    };
```

### 测试代码
终于是完成了日志系统的大部分代码，由于前几个模块关联性过高，所以测试代码憋到现在来写，没关系，这就开测！

测试目标

+ 测试`GlobalLoggerBuilder`能否正常创建实例
+ 测试两种日志器能否正常工作

```C++
#include "logger.hpp"
#include <iostream>

using namespace std;

int main()
{
    std::string path = "./test/rollsink";
    suplog::LoggerManager& lm = suplog::LoggerManager::getInstance();

    suplog::GlobalLoggerBuilder glb;
    glb.buildLoggerName("test");
    glb.buildSink<suplog::RollSink>("./test/rollsink",10);
    glb.buildLoggerType(suplog::Logger::Type::LOGGER_ASYNC);

    suplog::Logger::ptr alogger= glb.build();
    suplog::Logger::ptr slogger = lm.rootLogger();

    alogger->warn("main.cpp",19,"这是一条警告测试信息，测试码:%d",0);
    alogger->warn("main.cpp",20,"这是一条警告测试信息，测试码:%d",1);

    slogger->debug("main.cpp",22,"这是一条标准输出测试信息");

    return 0;
}
```
结果如下

*标准输出*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410042158759.png)

*滚动文件输出*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410042201373.png)

可以看到，正常工作

## 日志宏&全局接口设计(代理模式)
目前`logger.h`暴露给用户的接口还是太多了，所以新增一个代理来简化和保护.提供全局的日志器获取接口

实现代理模式时，通过`全局函数`或`宏函数`来代理`Logger`类的`log`,`debug`,`info`,`warn`,`error`,`fatal`等接口,以便于控制源码文件名称和行号的输出控制，简化用户操作

设计上，**标准输出日志的功能由主日志器承担**。当仅需要标准输出日志的时候可以通过当日志器来打印日志。且操作时只需要通过宏函数直接进行输出即可

> suplog.hpp

```C++
#pragma once

#include "logger.hpp"

namespace suplog
{
    Logger::ptr getLogger(const std::string& name)
    {
        return LoggerManager::getInstance().getLogger(name);
    }

    Logger::ptr rootLogger()
    {
        return LoggerManager::getInstance().rootLogger();
    }

    #define WOW 6666

    //开始定义宏函数
    #define debug(fmt,...)  debug(__FILE__,__LINE__,fmt,##__VA_ARGS__)//利用宏，自动传入文件路径和行号
    #define info(fmt,...)  info(__FILE__,__LINE__,fmt,##__VA_ARGS__)
    #define warn(fmt,...)  warn(__FILE__,__LINE__,fmt,##__VA_ARGS__)
    #define error(fmt,...)  error(__FILE__,__LINE__,fmt,##__VA_ARGS__)
    #define fatal(fmt,...)  fatal(__FILE__,__LINE__,fmt,##__VA_ARGS__)

    //封装真正能用的宏函数
    #define LOG_DEBUG(logger,fmt,...) (logger)->debug(fmt,##__VA_ARGS__)//这里就用了上面的宏函数
    #define LOG_INFO(logger,fmt,...) (logger)->info(fmt,##__VA_ARGS__)
    #define LOG_WARN(logger,fmt,...) (logger)->warn(fmt,##__VA_ARGS__)
    #define LOG_ERROR(logger,fmt,...) (logger)->error(fmt,##__VA_ARGS__)
    #define LOG_FATAL(logger,fmt,...) (logger)->fatal(fmt,##__VA_ARGS__)

    //封装主函数的调用
    #define LOGD(fmt,...) LOG_DEBUG(suplog::rootLogger(),fmt,##__VA_ARGS__)
    #define LOGI(fmt,...) LOG_INFO(suplog::rootLogger(),fmt,##__VA_ARGS__)
    #define LOGW(fmt,...) LOG_WARN(suplog::rootLogger(),fmt,##__VA_ARGS__)
    #define LOGE(fmt,...) LOG_ERROR(suplog::rootLogger(),fmt,##__VA_ARGS__)
    #define LOGF(fmt,...) LOG_FATAL(suplog::rootLogger(),fmt,##__VA_ARGS__)
}
```

# 项目测试
至此项目代码的基本功能已经完成，剩下的便是对其进行一系列测试了

[戳我去代码发布页🔗](https://github.com/sis-shen/Asynchrinous-Logging-System/releases/tag/%E6%AD%A3%E5%BC%8F%E7%89%88)

## 功能测试
测试⼀个⽇志器中包含有所有的落地⽅向，观察是否每个⽅向都正常落地，分别测试`同步⽅式`和`异步⽅式`落地后数据是否正常。

```C++
#include "suplog.hpp"
#include <iostream>

using namespace std;

//功能测试

void loggerTest(const std::string& logger_name)
{
    suplog::Logger::ptr lp = suplog::getLogger(logger_name);

    assert(lp.get());//防止拿到空指针
    LOGD("------------example---------------");
    //测试原生日志器
    lp->debug("%s", "logger->debug");
    lp->info("%s", "logger->info");
    lp->warn("%s", "logger->warn");
    lp->error("%s", "logger->error");
    lp->fatal("%s", "logger->fatal");
    //测试代理模式
    LOG_DEBUG(lp, "%s", "LOG_DEBUG");
    LOG_INFO(lp, "%s", "LOG_INFO");
    LOG_WARN(lp, "%s", "LOG_WARN");
    LOG_ERROR(lp, "%s", "LOG_ERROR");
    LOG_FATAL(lp, "%s", "LOG_FATAL");
    LOGD("-----------------------------------");

    std::string log_msg = "hello supdriver test log msg -";
    size_t count = 0;
    while(count < 1000000)//输出一百万条日志
    {
        std::string msg = log_msg + std::to_string(count++);
        lp->error("%s",msg.c_str());
    }

}

int main()
{
    suplog::GlobalLoggerBuilder::ptr glb(new suplog::GlobalLoggerBuilder());
    glb->buildLoggerName("sync-logger");//设置日志器名称
    // glb->buidFormatter("[%d][%c][%f:%l][%p] %m%n");//设置日志输出格式
    glb->buildLoggerLevel(suplog::LogLevel::Level::DEBUG);
    glb->buildSink<suplog::StdoutSink>();//创建标准输出落地方向
    glb->buildSink<suplog::FileSink>("./testdir/logs/sync.log");//创建文件落地方向
    glb->buildSink<suplog::RollSink>("./testdir/roll_logs/roll-",10*1024*1024);
    glb->buildLoggerType(suplog::Logger::Type::LOGGER_SYNC);

    glb->build();//建造同步日志器

    glb->buildLoggerName("async-logger");
    glb->buildSink<suplog::FileSink>("./testdir/logs/async.log");//创建文件落地方向
    glb->buildSink<suplog::RollSink>("./testdir/async-roll/roll-",10*1024*1024);
    glb->buildLoggerType(suplog::Logger::Type::LOGGER_ASYNC);

    glb->build();//建造异步日志器

    loggerTest("sync-logger");
    loggerTest("async-logger");

    return 0;
}
```

测试结果：功能运行正常，但是由于`100万`条日志实在太多，就不把测试结果放出来了

## 性能测试
下⾯对⽇志系统做⼀个性能测试，测试⼀下平均每秒能打印多少条⽇志消息到⽂件。

主要的测试⽅法是：每秒能打印⽇志数 = 打印⽇志条数 / 总的打印⽇志消耗时间

主要测试要素：同步/异步 & 单线程/多线程

+ 100w+条指定⻓度的⽇志输出所耗时间
+ 每秒可以输出多少条⽇志
+ 每秒可以输出多少MB⽇志

测试环境
+ `CPU`:Intel(R) Xeon(R) Platinum 2.5GHZ*2
+ `RAM`:2G
+ `OS`:Ubuntu 22.04 64位

> bench.hpp

```C++
#pragma once
#include "suplog.hpp"
#include <chrono>//处理时间


namespace suplog
{
void bench(const std::string& logger_name,size_t thread_num,
    size_t msglen,size_t msg_count)
{
    Logger::ptr lp = getLogger(logger_name);
    if(lp.get() ==nullptr) return;
    std::string msg(msglen,'1');//用字符1补全长度
    size_t msg_count_per_thread = msg_count/thread_num;
    std::vector<double> cost_time(thread_num);
    std::vector<std::thread>threads;
    std::cout<<"输入线程数量"<<thread_num<<std::endl;
    std::cout<<"输出日志数量"<<msg_count <<std::endl;
    std::cout << "输出⽇志⼤⼩: " << msglen * msg_count / 1024 << "KB" <<std::endl;

    for(int i =0;i<thread_num;++i)
    {
        //empalce_back直接构造对象，即新增新的任务线程
        threads.emplace_back([&,i](){
            auto start = std::chrono::high_resolution_clock::now();
            for(size_t j = 0;j<msg_count_per_thread;++j)
            {
                lp->fatal("%s",msg.c_str());//输出日志
            }
            auto end = std::chrono::high_resolution_clock::now();
            //计算时间差
            auto cost = std::chrono::duration_cast<std::chrono::duration<double>>(end-start);
            cost_time[i] = cost.count();
            auto avg = msg_count_per_thread/cost_time[i];//每个线程中每秒打印多少日志
            std::cout<<"线程"<<i<<"耗时："<<cost_time[i]<<"s ";
            std::cout<<"平均：： "<<(size_t)avg<<"/s\n";
        });
    }

    for(auto& thr:threads)
    {
        thr.join();//回收子线程
    }

    double max_cost = 0;
    for(auto cost:cost_time) max_cost = max_cost <cost?cost:max_cost;
    std::cout<<"总消耗时间： "<<max_cost<<std::endl;
    std::cout<<"平均每秒输出: "<<(size_t)(msg_count/max_cost)<<std::endl;
}

}

```

> main.cpp

```C++
#include "suplog.hpp"
#include "bench.hpp"
#include <iostream>

using namespace std;

//性能测试
void sync_bench_thread_log(size_t thread_count,
    size_t msg_count,size_t msg_len)
{
    static int num = 1;//记录函数被调用的次数
    std::string logger_name = "sync_bench_logger" + std::to_string(num++);
    LOGI("*****************************************************");
    LOGI("同步日志测试：%u threads,%u messages",thread_count,msg_count);

    suplog::GlobalLoggerBuilder::ptr glb(new suplog::GlobalLoggerBuilder);
    glb->buildLoggerName(logger_name);
    glb->buidFormatter("%m%n");
    std::string path = std::string("./testdir/sync")+std::to_string(num)+std::string(".log");
    glb->buildSink<suplog::FileSink>(path);
    glb->buildLoggerType(suplog::Logger::Type::LOGGER_SYNC);
    glb->build();
    
    suplog::bench(logger_name,thread_count,msg_len,msg_count);
    LOGI("*****************************************************");

}

void async_bench_thread_log(size_t thread_count,
    size_t msg_count,size_t msg_len)
{
    static int num = 1;
    std::string logger_name = "async_bench_logger"+std::to_string(num++);
    LOGI("*****************************************************");
    LOGI("异步日志测试：%u threads,%u messages",thread_count,msg_count);

    suplog::GlobalLoggerBuilder::ptr glb(new suplog::GlobalLoggerBuilder);
    glb->buildLoggerName(logger_name);
    glb->buidFormatter("%m%n");
    std::string path = std::string("./testdir/async")+std::to_string(num)+std::string(".log");
    glb->buildSink<suplog::FileSink>(path);
    glb->buildLoggerType(suplog::Logger::Type::LOGGER_ASYNC);
    glb->build();
    
    suplog::bench(logger_name,thread_count,msg_len,msg_count);
    LOGI("*****************************************************");

}

void bench_test()
{
    //同步写日志
    sync_bench_thread_log(1,1000000,100);
    sync_bench_thread_log(5,1000000,100);
    //异步日志输出，为了避免因为等待落地影响时间，所以日志数量降低为小于双缓冲区大小进行测试
    async_bench_thread_log(1,100000,100);
    async_bench_thread_log(5,100000,100);
}

int main()
{
    bench_test();
    return 0;
}
```
输出结果如下
```shell
当前日志器： root 未检测到⽇志格式,默认设置为:  %d{%H:%M:%S}%T%t%T[%p]%T[%c]%T%f:%l%T%m%n
当前日志器: root问检测到落地方向，默认为标准输出!
DEBUG同步日志器创建成功...
[10:51:28][140705783203648][INFO][root][main.cpp:13] *****************************************************
[10:51:28][140705783203648][INFO][root][main.cpp:14] 同步日志测试：1 threads,1000000 messages
DEBUG同步日志器创建成功...
输入线程数量1
输出日志数量1000000
输出⽇志⼤⼩: 97656KB
线程0耗时：1.23733s 平均：： 808188/s
总消耗时间： 1.23733
平均每秒输出: 808188
[10:51:30][140705783203648][INFO][root][main.cpp:25] *****************************************************
[10:51:30][140705783203648][INFO][root][main.cpp:13] *****************************************************
[10:51:30][140705783203648][INFO][root][main.cpp:14] 同步日志测试：5 threads,1000000 messages
DEBUG同步日志器创建成功...
输入线程数量5
输出日志数量1000000
输出⽇志⼤⼩: 97656KB
线程2耗时：1.01608s 平均：： 196834/s
线程0耗时：1.04435s 平均：： 191506/s
线程1耗时：1.06357s 平均：： 188045/s
线程4耗时：1.09672s 平均：： 182361/s
线程3耗时：1.11074s 平均：： 180060/s
总消耗时间： 1.11074
平均每秒输出: 900303
[10:51:31][140705783203648][INFO][root][main.cpp:25] *****************************************************
[10:51:31][140705783203648][INFO][root][main.cpp:34] *****************************************************
[10:51:31][140705783203648][INFO][root][main.cpp:35] 异步日志测试：1 threads,100000 messages
DEBUG异步日志器创建成功...
输入线程数量1
输出日志数量100000
输出⽇志⼤⼩: 9765KB
线程0耗时：0.322045s 平均：： 310515/s
总消耗时间： 0.322045
平均每秒输出: 310515
[10:51:31][140705783203648][INFO][root][main.cpp:46] *****************************************************
[10:51:31][140705783203648][INFO][root][main.cpp:34] *****************************************************
[10:51:31][140705783203648][INFO][root][main.cpp:35] 异步日志测试：5 threads,100000 messages
DEBUG异步日志器创建成功...
输入线程数量5
输出日志数量100000
输出⽇志⼤⼩: 9765KB
线程3耗时：0.0818183s 平均：： 244444/s
线程0耗时：0.102184s 平均：： 195725/s
线程1耗时：0.101945s 平均：： 196183/s
线程4耗时：0.104484s 平均：： 191417/s
线程2耗时：0.116347s 平均：： 171899/s
总消耗时间： 0.116347
平均每秒输出: 859496
[10:51:31][140705783203648][INFO][root][main.cpp:46] *****************************************************
```

从上面的测试可以看出:

在**单线程情况下**: 异步效率看起来还没有同步高，实际上是由于,对于**同步操作**,现在的IO操作在⽤⼾态都会有**缓冲区**进⾏缓冲区，
因此我们当前测试⽤例看起来的同步其实 **⼤多时候也是在操作内存**，只有在缓冲区满了才会涉及到阻塞写磁盘操作，即磁盘的`IO`操作占比很少。
而对于**异步操作**,⼀个很重要的原因就是单线程同步操作中不存在锁冲突，⽽单线程异**步⽇志操作存在⼤量的锁冲突**，因此性能也会有⼀定的降低。

但是，我们也要看到限制同步⽇志效率的最⼤原因是磁盘性能，打⽇志的线程多少并⽆明显区别，线程多了反⽽会降低，因为增加了磁盘的读写争抢

⽽对于异步⽇志的限制，并⾮磁盘的性能，⽽是cpu的处理性能，打⽇志并不会因为落地⽽阻塞，因此在多线程打⽇志的情况下性能有了显著的提⾼。

总结:

+ `同步日志器`适合`大内存` `单线程` `小规模`日志操作
+ `异步日志器`适合`强CPU` `多线程` `大规模`日志操作

# 扩展
到这里，整个项目已经基本完善，但关于日志系统还有很多可以扩展的地方，甚至把日志系统扩展为有日志服务器的`CS`服务模式

如下更新将会在以后的博客中实现,敬请期待~


+ 实现`日志服务器`和`客户端`,服务端负责存储日志，并提供检索，分析，展示等功能
+ 丰富Sink类
  + 支持按小时按天滚动文件
  + 支持将log通过**网络传输**落地到日志服务器(tcp/udp)
  + 支持在控制台通过日志等级渲染**不同颜色**输出方便定位
  + 支持落地日志到**数据库** 
  + 支持配置服务器地址，将日志落地到远程服务器
