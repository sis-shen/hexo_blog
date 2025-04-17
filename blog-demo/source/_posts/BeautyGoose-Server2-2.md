---
title: v2.1~V2.2美鹅外卖平台更新内容
date: 2025-03-11 14:39:37
tags:
---

# v2.1 命令行参数更新

## 1. gflags库引入
[戳我去gflags的介绍博客](https://www.supdriver.top/2025/03/06/gflags/)

`gflags`是`Google`开发的一个开源库，用于`C++`应用程序中**命令行参数**的声明、定义和解析。`gflags`库提供了一种简单的方式来`添加`、`解析`和`文档化`命令行标志（`flags`），使得程序可以根据不同的运行时配置进行调整。

有了`gflags`后，我们可以统一地使用`命令行参数`或者`统一的配置文件`方便地配置服务端的相关参数
## 2. 删除JSON配置文件
相比于命令行参数的配置文件，`JSON`文件写起来相对繁琐，且不够灵活，因此删除了`JSON`文件，全面使用命令行参数替代

## 3. 在main.cpp中定义命令行参数

因为如果要解析命令行参数，就必须使用`google::ParseCommandLineFlags(&argc, &argv, true);`解析`main`函数的传参，所以直接把命令行参数的定义写在`main.cpp`的全局区中
> main.cpp
```C++
#include <gflags/gflags.h>

DEFINE_string(host,"0.0.0.0","服务器监听的源ip地址范围");
DEFINE_uint32(port,80,"服务器的监听端口");

DEFINE_string(db_user,"root","数据库的连接用户");
DEFINE_string(db_password,"root_password","数据库用户的密码");
DEFINE_string(db_host,"localhost","数据库服务所在的ip");
DEFINE_string(db_port,"3306","数据库服务开放的端口");
DEFINE_string(db_database,"db","数据库中存储数据的数据库");

DEFINE_string(log_logfile,"./log/logfile","日志落地文件及路径");
DEFINE_bool(log_release_mode,false,"设置是否为发布模式输出日志,\ntrue:过滤低等级日志，输出到文件\nfalse:输出所有日志，输出到标准输出");
DEFINE_int32(log_release_level,spdlog::level::warn,"设置release模式下过滤的日志等级");
```

## 4. 修改HTTPServer和DatabaseClient的接口，使之能被显示初始化
部分修改代码接口如下(*下面的代码含v2.2更新内容，由后文介绍*)
```C++
btyGoose::HTTPServer::HTTPServer()
{

}

void btyGoose::HTTPServer::initDB(const string&_user,const string&_password,const string&_host,const string&_port,const string&_database)
{
    // LOG_TRACE("HTTPServer: 开始初始化db");
    db.init(_user,_password,_host,_port,_database);
    db.start();
}

void btyGoose::HTTPServer::init(const std::string& _host,const uint32_t _port)
{
    host = _host;
    port = _port;
    initTestAPI();
    initAccountAPI();
    initConsumerAPI();
    initMerchantAPI();
    initAdminAPI();
    LOG_DEBUG("HTTPServer初始化完成");
}

void btyGoose::HTTPServer::start()
{
    LOG_DEBUG("HTTPServer开始监听{}:{}",host,port);
    cout <<svr.listen(host, port);
}

DatabaseClient::DatabaseClient()
{

}

void DatabaseClient::init(const string&_user,const string&_password,const string&_host,const string&_port,const string&_database)
{
    // LOG_TRACE("数据库开始初始化参数");
    user = _user;
    password = _password;
    host = _host;
    port = _port;
    database = _database;
    LOG_DEBUG("数据库初始化参数:\n\tuser:{}\n\tpassword:{}\n\thost:{}\n\tport:{}\n\tdatabase:{}",user,password,host,port,database);
}

void DatabaseClient::start()
{
    try {
        sql::mysql::MySQL_Driver *driver = sql::mysql::get_mysql_driver_instance();
        // cout<< << "tcp://"+host+":"+port << "\nuser: " << user <<"\npassword: "<<password<<"\n";
        LOG_INFO("开始连接数据库 tcp://{}:{}\n\tuser:{}",host,port,user);
        con = driver->connect("tcp://"+host+":"+port, user, password);
        con->setSchema("BeautyGoose");
        stmt = con->createStatement();
    } catch (sql::SQLException &e) {
        LOG_FATAL( "数据库连接失败喵！\n\t错误码: {}\n\t信息:",e.getErrorCode(),e.what());
        exit(-1);
    }
}

```

## 修改main()函数体内容

```C++
int main(int argc, char *argv[])
{
    google::ParseCommandLineFlags(&argc,&argv,true);//解析命令行参数
    btyGoose::init_logger(FLAGS_log_release_mode,FLAGS_log_logfile,FLAGS_log_release_level);//初始化日志器
    LOG_DEBUG("全局日志器初始化成功");
    btyGoose::HTTPServer::getInstance()->initDB(FLAGS_db_user,FLAGS_db_password,FLAGS_db_host,FLAGS_db_port,FLAGS_db_database);
    btyGoose::HTTPServer::getInstance()->init(FLAGS_host,FLAGS_port);
    btyGoose::HTTPServer::getInstance()->start();
    return 0;
}
```

# v2.2 日志输出更新
[戳我去spdlog的介绍博客](https://www.supdriver.top/2025/03/07/spdlog/)

## 1. spdlog日志库引入
`spdlog` 是一个**高性能**、**超快速**、**零配置**的`C++`日志库，它旨在提供简洁的`API`和丰富的功能，同时保持**高性能**的日志记录。它支持**多种输出目标**、**格式化选项**、**线程安全**以及**异步**日志记录

## 2. spdlog二次封装
因为spdlog的日志输出对文件名和行号并不是很友好，以及因为spdlog本身实现了线程安全，如果使用默认日志器每次进行单例获取，效率会有降低，因此进行二次封装，简化使用。 

同时因为多次包含头文件，如果声明和实现**都写在.h文件中时**,就会造成`重复定义`的严重问题，所以尽管二次封装的内容不多，还是采用了声明与实现**分离地**写在不同文件中。

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

namespace btyGoose{
extern std::shared_ptr<spdlog::logger> g_default_logger;
void init_logger(bool mode, const std::string &file, int32_t level);

#define LOG_TRACE(format, ...) btyGoose::g_default_logger->trace(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_DEBUG(format, ...) btyGoose::g_default_logger->debug(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_INFO(format, ...) btyGoose::g_default_logger->info(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_WARN(format, ...) btyGoose::g_default_logger->warn(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_ERROR(format, ...) btyGoose::g_default_logger->error(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
#define LOG_FATAL(format, ...) btyGoose::g_default_logger->critical(std::string("[{}:{}] ") + format, __FILE__, __LINE__, ##__VA_ARGS__)
}
```
> logger.cpp
```C++
#include "logger.hpp"

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
}
```
## 3. 替换原本的日志输出方式为二次封装的宏函数输出日志
原本`HTTPServer`类和`DatabaseClient`类的日志输出仅支持使用标准输出和标准错误输出的简单输出，因此我们需要使用封装好的`宏函数`将其替换，使服务端能够更加规范、高可读、高效率地输出日志

相关改动的文件:

+ `HTTPServer.cpp`
+ `DatabaseClient.cpp`