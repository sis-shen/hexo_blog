---
title: spdlog高速C++日志库使用探究
date: 2025-03-07 10:26:47
tags:
---
# spdlog介绍
`spdlog` 是一个**高性能**、**超快速**、**零配置**的`C++`日志库，它旨在提供简洁的`API`和丰富的功能，同时保持**高性能**的日志记录。它支持**多种输出目标**、**格式化选项**、**线程安全**以及**异步**日志记录

## 为什么用spdlog
它有如下优点
+ **高性能**：spdlog 专为速度而设计，即使在高负载情况下也能保持良好的性能。 
+ **零配置**：无需复杂的配置，只需包含头文件即可在项目中使用。 
+ **异步日志**：支持异步日志记录，减少对主线程的影响。 
+ **格式化**：支持自定义日志消息的格式化，包括时间戳、线程 ID、日志级别等。 
+ **多平台**：跨平台兼容，支持 Windows、Linux、macOS 等操作系统。 
+ **丰富的 API**：提供丰富的日志级别和操作符重载，方便记录各种类型的日志。

## spdlog安装

### 前置依赖库安装
`spdlog`依赖`fmt`库，如果没有，需要提前安装
```shell
sudo apt update -y

sudo apt install -y  libfmt-dev
```
**同时编译的时候要加上选项 -lfmt**
### apt安装

```shell
sudo apt update
sudo apt install -y libspdlog-dev
```

### 源码安装

```shell
git clone https://github.com/gabime/spdlog.git
cd spdlog/ 
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
make && sudo make install
```

# spdlog使用
目前用到的头文件均来自于:

```C++
#include <spdlog/spdlog.h>  //核心库入口
#include <spdlog/logger.h>//提供 spdlog::logger
#include <spdlog/async_logger.h>    //提供spdlog::async_logger
#include <spdlog/async.h>       //提供工厂模式

```

**编译时要添加 -lpthread 和 -fmt 库链接**

## 日志输出等级枚举

```C++
namespace spdlog{
// Log level enum
namespace level {
enum level_enum
{
    trace = SPDLOG_LEVEL_TRACE,
    debug = SPDLOG_LEVEL_DEBUG,
    info = SPDLOG_LEVEL_INFO,
    warn = SPDLOG_LEVEL_WARN,
    err = SPDLOG_LEVEL_ERROR,
    critical = SPDLOG_LEVEL_CRITICAL,
    off = SPDLOG_LEVEL_OFF,
    n_levels
};
}
}
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503101358833.webp)

## 日志输出格式自定义
假定我们现在有一个日志器对象的指针`logger`,我们可以调用它的`set_pattern`接口来设置格式

```C++
logger->set_pattern("%Y-%m-%d %H:%M:%S [%t] [%-7l] %v");

/////////////可用的格式//////////
/*
%t - 线程ID（Thread ID）。 
%n - 日志器名称（Logger name）。 
%l - 日志级别名称（Level name），如 INFO, DEBUG, ERROR 等。 
%v - 日志内容（message）。
%Y - 年（Year）。 
%m - 月（Month）。 
%d - 日（Day）。 
%H - 小时（24-hour format）。 
%M - 分钟（Minute）。 
%S - 秒（Second）
*/////////////////////////////////
```

## 日志记录器类的接口声明
`spdlog`库提供了日志记录器的**工厂模式**，我们可以直接构造日志器对象`sdplog::logger`，也可以通过工厂构造。

下面是它的接口声明

```C++
namespace spdlog { 
class logger { 
    //构造函数
    logger(std::string name)； 
    logger(std::string name, sink_ptr single_sink) 
    logger(std::string name, sinks_init_list sinks) 
    //配置接口
    void set_level(level::level_enum log_level); 
    void set_formatter(std::unique_ptr<formatter> f); 
    //日志接口,支持不定参数列表
    template<typename... Args> 
    void trace(fmt::format_string<Args...> fmt, Args &&...args) ;
    template<typename... Args> 
    void debug(fmt::format_string<Args...> fmt, Args &&...args) ;
    template<typename... Args> 
    void info(fmt::format_string<Args...> fmt, Args &&...args) ;
    template<typename... Args> 
    void warn(fmt::format_string<Args...> fmt, Args &&...args) ;
    template<typename... Args> 
    void error(fmt::format_string<Args...> fmt, Args &&...args) ;
    template<typename... Args> 
    void critical(fmt::format_string<Args...> fmt, Args &&...args) ;
    //刷新日志 
    void flush(); 
    //策略刷新--触发指定等级日志的时候立即刷新日志的输出 
    void flush_on(level::level_enum log_level); 
}； 
}
```
## 异步日志记录器类
上面的那个只是基类，而现在这个`spdlog::async_logger`支持异步写日志

接口声明如下
```C++
class async_logger final : public logger { 
    async_logger(std::string logger_name, 
                 sinks_init_list sinks_list, 
                 std::weak_ptr<details::thread_pool> tp, 
                 async_overflow_policy overflow_policy = 
async_overflow_policy::block); 
    async_logger(std::string logger_name, 
                 sink_ptr single_sink, 
                 std::weak_ptr<details::thread_pool> tp, 
                 async_overflow_policy overflow_policy = 
async_overflow_policy::block); 

}
//异步日志输出需要异步工作线程的支持，这里是线程池类 
class SPDLOG_API thread_pool { 
    thread_pool(size_t q_max_items, 
        size_t threads_n, 
        std::function<void()> on_thread_start, 
        std::function<void()> on_thread_stop); 
    thread_pool(size_t q_max_items, size_t threads_n,  
        std::function<void()> on_thread_start); 
    thread_pool(size_t q_max_items, size_t threads_n); 
}; 
 
```

## 日志记录器工厂类
异步日志器工厂模式由`spdlog/async.h`提供

```C++
namespace spdlog{
template<async_overflow_policy OverflowPolicy = async_overflow_policy::block>
struct async_factory_impl{
    template<typename Sink, typename... SinkArgs>
    static std::shared_ptr<async_logger> create(std::string logger_name, SinkArgs &&...args);
};
using async_factory = async_factory_impl<async_overflow_policy::block>;
using async_factory_nonblock = async_factory_impl<async_overflow_policy::overrun_oldest>;


template<typename Sink, typename... SinkArgs>
inline std::shared_ptr<spdlog::logger> create_async(std::string logger_name, SinkArgs &&...sink_args);

template<typename Sink, typename... SinkArgs>
inline std::shared_ptr<spdlog::logger> create_async_nb(std::string logger_name, SinkArgs &&...sink_args);

//设置全局的线程池
inline void init_thread_pool(size_t q_size, size_t thread_count, std::function<void()> on_thread_start);

inline void init_thread_pool(size_t q_size, size_t thread_count);

inline std::shared_ptr<spdlog::details::thread_pool> thread_pool()
{
    return details::registry::instance().get_tp();
}
}
```

各种同步日志器工厂模式由`spdlog/sinks/`下的特殊头文件提供

比如下面的工厂来自头文件`<spdlog/sinks/stdout_color_sinks.h>`

```C++
namespace spdlog{
template<typename Factory = spdlog::synchronous_factory>
std::shared_ptr<logger> stdout_color_mt(const std::string &logger_name, color_mode mode = color_mode::automatic);

template<typename Factory = spdlog::synchronous_factory>
std::shared_ptr<logger> stdout_color_st(const std::string &logger_name, color_mode mode = color_mode::automatic);

template<typename Factory = spdlog::synchronous_factory>
std::shared_ptr<logger> stderr_color_mt(const std::string &logger_name, color_mode mode = color_mode::automatic);

template<typename Factory = spdlog::synchronous_factory>
std::shared_ptr<logger> stderr_color_st(const std::string &logger_name, color_mode mode = color_mode::automatic);

} // namespace spdlog
```

## 日志落地类
日志落地类的实现方式有很多,其中还区分了`单线程无锁版本`和`多线程锁保护版本`，这里不多赘述，只展示基类

+ `*_st`：*single thread*单线程版本，不用加锁，效率更高。
+ `*_mt`：*multi thread*多线程版本，用于多线程程序是线程安全的

```C++
namespace spdlog {

namespace sinks {
class SPDLOG_API sink
{
public:
    virtual ~sink() = default;
    virtual void log(const details::log_msg &msg) = 0;
    virtual void flush() = 0;
    virtual void set_pattern(const std::string &pattern) = 0;
    virtual void set_formatter(std::unique_ptr<spdlog::formatter> sink_formatter) = 0;

    void set_level(level::level_enum log_level);
    level::level_enum level() const;
    bool should_log(level::level_enum msg_level) const;

protected:
    // sink log level - default is all
    level_t level_{level::trace};
};

} // namespace sinks
} // namespace spdlog

```

## 全局接口
```C++
//输出等级设置接口 
void set_level(level::level_enum log_level); 
//日志刷新策略-每隔N秒刷新一次 
void flush_every(std::chrono::seconds interval) 
//日志刷新策略-触发指定等级立即刷新 
void flush_on(level::level_enum log_level);
```

## 代码样例
我们来测试如下要点

+ 多落地方向同步输出
  + 文件落地 
  + 标准输出落地
+ 设置输出等级下限
+ 异步输出日志

```C++
#include <spdlog/spdlog.h> 
#include <spdlog/sinks/stdout_color_sinks.h> 
#include <spdlog/sinks/basic_file_sink.h> 
#include <spdlog/async.h> 
 
void multi_sink_example() { 
    //创建一个标准输出的落地方向对象 
    auto console_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>(); 
    //该方向仅允许warn等级以上的日志输出 
    console_sink->set_level(spdlog::level::warn); 
    console_sink->set_pattern("[multi_sink_example] [%^%l%$] %v"); 
    //创建一个文件输出的落地方向对象 
    auto file_sink = std::make_shared<spdlog::sinks::basic_file_sink_mt>( 
        "logs/multisink.txt", true); 
    //该方向允许trace等级以上的日志输出 
    file_sink->set_level(spdlog::level::trace); 
 
    //构造一个日志器对象，用于输出日志 
    spdlog::logger logger("multi_sink", {console_sink, file_sink});

    //每个日志器都可以设置初步过滤等级，其次内部每个sink也可以设置自己独立的过滤等级 
    logger.set_level(spdlog::level::debug); 
    logger.set_pattern("%Y-%m-%d %H:%M:%S [%l] %v"); 
    logger.warn("this should appear in both console and file"); 
    logger.info("this message should not appear in the console, only in the file"); 
} 
 
void async_example() { 
    //void init_thread_pool(size_t q_size, size_t thread_count); 
    spdlog::init_thread_pool(32768, 1);//设置默认线程池属性信息: 任务队列最大任务缓存数为32768,最大异步线程数为1 
    //通过工厂模式创建异步日志记录器的同时，会在内部创建默认线程池作为异步线程 
    auto async_logger = spdlog::basic_logger_mt<spdlog::async_factory>( 
        "async_file_logger", "logs/async_log.txt"); 
    async_logger->set_pattern("%Y-%m-%d %H:%M:%S [%l] %v"); 
    for (int i = 1; i < 101; ++i) { 
        //需要注意的是，在多参数的时候，spdlog并非使用 %s %d这种通配符来匹配参数 
        //而是使用 {}, spdlog可以自己识别数据类型 
        async_logger->info("Async message #{} {}", i, "hello"); 
    } 
} 
 
int main() 
{ 
    multi_sink_example(); 
    // async_example(); 
    return 0; 
}
```

*运行结果如下*

1. 多落地方向

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503101003296.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503101005730.webp)

2. 异步输出

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503101007745.webp)

# 二次封装spdlog
因为spdlog的日志输出对文件名和行号并不是很友好，以及因为spdlog本身实现了线程安全，如果使用默认日志器每次进行单例获取，效率会有降低，因此进行二次封装，简化使用。 

我们主要实现如下几点功能:

1. 日志的初始化封装接口
2. 日志的输出接口封装宏 
3. 仅使用唯一的全局日志器对象

> logger.hpp
```C++
#pragma once
#include <spdlog/spdlog.h>
#include <spdlog/sinks/stdout_color_sinks.h>
#include <spdlog/sinks/basic_file_sink.h>
#include <spdlog/async.h>
#include <iostream>

// mode - 运行模式： true-发布模式； false调试模式

namespace btyGoose{
std::shared_ptr<spdlog::logger> g_default_logger;
void init_logger(bool mode, const std::string &file, int32_t level)
{
    if (mode == false) {
        //如果是调试模式，则创建标准输出日志器，输出等级为最低
        g_default_logger = spdlog::stdout_color_mt("default-logger");
        g_default_logger->set_level(spdlog::level::level_enum::trace);
        g_default_logger->flush_on(spdlog::level::level_enum::trace);
        g_default_logger->set_pattern("[%n][%H:%M:%S][%t]%^[%-8l]%$%v");//标记颜色的起始位置
    }else {
        //否则是发布模式，则创建文件输出日志器，输出等级根据参数而定
        g_default_logger = spdlog::basic_logger_mt("default-logger", file);
        g_default_logger->set_level((spdlog::level::level_enum)level);
        g_default_logger->flush_on((spdlog::level::level_enum)level);
        g_default_logger->set_pattern("[%n][%H:%M:%S][%t][%-8l]%v");
    }
    
}

#define LOG_TRACE(format, ...) btyGoose::g_default_logger->trace(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_DEBUG(format, ...) btyGoose::g_default_logger->debug(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_INFO(format, ...) btyGoose::g_default_logger->info(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_WARN(format, ...) btyGoose::g_default_logger->warn(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_ERROR(format, ...) btyGoose::g_default_logger->error(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_FATAL(format, ...) btyGoose::g_default_logger->critical(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
}

```

## 小小地测试一下

```C++
#include "logger.hpp"
#include <string>
#include <string.h>
const std::string log_file("log_file.txt");
const int32_t level = spdlog::level::warn;
//这次是DEBUG模式
const bool release_mode = false;
int main() 
{ 
    btyGoose::init_logger(release_mode,log_file,level);

    LOG_TRACE("trace");
    LOG_DEBUG("I have {} bugs",555);
    LOG_WARN("this is a warn");
    LOG_ERROR("error! error str: {}"," I am a error str");
    LOG_FATAL("fatal, fail to solve");
    return 0; 
}
```

输出结果

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503101100059.webp)