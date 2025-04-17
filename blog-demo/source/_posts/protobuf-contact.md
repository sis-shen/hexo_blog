---
title: protobuf实战】网络通讯录
date: 2024-12-20 11:22:00
tags:
---

# protobuf简介
Protocol Buffers 是 Google 的⼀种语⾔⽆关、平台⽆关、可扩展的序列化结构数据的⽅法，它可⽤于（数据）通信协议、数据存储等。

Protocol Buffers 类⽐于 XML，是⼀种灵活，⾼效，⾃动化机制的结构数据序列化⽅法，但是⽐XML 更⼩、更快、更为简单。你可以定义数据的结构，然后使⽤特殊⽣成的源代码轻松的在各种数据流中使⽤各种语⾔进⾏编写和读取结构数据。你甚⾄可以更新数据结构，⽽不破坏由旧数据结构编译的已部署程序。

简单来讲， ProtoBuf（全称为 Protocol Buffer）是让结构数据序列化的⽅法，其具有以下特点：

+ **语⾔⽆关、平台⽆关**：即 ProtoBuf ⽀持 Java、C++、Python 等多种语⾔，⽀持多个平台。
+ **⾼效**：即⽐ XML 更⼩、更快、更为简单。
+ **扩展性、兼容性好**：你可以更新数据结构，⽽不影响和破坏原有的旧程序

## 使用特点

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412201506428.webp)

## 安装protobuf

下载 ProtoBuf 前⼀定要安装依赖库：autoconf automake libtool curl make g++ unzip

安装命令如下

对于Ubuntu用户:

```shell
sudo apt-get install autoconf automake libtool curl make g++ unzip -y
```

接下来正式安装protobuf

下载地址: `https://github.com/protocolbuffers/protobuf/releases`

这里使用的版本是 `Protocol Buffers v21.12`

具体的安装步骤请自行参考网络资料

# cpp-httplib简介
该库是一个**跨平台**的只有单个头文件的`HTTP/HTTPS`库

它十分的简单易用，仅需把`httplib.h`引入到代码中,并在编译选项中添加`-lpthread`引入多线程库即可

> 注意：库内使用的是阻塞式的socket,如果需要使用非阻塞式的socket，需要使用别的库

下面是服务端和客户端的简单示例
```C++
//服务端
#define CPPHTTPLIB_OPENSSL_SUPPORT
#include "path/to/httplib.h"

// HTTP
httplib::Server svr;

// HTTPS
httplib::SSLServer svr;

svr.Get("/hi", [](const httplib::Request &, httplib::Response &res) {
  res.set_content("Hello World!", "text/plain");
});

svr.listen("0.0.0.0", 8080);

```

```C++
//客户端
#define CPPHTTPLIB_OPENSSL_SUPPORT
#include "path/to/httplib.h"

// HTTP
httplib::Client cli("http://cpp-httplib-server.yhirose.repl.co");

// HTTPS
httplib::Client cli("https://cpp-httplib-server.yhirose.repl.co");

auto res = cli.Get("/hi");
res->status;
res->body;
```

# 实战项目

## 设计

+ **名称**: 网络通讯录
+ **需求列表**:
  + 增加联系人
  + 删除联系人
  + 查询联系人
  + 列出联系人
+ **使用技术**: 
  + `protobuf`用于网络通讯和文件持久化
  + `cpp-httplib`用于快速搭建http服务器
+ **通信示意图**
  + ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412201631715.webp)
+ **服务端类图结构**:这边展示了部分类，protobuf自动生成的类数量会比较多，就不完全展示了
  + ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412201631715.webp)
+ **客户端类图结构**:与服务端的类图结构差别不会太大，仅仅是接口上有部分差别

+ HTTP接口约定

| 路径 | 接口关键词 | 业务功能 |
| ---- | -------- | ------- |
| `/add` | `addContact` | 增加联系人 |
| `/del` | `delContact` | 删除联系人 |
| `/find` | `findContact` | 查询联系人 |
| `/list` | `listContact` | 列出联系人 |
## protobuf文件编写

对于每一类通信，都要写一个对应的`protobuf`文件

### baseContact.proto 
