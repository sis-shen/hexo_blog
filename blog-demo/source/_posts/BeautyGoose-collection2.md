---
title: C++项目】v2.3美鹅外卖-总集介绍博客
date: 2025-02-23 09:55:40
tags:
---
# 项目介绍
为客户提供一个C/S架构的外卖平台，使用Qt框架和cpp-httplib 等模块实现其业务功能。能够服务消费者，商家和管理员的使用需求，目前支持的客户端平台为Windows平台。服务端平台只需支持Docker即可

使用的技术点有:

+ QT界面框架
+ Json序列化与反序列化
+ cpp-httplib库
+ MySQL connector/c++
+ Redis 缓存层
+ nginx 负载均衡
+ Docker容器与镜像管理技术

## 运行环境
*当前版本v2.5*

客户端: windows 11

服务端: 阿里云ECS云服务器 ubuntu 22.04
## 需求分析

### 用户需求
当前目标实现的用户需求有

1. 用户注册、登录
2. 商家添加产品
3. 商家查看和修改产品
4. 商家查看和处理订单
5. 消费者查看和添加购物车
6. 消费者查看和结算订单
7. 消费者优惠等级变化
8. 管理员查看销售日志

## 角色抽象归纳
根据需求的功能点，可以将系统的参与者抽象成两层三类。

+ `用户`: 所有具体参与者的总称，更高一级的抽象
+ `商家`：提供菜品和处理订单的一类人
+ `消费者`：点餐和请求订单的一类人
+ `管理员`：有权限查看销售日志的一类人

### 用例图
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501171350578.webp)


## 功能需求
根据用户的需求，我们提出更具体，更偏向实现的功能需求

*阅读提示：点击右侧目录可以跳转*

1. 用户注册功能
    + 用户可以使用唯一的***账号名和手机号**和进行账号注册

2. 用户登录功能
    + 用户可以使用**账号名或手机号**进行账号登录

3. 商家菜品注册功能
    + 商家可以使用菜品注册功能来注册新的菜品

4. 商家菜品查看功能
    + 商家可以看到自己的所有菜品列表，也能够看到某个菜品的详情

5. 商家菜品修改功能
    + 商家可以修改菜品的信息，也可以删除菜品

6. 商家订单查看功能
    + 商家能够看到自己的订单列表，也可以看到某个订单的详情

7. 商家订单处理功能
    + 商家能够选择接单或拒单

8. 消费者查看菜品功能
    + 消费者可以看到所有菜品列表，也可以看到某一个菜品的详情

9. 消费者购物车添加功能
    + 消费者能够将菜品添加到对应商家的购物车中

10. 消费者查看购物车功能
    + 消费者能够看到所有商家的购物车列表

11. 消费者订单生成功能
    + 消费者能够将指定的购物车生成为订单

12. 消费者第三方支付功能
    + 消费者能够使用第三方支付订单

13. 消费者订单支付功能
    + 消费者能够支付订单并支付完成的订单被商家看到

14. 消费者提升优惠等级功能
    + 消费者能够看到自己的优惠等级，并通过购买VIP的方式提升优惠等级

15. 管理员日志查看功能
    + 使用管理员账户登录后，可以看到所有的销售日志

## 与其它系统的接口

### 数据源
本平台仅主要作为商家和消费者交互的平台，因此绝大部分数据由用户产生，而平台本身只产生日志数据

### 线上支付服务
本平台只提用于接入第三方支付的接口，而不自己实现任何的支付服务。因此所有的支付服务均由第三方系统提供

### 数据库服务
本平台依托于`MySQL 8.0`提供的数据库管理服务,只实现与其交互的接口。

## 软件需求规格

### 系统架构

#### 系统结构图
+ 组件图
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501171433499.webp)

+ 应用数据架构图
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501191456115.webp)

本版本采用在开发机上部署服务端，云服务器上部署数据库，并使用云服务器提供内网穿透服务的方式为客户端提供服务
### 系统层次结构
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501191347270.webp)

## 软件模块设计

### 账户管理模块
1. 账户注册功能         accountRegister
2. 账户用户名登录功能   accountLoginByName
3. 账户手机号登录功能   accountLoginByPhone
4. 账户优惠等级升级功能 accountUpdateLevel

### 第三方支付模块
1. 第三方支付功能       thirdPay

### 消费者模块
1. 消费者查看菜品列表功能   consumerToDishList
2. 消费者查看菜品详情功能   consumerToDishDetail
3. 消费者购物车添加功能     consumerCartAdd
4. 消费者购物车移除功能     consumerCartPop
5. 消费者购物车列表查看功能 consumerToCartList
6. 消费者订单生成功能       consumerOrderGenerate
7. 消费者订单详情查看功能   consumerOrderDetail
8. 消费者订单支付功能能     consumerOrderPay
9. 消费者订单取消功能       consumerOrderCancel

### 商家模块
1. 商家菜品列表查看功能     merchantToDishList
2. 商家菜品注册功能         merchantDishRegister
3. 商家菜品详情查看功能     merchantDishDetail
4. 商家菜品修改功能         merchantDishEdit
5. 商家菜品删除功能         merchantDishDel 
6. 商家订单列表查看功能     merchantToOrderList
7. 商家订单详情查看功能     merchantOrderDetail
8. 商家订单接单功能         merchantOrderAccept
9. 商家订单拒单功能         merchantOrderReject

### 管理员模块
1. 日志查询功能             adminHistoryLog

## 数据库设计
本项目采用MySQL 8.0.40-0ubuntu0.22.04.1

目前按项目初期的小访问量设计的数据库，因此只使用单一的数据库存储所有文件，表按数据对象的类型划分。

### 数据库文件和连接账户
#### 建数据库
```sql
create database BeautyGoose;
use BeautyGoose;
```

#### 创建专门的连接用用户
```sql
create user 'btyGooseUser'@'%' identified by 'Password!1';
grant all on BeautyGoose.* to 'btyGooseUser'@'%';
flush privileges;
```
### 数据表设计

![account](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192017506.webp)

![dish](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192024255.webp)

![order](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192026572.webp)

![history](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192025124.webp)

![orderDish](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192027578.webp)

![history](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192027578.webp)

#### 建立账户表
因为本项目的重点是功能实现，数据安全目前暂不考虑，所以所有数据采用明文存储的方式

```sql
create table account(
    uuid varchar(13) unique,
    phone varchar(12) unique,
    name varchar(10) not null,
    password varchar(64) not null,
    nickname varchar(32),
    level tinyint not null default 0,
    icon_data blob,
    type tinyint not null default 0,
    balance int unsigned default 0
);

```

#### 建立菜品表

```sql
create table dishes(
    uuid varchar(13) unique,
    merchant_id varchar(13) not null,
    merchant_name varchar(10) not null,
    name varchar(10) not null,
    icon_path varchar(150),
    description text,
    base_price decimal(6,2),
    price_factor float(4,2) unsigned,
    is_delete boolean not null default 1
);
```

#### 建立订单表

```sql
create table orders(
    time varchar(32) not null, 
    uuid varchar(13) unique,
    merchant_id varchar(13) not null,
    merchant_name varchar(10) not null,
    consumer_id varchar(13) not null,
    consumer_name varchar(10) not null,
    level tinyint not null default 0,
    pay decimal(10,2) not null default 0,
    status tinyint not null default 0,
    sum int not null
);
```

#### 建立历史订单表

```sql
create table history(
    time varchar(32) not null, 
    uuid varchar(13) unique,
    merchant_id varchar(13) not null,
    merchant_name varchar(10) not null,
    consumer_id varchar(13) not null,
    consumer_name varchar(10) not null,
    level tinyint not null default 0,
    pay decimal(10,2) not null default 0,
    status tinyint not null default 0,
    sum int not null
);
```

#### 建立orderDish表

```sql
create table orderDish(
    order_id varchar(13) not null,
    dish_id varchar(13) not null,
    merchant_id varchar(13) not null,
    name varchar(10) not null,
    dish_price decimal(6,2),
    count int not null default 0
);
```

#### 建立historyDish表

```sql
create table historyDish(
    order_id varchar(13) not null,
    dish_id varchar(13) not null,
    merchant_id varchar(13) not null,
    name varchar(10) not null,
    dish_price decimal(6,2),
    count int not null default 0
);
```



# 服务端重大更新

## 平台迁移

服务端的开发环境和部署环境均完成了**操作系统平台迁移**。从`Windows 11`转移到了`Ubuntu 22.04`云服务器上。

## 第三方库依赖更新

1. **完全抛弃QT框架**
2. 新增`jsoncpp`替代`QJson`

## 部署升级

1. 引入`Dockerfile`完成镜像自动构建
2. 使用`docker-compose.yml`自动完成服务编排和部署启动,使单机开发测试更简单

# 服务端详细设计

## 版本信息
**当前版本**: v2.5 Redis缓存层更新

当前服务器使用了如下技术栈:
+ `MySQL`数据库服务：由`MySQL 8.0`提供数据持久化服务
+ `MySQL API`服务: 由`MySQL Connector/C++ 7`提供原生`API`,并在开发中封装成数据库对象及其接口
+ `序列化与反序列化`服务: 由`jsoncpp`库提供对象的序列化与反序列化
+ `HTTP短链接通信`服务 : 由开源项目`cpp-httplib`提供的单头文件库提供简单易用的HTTP服务
+ `gflags`命令行参数解析 ：由谷歌的开源项目`gflags`提供命令行解析服务
+ `spdlog`日志输出服务 ：由开源项目`spdlog`提供的异步、高速、易于配置的日志输出服务
+ 数据库客户端守护线程保证连接的健壮性
+ `nginx`反向代理，构建应用服务集群架构
+ `Redis`提供分布式数据缓存服务

## 当前架构图
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504170806632.webp)

## 服务端层次结构
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504161139779.webp)

## HTTP请求路径约定
客户端与服务端的交互对具体函数的调用，取决于`HTTP 请求路径`的约定，所以要提前约定好

### 账户相关 Account API
| 路径 | 业务 |
|--- | --- | 
| `/account/register` | 账户**注册** |
| `/account/login/username` | 账户使用**用户名登录** | 
| `/account/login/phone` | 账户使用**手机号登录** | 
| `/account/update/level` | 消费者账户**优惠等级更新** | 
| `/account/update/nickname` | 账号**更新昵称** |

### 消费者相关 Consumer API
| 路径 | 业务 |
|--- | --- | 
| `/consumer/order/detail/dishlist` | 消费者获取**订单**详情内的**菜品列表** |
| `/consumer/dish/list` | 消费者**菜品列表** |
| `/consumer/dish/dishInfo` | 消费者获取**菜品详情** | 
| `/consumer/order/generate` | 消费者获**取订单生成** |
| `/consumer/order/list` | 消费者**获取订单列表** |
| `/consumer/order/pay/confirm` | 消费者**完成订单支付** | 
| `/consumer/order/cancel` | 消费者**请求取消订单** |

### 商家相关 Merchant API
| 路径 | 业务 |
| --- | --- |
| `/merchant/dish/list` | 商家获取**菜品列表** | 
| `/merchant/dish/register` | 商家请求**注册菜品** | 
| `/merchant/dish/info` | 商家获取**菜品详情** | 
| `/merchant/dish/update` | 商家请求**更新菜品** |
| `/merchant/dish/del` | 商家请求**删除菜品** |
| `/merchant/order/list` | 商家请求**订单列表** |
| `/merchant/order/detail/dishlist` | 商家请求**订单详情的菜品列表** | 
| `/merchant/order/accept` | 商家**接单** | 
| `/merchant/order/reject` | 商家**拒单** |

## 管理员相关 Admin API
| 路径 | 业务 |
| --- | --- |
| `/admin/order/list` | 管理员获取订单列表 |

## 参数配置方法设计
这里使用第三方库`gflags`提供的命令行解析服务，因此所有的关键参数都使用命令行参数，运行时的参数既可以使用命令行配置，也可以导入配置文件自动解析和赋值。

## 日志输出方法设计
这里使用第三方库`spdlog`提供的异步日志输出服务，使程序能够异步地高效输出标准化的日志信息，便于代码开发和运行维护。

## 负载均衡方法实现
使用现成的`nginx`项目作为反向代理服务器，并实现负载均衡功能

## 开发环境
| Key | Value |
| ---- | --- |
| 开发平台 | 阿里ECS服务器 Ubuntu 22.04 | 
| MySQL Connector/C++ 版本 | Connector 7 | 
| C++语法标准 | C++17 | 

## 文件结构一览
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503111533646.webp)

## 编写 docker-compose.yml
在当前架构下，我们要运行如下容器

1. `web`: 由镜像`nginx`传入配置文件后二次制作镜像
2. `server`: 镜像`btygoose`,由项目源码构建而成
3. `server2`:第二个server服务器
4. `db`: 来自镜像`mysql:8.0`
5. `redismaster`: 来自镜像`redis:7.2.4`

### 容器独立网络配置
为了保持服务的网络独立性，我们先创建一个独立于物理机网络环境的子网

相关的部分`docker-compose.yml`代码如下

```YML
networks:  # 新增网络定义
  btygoose_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16  # 自定义子网
```


### 数据库容器准备
既然使用了容器技术，那么数据库我们选择使用官方镜像来启动`mysql`容器提供服务

其中我们将使用提前编写好的`数据库初始化脚本`和`存储卷技术`

初始化脚本我们放在`mysql/initdb/`目录下,目前只有一个文件，命名为`init.sql`

```sql
-- 创建数据库
DROP DATABASE if EXISTS BeautyGoose;
DROP USER IF EXISTS btyGooseUser;
CREATE DATABASE BeautyGoose DEFAULT CHARACTER SET utf8mb4;

-- 创建用户和赋予权限
CREATE USER 'btyGooseUser'@'%' IDENTIFIED BY 'Password!1';
grant all on BeautyGoose.* to 'btyGooseUser'@'%';
flush privileges;

-- 创建表结构
use BeautyGoose;

create table account(
    uuid varchar(13) unique,
    phone varchar(12) unique,
    name varchar(10) not null,
    password varchar(64) not null,
    nickname varchar(32),
    level tinyint not null default 0,
    icon_data blob,
    type tinyint not null default 0,
    balance int unsigned default 0
);

create table dishes(
    uuid varchar(13) unique,
    merchant_id varchar(13) not null,
    merchant_name varchar(10) not null,
    name varchar(10) not null,
    icon_path varchar(150),
    description text,
    base_price decimal(6,2),
    price_factor float(4,2) unsigned,
    is_delete boolean not null default 1
);

create table orders(
    time varchar(32) not null, 
    uuid varchar(13) unique,
    merchant_id varchar(13) not null,
    merchant_name varchar(10) not null,
    consumer_id varchar(13) not null,
    consumer_name varchar(10) not null,
    level tinyint not null default 0,
    pay decimal(10,2) not null default 0,
    status tinyint not null default 0,
    sum int not null
);

create table orderDish(
    order_id varchar(13) not null,
    dish_id varchar(13) not null,
    merchant_id varchar(13) not null,
    name varchar(10) not null,
    dish_price decimal(6,2),
    count int not null default 0
);

create table history(
    time varchar(32) not null, 
    uuid varchar(13) unique,
    merchant_id varchar(13) not null,
    merchant_name varchar(10) not null,
    consumer_id varchar(13) not null,
    consumer_name varchar(10) not null,
    level tinyint not null default 0,
    pay decimal(10,2) not null default 0,
    status tinyint not null default 0,
    sum int not null
);

create table historyDish(
    order_id varchar(13) not null,
    dish_id varchar(13) not null,
    merchant_id varchar(13) not null,
    name varchar(10) not null,
    dish_price decimal(6,2),
    count int not null default 0
);
```

相关的`docker-compose.yml`的部分代码如下

```YML
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: "Password!1"
    volumes:
      - ./mysql/varlib/:/var/lib/mysql 
      - ./mysql/initdb:/docker-entrypoint-initdb.d/
    healthcheck:
      test: mysql -uroot -pPassword!1 -e 'select 1'
      interval: 10s
      timeout: 5s
      retries: 5
    networks:  # 新增网络配置
      - btygoose_net
```

### 后端服务配置
+ 镜像的构建由`Dockerfile`自动完成
+ 端口映射：防止宿主机的80端口被占用，我们把容器的`80`端口映射到`30001`
+ 依赖： 后端服务依赖数据库服务的健康状态

*由于这是最后一部分代码，就合并到下一个标题展示了*

### 完整的docker-compose.yml
```YML
services:
  web:
    image: mynginx
    build: 
      context: ./nginx
    ports:
      - 30001:80
    depends_on:
      server:
        condition: service_started
      server2:
        condition: service_started
    networks:  # 新增网络配置
      - btygoose_net
  server:
    build:
      context: ./server
    image: btygoose
    environment:
      - TZ=Asia/Shanghai  # 设置时区变量
      - LANG=C.UTF-8      # 同步语言编码
    # ======= 仅测试使用  ======
    # privileged: true
    # ======== end =========
    # ports:
    #   - 30001:80
    depends_on:
      db:
        condition: service_healthy
      redismaster:  
        condition: service_healthy
    networks:  # 新增网络配置
      - btygoose_net

  server2:
    image: btygoose
    environment:
      - TZ=Asia/Shanghai  # 设置时区变量
      - LANG=C.UTF-8      # 同步语言编码
    depends_on:
      db:
        condition: service_healthy
      redismaster:  
        condition: service_healthy
    networks:  
      - btygoose_net
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: "Password!1"
    volumes:
      - ./mysql/varlib/:/var/lib/mysql 
      - ./mysql/initdb:/docker-entrypoint-initdb.d/
    healthcheck:
      test: mysql -uroot -pPassword!1 -e 'select 1'
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - 30006:3306
    networks:  
      - btygoose_net

  redismaster:
    image: redis:7.2.4
    restart: always
    volumes:
    - ./redis/data:/data
    - ./redis/conf/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf  # 加载配置
    environment:
    - TZ=Asia/Shanghai
    - REDIS_PASSWORD=redispass123  # 设置访问密码
    - maxmemory=512mb     # 限定内存大小
    - maxmemory-policy=allkeys-lru
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "$$REDIS_PASSWORD", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:  
      - btygoose_net

networks:  # 新增网络定义
  btygoose_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16  # 自定义子网


```

# 服务端源码编写
除了`httplib.h`来自`cpp-httplib`库外，剩下的源代码文件均需要我们编写

## CoreData.hpp
核心数据结构、工具函数和序列化与反序列化函数均实现在这个头文件中

```C++
#pragma once
#include <string>
#include <random>
#include <array>
#include <jsoncpp/json/json.h>
#include <chrono>
namespace btyGoose{

    namespace util{
        static uint64_t getSecTime()
        {
            return std::chrono::duration_cast<std::chrono::seconds>(
                std::chrono::system_clock::now().time_since_epoch()
            ).count();
        }

        static std::string makeId(const char pre = '#')
        {
            // 生成6字节随机数 (6*8=48bit)
            std::array<uint8_t, 6> bytes;
            std::random_device rd;
            std::mt19937 gen(rd());
            
            // 填充随机字节
            for (auto& byte : bytes) {
                byte = static_cast<uint8_t>(gen() & 0xFF);
            }

            // 转换字节为十六进制字符串
            constexpr char hexmap[] = "0123456789abcdef";
            std::string id(13, pre); // 预填充前缀+12字符
            for (int i = 0; i < 6; ++i) {
                id[1 + 2*i] = hexmap[(bytes[i] & 0xF0) >> 4]; // 高四位
                id[2 + 2*i] = hexmap[bytes[i] & 0x0F];        // 低四位
            }

            return id;
        }
    }

    /// <summary>
	/// 定义核心数据结构
	/// </summary>
    namespace data{
        //账户
        struct Account	
		{
			///账户类型
			enum Type
			{
				CONSUMER,	//消费者账户
				MERCHANT,	//商家账户
				ADMIN		//管理员账户
			};
			enum Level
			{
				MEMBER,		//普通用户
				VIP,		//VIP用户
				SUVIP		//超级VIP用户
			};

			std::string uuid = "";		//唯一id
			std::string name = "";		//账户名
			std::string password = "";	//密码
			std::string nickname = "";	//昵称
			std::string icon;	//图标
			Type type = CONSUMER;			//账户类型
			double balance = 0;		//账户余额
			std::string phone = "";		//电话号码
			Level level = MEMBER;		//优惠等级
        };
        struct Dish
        {
            std::string uuid = "";			//菜品id
            std::string merchant_id = "";	//商家uuid
            std::string merchant_name = "";	//商家uuid
            std::string name = "";			//菜品名称
            std::string icon_path = "";		//网络图片的url
            std::string description = "";	//菜品的描述
            double base_price = 0;		//基础价格
            double price_factor = 1;	//价格影响因素
            bool is_delete = false;		//是否被删除
        
        };

        struct Order
        {
            enum Status
            {
                UNPAYED,		//未支付状态
                WAITING,		//等待商家处理
                OVER_TIME,		//订单超时
                REJECTED,		//商家拒收
                SUCCESS,		//成功完成
                ERR,			//订单出错
                FATAL,			//订单出现致命错误
                CANCELED		//消费者取消订单
            };

            std::string merchant_id;		//商家id
            std::string merchant_name;		//商家账户名称
            std::string consumer_id;		//消费者id
            std::string consumer_name;		//消费者账户名称
            std::string time;				//订单时间
            Account::Level level;		//优惠等级
            double pay;					//订单价格
            std::string uuid;				//订单id
            Status status;				//订单状态
            int sum = 0;				//订单内总菜品数
        };

        struct OrderDish
        {
            std::string order_id = "";		//订单id
            std::string dish_id = "";		//菜品id
            std::string merchant_id = "";	//商家id
            std::string name = "";			//菜品id
            double dish_price = 0;		//当时的基础价格
            int count = 0;				//菜品数量

        };
        
    }//data

    //存放重载函数
    namespace json{
        static std::string toJson(data::Account acc)
        {
            Json::Value root;
        
            // 基本字段映射
            root["uuid"] = acc.uuid;
            root["name"] = acc.name;
            root["password"] = acc.password;
            root["nickname"] = acc.nickname;
            root["icon"] = acc.icon;
            root["type"] = static_cast<int>(acc.type);
            root["balance"] = acc.balance;
            root["phone"] = acc.phone;
            root["level"] = static_cast<int>(acc.level);
    
            // 配置漂亮的输出格式
            Json::StreamWriterBuilder writer;
            writer["indentation"] = "    ";  // 4空格缩进
            writer["commentStyle"] = "None"; // 不保留注释
            writer["enableYAMLCompatibility"] = false; 
            
            return Json::writeString(writer, root);
        }

        static data::Account createAccount(const std::string& jsonStr) {
            Json::Value root;
            JSONCPP_STRING errs;
            Json::CharReaderBuilder reader;
    
            // 创建字符流解析器
            std::unique_ptr<Json::CharReader> jsonReader(reader.newCharReader());
            if(!jsonReader->parse(jsonStr.data(), jsonStr.data()+jsonStr.size(), &root, &errs)) {
                throw std::runtime_error("JSON解析失败: " + errs);
            }
    
            data::Account acc;
    
            // 提取字段
            if(root.isMember("uuid")) acc.uuid = root["uuid"].asString();
            if(root.isMember("name")) acc.name = root["name"].asString();
            if(root.isMember("password")) acc.password = root["password"].asString();
            if(root.isMember("nickname")) acc.nickname = root["nickname"].asString();
            if(root.isMember("icon")) acc.icon = root["icon"].asString();
            if(root.isMember("type")) acc.type = static_cast<data::Account::Type>(root["type"].asInt());
            if(root.isMember("balance")) acc.balance = root["balance"].asDouble();
            if(root.isMember("phone")) acc.phone = root["phone"].asString();
            if(root.isMember("level")) acc.level = static_cast<data::Account::Level>(root["level"].asInt());
    
            return acc;
        }

        static std::string toJson(data::Dish dish)
        {
            Json::Value root;
            root["uuid"] = dish.uuid;
            root["merchant_id"] = dish.merchant_id;
            root["merchant_name"] = dish.merchant_name;
            root["name"] = dish.name;
            root["icon_path"] = dish.icon_path;
            root["description"] = dish.description;
            root["base_price"] = dish.base_price;
            root["price_factor"] = dish.price_factor;
            root["is_delete"] = dish.is_delete;
    
            Json::StreamWriterBuilder writer;
            writer["indentation"] = "    ";
            return Json::writeString(writer, root);
        }

        static data::Dish createDish(const std::string& jsonStr) {
            Json::Value root;
            JSONCPP_STRING errs;
            Json::CharReaderBuilder reader;
            
            std::unique_ptr<Json::CharReader> jsonReader(reader.newCharReader());
            if(!jsonReader->parse(jsonStr.data(), jsonStr.data()+jsonStr.size(), &root, &errs)) {
                throw std::runtime_error("Dish解析失败: " + errs);
            }
    
            data::Dish dish;
            if(root.isMember("uuid")) dish.uuid = root["uuid"].asString();
            if(root.isMember("merchant_id")) dish.merchant_id = root["merchant_id"].asString();
            if(root.isMember("merchant_name")) dish.merchant_name = root["merchant_name"].asString();
            if(root.isMember("name")) dish.name = root["name"].asString();
            if(root.isMember("icon_path")) dish.icon_path = root["icon_path"].asString();
            if(root.isMember("description")) dish.description = root["description"].asString();
            if(root.isMember("base_price")) dish.base_price = root["base_price"].asDouble();
            if(root.isMember("price_factor")) dish.price_factor = root["price_factor"].asDouble();
            if(root.isMember("is_delete")) dish.is_delete = root["is_delete"].asBool();
    
            return dish;
        }

            //====== Order结构体 =⭐️=//
            static std::string toJson(data::Order order) {
                Json::Value root;
                root["merchant_id"] = order.merchant_id;
                root["merchant_name"] = order.merchant_name;
                root["consumer_id"] = order.consumer_id;
                root["consumer_name"] = order.consumer_name;
                root["time"] = order.time;
                root["level"] = static_cast<int>(order.level);
                root["pay"] = order.pay;
                root["uuid"] = order.uuid;
                root["status"] = static_cast<int>(order.status);
                root["sum"] = order.sum;

                Json::StreamWriterBuilder writer;
                writer["indentation"] = "    ";
                return Json::writeString(writer, root);
            }

            static data::Order createOrder(const std::string& jsonStr) {
            Json::Value root;
            JSONCPP_STRING errs;
            Json::CharReaderBuilder reader;
            
            std::unique_ptr<Json::CharReader> jsonReader(reader.newCharReader());
            if(!jsonReader->parse(jsonStr.data(), jsonStr.data()+jsonStr.size(), &root, &errs)) {
                throw std::runtime_error("Order解析失败: " + errs);
            }

            data::Order order;
            if(root.isMember("merchant_id")) order.merchant_id = root["merchant_id"].asString();
            if(root.isMember("merchant_name")) order.merchant_name = root["merchant_name"].asString();
            if(root.isMember("consumer_id")) order.consumer_id = root["consumer_id"].asString();
            if(root.isMember("consumer_name")) order.consumer_name = root["consumer_name"].asString();
            if(root.isMember("time")) order.time = root["time"].asString();
            if(root.isMember("level")) order.level = static_cast<data::Account::Level>(root["level"].asInt());
            if(root.isMember("pay")) order.pay = root["pay"].asDouble();
            if(root.isMember("uuid")) order.uuid = root["uuid"].asString();
            if(root.isMember("status")) order.status = static_cast<data::Order::Status>(root["status"].asInt());
            if(root.isMember("sum")) order.sum = root["sum"].asInt();

            return order;
        }

    //====== OrderDish结构体 ======//
    static std::string toJson(data::OrderDish orderDish) {
        Json::Value root;
        root["order_id"] = orderDish.order_id;
        root["dish_id"] = orderDish.dish_id;
        root["merchant_id"] = orderDish.merchant_id;
        root["name"] = orderDish.name;
        root["dish_price"] = orderDish.dish_price;
        root["count"] = orderDish.count;

        Json::StreamWriterBuilder writer;
        writer["indentation"] = "    ";
        return Json::writeString(writer, root);
    }

    static data::OrderDish createOrderDish(const std::string& jsonStr) {
        Json::Value root;
        JSONCPP_STRING errs;
        Json::CharReaderBuilder reader;
        
        std::unique_ptr<Json::CharReader> jsonReader(reader.newCharReader());
        if(!jsonReader->parse(jsonStr.data(), jsonStr.data()+jsonStr.size(), &root, &errs)) {
            throw std::runtime_error("OrderDish解析失败: " + errs);
        }

        data::OrderDish od;
        if(root.isMember("order_id")) od.order_id = root["order_id"].asString();
        if(root.isMember("dish_id")) od.dish_id = root["dish_id"].asString();
        if(root.isMember("merchant_id")) od.merchant_id = root["merchant_id"].asString();
        if(root.isMember("name")) od.name = root["name"].asString();
        if(root.isMember("dish_price")) od.dish_price = root["dish_price"].asDouble();
        if(root.isMember("count")) od.count = root["count"].asInt();

        return od;
    }

    }//json
}
```

## logger.hpp

## DatabaseClient.hpp
我们将所有与数据库相关的接口封装到`DatabaseClient`类中，在抽象层次中单独封装一层，为`HTTP服务层`提供简单易用的数据库服务。

特别的，在`v2.3`版本中我们引入了`守护线程`，增强了数据库连接的健壮性。具体改动如下

+ 为`DatabaseClient`配置了一个负责重连的异步守护线程
+ 完成了`DatabaseClient`和`HTTPServer`的解耦合
+ 为每一个数据库客户端接口的异常处理处加入了被动检测连接状态
+ 异步线程循环中加入了计数器，能够定时自动触发连接状态检测
+ 解决了查询订单列表时导致客户端崩溃的问题

*这边只给出头文件，具体实现的.cpp文件就不展示了*

> DatabaseClient.hpp
```C++
#pragma once

#include "CoreData.hpp"
#include "logger.hpp"

#include <cppconn/prepared_statement.h>
#include <cppconn/statement.h>
#include <cppconn/resultset.h>
#include <cppconn/exception.h>
#include <cppconn/metadata.h>
#include "mysql_driver.h"
#include <mysql_connection.h>
#include <sstream>
#include <string>
#include <vector>
#include <mutex>
#include <atomic>
#include <condition_variable>

using std::vector;
using std::string;

namespace btyGoose
{

class DatabaseClient
{
public:
	DatabaseClient();
	virtual ~DatabaseClient();
private:
	//QSqlDatabase db;
	string user = "";
	string password = "";
	string host = "127.0.0.1";
	string port = "3306";
	string database = "";

	sql::Connection* con;
	sql::Statement* stmt;

	std::mutex mtx;

private:
	//守护线程相关成员变量
	std::atomic<bool> connected_= false;	//表示连接状态
    std::chrono::seconds reconnect_interval_;//两次重连的最小间隔时间
    std::condition_variable reconnect_cv_;	//重连条件变量
    std::mutex reconnect_mtx_;		//重连锁
    std::thread reconnect_thread_;	//守护线程
    bool stop_reconnect_ = false;	//停止重连的标记

	void reconnectLoop();
	bool checkConnection();
public:
	void init(const string&_user,const string&_password,const string&_host,const string&_port,const string&_database,const std::chrono::seconds _reconnect_interval);

	void start();

	//Account CURD
	bool addAccount(const data::Account&);
	bool updateAccount(const data::Account&);
	data::Account searchAccountByID(const string& id);
	data::Account searchAccountByName(const string& name);
	data::Account searchAccountByPhone(const string& phone);

	bool delAccountByID(const string& id);

	//dish CURD
	bool addDish(const data::Dish&);
	bool updateDish(const data::Dish&);
	data::Dish searchDishByID(const string& id);
	vector<data::Dish> getAllDishList();
	vector<data::Dish> getDishListByMerchant(const string& id);
	bool delDishByID(const string& id);

	//Order CURD
	bool addOrder(const data::Order&order);
	bool updateOrder(const data::Order&order);
	bool updateOrder(const string&id ,const data::Order::Status stat);
	data::Order searchOrderByID(const string& order_id);
	vector<data::Order> getOrderListByMerchant(const string& merchant_id);
	vector<data::Order> getOrderListByMerchantWaiting(const string& merchant_id);
	vector<data::Order> getOrderListByConsumer(const string& consumer_id);
	vector<data::Order> getAllOrderList();
	bool delOrderByID(const string& order_id);
	int clearOverTimeOrder();

	//OrderDish CURD
	bool addOrderDishesByID(const string& order_id, const vector<data::OrderDish>& list);
	bool delOrderDishesByID(const string& order_id);
	vector<data::OrderDish> getOrderDishesByID(const string& order_id);
	
	//History CURD
	bool addHistory(const data::Order& order);
	vector<data::Order> getAllHistoryList();

	//HistoryDish CURD
	bool addHistoryDishesByID(const string& order_id, const vector<data::OrderDish>& list);
	vector<data::OrderDish> getHistoryDishesByID(const string& order_id);
};
}

```

## 编写Redis客户端

我们封装一个`btyGoose::RedisClient`类，专门负责向业务层提供服务和向Redis层请求服务

我们在这个版本中要实现的接口有:

> RedisClient.hpp
```C++
#pragma once
#include <sw/redis++/redis.h>
#include <unordered_set>
#include <vector>
#include <unordered_map>
#include <memory>
#include "CoreData.hpp"
#include "logger.hpp"

namespace btyGoose{

using std::string;
using std::shared_ptr;
using std::make_shared;
using std::vector;




class RedisClient
{
    //设定前缀
    const std::string account_prefix = "data:Account:";
    const std::string dish_prefix = "data:Dish:";
    const std::string order_prefix = "data:Order:";
    const std::string history_prefix = "data:History:";
    const std::chrono::seconds expire_time = std::chrono::seconds(300);
public:
    RedisClient(const string&ip,const uint16_t port,const uint16_t db,const bool keep_alive,const string&password)
    {
        LOG_INFO("即将连接Redis服务器,地址 {}:{}",ip,port);
        sw::redis::ConnectionOptions opts;
        opts.host = ip;
        opts.port = port;
        opts.db = db;
        opts.keep_alive = keep_alive;
        opts.password = password;

        _redis = make_shared<sw::redis::Redis>(sw::redis::Redis(opts));
    }

    

public:
    bool isOK()
    {
        try {
            auto pong = _redis->ping();
            if (pong == "PONG") {
                return true;
            }
        } catch (const sw::redis::Error &e) {
            LOG_WARN("Redis连接失败：{}",e.what());
            return false;
        }
        return false;
    }    

    void flushall()
    {
        _redis->flushall();
    }

    //订单相关
    void setOrder(const data::Order& order);
    void setOrderDishListJson(const string&order_id,const string& dish_list_json);
    string getOrderDishListJson(const string&order_id);
    data::Order getOrderById(const string& id);
    void setOrderList(const vector<data::Order> order_list);
    void setOrderListByIdDone(const string&id);
    vector<data::Order> getOrderListByMerchant(const string&id);
    vector<data::Order> getOrderListByMerchantWaiting(const string&id);
    vector<data::Order> getOrderListByConsumer(const string&id);
    bool hasOrderListByUserId(const string& id);
    void delOrderById(const string&id);

    

    //菜品相关
    data::Dish getDishById(const string&id);
    void delDishById(const string&id);
    string getDishListJsonByMerchant(const string&id);
    void setDishListJsonByMerchant(const string&id,const string&json);
    void delDishListJsonByMerchant(const string&id);    //缓存会有失效的时候
    void setDish(const data::Dish& dish);
    
    //账号相关
    data::Account getAccountByName(const string&name);
    data::Account getAccountById(const string&id);
    data::Account getAccountByPhone(const string&phone);
    void setAccount(const data::Account& acc);

private:
    shared_ptr<sw::redis::Redis> _redis;
};
}
```

当然，对于缓存的具体实现，有如下问题要考虑

### 如何存储结构化的数据
像`Order`,`Dish`这种结构化的数据，在Redis中我们可以选择如下比较合理的存储方式

1. 用哈希表存储
2. 序列化后存储

其中用哈希存储的灵活性较高，易修改，而序列化后存储则是数据的完整强，不会出现误修改的问题,因此我们得出如下策略

1. 对于`Dish`,`Account`这种简单，不易改变的数据，两种方法皆可，我们使用更为简单的哈希表存储
2. 对于`Order`这种容易动态改变的数据，我们使用哈希表存储
3. 对于`vector<Order>`这种与`Order`相关联的数据，我们使用`set`无序集合来存储它们的`key`
4. 对于`vector<OrderDish>`这种要求数据完整性强的，数据不会变动的类型，我们使用序列化后存储

以下是项目中的实际例子

```C++
//示例1
void RedisClient::setAccount(const data::Account& acc)
{
    unordered_map<string,string> m;

    m["uuid"] = acc.uuid;
    m["name"] = acc.name;
    m["password"] = acc.password;
    m["nickname"] = acc.nickname;
    m["icon"] = acc.icon;
    m["type"] = to_string(static_cast<int>(acc.type));
    m["balance"] = to_string(acc.balance);
    m["phone"] = acc.phone;
    m["level"] = to_string(static_cast<int>(acc.level));

    _redis->hmset(account_prefix+"hash:"+acc.name,m.begin(),m.end());
    _redis->set(account_prefix+"id:"+acc.uuid,account_prefix+acc.name);
    _redis->set(account_prefix+"phone:"+acc.phone,account_prefix+acc.name);
}

//示例2和示例4

void RedisClient::setOrder(const data::Order&order)
{
    unordered_map<string,string> m;

    m["merchant_id"] = order.merchant_id;
    m["merchant_name"] = order.merchant_name;
    m["consumer_id"] = order.consumer_id;
    m["consumer_name"] = order.consumer_name;
    m["time"] = order.time;
    m["level"] = to_string(static_cast<int>(order.level));
    m["pay"] = to_string(order.pay);
    m["uuid"] = order.uuid;
    m["status"] = to_string(static_cast<int>(order.status));
    m["sum"] = to_string(order.sum);

    _redis->hset(order_prefix+"id:"+order.uuid,m.begin(),m.end());
    _redis->sadd(order_prefix+"merchant:"+order.merchant_id,order.uuid);
    _redis->sadd(order_prefix+"consumer:"+order.consumer_id,order.uuid);
}

//示例3
void RedisClient::setOrderDishListJson(const string&order_id,const string& dish_list_json)
{
    _redis->set(order_prefix+"dishlist:"+order_id,dish_list_json);
}


```

### 缓存一致性问题
一旦引入缓存，缓存一致性问题就绕不开了。这里采用偏向于简单的解决方法

1. 对于一般的数据，我们会把增删改即时同步到数据库中
2. 对于`OrderList`这样的数据，我们采用额外的集合来标记有效的`key`,当发生增删改时，使`key`失效，对应的每次查询缓存时会先检查`key`是否有效,若失效，则重新从数据库加载，并标记`key`为有效
3. 对于**被序列化的数据**,我们在增删改的时候采用删除`key`的方式进行懒加载，等待到下次查询时发现缓存不命中，自动从数据库加载打破缓存

### 缓存淘汰策略
部分较大的数据被设置了存活时间，过期自动淘汰，它们有:

1. `dishListJsonByMerchant`
2. `orderListById`



## HTTPServer.h
这里就是实现服务器的主要业务功能的地方,有如下特点

+ `单例模式`:全局范围只运行一个`HTTPServer`实例
+ 提供工具函数： 这里也提供了一点序列化和反序列化函数

*这边只给出头文件，具体实现的.cpp文件就不展示了*

> HTTPServer.h
```C++
#pragma once
#include "httplib.h"
#include "DatabaseClient.hpp"
using std::string;
using std::vector;
namespace btyGoose
{

class HTTPServer
{
	class HTTPException
	{
	private:
		string message;
	public:
		HTTPException(const string& msg = "a problem")
			:message(msg)
		{}

		string what()const {
			return message;
		}
	};
private: 
	HTTPServer();//单例模式化
	HTTPServer(const HTTPServer&) = delete;
	HTTPServer operator=(const HTTPServer&) =delete;

	static HTTPServer* _ins;
public:
	static HTTPServer* getInstance()
	{
		if(_ins == nullptr)
		{
			_ins = new HTTPServer();
		}

		return _ins;
	}
	// void initDB(const string&_user,const string&_password,const string&_host,const string&_port,const string&_database,const std::chrono::seconds _reconnect_interval);
	void setDB(std::shared_ptr<DatabaseClient>& ptr);
	void init(const std::string&ip,const uint32_t port);

	void start();

	void initTestAPI();
	void initAccountAPI();
	bool AuthenticateAuthCode(const string& phone,const string&auth_code);	//这里留着接入第三方API
	void initConsumerAPI();
	void initMerchantAPI();
	void initAdminAPI();

	bool getPay(double pay);
	//工具函数
	string DishListToJsonArray(const vector<data::Dish>& dishList);
	vector<data::Dish> DishListFromJsonArray(const string& jsonString);

	string OrderListToJsonArray(const vector<data::Order>&orderList);
	vector<data::Order> OrderListFromJsonArray(const string& jsonString);

	string OrderDishListToJsonArray(const vector<data::OrderDish>&dishList);


private:
	httplib::Server svr;
	std::shared_ptr<DatabaseClient> db;
	string host;
	int port;
};
}



```

## main.cpp
`main`函数写在`main.cpp`中，并在这个文件中实现如下功能

+ 定义命令行参数
+ 解析命令行参数
+ 初始化各个对象
+ 运行服务端

```C++
#include "HTTPServer.hpp"
#include "logger.hpp"
#include <gflags/gflags.h>

DEFINE_string(host,"0.0.0.0","服务器监听的源ip地址范围");
DEFINE_uint32(port,80,"服务器的监听端口");

DEFINE_string(db_user,"root","数据库的连接用户");
DEFINE_string(db_password,"root_password","数据库用户的密码");
DEFINE_string(db_host,"localhost","数据库服务所在的ip");
DEFINE_string(db_port,"3306","数据库服务开放的端口");
DEFINE_string(db_database,"db","数据库中存储数据的数据库");
DEFINE_int32(db_reconnect_interval,10,"数据库两次重连的最小间隔秒数");

//Redis客户端相关参数
DEFINE_string(redis_host,"127.0.0.1","Redis服务器的ip地址");
DEFINE_uint32(redis_port,6379,"Redis服务器的端口");
DEFINE_uint32(redis_db,0,"Redis数据库的变化，默认0，范围[0,15]");
DEFINE_bool(redis_keep_alive,true,"是否保持长连接");
DEFINE_string(redis_master_password,"password","redis主节点的登录密码");

DEFINE_string(log_logfile,"./log/logfile","日志落地文件及路径");
DEFINE_bool(log_release_mode,false,"设置是否为发布模式输出日志,\ntrue:过滤低等级日志，输出到文件\nfalse:输出所有日志，输出到标准输出");
DEFINE_int32(log_release_level,spdlog::level::warn,"设置release模式下过滤的日志等级");

int main(int argc, char *argv[])
{
    google::ParseCommandLineFlags(&argc,&argv,true);//解析命令行参数
    btyGoose::init_logger(FLAGS_log_release_mode,FLAGS_log_logfile,FLAGS_log_release_level);//初始化日志器
    LOG_DEBUG("全局日志器初始化成功");
    auto db = std::make_shared<btyGoose::DatabaseClient>();
    db->init(FLAGS_db_user,FLAGS_db_password,FLAGS_db_host,FLAGS_db_port,FLAGS_db_database,                         
        std::chrono::seconds(FLAGS_db_reconnect_interval));
    btyGoose::HTTPServer::getInstance()->setDB(db);
    btyGoose::HTTPServer::getInstance()->init(FLAGS_host,FLAGS_port);
    btyGoose::HTTPServer::getInstance()->start();
    return 0;
}
```

其中为其准备的参数配置文件`server.conf`内容如下

```conf
-host=0.0.0.0
-port=80

-db_user=btyGooseUser
-db_password=Password!1
-db_host=db
-db_port=3306
-db_database=BeautyGoose
-db_reconnect_interval=10

-redis_host=redismaster
-redis_port=6379
-redis_db=0
-redis_keep_alive=true
-redis_master_password=redispass123

-log_logfile=./log/logfile
-log_release_mode=false
#warn等级为3
-log_release_level=3
```

## makefile 构建项目

```makefile
LDLIBS = -lmysqlcppconn -ljsoncpp -lpthread -lgflags -lfmt

server:main.o DatabaseClient.o HTTPServer.o logger.o
	g++ -o $@ *.o -std=c++17 $(LDLIBS)

server-static:main.o DatabaseClient.o HTTPServer.o
	g++ 

main.o: main.cpp
	g++ -c main.cpp -o main.o -std=c++17 

DatabaseClient.o:DatabaseClient.cpp 
	g++ -c DatabaseClient.cpp -o DatabaseClient.o -std=c++17 

HTTPServer.o:HTTPServer.cpp
	g++ -c HTTPServer.cpp -o HTTPServer.o -std=c++17 

logger.o:logger.cpp
	g++ -c logger.cpp -o logger.o -std=c++17 

.PHONY:clean
clean:
	rm -f server *.o
```

## Dockerfile编写

主要逻辑如下

+ 这里使用ubuntu:22.04作为基础镜像
+ 使用apt安装相关的编译环境和运行环境
+ 使用编译安装部分库
+ COPY源代码
+ 覆写配置文件
+ 编译和执行程序

```Dockerfile
FROM ubuntu:22.04 AS build1
RUN apt update -y && apt install -y g++ \
    make \
    git \
    cmake \
    libmysqlcppconn-dev \
    libjsoncpp-dev \
    libboost-all-dev\
    libgflags-dev \ 
    libfmt-dev \
    libspdlog-dev \
    libhiredis-dev 

WORKDIR /redispp
RUN git clone https://github.com/sewenew/redis-plus-plus.git
RUN mkdir -p /redispp/redis-plus-plus/build
WORKDIR /redispp/redis-plus-plus/build
RUN cmake ..
RUN make 
RUN make install
RUN ldconfig

COPY ./src* /src

WORKDIR /src
RUN make && mkdir logs
#DEBUG
# RUN make server_g && mkdir logs
#end
CMD ["/src/server","--flagfile=server.conf"]
```

# 客户端详细设计

## 版本信息
**当前版本**: v1.0正式版

当前版本的界面属于类似于毛坯房的只有UI结构和功能而没有美化，且所有的图片显示均尚未处理,均使用了QLabel进行占位

## 用户界面设计
在使用代码工具编写界面前，可以先使用作图工具设计（“画”）一个UI界面出来，这样子有助于将UI设计与代码开发有**一定程度的解耦合**。降低开发难度

### 界面流转流程图
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501201347364.png)

### 注册界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192048136.png)

1.	注册按钮： 点击请求注册
2.	转到用户名登录按钮： 点击转到用户名登录界面
3.	转到手机号登录按钮：点击转到手机号登录

### 用户名登录界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192050550.png)

1.	登录按钮： 点击请求登录
2.	转到注册按钮： 点击转到注册界面
3.	转到手机号登录按钮：点击转到手机号登录

### 手机号登录界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192113705.png)

1.	登录按钮： 点击请求登录
2.	转到注册按钮： 点击转到注册界面
3.	转到用户名登录按钮：点击转到用户名登录

### 账户优惠等级升级界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192139112.png)

1.	升级为VIP按钮：由普通成员升级为VIP
2.	升级为SVIP按钮：由普通成员升级为SVIP

### 消费者菜品列表界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192140617.png)

1.	导航栏按钮： 点击切换对应界面
2.	菜品样式图： 点击转到菜品详情窗口
3.	菜品名称：点击转到菜品详情窗口

### 消费者菜品详情窗口
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192124542.png)
1.	添加到购物车按钮：往购物车内添加一个菜品
2.	从购物车移除：从购物车内减少一个对应菜品

### 消费者购物车列表界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192141879.png)
1.	导航栏按钮： 点击切换对应界面
2.	生成订单：点击生成订单
3.	清空：点击清除购物车

### 消费者查看订单列表界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192141740.png)
1.	查看：点击跳转到查看订单界面
2.	订单状态：显示订单完成状态

### 消费者查看订单详情界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192140856.png)
1.	支付订单：点击转到第三方支付窗口
2.	取消订单： 取消生成的订单
3.  返回列表: 返回订单列表界面
4.	订单状态：显示订单完成状态

### 商家菜品列表界面  
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192144065.png)
1.	导航栏按钮： 点击切换对应界面
2.	菜品名称：点击转到菜品详情窗口

### 商家菜品注册界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192149989.png)
1.	菜品图片： 输入菜品图片的链接
2.	菜品名称：输入菜品名称
3.	菜品价格：输入菜品价格
4.	菜品简介：输入菜品简介
5.	注册菜品按钮：点击请求注册菜品

### 商家菜品详情窗口
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192156330.png)
1.	编辑菜品：点击跳转到菜品编辑窗口

### 商家菜品编辑窗口
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192157733.png)
1.	菜品图片： 输入菜品图片的链接
2.	菜品名称：输入菜品名称
3.	菜品价格：输入菜品价格
4.	菜品简介：输入菜品简介
5.	注册菜品按钮：点击请求保存菜品信息
6.  删除菜品按钮：点击删除菜品

### 商家订单列表界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192159376.png)
1.	导航栏按钮： 点击切换对应界面
2.	订单内容：点击转到订单详情窗口

### 商家订单详情窗口
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192159279.png)
1.	接单按钮： 接受订单
2.	拒单按钮：拒绝订单

### 管理员日志查看界面
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192200292.png)
1.	显示平台订单情况

### 第三方支付窗口
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192201256.png)
1.  支付按钮：点击模拟支付成功

## 类图设计
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501201231850.webp)

# 客户端实现

## 开发环境
因为当时遇到了些网络问题，所以下载了`Qt 6.7.3`

+ `开发平台:Windows 11 家庭中文版`
+ `QT版本 QT6.7.3`
+ `QtCreator版本 15.0.0(community)`

## 文件结构
由于C++提供的特性，对函数的实现可以有两大选择:

1. 在`.h`文件中**声明**和**实现**同时完成
2. 在`.h`中声明，在`.cpp`中实现

对于一些比较简单的对象，就比如下面的核心数据结构，我们采用在一个`.h`文件中声明和实现。

而对于比较复杂的对象，比如对`QT`的界面对象的封装，我们就采用**声明与实现分离**的方式

这样有个好处，就是有哪些成员变量和成员函数可以很清晰地在`.h`文件中找到，而如果要`创建/跳转到`实现，可以方便地使用快捷键:

+ `在.cpp文件中创建实现`: `alt + enter`
+ `跳转到定义`: `ctrl + 鼠标左键` / `F2`(这个可设置)
+ `指针返回到上一个位置`: `鼠标侧键` / `alt + 方向键←`

## 核心数据结构实现
我们需要一套核心的数据结构用于在数据库，服务端和客户端之间进行数据传输,所以将核心的数据结构在单独的头文件中实现，并且保持与服务端同步

同时有一些通用的工具函数写实现在了这个文件中:

+ `getFileName`根据路径获取文件名(用于打印日志)
+ `formatTime`根据时间戳获取格式化的时间文本
+ `getSecTime`获取秒级时间戳
+ `TAG`宏定义规定日志打印的前缀
+ `LOG()`宏定义打印日志用的函数

**关于命名空间**:为了防止命名冲突，这里使用了命名空间`btyGoose`，即项目名`BeautyGoose`的缩写

> CoreData.h  

```C++
#pragma once
#include <qstring.h>
#include <qbytearray.h>

#include <string>
#include <QString>
#include <QUuid>
#include <QDateTime>
#include <QFile>
#include <QDebug>
#include <QFileInfo>
#include <iostream>
#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
#include <QJsonValue>
#include <QFile>
#include <QIcon>
#include <qpixmap.h>
namespace btyGoose
{
namespace util
{
static inline QString getFileName(const QString& path)
{
    QFileInfo fileInfo(path);
    return fileInfo.fileName();
}

//防止函数重复定义,static 或者 inline 必有一个
//秒级时间戳转换格式化时间
static inline QString formatTime(int64_t timestamp)
{
    //传入的是秒级时间戳
    QDateTime dateTime = QDateTime::fromSecsSinceEpoch(timestamp);
    //把QDateTime对象转换成格式化时间
    return dateTime.toString("MM-dd HH:mm::ss");
}

//获取秒级时间戳
static inline int64_t getSecTime()
{
    return QDateTime::currentSecsSinceEpoch();
}

//封装一个"宏"作为打印日志的方式
#define TAG (QString("[%1 :%2][%3]").arg(btyGoose::util::getFileName(__FILE__),QString::number(__LINE__),btyGoose::util::formatTime(btyGoose::util::getSecTime())))
#define LOG() qDebug().noquote() << TAG



//把QByteArray转换成QIcon
static inline QIcon makeIcon(const QByteArray& byteArray)
{
    QPixmap pixmap;
    pixmap.loadFromData(byteArray);
    QIcon icon(pixmap);
    return icon;
}

static QString makeId(const QString& pre) {
    return pre + QUuid::createUuid().toString().sliced(25, 12);//截取一部分来提高可读性
}
}

/// <summary>
/// 定义核心数据结构
/// </summary>
namespace data
{
struct Account
{
    ///账户类型
    enum Type
    {
        CONSUMER,	//消费者账户
        MERCHANT,	//商家账户
        ADMIN		//管理员账户
    };
    enum Level
    {
        MEMBER,		//普通用户
        VIP,		//VIP用户
        SUVIP		//超级VIP用户
    };

    QString uuid = "";		//唯一id
    QString name = "";		//账户名
    QString password = "";	//密码
    QString nickname = "";	//昵称
    QByteArray icon = QByteArray();	//图标
    Type type = CONSUMER;			//账户类型
    double balance = 0;		//账户余额
    QString phone = "";		//电话号码
    Level level = MEMBER;		//优惠等级

    QString toJson() const
    {
        QJsonObject jsonObj;

        // 填充基本字段
        jsonObj["uuid"] = uuid;
        jsonObj["name"] = name;
        jsonObj["password"] = password;
        jsonObj["nickname"] = nickname;
        jsonObj["icon"] = QString(icon.toBase64()); // 图标可能是二进制数据，使用Base64编码
        jsonObj["balance"] = balance;
        jsonObj["phone"] = phone;

        // 填充类型字段，转换枚举类型为字符串
        jsonObj["type"] = (int)type;
        jsonObj["level"] = (int)level;

        // 转换为 JSON 字符串并返回
        QJsonDocument doc(jsonObj);
        return QString(doc.toJson(QJsonDocument::Compact));
    }

    bool loadFromJson(const std::string& jsonStr)
    {
        QString qJsonString = QString::fromStdString(jsonStr);

        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            QJsonObject jsonObj = doc.object();

            // 从 JSON 中加载数据到结构体成员
            if (jsonObj.contains("uuid") && jsonObj["uuid"].isString()) {
                uuid = jsonObj["uuid"].toString();
            }

            if (jsonObj.contains("name") && jsonObj["name"].isString()) {
                name = jsonObj["name"].toString();
            }

            if (jsonObj.contains("password") && jsonObj["password"].isString()) {
                password = jsonObj["password"].toString();
            }

            if (jsonObj.contains("nickname") && jsonObj["nickname"].isString()) {
                nickname = jsonObj["nickname"].toString();
            }

            if (jsonObj.contains("icon") && jsonObj["icon"].isString()) {
                icon = QByteArray::fromBase64(jsonObj["icon"].toString().toUtf8());
            }

            if (jsonObj.contains("type") && jsonObj["type"].isDouble()) {
                type = (Type)jsonObj["type"].toInt();
            }

            if (jsonObj.contains("balance") && jsonObj["balance"].isDouble()) {
                balance = jsonObj["balance"].toDouble();
            }

            if (jsonObj.contains("phone") && jsonObj["phone"].isString()) {
                phone = jsonObj["phone"].toString();
            }

            if (jsonObj.contains("level") && jsonObj["level"].isDouble()) {
                level = (Level)jsonObj["level"].toInt();
            }


            return true;
        }
        else
        {
            qDebug() << "Invalid Json string : " << jsonStr;
            return false;
        }
    }


};

struct Dish
{
    QString uuid = "";			//菜品id
    QString merchant_id = "";	//商家uuid
    QString merchant_name = "";	//商家uuid
    QString name = "";			//菜品名称
    QString icon_path = "";		//网络图片的url
    QString description = "";	//菜品的描述
    double base_price = 0;		//基础价格
    double price_factor = 1;	//价格影响因素
    bool is_delete = false;		//是否被删除

    QString toJson() const
    {
        QJsonObject jsonObj;

        // 填充基本字段
        jsonObj["uuid"] = uuid;
        jsonObj["merchant_id"] = merchant_id;
        jsonObj["merchant_name"] = merchant_name;
        jsonObj["name"] = name;
        jsonObj["icon_path"] = icon_path;
        jsonObj["description"] = description;
        jsonObj["base_price"] = base_price;
        jsonObj["price_factor"] = price_factor;
        jsonObj["is_delete"] = is_delete;

        // 将 QJsonObject 转换为 JSON 字符串
        QJsonDocument doc(jsonObj);
        return QString(doc.toJson(QJsonDocument::Compact));
    }

    bool loadFromJson(const std::string& jsonStr)
    {
        QString qJsonString = QString::fromStdString(jsonStr);

        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            QJsonObject jsonObj = doc.object();

            // 从 JSON 中加载数据到成员变量
            if (jsonObj.contains("uuid") && jsonObj["uuid"].isString()) {
                uuid = jsonObj["uuid"].toString();
            }

            if (jsonObj.contains("merchant_id") && jsonObj["merchant_id"].isString()) {
                merchant_id = jsonObj["merchant_id"].toString();

            }
            if (jsonObj.contains("merchant_name") && jsonObj["merchant_name"].isString()) {
                merchant_name = jsonObj["merchant_name"].toString();
            }

            if (jsonObj.contains("name") && jsonObj["name"].isString()) {
                name = jsonObj["name"].toString();
            }

            if (jsonObj.contains("icon_path") && jsonObj["icon_path"].isString()) {
                icon_path = jsonObj["icon_path"].toString();
            }

            if (jsonObj.contains("description") && jsonObj["description"].isString()) {
                description = jsonObj["description"].toString();
            }

            if (jsonObj.contains("base_price") && jsonObj["base_price"].isDouble()) {
                base_price = jsonObj["base_price"].toDouble();
            }

            if (jsonObj.contains("price_factor") && jsonObj["price_factor"].isDouble()) {
                price_factor = jsonObj["price_factor"].toDouble();
            }


            if (jsonObj.contains("is_delete") && jsonObj["is_delete"].isBool()) {
                is_delete = jsonObj["is_delete"].toBool();
            }
            return true;

        }
        else
        {
            qDebug() << "Invalid Json string : " << jsonStr;
            return false;
        }
    }

};

struct Order
{
    enum Status
    {
        UNPAYED,		//未支付状态
        WAITING,		//等待商家处理
        OVER_TIME,		//订单超时
        REJECTED,		//商家拒收
        SUCCESS,		//成功完成
        ERR,			//订单出错
        FATAL,			//订单出现致命错误
        CANCELED		//消费者取消订单
    };

    QString merchant_id;		//商家id
    QString merchant_name;		//商家账户名称
    QString consumer_id;		//消费者id
    QString consumer_name;		//消费者账户名称
    QString time;				//订单时间
    Account::Level level;		//优惠等级
    double pay;					//订单价格
    QString uuid;				//订单id
    Status status;				//订单状态
    int sum = 0;				//订单内总菜品数

    QString toJson() const
    {
        QJsonObject jsonObj;

        // 填充字段
        jsonObj["merchant_id"] = merchant_id;
        jsonObj["merchant_name"] = merchant_name;
        jsonObj["consumer_id"] = consumer_id;
        jsonObj["consumer_name"] = consumer_name;
        jsonObj["time"] = time;
        jsonObj["pay"] = pay;
        jsonObj["uuid"] = uuid;
        jsonObj["sum"] = sum;

        // 转换枚举类型为字符串
        jsonObj["status"] = (int)status;

        // 转换 Account::Level 枚举为字符串
        jsonObj["level"] = (int)level;

        // 将 QJsonObject 转换为 JSON 字符串
        QJsonDocument doc(jsonObj);
        return QString(doc.toJson(QJsonDocument::Compact));
    }

    bool loadFromJson(const std::string& jsonStr)
    {
        QString qJsonString = QString::fromStdString(jsonStr);

        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            QJsonObject jsonObj = doc.object();

            // 从 JSON 中加载数据到成员变量
            if (jsonObj.contains("merchant_id") && jsonObj["merchant_id"].isString()) {
                merchant_id = jsonObj["merchant_id"].toString();
            }

            if (jsonObj.contains("merchant_name") && jsonObj["merchant_name"].isString()) {
                merchant_name = jsonObj["merchant_name"].toString();
            }

            if (jsonObj.contains("consumer_id") && jsonObj["consumer_id"].isString()) {
                consumer_id = jsonObj["consumer_id"].toString();
            }

            if (jsonObj.contains("consumer_name") && jsonObj["consumer_name"].isString()) {
                consumer_name = jsonObj["consumer_name"].toString();
            }

            if (jsonObj.contains("time") && jsonObj["time"].isString()) {
                time = jsonObj["time"].toString();
            }

            if (jsonObj.contains("level") && jsonObj["level"].isDouble()) {
                level = (Account::Level)jsonObj["level"].toInt();
            }

            if (jsonObj.contains("pay") && jsonObj["pay"].isDouble()) {
                pay = jsonObj["pay"].toDouble();
            }

            if (jsonObj.contains("uuid") && jsonObj["uuid"].isString()) {
                uuid = jsonObj["uuid"].toString();
            }

            if (jsonObj.contains("status") && jsonObj["status"].isDouble()) {
                status = static_cast<Status>(jsonObj["status"].toInt());
            }

            if (jsonObj.contains("sum") && jsonObj["sum"].isDouble()) {
                sum = jsonObj["sum"].toInt();
            }

            return true;

        }
        else
        {
            qDebug() << "Invalid Json string : " << jsonStr;
            return false;
        }
    }
};

struct OrderDish
{
    QString order_id = "";		//订单id
    QString dish_id = "";		//菜品id
    QString merchant_id = "";	//商家id
    QString name = "";			//菜品id
    double dish_price = 0;		//当时的基础价格
    int count = 0;				//菜品数量

    // 将结构体转换为JSON字符串
    QString toJson()
    {
        QJsonObject jsonObj;

        // 使用operator[]插入各个字段到JSON对象中
        jsonObj["order_id"] = order_id;
        jsonObj["dish_id"] = dish_id;
        jsonObj["merchant_id"] = merchant_id;
        jsonObj["name"] = name;
        jsonObj["dish_price"] = dish_price;
        jsonObj["count"] = count;

        // 使用QJsonDocument将QJsonObject转为JSON字符串
        QJsonDocument doc(jsonObj);
        return QString(doc.toJson(QJsonDocument::Compact));  // 返回紧凑的JSON字符串
    }

    // 从JSON字符串加载数据
    bool loadFromJson(const std::string& jsonStr)
    {
        // 将std::string转换为QString
        QString jsonString = QString::fromStdString(jsonStr);

        // 解析JSON字符串为QJsonDocument
        QJsonDocument doc = QJsonDocument::fromJson(jsonString.toUtf8());

        // 如果JSON解析失败，返回false
        if (!doc.isObject())
        {
            qDebug() << "Invalid JSON format.";
            return false;
        }

        // 获取QJsonObject
        QJsonObject jsonObj = doc.object();

        // 检查每个字段是否存在且类型正确，如果存在，则赋值
        if (jsonObj.contains("order_id") && jsonObj["order_id"].isString()) {
            order_id = jsonObj["order_id"].toString();
        }

        if (jsonObj.contains("dish_id") && jsonObj["dish_id"].isString()) {
            dish_id = jsonObj["dish_id"].toString();
        }

        if (jsonObj.contains("merchant_id") && jsonObj["merchant_id"].isString()) {
            merchant_id = jsonObj["merchant_id"].toString();
        }

        if (jsonObj.contains("name") && jsonObj["name"].isString()) {
            name = jsonObj["name"].toString();
        }

        if (jsonObj.contains("dish_price") && jsonObj["dish_price"].isDouble()) {
            dish_price = jsonObj["dish_price"].toDouble();
        }


        if (jsonObj.contains("count") && jsonObj["count"].isDouble()) {
            count = jsonObj["count"].toInt();  // 由于JSON可能存储为double类型，使用toInt()转换
        }

        return true;  // 成功加载
    }
};
};
}
```

## MainWidget界面流转实现
本项目的界面流转方案有如下要点:

1. `静态页面`:页面在生成后基本不会自动发生变化，除非触发刷新**重新生成**页面
2. `临时性`: 所有的页面都属于临时生成，在切换时原本的界面直接删除，释放内存，不作保留
3. `自动生成`: `MainWidget`在控制界面的生成时，仅负责传入数据，具体的界面生成由界面类的**构造函数自动生成**
4. `信号槽机制通信`: `MainWidget`与前台的界面对象的交互仅有**界面对象发信号**和`MainWidget`接收信号并执行槽函数，因此每一次生成界面时，都要连接界面对象的信号和自己的槽函数

对于界面流转，我们重点实现两个函数:

1. 构造函数: 为界面切换准备
2. `clearAll()函数`： 清除界面内容

*下面仅展示部分代码*

```C++
MainWidget::MainWidget(QWidget *parent)
    : QWidget(parent)
    , ui(new Ui::MainWidget)
{
    ui->setupUi(this);
    this->setFixedSize(400,600);
    this->setWindowIcon(QIcon(QPixmap(":/qsrc/icon.png")));
    this->setWindowTitle("美鹅外卖");
    //设置全局样式
    if(qApp)
    {
        QApplication* app = qApp;
        app->setFont(QFont("微软雅黑",8));
    }
    this->setStyleSheet(QString("QWidget{background-color: %1}").arg(ColorConfig::getInstance()->background_color));
    QGridLayout* layout = new QGridLayout;
    this->setLayout(layout);
    //初始化信号连接
    initAccountResponseConnection();
    initConsumerResponeConnection();
    initMerchantResponseConnection();
    initAdminResponseConnection();
    // toRegisterSlot();
    toNameLoginSlot();      //转到用户名登录界面
}

void MainWidget::clearAll()
{
    QLayout* layout = this->layout();
    while(layout->count())
    {
        QLayoutItem* obj=layout->takeAt(0);
        if(obj->widget())
        {
            delete obj->widget();
        }
    }
}
```


## 注册界面实现
目前注册界面为简单实现，对输入的合法性判断较少，主要使用了如下QT类

+ `<QRadioButton>`提供了单选按钮,用于选择账户类型
+ `<QLineEdit>`用于输入单行文本，也提供了密码输入模式
+ `<QPushButton>`提供了可点击的按钮
+ `<QTextEdit>`用于输入多行文本，这里属于误用，使用`QLineEdit`更合适
+ `<QMessageBox>`用于简单地通过文本弹窗的方式输出提示信息
+ `<QRegularExpression>,<QRegularExpressionValidator>`提供正则表达式对手机号码等输入进行合法性判断
+ `<QGridLayout>`表格布局，用于较为简单地使用QT的布局功能

> registerwidget.h
```C++
#ifndef REGISTERWIDGET_H
#define REGISTERWIDGET_H

#include <QWidget>
#include <QRadioButton>
#include <QTextEdit>
#include <QPushButton>
#include <QLineEdit>
class RegisterWidget : public QWidget
{
    Q_OBJECT
public:
    explicit RegisterWidget(QWidget *parent = nullptr);

    struct Input
    {
        QString type;
        QString name;
        QString nickname;
        QString password;
        QString phone;
        QString auth_code;
    };

private:
    QRadioButton* consumerRadio;
    QRadioButton* merchantRadio;
    QTextEdit* nameInput;
    QTextEdit* nicknameInput;
    QLineEdit* passwordInput;
    QLineEdit* passwordRepeatInput;
    QTextEdit* phoneInput;
    QTextEdit* auth_codeInput;

    QPushButton* registerBtn;
    QPushButton* getCodeBtn;
    QPushButton* toNameLoginBtn;
    QPushButton* toPhoneLoginBtn;

    bool check();
public slots:
    void registerSlot();
    void getCodeSlot();
    void toNameSlot();
    void toPhoneSlot();
signals:
    void toNameLoginSignal();
    void toPhoneLoginSignal();
    void authCodeSignal();
    void registerSiganl(Input input);
};

#endif // REGISTERWIDGET_H
```

> registerwidget.cpp 
```C++
#include "registerwidget.h"
#include "QGridLayout"
#include "ColorConfig.h"
#include <QLabel>
#include <QMessageBox>
#include <QRegularExpression>
#include <QRegularExpressionValidator>
RegisterWidget::RegisterWidget(QWidget *parent)
    : QWidget{parent}
{
    auto color = ColorConfig::getInstance();
    this->setFixedSize(400,600);
    QGridLayout* layout = new QGridLayout;

    this->setStyleSheet(QString("QLabel{"
                                "background-color: %1;"
                                "border-radius: 10px;"
                                "border: 2px solid %2;"
                                "font-size: 15px;"
                                "font-weight: 500;"
                                "font-family: 'Microsoft YaHei';}").arg(color->main_color,color->main_color));

    int m = 10;
    layout->setContentsMargins(m,m,m,m);
    //第一行
    QLabel* label = new QLabel;
    label->setText("账户类型");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,0,0);

    QWidget* typeW = new QWidget;
    // typeW->setStyleSheet("QWidget{border:1px solid black}");
    typeW->setFixedHeight(50);
    layout->addWidget(typeW,0,1,1,4);

    QGridLayout* typeLayout = new QGridLayout;
    typeLayout->setSpacing(0);
    typeW->setLayout(typeLayout);

    consumerRadio = new QRadioButton;
    consumerRadio->setChecked(true);
    consumerRadio->setFixedSize(25,25);
    typeLayout->addWidget(consumerRadio,0,1);

    label = new QLabel ;
    label->setText("我是消费者");
    label->setAlignment(Qt::AlignCenter);//文字居中
    typeLayout->addWidget(label,0,2);
    merchantRadio = new QRadioButton;
    merchantRadio->setFixedSize(25,25);
    typeLayout->addWidget(merchantRadio,0,3);
    label = new QLabel;
    label->setText("我是商家");
    label->setAlignment(Qt::AlignCenter);//文字居中
    typeLayout->addWidget(label,0,4);

    //第二三四五六七八行
    label = new QLabel("用户名");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,1,0);
    nameInput = new QLineEdit;
    nameInput->setFixedHeight(50);
    layout->addWidget(nameInput,1,1,1,4);
    label = new QLabel("昵称");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,2,0);
    nicknameInput = new QLineEdit;
        nicknameInput->setFixedHeight(50);
    layout->addWidget(nicknameInput,2,1,1,4);
    label = new QLabel("密码");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,3,0);
    passwordInput = new QLineEdit;
        passwordInput->setFixedHeight(50);
    passwordInput->setEchoMode(QLineEdit::Password);
    layout->addWidget(passwordInput,3,1,1,4);
    label = new QLabel("重复密码");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,4,0);
    passwordRepeatInput = new QLineEdit;
        passwordRepeatInput->setFixedHeight(50);
        passwordRepeatInput->setEchoMode(QLineEdit::Password);
    layout->addWidget(passwordRepeatInput,4,1,1,4);
    label = new QLabel("手机号");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,5,0);
    phoneInput = new QLineEdit;
        phoneInput->setFixedHeight(50);
    layout->addWidget(phoneInput,5,1,1,4);
    label = new QLabel("短信验证码");
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(label,6,0);
    auth_codeInput = new QLineEdit;
        auth_codeInput->setFixedHeight(50);
    auth_codeInput->setFixedWidth(100);
    layout->addWidget(auth_codeInput,6,1);

    getCodeBtn = new QPushButton("获取验证码");
    label->setAlignment(Qt::AlignCenter);//文字居中
    getCodeBtn->setFixedHeight(50);
    layout->addWidget(getCodeBtn,6,2);

    registerBtn = new QPushButton("注册");
    label->setAlignment(Qt::AlignCenter);//文字居中

    registerBtn->setFixedHeight(50);
    layout->addWidget(registerBtn,7,1,1,4);

    toNameLoginBtn = new QPushButton("转到用户名登录");
    toNameLoginBtn->setStyleSheet(QString("QPushButton {"
                                  "background-color: %1;"  // 设置背景色
                                  "color: %2;"
                                  "border: 0;}").arg(color->background_color,color->grey));
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(toNameLoginBtn,8,2,1,2);
    toPhoneLoginBtn = new QPushButton("转到手机号登录");
    toPhoneLoginBtn->setStyleSheet(QString("QPushButton {"
                                          "background-color: %1;"  // 设置背景色
                                          "color: %2;"
                                          "border: 0;}").arg(color->background_color,color->grey));
    label->setAlignment(Qt::AlignCenter);//文字居中
    layout->addWidget(toPhoneLoginBtn,8,4,1,2);


    this->setLayout(layout);

    connect(registerBtn,&QPushButton::clicked,this,&RegisterWidget::registerSlot);
    connect(getCodeBtn,&QPushButton::clicked,this,&RegisterWidget::getCodeSlot);
    connect(toNameLoginBtn,&QPushButton::clicked,this,&RegisterWidget::toNameLoginSignal);
    connect(toPhoneLoginBtn,&QPushButton::clicked,this,&RegisterWidget::toPhoneLoginSignal);
}

bool RegisterWidget::check()
{
    QString name = nameInput->text();
    if(name.isEmpty())
    {
        QMessageBox::information(this, "Info", "用户名不能为空！");
        return false;
    }
    QString password = passwordInput->text();
    if(password.size()<8)
    {
        QMessageBox::information(this, "Info", "密码不能小于8位！");
        return false;
    }

    QString passwordR = passwordRepeatInput->text();
    if(password != passwordR)
    {
        QMessageBox::information(this, "Info", "两次i密码不一样");
        return false;
    }
    QRegularExpression regex("^[1][3-9][0-9]{9}$");
    QString phone = phoneInput->text();
    if (regex.match(phone).hasMatch() == false)
    {
        QMessageBox::information(this, "Info", "非法手机号！");
        return false;
    }
    return true;
}

void RegisterWidget::registerSlot()
{
    if(!check())
    {
        return;
    }
    QString code = auth_codeInput->text();
    if(code.size() != 6)
    {
        QMessageBox::information(this, "Info", "请输入六位验证码！");
        return;
    }

    Input input;
    if(consumerRadio->isChecked())
    {
        input.type = "CONSUMER";
    }
    else
    {
        input.type = "MERCHANT";
    }
    input.name = nameInput->text();
    QString nickname = nicknameInput->text();
    if(nickname.isEmpty())
    {
        input.nickname = input.name;
    }
    else
    {
        input.nickname = nickname;
    }
    input.password = passwordInput->text();
    input.phone = phoneInput->text();
    input.auth_code = auth_codeInput->text();

    emit registerSiganl(input);
}

void RegisterWidget::getCodeSlot()
{
    if(check())
    {
        emit authCodeSignal();
    }
}

void RegisterWidget::toNameSlot()
{
    emit toNameLoginSignal();
}

void RegisterWidget::toPhoneSlot()
{
    emit toPhoneLoginSignal();
}


```

实际效果如下:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231513388.webp)

## 导航栏实现
参照前面的界面设计图我们可以看到很多界面都有个侧导航栏是一样的，因此我们大可以继承`QWidget`封装一个导航栏控件类。


![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501192140617.png)

因为单纯的导航栏控件比较简单，所以我们只使用一个`Nav.h`头文件

由于`消费者`和`商家`的导航栏切换到的是不同的界面，所以我们这边简单处理，单独实现两种导航栏类

```C++
#ifndef NAV_H
#define NAV_H
#include <QWidget>
#include <QPushButton>
#include <QGridLayout>
#include <QPainter>
#include <QStyleOption>
#include <QStyle>
#include <QDebug>
#include <QStyleFactory>

#include "ColorConfig.h"
#include "datacenter.h"
class ConsumerNavWidget:public QWidget
{
    Q_OBJECT
public:

    //导航栏切换界面的按钮
    QPushButton* viewUserInfo;
    QPushButton* viewDishList;
    QPushButton* viewOrderList;
    QPushButton* viewCartList;
    QPushButton* viewVIP;
    void paintEvent(QPaintEvent* event) override
    {
        (void)event;
        QStyleOption opt;
        opt.initFrom(this);
        QPainter p(this);
        style()->drawPrimitive(QStyle::PE_Widget,&opt,&p,this);
    }
    ConsumerNavWidget()
    {
        this->setFixedWidth(150);
        this->setFixedHeight(800);
        auto color = ColorConfig::getInstance();
        QGridLayout* mainLayout = new QGridLayout;
        QWidget* mainW = new QWidget;
        this->setLayout(mainLayout);
        mainLayout->addWidget(mainW);
        mainW->setObjectName("Nav");
        // mainW->setStyleSheet(QString("QPushButton { background-color: %1;}").arg(color->background_color));
        mainW->setStyleSheet("QWidget#Nav { border: 2px solid black;}");
        QGridLayout* layout = new QGridLayout;
        mainW->setLayout(layout);
        layout->setAlignment(Qt::AlignTop);

        //个人信息
        QWidget* widget = new QWidget;
        QGridLayout* lay = new QGridLayout;
        lay->setAlignment(Qt::AlignHCenter);
        // widget->setFixedHeight(125);
        widget->setLayout(lay);
        viewUserInfo = new QPushButton("查看个人信息");
        QPushButton* userInfoIcon = new QPushButton;
        userInfoIcon->setIcon(QIcon(QPixmap("://qsrc/avatar.png")));
        userInfoIcon->setIconSize(QSize(100,100));
        viewUserInfo->setStyleSheet("QPushButton { text-align: center; }");
        userInfoIcon->setFixedSize(100,100);
        viewUserInfo->setFixedSize(100,25);
        lay->addWidget(userInfoIcon,0,0);
        lay->addWidget(viewUserInfo,1,0);
        lay->setSpacing(0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget);



        viewDishList = new QPushButton("查看菜品列表");
        widget = new QWidget;
        // widget->setFixedHeight(100);
        lay = new QGridLayout;
        widget->setLayout(lay);
        QPushButton* dishListIcon = new QPushButton;
        dishListIcon->setIcon(QIcon(QPixmap("://qsrc/dishList.png")));
        dishListIcon->setIconSize(QSize(75,75));
        viewDishList->setStyleSheet("QPushButton { text-align: center; }");
        dishListIcon->setFixedSize(75,75);
        viewDishList->setFixedSize(100,25);
        lay->addWidget(dishListIcon,0,0);
        lay->addWidget(viewDishList,1,0);
        lay->setSpacing(0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget);





        viewCartList = new QPushButton("查看购物车列表");
        widget = new QWidget;
        // widget->setFixedHeight(100);
        lay = new QGridLayout;
        widget->setLayout(lay);
        QPushButton* cartListIcon = new QPushButton;
        cartListIcon->setIcon(QIcon(QPixmap("://qsrc/cart.png")));
        cartListIcon->setIconSize(QSize(75,75));
        viewCartList->setStyleSheet("QPushButton { text-align: center; }");
        cartListIcon->setFixedSize(75,75);
        viewCartList->setFixedSize(100,25);
        lay->addWidget(cartListIcon,0,0);
        lay->addWidget(viewCartList,1,0);
        lay->setSpacing(0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget);




        viewOrderList = new QPushButton("查看订单列表");
        widget = new QWidget;
        // widget->setFixedHeight(100);
        lay = new QGridLayout;
        widget->setLayout(lay);
        QPushButton* orderListIcon = new QPushButton;
        orderListIcon->setIcon(QIcon(QPixmap("://qsrc/order.png")));
        orderListIcon->setIconSize(QSize(75,75));
        viewOrderList->setStyleSheet("QPushButton { text-align: center; }");
        orderListIcon->setFixedSize(75,75);
        viewOrderList->setFixedSize(100,25);
        lay->addWidget(orderListIcon,0,0);
        lay->addWidget(viewOrderList,1,0);
        lay->setSpacing(0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget);


        viewVIP = new QPushButton("VIP");
        layout->addWidget(viewVIP);


        // layout->addWidget(viewUserInfo,0,0);
        // layout->addWidget(viewDishList,1,0);
        // layout->addWidget(viewCartList,2,0);
        // layout->addWidget(viewOrderList,3,0);
        // layout->addWidget(viewVIP,4,0);

        connect(viewUserInfo,&QPushButton::clicked,this,&ConsumerNavWidget::toUserInfoSlot);
        connect(userInfoIcon,&QPushButton::clicked,this,&ConsumerNavWidget::toUserInfoSlot);
        connect(viewDishList,&QPushButton::clicked,this,&ConsumerNavWidget::toDishListSlot);
        connect(dishListIcon,&QPushButton::clicked,this,&ConsumerNavWidget::toDishListSlot);
        connect(viewCartList,&QPushButton::clicked,this,&ConsumerNavWidget::toCartListSlot);
        connect(cartListIcon,&QPushButton::clicked,this,&ConsumerNavWidget::toCartListSlot);
        connect(viewOrderList,&QPushButton::clicked,this,&ConsumerNavWidget::toOrderListSlot);
        connect(orderListIcon,&QPushButton::clicked,this,&ConsumerNavWidget::toOrderListSlot);
        connect(viewVIP,&QPushButton::clicked,this,&ConsumerNavWidget::toVIPSlot);
    }
public slots:
    void toUserInfoSlot(){emit toUesrInfoSignal();}
    void toDishListSlot(){emit toDishListSignal();}
    void toCartListSlot(){emit toCartListSignal();}
    void toOrderListSlot(){emit toOrderListSignal();}
    void toVIPSlot(){emit toVIPSignal();}

signals:
    void toUesrInfoSignal();
    void toDishListSignal();
    void toCartListSignal();
    void toOrderListSignal();
    void toVIPSignal();
};

class MerchantNavWidget:public QWidget
{
    Q_OBJECT
public:
    //导航栏切换界面的按钮
    QPushButton* viewUserInfo;
    QPushButton* viewDishList;
    QPushButton* viewOrderList;
    QPushButton* dishRegister;

    void paintEvent(QPaintEvent* event) override
    {
        (void)event;
        QStyleOption opt;
        opt.initFrom(this);
        QPainter p(this);
        style()->drawPrimitive(QStyle::PE_Widget,&opt,&p,this);
    }

    MerchantNavWidget()
    {
        this->setFixedWidth(150);
        this->setFixedHeight(800);
        QGridLayout* mainLayout = new QGridLayout;
        QWidget* mainW = new QWidget;
        this->setLayout(mainLayout);
        mainLayout->addWidget(mainW);
        mainW->setStyleSheet("QWidget { border: 2px solid black;}");
        QGridLayout* layout = new QGridLayout;
        mainW->setLayout(layout);

        QGridLayout* lay = nullptr;
        QWidget* widget = nullptr;

        viewUserInfo = new QPushButton("查看个人信息");
        viewUserInfo->setFixedSize(100,25);
        viewUserInfo->setStyleSheet("QPushButton{text-align:center;}");
        QPushButton* userInfoBtn = new QPushButton;
        // QIcon icon(QPixmap(DataCenter::getInstance()->account->icon));
        userInfoBtn->setIcon(QIcon(QPixmap("://qsrc/avatar.png")));
        userInfoBtn->setIconSize(QSize(100,100));
        widget = new QWidget;
        lay = new QGridLayout;
        widget->setLayout(lay);
        lay->setSpacing(0);
        lay->addWidget(userInfoBtn,0,0);
        lay->addWidget(viewUserInfo,1,0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget,0,0);

        viewDishList = new QPushButton("查看菜品列表");
        viewDishList->setFixedSize(100,25);
        viewDishList->setStyleSheet("QPushButton{text-align:center;}");
        QPushButton* dishListBtn = new QPushButton;
        dishListBtn->setIcon(QIcon(QPixmap("://qsrc/dishList.png")));
        dishListBtn->setIconSize(QSize(100,100));
        widget = new QWidget;
        lay = new QGridLayout;
        widget->setLayout(lay);
        lay->setSpacing(0);
        lay->addWidget(dishListBtn,0,0);
        lay->addWidget(viewDishList,1,0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget,1,0);

        viewOrderList = new QPushButton("查看订单列表");
        viewOrderList->setFixedSize(100,25);
        viewOrderList->setStyleSheet("QPushButton{text-align:center;}");
        QPushButton* orderListBtn = new QPushButton;
        orderListBtn->setIcon(QIcon(QPixmap("://qsrc/order.png")));
        orderListBtn->setIconSize(QSize(100,100));
        widget = new QWidget;
        lay = new QGridLayout;
        widget->setLayout(lay);
        lay->setSpacing(0);
        lay->addWidget(orderListBtn,0,0);
        lay->addWidget(viewOrderList,1,0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget,2,0);

        dishRegister = new QPushButton("注册菜品");
        dishRegister->setFixedSize(100,25);
        dishRegister->setStyleSheet("QPushButton{text-align:center;}");
        QPushButton* dishRegisterBtn = new QPushButton;
        dishRegisterBtn->setIcon(QIcon(QPixmap("://qsrc/edit.png")));
        dishRegisterBtn->setIconSize(QSize(100,100));
        widget = new QWidget;
        lay = new QGridLayout;
        widget->setLayout(lay);
        lay->setSpacing(0);
        lay->addWidget(dishRegisterBtn,0,0);
        lay->addWidget(dishRegister,1,0);
        lay->setAlignment(Qt::AlignTop);
        layout->addWidget(widget,3,0);


        // layout->addWidget(viewUserInfo,0,0);
        // layout->addWidget(viewDishList,1,0);
        // layout->addWidget(viewOrderList,2,0);
        // layout->addWidget(dishRegister,3,0);

        connect(viewUserInfo,&QPushButton::clicked,this,&MerchantNavWidget::toUserInfoSlot);
        connect(userInfoBtn,&QPushButton::clicked,this,&MerchantNavWidget::toUserInfoSlot);
        connect(viewDishList,&QPushButton::clicked,this,&MerchantNavWidget::toDishListSlot);
        connect(dishListBtn,&QPushButton::clicked,this,&MerchantNavWidget::toDishListSlot);
        connect(viewOrderList,&QPushButton::clicked,this,&MerchantNavWidget::toOrderListSlot);
        connect(orderListBtn,&QPushButton::clicked,this,&MerchantNavWidget::toOrderListSlot);
        connect(dishRegister,&QPushButton::clicked,this,&MerchantNavWidget::toDishRegisterWindowSlot);
        connect(dishRegisterBtn,&QPushButton::clicked,this,&MerchantNavWidget::toDishRegisterWindowSlot);

    }
public slots:
    void toUserInfoSlot(){emit toUserInfoSignal();}
    void toDishListSlot(){emit toDishListSignal();}
    void toOrderListSlot(){emit toOrderListSignal();}
    void toDishRegisterWindowSlot(){emit toDishRegisterWindowSignal();}

signals:
    void toUserInfoSignal();
    void toDishListSignal();
    void toOrderListSignal();
    void toDishRegisterWindowSignal();
};


#endif // NAV_H

```
## 消费者菜品列表界面实现
这个界面就属于比较复杂的界面了，会采用多层嵌套,这样开发的好处如下:

1. `简化开发难度`:将写一个大的类拆分成多层嵌套聚合的类，这样一层一层地独立开发能够降低开发难度
2. `降低数据传递难度`: 我们对打包好的数据的处理也是逐层解包拿到每一层需要的数据
3. `灵活性高`: 这样的多层嵌套可以很好地解耦合，提高灵活性
4. `聚焦信号`: 如下图所示，虽然在这个界面中，可能会有很多空间，但通过信号的层层转发的方式，通过一次或多次`多对一`的转发方式，最终让信号只从这个最大的界面控件发送传递给`MainWidget`用于处理。
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501201334881.webp)

> consumerdishlistwidget.h  
```C++
#ifndef CONSUMERDISHLISTWIDGET_H
#define CONSUMERDISHLISTWIDGET_H

#include <QWidget>
#include <QGridLayout>
#include <QScrollBar>
#include <QPushButton>
#include <QLabel>
#include <QString>
#include <QSpacerItem>
#include <QList>
#include <QPointer>
#include <QScrollArea>
#include <QHash>
#include <QEvent>
#include "Nav.h"

#include "CoreData.h"
#include "DataCenterCoreData.h"
#include "ColorConfig.h"

namespace dishList
{
struct DishItem:public QWidget
{
    Q_OBJECT;
public:
    using ptr = QSharedPointer<DishItem>;
    QString dish_id;
    QPushButton* dish_name;
    QPushButton* single_picture;
    QLabel* single_price;


    DishItem(const btyGoose::data::Dish& dish)
        :dish_id(dish.uuid)
    {
        auto color = ColorConfig::getInstance();
        QGridLayout* layout = new QGridLayout;
        QWidget* mainW = new QWidget;
        QGridLayout* mainL = new QGridLayout;
        mainL->addWidget(mainW);
        mainL->setContentsMargins(0,5,0,0);
        mainW->setLayout(layout);
        this->setLayout(mainL);
        mainW->setStyleSheet(QString("QWidget{border-radius: 15px;"
                                     " border: 2px solid %1;"
                                     "background-color: %2;}").arg(color->main_color_light,color->main_color_light));
        single_picture = new QPushButton();
        dish_name = new QPushButton(dish.name);
        QString price = "单价:";
        price += QString::number(dish.price_factor*dish.base_price);
        price += "元";
        single_price = new QLabel(price);

        QWidget* leftW = new QWidget;
        QWidget* rightW = new QWidget;

        single_picture->setFixedSize(100,100);
        QIcon icon = btyGoose::util::getIconFromLink(dish.icon_path);
        if(icon.isNull())
        {
            //如果图片下载失败，则使用默认图片
            icon = QIcon(QPixmap("://qsrc/defaultDish.png"));
        }
        single_picture->setIcon(icon);
        single_picture->setIconSize(QSize(100,100));
        dish_name->setFixedWidth(100);
        single_price->setFixedWidth(100);


        QGridLayout* leftLayout = new QGridLayout;
        leftLayout->setAlignment(Qt::AlignLeft);
        leftW->setLayout(leftLayout);
        leftLayout->addWidget(single_picture,0,0,2,1);
        leftLayout->addWidget(dish_name,0,1);
        QSpacerItem* space = new QSpacerItem(100,50);
        leftLayout->addItem(space,1,1);



        QGridLayout* rightLayout = new QGridLayout;
        rightLayout->setAlignment(Qt::AlignRight);
        rightLayout->addWidget(single_price);
        single_price->setFixedHeight(50);
        single_price->setStyleSheet(QString("QLabel{border-radius: 15px;"
                                            " border: 2px solid %1;"
                                            "background-color: %1;}").arg(color->main_color));
        rightW->setLayout(rightLayout);
        layout->addWidget(leftW,0,0);
        layout->addWidget(rightW,0,1);


        connect(dish_name,&QPushButton::clicked,this,&DishItem::dishNameBtnSlot);
        connect(single_picture,&QPushButton::clicked,this,&DishItem::dishNameBtnSlot);

    }
protected:
    void mousePressEvent(QMouseEvent *event) override {
        qDebug()<<"菜品item鼠标点击事件";
        emit dishInfoSignal(dish_id);//向外传递信号
        QWidget::mousePressEvent(event);
    }

public slots:
    void dishNameBtnSlot()
    {
        emit dishInfoSignal(dish_id);//向外传递信号
    }
signals:
    void dishInfoSignal(QString dish_id);
};
struct MerchantItem:public QWidget
{
    Q_OBJECT
public:
    using ptr = QSharedPointer<MerchantItem>;
    QString merchant_id;
    QLabel* merchant_name;


    QList<DishItem::ptr> list;
    MerchantItem(const QList<btyGoose::data::Dish>& lst)
    {
        auto color = ColorConfig::getInstance();


        merchant_id = lst[0].merchant_id;
        //默认加三个
        // qDebug()<<"开始往缓冲区插入菜品";
        for(auto dish:lst)
        {
            list.push_back(DishItem::ptr(new DishItem(dish)));
        }
        // qDebug()<<"插入完成";
        QWidget* mainW = new QWidget;
        QGridLayout* mainLayout = new QGridLayout;
        mainLayout->addWidget(mainW);
        QGridLayout* layout = new QGridLayout;
        this->setLayout(mainLayout);
        mainLayout->addWidget(mainW);
        mainW->setLayout(layout);

        merchant_name = new QLabel(lst[0].merchant_name);
        QWidget* nameW = new QWidget;
        nameW->setStyleSheet(QString("QWidget{background-color: %1;border-radius: 5px;}").arg(color->main_color));
        // nameW->setFixedWidth(this->width());
        QGridLayout* nameLayout = new QGridLayout;
        nameW->setLayout(nameLayout);
        nameLayout->addWidget(merchant_name);
        merchant_name->setParent(nameW);
        merchant_name->setStyleSheet(QString("QLabel{border-radius: 5px;"
                                             " border: 2px solid %1;}").arg(color->background_color2));
        layout->addWidget(nameW,0,0,1,4);//添加商家名称
        layout->setSpacing(0);

        mainW->setStyleSheet(QString("QWidget{background-color: %1;"
                                     "border: 2px solid %2}").arg(color->background_color2,color->background_color));

        int line  = 1;
        for(DishItem::ptr p:list)
        {
            //全都绑定到同一个槽函数
            connect(p.data(),&DishItem::dishInfoSignal,this,&MerchantItem::dishItemBtnSlot);
            layout->addWidget(p.data(),line,0,1,4);
            line++;
        }
    }

public slots:
    void dishItemBtnSlot(QString dish_id)
    {
        emit dishInfoSignal(dish_id);//向外传递信号
    }

signals:
    void dishInfoSignal(QString dish_id);

};
}
class ConsumerDishListWidget : public QWidget
{
    Q_OBJECT
public:
    explicit ConsumerDishListWidget(QHash<QString,QList<btyGoose::data::Dish>>* dish_list_table);
    ConsumerNavWidget* leftNavW;
    QWidget* rightW;
    void initRightW();

    QHash<QString,dishList::MerchantItem::ptr> merchantTable;
public:


public slots:
    //接收右子控件的槽
    void dishInfoSlot(QString dish_id);

signals:
    //右子控件的信号转发
    void dishInfoSignal(QString dish_id);


};

#endif // CONSUMERCARTLISTWIDGET_H


```
> consumerdishlistwidget.cpp  
```C++
#include "consumerdishlistwidget.h"
#include <QStyle>
#include <QBoxLayout>
using namespace dishList;
ConsumerDishListWidget::ConsumerDishListWidget(QHash<QString,QList<btyGoose::data::Dish>>* dish_list_table)
: QWidget{nullptr}
{
    QGridLayout* layout = new QGridLayout;
    this->setLayout(layout);
    leftNavW = new ConsumerNavWidget;
    rightW = new QWidget;
    layout->addWidget(leftNavW,0,0);
    layout->addWidget(rightW,0,1);
    //DEBUG //TODO
    // qDebug()<<"开始往缓冲区插入菜品列表";
    for(auto it = dish_list_table->begin();it!= dish_list_table->end();++it)
    {
        // qDebug()<<"第一个列表"<<it.value().size();
        merchantTable.insert(it.key(),MerchantItem::ptr(new MerchantItem(it.value())));
    }
    // qDebug()<<"插入完成";
    initRightW();

}

void ConsumerDishListWidget::initRightW()
{
    rightW->setFixedWidth(1000);
    rightW->setStyleSheet("QWidget { border: 2px solid black; }");
    QGridLayout* glayout = new QGridLayout;
    rightW->setLayout(glayout);
    QScrollArea* sa = new QScrollArea;
    glayout->addWidget(sa);
    sa->setWidgetResizable(true);
    sa->verticalScrollBar()->setStyleSheet("QScrollBar:vertical{width:2px;background-color: rgb(46,46,46);}");
    sa->horizontalScrollBar()->setStyleSheet("QScrollBar:horizontal{height:0;}");

    //3.实例化container
    QWidget* container =new QWidget;
    container->setFixedWidth(1000);//设置固定宽度
    sa->setWidget(container);//告诉滚动区和哪个控件配合

    //4.指定布局管理器
    QVBoxLayout* layout = new QVBoxLayout;
    layout->setContentsMargins(0,0,0,0);
    layout->setSpacing(0);
    layout->setAlignment(Qt::AlignTop);
    container->setLayout(layout);

    //插入每个商家的购物车控件
    for(auto it = merchantTable.begin();it!=merchantTable.end();++it)
    {
       MerchantItem::ptr p = it.value();
        layout->addWidget(p.data());

        connect(p.data(),&MerchantItem::dishInfoSignal,this,&ConsumerDishListWidget::dishInfoSlot);

    }
}

void ConsumerDishListWidget::dishInfoSlot(QString dish_id)
{
    emit dishInfoSignal(dish_id);
}

```

实际效果如下  
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231521568.webp)

## 消费者菜品详情窗口实现
虽然我们把它称为窗口，但实际上我们使用的是`<QDialog>`类

可能是出于开发失误，或者有更多的考量，具体的界面实现还是继承了`QWidget`类封装在`ConsumerDishDetailWidget`中，而对窗口的实现，则是另外封装`ConsumerDishDetailWindow`类并持有`ConsumerDishDetailWidget`指针的方式来实现窗口。

可能是为了使窗口能够更方便地加入动态性，对于所有的窗口，`MainWidget`都存有它们的指针。

> consumerdishdetailwidget.h  
```C++
#ifndef CONSUMERDISHDETAILWIDGET_H
#define CONSUMERDISHDETAILWIDGET_H

#include <QWidget>
#include <QGridLayout>
#include <QLabel>
#include <QIcon>
#include <QPushButton>
#include "CoreData.h"
class ConsumerDishDetailWidget : public QWidget
{
    Q_OBJECT
public:
    explicit ConsumerDishDetailWidget(const btyGoose::data::Dish* dish,int num);

    QString merchant_id="merchant_id";
    QString dish_id = "order_id";
    int num;//购物车中初识数量
    const btyGoose::data::Dish* _dish;

    // QIcon* icon;//TODO
    QLabel* iconTMP;
    // QIcon* iconTMP;

    QWidget* upW;
    QWidget* downW;
    QWidget* ulW;
    QWidget* urW;

    QLabel* cnt;
    void setCnt(int n);
    void initUp();
    void initDown();
public slots:
    void addSlot();
    void popSlot();

signals:
    void cartDishAddSignal(QString merchant_id,QString order_id);
    void cartDishPopSignal(QString merchant_id,QString order_id);
};

#endif // CONSUMERDISHDETAILWIDGET_H

```

> consumerdishdetailwidget.cpp  
```C++
#include "consumerdishdetailwidget.h"

ConsumerDishDetailWidget::ConsumerDishDetailWidget(const btyGoose::data::Dish* dish,int num)
    :_dish(dish),num(num)
{
    dish_id = _dish->uuid;
    merchant_id = _dish->merchant_id;

    this->setFixedSize(1080,600);
    QWidget* MW = new QWidget;
    QGridLayout* mwl = new QGridLayout;
    MW->setStyleSheet("QWidget{border:0}");//取消边框显示

    this->setLayout(mwl);
    mwl->addWidget(MW);
    QGridLayout* layout = new QGridLayout;
    MW->setLayout(layout);
    MW->setStyleSheet(QString("QWidget { border: 0px solid black;background-color: %1 }").arg(ColorConfig::getInstance()->background_color));
    upW = new QWidget;
    downW = new QWidget;
    ulW = new QWidget;
    urW = new QWidget;

    upW->setFixedSize(1080,400);
    layout->addWidget(upW,0,0);
    layout->addWidget(downW,1,0);

    initUp();
    initDown();
}

void ConsumerDishDetailWidget::setCnt(int n)
{
    QString content = QString("购物车内数量:") + QString::number(n);
    cnt->setText(content);
}

void ConsumerDishDetailWidget::initUp()
{
    auto color = ColorConfig::getInstance();
    QGridLayout* ulayout = new QGridLayout;
    upW->setLayout(ulayout);
    ulayout->addWidget(ulW,0,0);
    ulayout->addWidget(urW,0,1);
    //左上角初始化
    ulW->setFixedWidth(200);
    QGridLayout* lLayout = new QGridLayout;
    ulW->setLayout(lLayout);

    QIcon icon = btyGoose::util::getIconFromLink(_dish->icon_path);
    if(icon.isNull())
    {
        //如果图片下载失败，则使用默认图片
        icon = QIcon(QPixmap("://qsrc/defaultDish.png"));
    }
    icon_label = new QLabel();
    icon_label->setFixedSize(200,200);
    icon_label->setPixmap(icon.pixmap(200,200));

    QLabel* price = new QLabel("价格：" + QString::number(_dish->base_price*_dish->price_factor)+"元");
    price->setStyleSheet(QString("QLabel{/*border-radius: 15px;*/"
                                 " border: 2px solid %1;"
                                 "background-color: %1;}").arg(color->main_color));
    price->setFixedSize(100,50);

    lLayout->addWidget(icon_label,0,0);
    lLayout->addWidget(price,1,0);

    //右上角初始化
    QGridLayout* rLayout = new QGridLayout;
    urW->setLayout(rLayout);
    QLabel* name = new QLabel(_dish->name);
    name->setStyleSheet(QString("QLabel{/*border-radius: 15px;*/"
                                " border: 2px solid %1;"
                                "background-color: %1;}").arg(color->main_color));
    name->setFixedHeight(40);
    // QLabel* profile = new QLabel("这是一个多行文本\n这是一个多行文本\n这是一个多行文本\n这是一个多行文本\n");
    QLabel* profile = new QLabel(_dish->description);
    profile->setAlignment(Qt::AlignLeft | Qt::AlignTop);
    profile->setWordWrap(true);
    profile->setFixedWidth(600);

    rLayout->setAlignment(name,Qt::AlignLeft);
    rLayout->setAlignment(profile,Qt::AlignLeft);

    rLayout->addWidget(name,0,0);
    rLayout->addWidget(profile,1,0);
}

void ConsumerDishDetailWidget::initDown()
{
    QGridLayout* layout = new QGridLayout;
    downW->setLayout(layout);
    QPushButton* addBtn = new QPushButton("添加到购物车");
    QPushButton* popBtn = new QPushButton("从购物车移除");
    cnt = new QLabel;
    setCnt(num);

    addBtn->setFixedHeight(50);
    popBtn->setFixedHeight(50);
    cnt->setFixedHeight(50);
    auto color = ColorConfig::getInstance();
    QString btnStyle = QString("QPushButton{border-radius: 15px;"
                                " border: 2px solid %1;"
                                "background-color: %1;}").arg(color->main_color);

    addBtn->setStyleSheet(btnStyle);
    popBtn->setStyleSheet(btnStyle);
    layout->addWidget(addBtn,0,0);
    layout->addWidget(popBtn,1,0);
    layout->addWidget(cnt,2,0);

    layout->setAlignment(addBtn,Qt::AlignRight);
    layout->setAlignment(popBtn,Qt::AlignRight);
    layout->setAlignment(cnt,Qt::AlignRight);

    connect(addBtn,&QPushButton::clicked,this,&ConsumerDishDetailWidget::addSlot);
    connect(popBtn,&QPushButton::clicked,this,&ConsumerDishDetailWidget::popSlot);
}

void ConsumerDishDetailWidget::addSlot()
{
    emit cartDishAddSignal(merchant_id,dish_id);
}

void ConsumerDishDetailWidget::popSlot()
{
    emit cartDishPopSignal(merchant_id,dish_id);
}


```

> consumerdishdetailwindow.h  
```C++
#ifndef CONSUMERDISHDETAILWINDOW_H
#define CONSUMERDISHDETAILWINDOW_H
#include <QDialog>
#include <QVBoxLayout>
#include "consumerdishdetailwidget.h"
#include "CoreData.h"
class ConsumerDishDetailWindow : public QDialog {
    Q_OBJECT
public:
    ConsumerDishDetailWindow(const btyGoose::data::Dish* dish,int n) {
        // 设置窗口的基本属性
        setWindowTitle("Custom Sub Window");
        setFixedSize(1080, 600);
        // 创建控件
        QWidget *centralWidget = new QWidget(this);
        QVBoxLayout *layout = new QVBoxLayout(centralWidget);
        cdd = new ConsumerDishDetailWidget(dish,n);
        layout->addWidget(cdd);
        centralWidget->setLayout(layout);
    }

    ConsumerDishDetailWidget* cdd;
// public slots:
//     void onClose()
//     {
//         emit closeEvent();
//     }
// signals:
//     void closeEvent();
};
#endif // CONSUMERDISHDETAILWINDOW_H

```

效果如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231522231.webp)
## 数据中心类实现
按照之前的分层，数据中心便是客户端端业务逻辑层之下的客户端数据层，主要功能如下：

1. `发起异步数据获取请求`: 当`MainWidget`需要获取某个数据用于构建界面时，需要调用`DataCenter`对象进行异步获取数据
2. `缓存数据`: 所有从网络层传输到客户端的数据都缓存在数据中心中，同时可供`MainWidget`和`NetClient`（网络客户端）统一获取数据
3. `提供数据就绪信号`: 当`NetClient`把数据中心的数据就绪时，它会发送数据中心提供的就绪信号，以此通知`MainWidget`
4. `读取配置文件`: 为了能够方便地修改可配置参数，我们选择使用配置文件，这里用的是`Json`,而对配置文件的读取由数据中心类完成

**特点**: `单例模式`。我们采用1.构造函数私有化 2.删除拷贝构造 3.删除赋值拷贝 4.提供静态`getInstance()`接口 四步走来实现数据中心类的单例模式，因为整个客户端进程只需要一个数据中心，所以适合使用严格的单例模式

> 配置文件ServerConfig.json
```C++
{
    "httpUrl": "http://127.0.0.1:80",
    "sockUrl": "http://127.0.0.1:8001/websock"
}
```

> datacenter.h  
```C++
#ifndef DATACENTER_H
#define DATACENTER_H
#include "CoreData.h"
#include "DataCenterCoreData.h"
#include "NetClient.h"
#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
#include <QJsonValue>

#include <QCryptographicHash>


//网络
#include <QNetworkAccessManager>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QUrl>

#include <QFile>
#include <QObject>
#include <QIcon>
#include <QPointer>

using namespace btyGoose;
class DataCenter:public QObject
{
    Q_OBJECT;
public:
    //单例模式化
    static DataCenter* getInstance()
    {
        if(datacenter == nullptr)
        {
            datacenter = new DataCenter;
        }

        return datacenter;
    }
private:
    DataCenter();
    static DataCenter* datacenter;
    DataCenter(const DataCenter&) = delete;
    DataCenter operator=(const DataCenter&) =delete;
private:
    network::NetClient* client = nullptr;
public:
    //工具函数

    QList<data::Dish> DishListFromJsonArray(const QString& jsonString);
    QString DishListToJsonArray(const QList<data::Dish>& dishList);
    //处理短链连接的http请求
    QString configFileName = "ServerConfig.json";
    QString httpUrl = "http://127.0.0.1:80";
    QString sockUrl = "127.0.0.1:8001/websock";
    bool loadConfig();

public:
//////////////////////
///账户子系统
////////////////////////
/// 核心数据缓存
    data::Account* account = nullptr;
    //接口
    bool getAuthcode(const QString&phoneNumber);
    void accountRegisterAsync(const QString&name,const QString&password,const QString& phone,
                        const QString&nickname,const QString auth_code ,int type);
    void loginByNameAsync(const QString&name,const QString&password);
    void loginByPhoneAsync(const QString&phone,const QString&password);
    void accountUpdateLevelAsync();
    void accountChangeNicknameAsync(const QString&nickname);
    void accountUpdateLevelAsync(const QString& level);
public:
    QIcon QRCode;
    //第三方支付子系统
    bool getQRCode();
    void PaySuccessSignal();
    void PayFailSignal();
public:
//公共数据缓存
    data::Dish* dish = nullptr;
    data::Order* order = nullptr;
//////////////////////
///消费者子系统
////////////////////////
//缓存数据
    QHash<QString,QList<data::Dish>>* dish_list_table = nullptr;

    CartList* cart_list = nullptr;
    // ConsumerOrderList* consumer_order_list =nullptr;
    // ConsumerOrderItem* consumer_order_item = nullptr;
    QList<data::OrderDish>* consumer_order_dish_list = nullptr;
    QHash<QString,data::Order>* order_table = nullptr;//order_id -> order
//接口
    void consumerGetDishListAsync();    //用户数据在本对象的account里
    void consumerGetDishInfoAsync(const QString&dihs_id);
    void consumerOrderGenerateAsync(const QString&merchant_id);//得知道生成的是谁的
    void consumerGetOrderListAsync();//获取订单列表
    void consumerGetOrderDishListAsync(const QString&order_id);//订单菜品列表
    int getCartDishNum(const QString&merchant_id,const QString&dish_id);
    void consumerOderPayConfirmAsync(const QString&order_id);
    void consumerOrderCancelAsync(const QString& order_id);
public:
//////////////////////
///商家子系统
////////////////////////
/// 数据缓存
    QHash<QString,data::Dish>* merchant_dish_table = nullptr;
    QHash<QString,data::Order>* merchant_order_table =nullptr;
    QList<data::OrderDish>* merchant_order_dish_list = nullptr;//菜品列表缓冲
//接口
    void merchantGetDishListAysnc();
    void merchantDishRegisterAsync(const QString& name,const QString&link,
                                   double price,double price_factor = 1,
                                   const QString& introduction = "");
    //编辑菜品保存
    void merchantDishEditSaveAsync(const QString&dish_id,const QString& name,const QString&link,
                                   double price,double price_factor = 1,
                                   const QString& introduction = "");
    void merchantGetDishInfoAsync(const QString&dish_id);
    void merchantDishEditDelAsync(const QString&dish_id);
    void merchantGetOrderListAsync();
    void merchantGetOrderDishListAsync(const QString&order_id);
    void merchantOrderAcceptAsync(const QString&order_id);
    void merchantOrderRejectAsync(const QString&order_id);

    QList<data::Order> OrderListFromJsonArray(const QString& jsonString);
    QList<data::OrderDish> OrderDishListFromJsonArray(const QString&jsonString);

public:
//////////////////////
///管理员子系统
////////////////////////
/// 数据缓存
    QList<data::Order>* admin_order_list = nullptr;

//接口
    void adminGetOrderListAsync();
signals:
///////////////////////////////////
///账户子系统
//////////////////////////////////
    void getAccountDone();
    void getRegisterDone(bool ok,const QString& reason);
    void getLoginByNameDone(bool ok,const QString& reason);
    void getLoginByPhoneDone(bool ok,const QString& reason);
    void accountChangeNicknameAsyncDone();
    void accountUpdateLevelDone();
    void accountUpdateNicknameDone();
///////////////////////////////////
///消费者子系统
//////////////////////////////////

    void consumerGetDishListDone();
    void consumergetDishInfoDone();
    void consumerOrderGenerateDone();
    void consumerGetOrderListDone();
    void consumerGetOrderDishListDone(const QString&order_id);
    void consumerOrderPayConfirmDone();
    void consumerOrderCancelDone(const QString&reason);
///////////////////////////////////
///商家者子系统
//////////////////////////////////
    void merchantGetDishListDone();
    void merchantDishRegisterDone(bool ok,const QString&reason);
    void merchantGetDishInfoDone();
    void merchantDishEditSaveDone(bool ok,const QString&reason);
    void merchantDishEditDelDone();
    void merchantGetOrderListDone();
    void merchantGetOrderDishListDone(const QString&order_id);
    void merchantOrderAcceptDone();
    void merchantOrderRejectDone();

///////////////////////////////////
///管理员子系统
//////////////////////////////////

    void adminGetOrderListDone();
};


#endif // DATACENTER_H

```
> datacenter.cpp 
```C++
#include "datacenter.h"

DataCenter* DataCenter::datacenter = nullptr;

DataCenter::DataCenter()
{
    if(!loadConfig())
    {
        qDebug()<<configFileName<<"服务器地址配置错误";
        exit(1);
    }

    client = new network::NetClient(this,httpUrl,sockUrl);

    dish_list_table = new QHash<QString,QList<data::Dish>>;
    cart_list = new CartList;
    cart_list->cart_table = new QHash<QString,Cart::ptr>;
    // consumer_order_list = new ConsumerOrderList;
    // consumer_order_list->orderTable = new QHash<QString,ConsumerOrderItem::ptr>;
    order_table =  new QHash<QString,data::Order>;

    merchant_dish_table = new QHash<QString,data::Dish>;
    merchant_order_table = new QHash<QString,data::Order>;
}

QList<data::Dish> DataCenter::DishListFromJsonArray(const QString &jsonString)
{
    // qDebug()<<jsonString;
    QList<data::Dish> dishList;

    // 解析 JSON 字符串为 QJsonDocument
    QJsonDocument doc = QJsonDocument::fromJson(jsonString.toUtf8());

    // 如果文档有效且是一个 JSON 数组
    if (doc.isArray()) {
        QJsonArray jsonArray = doc.array();

        // 遍历数组中的每个元素
        for (const QJsonValue& value : jsonArray) {
            if (value.isObject()) {
                // 通过 Dish 类的 fromJson 方法将 QJsonObject 转换为 Dish 对象
                QJsonDocument doc(value.toObject());
                data::Dish dish;
                // qDebug()<<QString(doc.toJson()).toStdString();
                dish.loadFromJson(QString(doc.toJson()).toStdString());
                // qDebug()<<dish.merchant_name;
                dishList.append(dish);
            }
        }
    }

    return dishList;
}

QString DataCenter::DishListToJsonArray(const QList<data::Dish> &dishList)
{
    QJsonArray jsonArray;

    // 遍历 dishList，将每个 Dish 对象的 JSON 添加到 jsonArray 中
    for (const auto& dish : dishList)
    {
        // 将每个 Dish 对象转换为 JSON 字符串并加入到 JSON 数组中
        jsonArray.append(QJsonDocument::fromJson(dish.toJson().toUtf8()).object());
    }

    // 将 QJsonArray 转换为 JSON 字符串
    QJsonDocument doc(jsonArray);
    return QString(doc.toJson(QJsonDocument::Compact));
}

bool DataCenter::loadConfig()
{
    QFile file(configFileName);
    if (file.open(QIODevice::ReadOnly)) {
        QByteArray data = file.readAll();
        QJsonDocument doc = QJsonDocument::fromJson(data);

        if (doc.isObject()) {
            QJsonObject jsonObject = doc.object();
            httpUrl = jsonObject["httpUrl"].toString();
            sockUrl = jsonObject["sockUrl"].toString();

            return true;
        }
        else
        {
            qDebug()<<"无法打开为json";
        }
    }
    else
    {
        qDebug()<<"文件打开错误";
    }
    return false;
}

bool DataCenter::getAuthcode(const QString &phoneNumber)
{
    return true;
}

void DataCenter::accountRegisterAsync(const QString &name, const QString &password, const QString &phone, const QString &nickname, const QString auth_code, int type)
{
    //加密password
    QByteArray hashPass = QCryptographicHash::hash(password.toUtf8(), QCryptographicHash::Sha256);
    //调用接口
    client->accountRegister(name,hashPass,phone,nickname,auth_code,type);
}

void DataCenter::loginByNameAsync(const QString &name, const QString &password)
{
    //加密password
    QByteArray hashPass = QCryptographicHash::hash(password.toUtf8(), QCryptographicHash::Sha256);
    client->accountLoginByName(name,hashPass);
}

void DataCenter::loginByPhoneAsync(const QString &phone, const QString &password)
{
    QByteArray hashPass = QCryptographicHash::hash(password.toUtf8(), QCryptographicHash::Sha256);
    client->accountLoginByPhone(phone,hashPass);
}



void DataCenter::accountChangeNicknameAsync(const QString &nickname)
{
    client->accountChangeNickName(nickname);
}

void DataCenter::accountUpdateLevelAsync(const QString &level)
{
    client->accountUpdateLevel(account->uuid,level);
}

void DataCenter::consumerGetDishListAsync()
{
    client->consumerGetDishList();
    qDebug()<<"转到消费者界面";
}

void DataCenter::consumerGetDishInfoAsync(const QString &dish_id)
{
    client->consumerGetDishInfo(dish_id);
}

void DataCenter::consumerOrderGenerateAsync(const QString &merchant_id)
{
    auto it = cart_list->cart_table->find(merchant_id);
    if(it == cart_list->cart_table->end()) return;//购物车不存在

    client->consumerOrderGenerate(merchant_id);
}

void DataCenter::consumerGetOrderListAsync()
{
    client->consumerGetOrderList(account->uuid);
}

void DataCenter::consumerGetOrderDishListAsync(const QString &order_id)
{
    client->consumerGetOrdrerDishList(order_id);
}

int DataCenter::getCartDishNum(const QString &merchant_id, const QString &dish_id)
{
    // return cart_list->cart_table->value(merchant_id)->dish_table->value(dish_id)->cnt;
    auto cart_table = cart_list->cart_table;
    if(cart_table->find(merchant_id) == cart_table->end())
    {
        return 0;//尚未加入购物车
    }
    auto dish_table = cart_table->value(merchant_id)->dish_table;
    if(dish_table->find(dish_id) == dish_table->end())
    {
        return 0;//尚未加入购物车
    }
    return dish_table->value(dish_id)->cnt;
}

void DataCenter::consumerOderPayConfirmAsync(const QString &order_id)
{
    client->consumerOrderPayConfirm(order_id);
}

void DataCenter::consumerOrderCancelAsync(const QString &order_id)
{
    client->consumerOrderCancel(order_id);
}

void DataCenter::merchantGetDishListAysnc()
{
    client->merchantGetDishList(account->uuid);
}

void DataCenter::merchantDishRegisterAsync(const QString &name, const QString &link, double price,double price_factor , const QString &introduction)
{
    client->merchantDishRegister(name,link,price,price_factor,introduction);
}

void DataCenter::merchantDishEditSaveAsync(const QString &dish_id, const QString &name, const QString &link, double price, double price_factor, const QString &introduction)
{
    client->merchantDishEditSave(dish_id,name,link,price,price_factor,introduction);
}

void DataCenter::merchantGetDishInfoAsync(const QString &dish_id)
{
    client->merchantGetDishInfo(dish_id);
}

void DataCenter::merchantDishEditDelAsync(const QString &dish_id)
{
    client->merchantDishEditDel(dish_id);
}

void DataCenter::merchantGetOrderListAsync()
{
    client->merchantGetOrderList(account->uuid);
}

void DataCenter::merchantGetOrderDishListAsync(const QString &order_id)
{
    client->merchantGetOrderDishList(order_id);
}

void DataCenter::merchantOrderAcceptAsync(const QString &order_id)
{
    client->merchantOrderAccept(order_id);
}

void DataCenter::merchantOrderRejectAsync(const QString &order_id)
{
    client->merchantOrderReject(order_id);
}

QList<data::Order> DataCenter::OrderListFromJsonArray(const QString &jsonString)
{
    QList<btyGoose::data::Order> orderList;

    // 解析 JSON 字符串为 QJsonDocument
    QJsonDocument doc = QJsonDocument::fromJson(jsonString.toUtf8());

    // 如果文档有效且是一个 JSON 数组
    if (doc.isArray()) {
        QJsonArray jsonArray = doc.array();

        // 遍历数组中的每个元素
        for (const QJsonValue& value : jsonArray) {
            if (value.isObject()) {
                // 通过 Dish 类的 fromJson 方法将 QJsonObject 转换为 Dish 对象
                QJsonDocument doc(value.toObject());
                data::Order order;
                order.loadFromJson(QString(doc.toJson()).toStdString());
                orderList.append(order);
            }
        }
    }

    return orderList;
}

QList<data::OrderDish> DataCenter::OrderDishListFromJsonArray(const QString &jsonString)
{
    QList<btyGoose::data::OrderDish> dishList;

    // 解析 JSON 字符串为 QJsonDocument
    QJsonDocument doc = QJsonDocument::fromJson(jsonString.toUtf8());

    // 如果文档有效且是一个 JSON 数组
    if (doc.isArray()) {
        QJsonArray jsonArray = doc.array();

        // 遍历数组中的每个元素
        for (const QJsonValue& value : jsonArray) {
            if (value.isObject()) {
                // 通过 Dish 类的 fromJson 方法将 QJsonObject 转换为 Dish 对象
                QJsonDocument doc(value.toObject());
                data::OrderDish dish;
                dish.loadFromJson(QString(doc.toJson()).toStdString());
                dishList.append(dish);
            }
        }
    }

    return dishList;
}

void DataCenter::adminGetOrderListAsync()
{
    client->adminGetOrderList();
}

```

## 网络客户端
`NetClient`负责把参数和数据中心的数据进行序列化，通过`HTTP`的`POST`请求的`body正文`发送给服务端，并通过绑定响应信号到`lambda`表达式处理响应的报文，将响应的数据存入数据中心，或者通过信号传参发送出去

当前版本使用的是`Json`数据格式，以后也可以升级为效率更高的`protobuf`

>NetClient.h 
```C++
#ifndef NETCLIENT_H
#define NETCLIENT_H

#include <QObject>
#include <QString>
#include <QUrl>
#include <QWebSocket>
#include <QNetworkAccessManager>
#include <QNetworkReply>

#include <QJsonDocument>
#include <QJsonObject>
#include <QJsonArray>
#include <QJsonValue>
#include "DataCenterCoreData.h"

//提前声明
class DataCenter;

namespace network
{
class NetClient : public QObject
{
    Q_OBJECT
public:
    explicit NetClient(DataCenter* datacenter,const QString& hUrl,const QString& sUrl);

    DataCenter* datacenter;
    QString httpUrl;
    QString sockUrl;

    //http客户端
    QNetworkAccessManager httpClient;
    //socket客户端
    QWebSocket websocktClient;



//网络接口
    void ping();

public:
/////////////////////////////////////////
/// Account子系统
/////////////////////////////////////////
    void accountRegister(const QString&name,const QString&password,const QString& phone,
                         const QString&nickname,const QString auth_code ,int type);
    void accountLoginByName(const QString&name,const QString&password);
    void accountLoginByPhone(const QString&phone,const QString&password);
    void accountUpdateLevel(const QString&id,const QString& level);
    void accountChangeNickName(const QString&nickname);
/////////////////////////////////////////
/// Consumer子系统
/////////////////////////////////////////
    void consumerGetDishList();
    void consumerGetDishInfo(const QString& dish_id);
    void consumerOrderGenerate(const QString& merchant_id);
    void consumerGetOrderList(const QString&consumer_id);
    void consumerGetOrdrerDishList(const QString&order_id);
    void consumerOrderPayConfirm(const QString&order_id);
    void consumerOrderCancel(const QString&order_id);
/////////////////////////////////////////
/// Merchant子系统
/////////////////////////////////////////
    void merchantGetDishList(const QString&merchant_id);
    void merchantDishRegister(const QString& name,const QString&link,
                                   double price,double price_factor = 1,
                              const QString& introduction = "");
    void merchantDishEditSave(const QString&dish_id,const QString& name,const QString&link,
                              double price,double price_factor = 1,
                              const QString& introduction = "");
    void merchantGetDishInfo(const QString& dish_id);
    void merchantDishEditDel(const QString&dish_id);
    void merchantGetOrderList(const QString&merchant_id);
    void merchantGetOrderDishList(const QString&order_id);
    void merchantOrderAccept(const QString&order_id);
    void merchantOrderReject(const QString&order_id);

/////////////////////////////////////////
/// Admin 子系统
/////////////////////////////////////////
    void adminGetOrderList();
signals:
};
}
#endif // NETCLIENT_H

```
>NetClient.cpp 
```C++
#include "NetClient.h"
#include "datacenter.h"
namespace network
{

NetClient::NetClient(DataCenter *datacenter, const QString &hUrl, const QString &sUrl)
    :datacenter(datacenter),httpUrl(hUrl),sockUrl(sUrl)
{
    qDebug()<<"ping";
    ping();
}

void NetClient::ping()
{
    QNetworkRequest httpReq;
    httpReq.setUrl(httpUrl+"/ping");

    QNetworkReply* httpRes = httpClient.get(httpReq);
    connect(httpRes,&QNetworkReply::finished,this,[=](){
        //这里面的槽函数说明响应已经来了
        if(httpRes->error() == QNetworkReply::NoError)
        {
            //请求失败
            qDebug()<<"HTTP请求失败:"<<httpRes->errorString();
        }
        //获取响应到的body
        QByteArray body = httpRes->readAll();
        qDebug()<<"响应内容"<<body;
    });
}

void NetClient::accountRegister(const QString &name, const QString &password, const QString &phone, const QString &nickname, const QString auth_code, int type)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["name"] = name;
    jsonObj["password"] = password;  // 直接使用传入的已加密密码
    jsonObj["phone"] = phone;
    jsonObj["nickname"] = nickname;
    jsonObj["auth_code"] = auth_code;
    jsonObj["type"] = type;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/account/register");  // 注册接口是 /account/register
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                bool success = responseObj["success"].toBool();  // 假设响应中包含 success 字段
                QString Message = responseObj["message"].toString();
                if (success) {
                    qDebug() << "Account registration successful!";
                } else {
                    QString errorMessage = responseObj["message"].toString();
                    qDebug() << "Account registration failed: " << errorMessage;
                    // 处理失败的情况
                }
                emit datacenter->getRegisterDone(success,Message);
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::accountLoginByName(const QString &name, const QString &password)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["name"] = name;
    jsonObj["password"] = password;  // 直接使用传入的已加密密码

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/account/login/username");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                bool success = responseObj["success"].toBool();  // 假设响应中包含 success 字段
                QString Message = responseObj["message"].toString();
                if (success) {
                    qDebug() << "Account login by name successful!";
                    QString accJson = responseObj["account"].toString();
                    data::Account* acc = new data::Account;
                    acc->loadFromJson(accJson.toStdString());
                    datacenter->account = acc;//向数据中心存入账户信息
                } else {
                    QString errorMessage = responseObj["message"].toString();
                    qDebug() << "Account login by name failed: " << errorMessage;
                    // 处理失败的情况
                }
                emit datacenter->getLoginByNameDone(success,Message);
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::accountLoginByPhone(const QString &phone, const QString &password)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["phone"] = phone;
    jsonObj["password"] = password;  // 直接使用传入的已加密密码

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/account/login/phone");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                bool success = responseObj["success"].toBool();  // 假设响应中包含 success 字段
                QString Message = responseObj["message"].toString();
                if (success) {
                    qDebug() << "Account login by name successful!";
                    QString accJson = responseObj["account"].toString();
                    data::Account* acc = new data::Account;
                    acc->loadFromJson(accJson.toStdString());
                    datacenter->account = acc;//向数据中心存入账户信息
                } else {
                    QString errorMessage = responseObj["message"].toString();
                    qDebug() << "Account login by name failed: " << errorMessage;
                    // 处理失败的情况
                }
                emit datacenter->getLoginByPhoneDone(success,Message);
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::accountUpdateLevel(const QString&id,const QString &level)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["id"] = id;
    jsonObj["level"] = level;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/account/update/level");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                DataCenter::getInstance()->account->level = static_cast<data::Account::Level>(responseObj["level"].toInt());
                qDebug()<<"level change to :"<<DataCenter::getInstance()->account->level;
                emit datacenter->accountUpdateLevelDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::accountChangeNickName(const QString &nickname)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["nickname"] = nickname;
    jsonObj["id"] = datacenter->account->uuid;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/account/update/nickname");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                DataCenter::getInstance()->account->nickname = responseObj["nickname"].toString();
                // qDebug()<<"nickname change to :"<<DataCenter::getInstance()->account->level;
                emit datacenter->accountUpdateNicknameDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerGetDishList()
{
    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/dish/list");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request,"");//没有报文

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();
            // qDebug()<<responseData;
            auto list  = datacenter->DishListFromJsonArray(responseObj["dish_list"].toString());
            // qDebug()<<"获取到的菜品数量:"<<list.size();
            auto table = datacenter->dish_list_table;
            if(!table->empty())
            {
                table->clear();
            }
            for(const auto&dish:list)
            {

                if(table->find(dish.merchant_id) == table->end())
                {
                    // qDebug()<<"新增了一个商家";
                    table->insert(dish.merchant_id,QList<data::Dish>());
                }
                auto it = table->find(dish.merchant_id);
                (*it).append(dish);
                // qDebug()<<dish.merchant_name;
                // qDebug()<<dish.merchant_id<<"菜品数量"<<table->value(dish.merchant_id).size();
            }
            emit datacenter->consumerGetDishListDone();
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerGetDishInfo(const QString &dish_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["dish_id"] = dish_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/dish/dishInfo");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                data::Dish* dish = new data::Dish;
                dish->loadFromJson(responseObj["dish"].toString().toStdString());
                if(datacenter->dish)
                {
                    delete datacenter->dish;
                    datacenter->dish = nullptr;
                }
                datacenter->dish = dish;    //导入缓冲区
                emit datacenter->consumergetDishInfoDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerOrderGenerate(const QString &merchant_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;

    if(datacenter->cart_list->cart_table->find(merchant_id) == datacenter->cart_list->cart_table->end())
    {
        qDebug()<<"商家不存在";
        return;
    }
    Cart::ptr cart = *(datacenter->cart_list->cart_table->find(merchant_id));
    jsonObj["merchant_id"] =cart->merchant_id;
    jsonObj["merchant_name"] = cart->merchant_name;
    jsonObj["consumer_id"] = datacenter->account->uuid;
    jsonObj["consumer_name"] = datacenter->account->name;
    jsonObj["level"] = static_cast<int>(datacenter->account->level);
    jsonObj["pay"] = cart->pay;
    jsonObj["count"] = cart->cnt;

    QJsonArray jsonArr;

    for(auto it = cart->dish_table->begin();it!=cart->dish_table->end();++it)
    {
        QJsonObject obj;
        obj["dish_id"] = it.value()->dish_id;
        obj["merchant_id"] = cart->merchant_id;
        obj["dish_name"] = it.value()->dish_name;
        obj["dish_price"] = it.value()->dish_price;
        obj["count"] = it.value()->cnt;
        jsonArr.append(obj);
    }

    jsonObj["dish_arr"] = jsonArr;

    //清空购物车
    datacenter->cart_list->cart_table->erase(datacenter->cart_list->cart_table->find(merchant_id));
    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/order/generate");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                // qDebug()<<doc.toJson();
                QJsonObject responseObj = doc.object();
                data::Order order;
                // qDebug()<<order.status;
                // qDebug()<<responseObj["order"].toString();
                order.loadFromJson(responseObj["order"].toString().toStdString());
                datacenter->order_table->insert(order.uuid,order);    //导入订单列表
                // qDebug()<<"NetClient order stat"<<order.status;
                emit datacenter->consumerOrderGenerateDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerGetOrderList(const QString &consumer_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["consumer_id"] = consumer_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/order/list");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();
            // qDebug()<<responseData;
            auto list  = datacenter->OrderListFromJsonArray(responseObj["order_list"].toString());
            //先清空旧数据
            datacenter->order_table->clear();
            for(const data::Order&order:list)
            {
                // qDebug()<<order.name;
                datacenter->order_table->insert(order.uuid,order);
            }
            emit datacenter->consumerGetOrderListDone();

        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerGetOrdrerDishList(const QString &order_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["order_id"] = order_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/order/detail/dishlist");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply,order_id]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                data::Order* order = new data::Order;
                auto list  = datacenter->OrderDishListFromJsonArray(responseObj["dish_list"].toString());
                qDebug()<<"获取到的菜品数量:"<<list.size();
                QList<data::OrderDish>* dish_list = datacenter->consumer_order_dish_list;
                if(dish_list != nullptr)
                {
                    delete dish_list;
                    dish_list = nullptr;
                }
                datacenter->consumer_order_dish_list = new QList<data::OrderDish>(list);
                emit datacenter->consumerGetOrderDishListDone(order_id);
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerOrderPayConfirm(const QString &order_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["order_id"] = order_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/order/pay/confirm");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply,order_id]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();

                emit datacenter->consumerOrderPayConfirmDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::consumerOrderCancel(const QString &order_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["order_id"] = order_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/consumer/order/cancel");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply,order_id]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();

                emit datacenter->consumerOrderCancelDone(responseObj["reason"].toString());
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantGetDishList(const QString &merchant_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["merchant_id"] = merchant_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/dish/list");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();
            // qDebug()<<responseData;
            auto list  = datacenter->DishListFromJsonArray(responseObj["dish_list"].toString());
            //先清空旧数据
            datacenter->merchant_dish_table->clear();
            for(const data::Dish&dish:list)
            {
                qDebug()<<dish.name;
                datacenter->merchant_dish_table->insert(dish.uuid,dish);
            }
            emit datacenter->merchantGetDishListDone();

        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantDishRegister(const QString &name, const QString &link, double price,double price_factor, const QString &introduction)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["name"] = name;
    jsonObj["icon_path"] = link;  // 直接使用传入的已加密密码
    jsonObj["price"] = price;
    jsonObj["price_factor"] = price_factor;
    jsonObj["introduction"] = introduction;
    jsonObj["merchant_id"] = datacenter->account->uuid;
    jsonObj["merchant_name"] = datacenter->account->nickname;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/dish/register");  // 注册接口是 /account/register
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                bool success = responseObj["success"].toBool();  // 假设响应中包含 success 字段
                QString Message = responseObj["message"].toString();
                if (success) {
                    qDebug() << "Register registration successful!";
                } else {
                    QString errorMessage = responseObj["message"].toString();
                    qDebug() << "Register registration failed: " << errorMessage;
                    // 处理失败的情况
                }
                emit datacenter->merchantDishRegisterDone(success,Message);
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantDishEditSave(const QString &dish_id, const QString &name, const QString &link, double price, double price_factor, const QString &introduction)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["dish_id"] = dish_id;
    jsonObj["name"] = name;
    jsonObj["icon_path"] = link;  // 直接使用传入的已加密密码
    jsonObj["price"] = price;
    jsonObj["price_factor"] = price_factor;
    jsonObj["introduction"] = introduction;
    jsonObj["merchant_id"] = datacenter->account->uuid;
    jsonObj["merchant_name"] = datacenter->account->nickname;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/dish/update");  // 注册接口是 /account/register
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                bool success = responseObj["success"].toBool();  // 假设响应中包含 success 字段
                QString Message = responseObj["message"].toString();
                if (success) {
                    qDebug() << "Register registration successful!";
                } else {
                    QString errorMessage = responseObj["message"].toString();
                    qDebug() << "Register registration failed: " << errorMessage;
                    // 处理失败的情况
                }
                emit datacenter->merchantDishEditSaveDone(success,Message);
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantGetDishInfo(const QString &dish_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["dish_id"] = dish_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/dish/info");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                QJsonObject responseObj = doc.object();
                data::Dish* dish = new data::Dish;
                dish->loadFromJson(responseObj["dish"].toString().toStdString());
                if(datacenter->dish)
                {
                    delete datacenter->dish;
                    datacenter->dish = nullptr;
                }
                datacenter->dish = dish;    //导入缓冲区
                emit datacenter->merchantGetDishInfoDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantDishEditDel(const QString &dish_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["dish_id"] = dish_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/dish/del");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            if (!doc.isNull()) {
                emit datacenter->merchantDishEditDelDone();
            } else {
                qDebug() << "Invalid JSON response!";
            }
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantGetOrderList(const QString &merchant_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["merchant_id"] = merchant_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/order/list");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();
            // qDebug()<<responseData;
            auto list  = datacenter->OrderListFromJsonArray(responseObj["order_list"].toString());
            //先清空旧数据
            datacenter->merchant_order_table->clear();
            for(const data::Order&order:list)
            {
                // qDebug()<<order.name;
                datacenter->merchant_order_table->insert(order.uuid,order);
            }
            emit datacenter->merchantGetOrderListDone();

        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantGetOrderDishList(const QString &order_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["order_id"] = order_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/order/detail/dishlist");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply,order_id]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();
            // qDebug()<<responseData;
            auto list  = datacenter->OrderDishListFromJsonArray(responseObj["dish_list"].toString());
            qDebug()<<"获取到的菜品数量:"<<list.size();
            QList<data::OrderDish>* dish_list = datacenter->merchant_order_dish_list;
            if(dish_list != nullptr)
            {
                delete dish_list;
                dish_list = nullptr;
            }
            datacenter->merchant_order_dish_list = new QList<data::OrderDish>(list);
            emit datacenter->merchantGetOrderDishListDone(order_id);
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantOrderReject(const QString &order_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["order_id"] = order_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/order/reject");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply,order_id]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();

            emit datacenter->merchantOrderRejectDone();
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::merchantOrderAccept(const QString &order_id)
{
    // 1. 构造 JSON 数据
    QJsonObject jsonObj;
    jsonObj["order_id"] = order_id;

    // 将 JSON 对象转换为字符串
    QJsonDocument doc(jsonObj);
    QByteArray jsonData = doc.toJson();

    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/merchant/order/accept");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, jsonData);

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply,order_id]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();

            emit datacenter->merchantOrderAcceptDone();
        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}

void NetClient::adminGetOrderList()
{
    // 2. 创建 HTTP 请求并设置 URL 和请求头
    QUrl url(httpUrl + "/admin/order/list");  //
    QNetworkRequest request(url);
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json;charset=UTF-8");

    // 3. 发送 POST 请求
    QNetworkReply *reply = httpClient.post(request, "");

    // 4. 连接信号和槽以处理响应
    connect(reply, &QNetworkReply::finished, [this, reply]() {
        if (reply->error() == QNetworkReply::NoError) {
            // 处理服务器返回的数据
            QByteArray responseData = reply->readAll();
            QJsonDocument doc = QJsonDocument::fromJson(responseData);
            QJsonObject responseObj = doc.object();
            // qDebug()<<responseData;
            auto list  = datacenter->OrderListFromJsonArray(responseObj["order_list"].toString());
            //先清空旧数据
            if(datacenter->admin_order_list == nullptr)
            {
                datacenter->admin_order_list=new QList<data::Order>;
            }
            datacenter->admin_order_list->clear();
            datacenter->admin_order_list->append(list.begin(),list.end());
            emit datacenter->adminGetOrderListDone();

        } else {
            qDebug() << "Request failed: " << reply->errorString();
        }

        // 释放回复对象
        reply->deleteLater();
    });
}


}
```

## 其余代码
上面的代码已经体现了客户端代码的大部分技术特点，其余界面的代码详见项目源码


