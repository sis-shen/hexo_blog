---
title: C++异步web框架drogon使用探究
date: 2025-04-21 09:48:46
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504210959723.webp
---
# 简介
Drogon是一个基于C++17/20的Http应用框架，使用Drogon可以方便的使用C++构建各种类型的Web应用服务端程序。Drogon是作者非常喜欢的美剧《权力的游戏》中的一条龙的名字(汉译作卓耿)，和龙有关但并不是dragon的误写，为了不至于引起不必要的误会这里说明一下。

Drogon是一个跨平台框架，它支持Linux，也支持macOS、FreeBSD，OpenBSD，HaikuOS，和Windows。它的主要特点如下：

+ 网络层使用基于epoll(macOS/FreeBSD下是kqueue)的非阻塞IO框架，提供高并发、高性能的网络IO。
+ 全异步编程模式；
+ 支持Http1.0/1.1(server端和client端)；
+ 基于template实现了简单的反射机制，使主程序框架、控制器(controller)和视图(view)完全解耦；
+ 支持cookies和内建的session；
+ 支持后端渲染，把控制器生成的数据交给视图生成Html页面，视图由CSP模板文件描述，通过CSP标签把C++代码嵌入到Html页面，由drogon的命令行工具在编译阶段自动生成C++代码并编译；
支持运行期的视图页面动态加载(动态编译和加载so文件)；
+ 非常方便灵活的路径(path)到控制器处理函数(handler)的映射方案；
+ 支持过滤器(filter)链，方便在控制器之前执行统一的逻辑(如登录验证、Http Method约束验证等)；
+ 支持https(基于OpenSSL实现);
+ 支持websocket(server端和client端);
+ 支持Json格式请求和应答, 对Restful API应用开发非常友好;
+ 支持文件下载和上传,支持sendfile系统调用；
+ 支持gzip/brotli压缩传输；
+ 支持pipelining；
+ **提供一个轻量的命令行工具drogon_ctl**，帮助简化各种类的创建和视图代码的生成过程；
+ 基于非阻塞IO实现的异步数据库读写，目前支持PostgreSQL和MySQL(MariaDB)数据库；
+ 基于线程池实现sqlite3数据库的异步读写，提供与上文数据库相同的接口；
+ 支持Redis异步读写；
+ 支持ARM架构；
+ 方便的轻量级ORM实现，支持常规的对象到数据库的双向映射操作；
+ 支持插件，可通过配置文件在加载期动态拆装；
+ 支持内建插入点的AOP
+ 支持C++协程

## 关于子模块
该项目还包含了一个日志子模块，也就是说它把日志也集成进了框架中。

# Ubuntu安装
## 安装方式
为了获取最新的特性，我们使用源码编译的方式安装

## 依赖库
除了最基本的编译套件，它还依赖于`jsoncpp`,`ssl`等第三方库

## 安装命令
```shell
#安装依赖
sudo apt install -y gcc g++ cmake libssl-dev libjsoncpp-dev uuid-dev zlib1g-dev
#克隆仓库
git clone https://github.com/drogonframework/drogon.git
cd drogon
# 下载子模块
git submodule update --init
#编译安装（约5-10分钟）
mkdir build
cd build
cmake ..
# 多进程编译
make -j$(nproc) 
sudo make install
```

# 简单测试程序
我们可以简单地在`main()`函数中注册路由和创建服务器程序

```C++
#include <drogon/drogon.h>
#include <jsoncpp/json/json.h>
using namespace drogon;
int main()
{
    //注册路由
    app().registerHandler("/test?username={name}",
        [](const HttpRequestPtr& req,
           std::function<void (const HttpResponsePtr &)> &&callback,
           const std::string &name)
        {
            Json::Value json;
            json["result"]="ok";
            json["message"]=std::string("hello,")+name;
            auto resp=HttpResponse::newHttpJsonResponse(json);
            callback(resp);
        },
        {Get,"LoginFilter"});
    //启动服务器
    app().setLogPath("./")
         .setLogLevel(trantor::Logger::kWarn)
         .addListener("0.0.0.0", 8088)
         .setThreadNum(16)
        //  .enableRunAsDaemon()
         .run();
}
```
效果如下
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504211038157.webp)

但是，这并不适用于复杂的应用，试想假如有数十个或者数百个处理函数要注册进框架，main()函数将膨胀到不可读的程度。显然，让每个包含处理函数的类在自己的定义中完成注册是更好的选择。所以，除非你的应用逻辑非常简单，我们不推荐使用上述接口，更好的实践是，我们可以创建一个HttpSimpleController对象,我们使用命令行工具drogon_ctl命令创建控制器类，这样开发者只需要关心如何填入业务代码即可

# 认识 drogon_ctl
1. 创建项目
```shell
# 创建新项目（默认创建CMake项目）
drogon_ctl create project 项目名

# 创建简单项目（无CMake）
drogon_ctl create simple 项目名
```

2. 创建三种控制器
```shell
# 创建HTTP控制器
drogon_ctl create controller 控制器名

# 创建WebSocket控制器
drogon_ctl create controller -w 控制器名

# 创建RESTful API控制器
drogon_ctl create controller -r 资源名
```
3. 创建orm模型
```shell
# 从数据库生成ORM模型
drogon_ctl create model 表名 -f 数据库配置.json
```
配置文件示例
```json
# 示例数据库配置：
{
  "dbname": "test",
  "host": "127.0.0.1",
  "port": 5432,
  "user": "postgres",
  "passwd": "123456",
  "tables": ["users", "posts"]
}
```
4. 插件管理
```shell
# 创建插件
drogon_ctl create plugin 插件名

# 安装插件模板
drogon_ctl install 插件名
```

5. 视图模版
```shell
# 创建CSP模板
drogon_ctl create view 模板名.csp

# 创建Jade模板
drogon_ctl create view -t jade 模板名.jade
```

6. 其它功能
```shell
# 压力测试（需先安装wrk）
drogon_ctl benchmark http://localhost:8080/api -t 10s -c 100

# 配置文件校验
drogon_ctl config check config.json

# 生成API文档
drogon_ctl create docs -o ./docs
```

# 封装控制器类
因为我们只是简单的测试，所以我们不使用`drogon_ctl`创建项目，而是简单的自己编写一个`CMakeLists.txt`

然后我们使用命令行工具drogon_ctl命令创建控制器类，这样开发者只需要关心如何填入业务代码即可

```shell
mkdir controllers
cd controllers
drogon_ctl create controller -h SupCtr
```

然后写入业务代码，特别的，我们只需要在`SupCtr`内注册路由和业务函数即可

> main.cpp
```C++
#include <drogon/drogon.h>
#include <jsoncpp/json/json.h>
#include "controllers/SupCtr.h"
using namespace drogon;
int main()
{
    //启动服务器
    app().setLogPath("./log/")
         .setLogLevel(trantor::Logger::kWarn)
         .addListener("0.0.0.0", 8088)
         .setThreadNum(16)
        //  .enableRunAsDaemon()
         .run();
}
```

> controllers/SupCtr.h
```C++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

class SupCtr : public drogon::HttpController<SupCtr>
{
  public:
    METHOD_LIST_BEGIN
    // use METHOD_ADD to add your custom processing function here;
    // METHOD_ADD(SupCtr::get, "/{2}/{1}", Get); // path is /SupCtr/{arg2}/{arg1}
    // METHOD_ADD(SupCtr::your_method_name, "/{1}/{2}/list", Get); // path is /SupCtr/{arg1}/{arg2}/list
    // ADD_METHOD_TO(SupCtr::your_method_name, "/absolute/path/{1}/{2}/list", Get); // path is /absolute/path/{arg1}/{arg2}/list

    METHOD_ADD(SupCtr::hello,"/hello/{1:p1}",Get);

    METHOD_LIST_END
    void hello(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, std::string p1);
    // your declaration of processing function maybe like this:
    // void get(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, int p1, std::string p2);
    // void your_method_name(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, double p1, int p2) const;
};
```

> controllers/SupCtr.cpp
```C++
#include "SupCtr.h"

// Add definition of your processing function here
void SupCtr::hello(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, std::string p1)
{
    std::string ret = "hello " + p1;
    auto resp = HttpResponse::newHttpResponse();
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody(ret);
    callback(resp);
}
```

