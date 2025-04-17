---
title:  C++项目】v2.0美鹅外卖平台的服务端实现
date: 2025-02-23 09:55:22
tags:
---
# 重大更新

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
**当前版本**: v2.0重大更新

当前服务器使用了如下技术栈:
+ `MySQL`数据库服务：由`MySQL 8.0`提供数据持久化服务
+ `MySQL API`服务: 由`MySQL Connector/C++ 7`提供原生`API`,并在开发中封装成数据库对象及其接口
+ `序列化与反序列化`服务: 由`jsoncpp`库提供对象的序列化与反序列化
+ `HTTP短链接通信`服务 : 由开源项目`cpp-httplib`提供的单头文件库提供简单易用的HTTP服务

## 当前架构图
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231424942.webp)

## 服务端层次结构
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231446306.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231450675.webp)

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

## 开发环境
| 开发平台 | 阿里ECS服务器 Ubuntu 22.04 | 
| MySQL Connector/C++ 版本 | Connector 7 | 
| C++语法标准 | C++17 | 

## 文件结构一览
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502231102725.webp)

## 编写 docker-compose.yml
目前服务的架构比较简单，我们只需要用到两个`Service`

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

## 后端服务配置
+ 镜像的构建由`Dockerfile`自动完成
+ 端口映射：防止宿主机的80端口被占用，我们把容器的`80`端口映射到`30001`
+ 依赖： 后端服务依赖数据库服务的健康状态

*由于这是最后一部分代码，就合并到下一个标题展示了*

## 完整的docker-compose.yml
```YML
services:
  server:
    build:
      context: ./server
    image: btygoose:v1.0
    ports:
      - 30001:80
    depends_on:
      db:
        condition: service_healthy
    networks:  # 新增网络配置
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
    networks:  # 新增网络配置
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

## DatabaseClient.hpp
我们将所有与数据库相关的接口封装到`DatabaseClient`类中，在抽象层次中单独封装一层，为`HTTP服务层`提供简单易用的数据库服务。

*这边只给出头文件，具体实现的.cpp文件就不展示了*

> DatabaseClient.hpp
```C++
#pragma once

#include "CoreData.hpp"

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

using std::vector;
using std::string;

namespace btyGoose
{

class DatabaseClient
{
public:
	DatabaseClient();
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
public:

	bool loadConfig();
	void saveConfig();

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


下面的是配置文件，不过在Dockerfile中会重新生成一遍

```json
{
    "database" : "BeautyGoose",
    "host" : "127.0.0.1",
    "password" : "Password!1",
    "port" : "3306",
    "user" : "btyGooseUser"
}
```

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

	void start();

	bool loadConfig();
	void saveConfig();

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
	DatabaseClient db;
	string host = "127.0.0.1";
	int port = 80;
};
}

```

## main.cpp
为程序提供main函数入口

```C++
#include "HTTPServer.hpp"

int main()
{
    btyGoose::HTTPServer::getInstance()->start();
    return 0;
}
```

## makefile 构建项目

```makefile
LDLIBS = -lmysqlcppconn -ljsoncpp -lpthread

server:main.o DatabaseClient.o HTTPServer.o
	g++ -o $@ *.o -std=c++17 $(LDLIBS)

server-static:main.o DatabaseClient.o HTTPServer.o
	g++ 

main.o: main.cpp
	g++ -c main.cpp -o main.o -std=c++17 

DatabaseClient.o:DatabaseClient.cpp 
	g++ -c DatabaseClient.cpp -o DatabaseClient.o -std=c++17 

HTTPServer.o:HTTPServer.cpp
	g++ -c HTTPServer.cpp -o HTTPServer.o -std=c++17 

.PHONY:clean
clean:
	rm -f server *.o
```

## Dockerfile编写

主要逻辑如下

+ 这里使用ubuntu:22.04作为基础镜像
+ 使用apt安装相关的编译环境和运行环境
+ COPY源代码
+ 覆写配置文件
+ 编译和执行程序

```Dockerfile
FROM ubuntu:22.04 AS build1
RUN apt update -y && apt install -y g++ \
    make \
    libmysqlcppconn-dev \
    libjsoncpp-dev \
    libboost-system-dev
COPY ./src* /src
RUN echo '{"host" : "0.0.0.0","port" : 80}' > /src/serverConfig.json && \
    echo  '{"database" : "BeautyGoose","host" : "db","password" : "Password!1","port" : "3306","user" : "btyGooseUser"}' > /src/DBConfig.json

WORKDIR /src
RUN make
CMD ["/src/server"]
```
