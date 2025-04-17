---
title: C++项目】美鹅外卖平台的服务端实现
date: 2024-12-18 10:26:20
tags:
---

# 服务端详细设计
## 版本信息
**当前版本**: v0.9测试版

当前版本的服务器使用了如下技术栈:

+ `QT框架`: 利用QT的全平台特性，这里的开发环境也使用了QT框架。主要使用了QT里的数据结构
+ `QT Json框架`: 本版本对报文的序列化与反序列化技术依然使用了`Json`格式
+ `cpp-httplib`:一个开源的单文件的http库，用于提供简单易操作的短链接HTTP服务
+ `MySQL Connector/C++  Windows平台`: MySQL官方提供的Windows平台C++连接数据库的API

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

# 服务端实现
[戳我🔗去github的release页面获取当前版本的源码](https://github.com/sis-shen/BeautyGoose/releases/tag/v0.9)

## 开发环境
+ `开发平台:Windows 11 家庭中文版`
+ `QT版本 QT6.7.3`
+ `Visual Studio 2022`
+ `Qt Visual Studio Tools` 版本3.3.1.1（VS 插件）
 
## 文件结构一览
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501211959689.webp)

## 数据库准备
这里采用手动创建的方式准备数据库,本项目使用的数据库为`MySQL`

### 建数据库
```sql
create database BeautyGoose;
use BeautyGoose;
```

### 创建专门的连接用用户
```sql
set global validate_password_policy = 'LOW';
create user 'btyGooseUser'@'%' identified by 'BueatyGoose';
grant all on BeautyGoose.* to 'btyGooseUser'@'%';
flush privileges;
```

### 建立账户表
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

### 建立菜品表

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

### 建立订单表

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

### 建立历史订单表

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

### 建立orderDish表

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

### 建立historyDish表

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
## 主文件 main.cpp
主文件没有特殊功能，仅仅只是启动`QT 框架`的核心应用，以及生成一个自定义类`btyGoose::HTTPServer`的实例对象

> main.cpp
```C++
#include <QtCore/QCoreApplication>
#include "HTTPServer.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    btyGoose::HTTPServer svr;
    return a.exec();
}

```

## 核心数据结构
这部分的代码与客户端相同

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

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

## 数据库客户端类
我们与数据库交互的**代码封装进函数**,再把**函数封装成客户端类**，这样我们可以大大简化处理上层业务逻时，对数据库操作的易用性

**配置文件**: 在配置文件`databaseConfig.json`中写入所用数据库的相关信息

> databaseConfig.json
```Json
{
  "database": "BeautyGoose",
  "host": "127.0.0.1",
  "password": "BueatyGoose",
  "port": "3306",
  "user": "btyGooseUser"
}
```

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

> DatabaseClient.h
```C++
#pragma once

#include <QtCore/QCoreApplication>
#include <QTextStream>
#include <QList>
#include <QDebug>
#include <QThread>
#include "CoreData.h"

#include <jdbc/cppconn/prepared_statement.h>
#include <jdbc/cppconn/statement.h>
#include <jdbc/cppconn/resultset.h>
#include <jdbc/cppconn/exception.h>
#include <jdbc/cppconn/metadata.h>
#include "jdbc/mysql_driver.h"
#include <jdbc/mysql_connection.h>
#include <sstream>

#include <QMutex>
namespace btyGoose
{

class DatabaseClient
{
public:
	DatabaseClient();
private:
	//QSqlDatabase db;
	QString user = "";
	QString password = "";
	QString host = "127.0.0.1";
	QString port = "3306";
	QString database = "";

	sql::Connection* con;
	sql::Statement* stmt;

	QMutex mtx;
public:

	bool loadConfig();
	void saveConfig();

	//Account CURD
	bool addAccount(const data::Account&);
	bool updateAccount(const data::Account&);
	data::Account searchAccountByID(const QString& id);
	data::Account searchAccountByName(const QString& name);
	data::Account searchAccountByPhone(const QString& phone);

	bool delAccountByID(const QString& id);

	//dish CURD
	bool addDish(const data::Dish&);
	bool updateDish(const data::Dish&);
	data::Dish searchDishByID(const QString& id);
	QList<data::Dish> getAllDishList();
	QList<data::Dish> getDishListByMerchant(const QString& id);
	bool delDishByID(const QString& id);

	//Order CURD
	bool addOrder(const data::Order&order);
	bool updateOrder(const data::Order&order);
	bool updateOrder(const QString&id ,const data::Order::Status stat);
	data::Order searchOrderByID(const QString& order_id);
	QList<data::Order> getOrderListByMerchant(const QString& merchant_id);
	QList<data::Order> getOrderListByMerchantWaiting(const QString& merchant_id);
	QList<data::Order> getOrderListByConsumer(const QString& consumer_id);
	QList<data::Order> getAllOrderList();
	bool delOrderByID(const QString& order_id);
	int clearOverTimeOrder();

	//OrderDish CURD
	bool addOrderDishesByID(const QString& order_id, const QList<data::OrderDish>& list);
	bool delOrderDishesByID(const QString& order_id);
	QList<data::OrderDish> getOrderDishesByID(const QString& order_id);
	
	//History CURD
	bool addHistory(const data::Order& order);
	QList<data::Order> getAllHistoryList();

	//HistoryDish CURD
	bool addHistoryDishesByID(const QString& order_id, const QList<data::OrderDish>& list);
	QList<data::OrderDish> getHistoryDishesByID(const QString& order_id);
};
}


```

> DatabaseClient.cpp
```C++
#include "DatabaseClient.h"
using namespace btyGoose;
btyGoose::DatabaseClient::DatabaseClient()
{
    if (!loadConfig())
    {
        saveConfig();
        exit(1);
    }
    sql::mysql::MySQL_Driver* driver = sql::mysql::get_mysql_driver_instance();
    std::string hostName = "tcp://";
    hostName += host.toStdString();
    hostName += ":";
    hostName += port.toStdString();
    con = driver->connect(hostName, user.toStdString(), password.toStdString());
    con->setSchema(database.toStdString());
    LOG() << "数据库初始连接状态:" << !con->isClosed();
    qDebug() << "数据库地址:" << hostName;
    qDebug() << "数据库用户名:" << user;
    qDebug() << "数据库名称" << database;
}

bool btyGoose::DatabaseClient::loadConfig()
{
    QFile file("databaseConfig.json");
    if (file.open(QIODevice::ReadOnly)) {
        QByteArray data = file.readAll();
        QJsonDocument doc = QJsonDocument::fromJson(data);

        if (doc.isObject()) {
            QJsonObject jsonObject = doc.object();
            user = jsonObject["user"].toString();
            password = jsonObject["password"].toString();
            host = jsonObject["host"].toString();
            port = jsonObject["port"].toString();
            database = jsonObject["database"].toString();
            return true;
        }
    }
    //反序列化失败
    user = "unkown";
    password = "unkown";
    database = "unkown";
    saveConfig();
    return false;
}

void btyGoose::DatabaseClient::saveConfig()
{
    QFile file("databaseConfig.json");
    if (file.open(QIODevice::WriteOnly)) {
        QJsonObject jsonObject;
        jsonObject["user"] = user;
        jsonObject["password"] = password;
        jsonObject["host"] = host;
        jsonObject["port"] = port;
        jsonObject["database"] = database;

        QJsonDocument doc(jsonObject);
        file.write(doc.toJson());
        file.close();
    }
    else
    {
        std::cerr << "database配置文件打开错误";
    }
}



bool btyGoose::DatabaseClient::addAccount(const data::Account& account)
{
    // 使用 QMutexLocker 确保线程安全
    QMutexLocker locker(&mtx);

    try {
        // 构建 SQL 插入语句
        QString sql = QString(
            "INSERT INTO account (uuid, phone, name, password, nickname, level, icon_data, type, balance) "
            "VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)"
        );

        // 创建预处理语句
        sql::PreparedStatement* pstmt = con->prepareStatement(sql.toStdString());

        // 绑定参数
        pstmt->setString(1, account.uuid.toStdString());
        pstmt->setString(2, account.phone.toStdString());
        pstmt->setString(3, account.name.toStdString());
        pstmt->setString(4, account.password.toStdString());
        pstmt->setString(5, account.nickname.toStdString());
        pstmt->setInt(6, static_cast<int>(account.level));  // 使用 int 存储 Level 枚举
        // 使用 std::istringstream 来处理图标数据
        std::istringstream iconStream(std::string(account.icon.data(), account.icon.size()));
        pstmt->setBlob(7, &iconStream);  // 使用 std::istream* 作为 BLOB 类型
        pstmt->setInt(8, static_cast<int>(account.type));  // 使用 int 存储 Type 枚举
        pstmt->setInt64(9, static_cast<long long>(account.balance));  // balance 作为 int unsigned，转为 long long

        // 执行插入操作
        pstmt->executeUpdate();

        // 清理资源
        delete pstmt;

        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error while inserting account: " << e.what();
        return false;
    }
}

bool btyGoose::DatabaseClient::updateAccount(const data::Account& account)
{
    QMutexLocker locker(&mtx);  // 确保线程安全

    try {
        // 准备 SQL 更新语句
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "UPDATE account SET name = ?, password = ?, nickname = ?, icon_data = ?, type = ?, balance = ?, phone = ?, level = ? "
            "WHERE uuid = ?"
        );

        // 设置参数
        pstmt->setString(1, account.name.toStdString());
        pstmt->setString(2, account.password.toStdString());
        pstmt->setString(3, account.nickname.toStdString());
        std::istringstream iconStream(std::string(account.icon.data(), account.icon.size()));
        pstmt->setBlob(4, &iconStream);  // 图标是二进制数据（BLOB）
        pstmt->setInt(5, static_cast<int>(account.type));  // 类型转换为整数
        pstmt->setDouble(6, account.balance);  // 账户余额
        pstmt->setString(7, account.phone.toStdString());
        pstmt->setInt(8, static_cast<int>(account.level));  // 等级转换为整数
        pstmt->setString(9, account.uuid.toStdString());  // 用uuid作为更新的条件

        // 执行更新操作
        int affectedRows = pstmt->executeUpdate();

        // 如果受影响的行数为 0，表示没有记录被更新
        if (affectedRows == 0) {
            qDebug() << "No account updated, check if uuid exists: " << account.uuid;
        }

        // 清理资源
        delete pstmt;

        return affectedRows > 0;  // 如果有受影响的行，则返回 true，表示更新成功
    }
    catch (sql::SQLException& e) {
        // 捕获并打印 SQL 异常信息
        qDebug() << "Error updating account: " << e.what();
        return false;
    }
}

//bool btyGoose::DatabaseClient::updataAccount(const data::Account&)
//{
//    return false;
//}
//btyGoose::data::Account btyGoose::DatabaseClient::searchAccountByID(const QString& id)
//{
//    return btyGoose::data::Account();
//}
//
//btyGoose::data::Account btyGoose::DatabaseClient::searchAccountByName(const QString& name)
//{
//    return btyGoose::data::Account();
//}
//
//btyGoose::data::Account btyGoose::DatabaseClient::searchAccountByPhone(const QString& phone)
//{
//    return btyGoose::data::Account();
//}

btyGoose::data::Account btyGoose::DatabaseClient::searchAccountByID(const QString& id)
{
    QMutexLocker locker(&mtx);  // 确保线程安全

    try {
        // 准备 SQL 查询
        QString sql = QString("SELECT * FROM account WHERE uuid = ?");

        // 创建 SQL 语句对象
        sql::PreparedStatement* pstmt = con->prepareStatement(sql.toStdString());

        // 绑定参数
        pstmt->setString(1, id.toStdString());

        // 执行查询
        sql::ResultSet* res = pstmt->executeQuery();

        // 检查是否有查询结果
        if (res->next()) {
            // 将查询结果映射到 Account 结构体
            btyGoose::data::Account account;
            account.uuid = QString::fromStdString(res->getString("uuid"));
            account.phone = QString::fromStdString(res->getString("phone"));
            account.name = QString::fromStdString(res->getString("name"));
            account.password = QString::fromStdString(res->getString("password"));
            account.nickname = QString::fromStdString(res->getString("nickname"));
            account.level = static_cast<btyGoose::data::Account::Level>(res->getInt("level"));
            account.type = static_cast<btyGoose::data::Account::Type>(res->getInt("type"));
            account.balance = res->getDouble("balance");

            // 处理 BLOB 数据
            std::istream* blobStream = res->getBlob("icon_data");
            if (blobStream) {
                std::ostringstream oss;
                oss << blobStream->rdbuf();
                account.icon = QByteArray::fromStdString(oss.str());
            }

            // 清理资源
            delete res;
            delete pstmt;

            return account;
        }

        // 如果没有找到记录，返回一个空账户
        delete res;
        delete pstmt;
        return btyGoose::data::Account();
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error while searching account by ID: " << e.what();
        return btyGoose::data::Account();
    }
}

btyGoose::data::Account btyGoose::DatabaseClient::searchAccountByName(const QString& name)
{
    QMutexLocker locker(&mtx);  // 确保线程安全

    try {
        // 准备 SQL 查询
        QString sql = QString("SELECT * FROM account WHERE name = ?");

        // 创建 SQL 语句对象
        sql::PreparedStatement* pstmt = con->prepareStatement(sql.toStdString());

        // 绑定参数
        pstmt->setString(1, name.toStdString());

        // 执行查询
        sql::ResultSet* res = pstmt->executeQuery();

        // 检查是否有查询结果
        if (res->next()) {
            // 将查询结果映射到 Account 结构体
            btyGoose::data::Account account;
            account.uuid = QString::fromStdString(res->getString("uuid"));
            account.phone = QString::fromStdString(res->getString("phone"));
            account.name = QString::fromStdString(res->getString("name"));
            account.password = QString::fromStdString(res->getString("password"));
            account.nickname = QString::fromStdString(res->getString("nickname"));
            account.level = static_cast<btyGoose::data::Account::Level>(res->getInt("level"));
            account.type = static_cast<btyGoose::data::Account::Type>(res->getInt("type"));
            account.balance = res->getDouble("balance");

            // 处理 BLOB 数据
            std::istream* blobStream = res->getBlob("icon_data");
            if (blobStream) {
                std::ostringstream oss;
                oss << blobStream->rdbuf();
                account.icon = QByteArray::fromStdString(oss.str());
            }

            // 清理资源
            delete res;
            delete pstmt;

            return account;
        }

        // 如果没有找到记录，返回一个空账户
        delete res;
        delete pstmt;
        return btyGoose::data::Account();
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error while searching account by name: " << e.what();
        return btyGoose::data::Account();
    }
}

btyGoose::data::Account btyGoose::DatabaseClient::searchAccountByPhone(const QString& phone)
{
    QMutexLocker locker(&mtx);  // 确保线程安全

    try {
        // 准备 SQL 查询
        QString sql = QString("SELECT * FROM account WHERE phone = ?");

        // 创建 SQL 语句对象
        sql::PreparedStatement* pstmt = con->prepareStatement(sql.toStdString());

        // 绑定参数
        pstmt->setString(1, phone.toStdString());

        // 执行查询
        sql::ResultSet* res = pstmt->executeQuery();

        // 检查是否有查询结果
        if (res->next()) {
            // 将查询结果映射到 Account 结构体
            btyGoose::data::Account account;
            account.uuid = QString::fromStdString(res->getString("uuid"));
            account.phone = QString::fromStdString(res->getString("phone"));
            account.name = QString::fromStdString(res->getString("name"));
            account.password = QString::fromStdString(res->getString("password"));
            account.nickname = QString::fromStdString(res->getString("nickname"));
            account.level = static_cast<btyGoose::data::Account::Level>(res->getInt("level"));
            account.type = static_cast<btyGoose::data::Account::Type>(res->getInt("type"));
            account.balance = res->getDouble("balance");

            // 处理 BLOB 数据
            std::istream* blobStream = res->getBlob("icon_data");
            if (blobStream) {
                std::ostringstream oss;
                oss << blobStream->rdbuf();
                account.icon = QByteArray::fromStdString(oss.str());
            }

            // 清理资源
            delete res;
            delete pstmt;

            return account;
        }

        // 如果没有找到记录，返回一个空账户
        delete res;
        delete pstmt;
        return btyGoose::data::Account();
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error while searching account by phone: " << e.what();
        return btyGoose::data::Account();
    }
}

bool btyGoose::DatabaseClient::delAccountByID(const QString& id)
{
    return false;
}

bool btyGoose::DatabaseClient::addDish(const data::Dish&dish)
{
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "INSERT INTO dishes (uuid, merchant_id, name, icon_path, description, base_price, price_factor, is_delete,merchant_name) "
            "VALUES (?, ?, ?, ?, ?, ?, ?, ?,?)"
        );
        pstmt->setString(1, dish.uuid.toStdString());
        pstmt->setString(2, dish.merchant_id.toStdString());
        pstmt->setString(3, dish.name.toStdString());
        pstmt->setString(4, dish.icon_path.toStdString());
        pstmt->setString(5, dish.description.toStdString());
        pstmt->setDouble(6, dish.base_price);
        pstmt->setDouble(7, dish.price_factor);
        pstmt->setBoolean(8, dish.is_delete);
        pstmt->setString(9, dish.merchant_name.toStdString());
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error inserting dish: " << e.what();
        return false;
    }
}

bool btyGoose::DatabaseClient::updateDish(const data::Dish&dish)
{
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "UPDATE dishes SET name = ?, icon_path = ?, description = ?, base_price = ?, price_factor = ?, is_delete = ?,merchant_name = ? "
            "WHERE uuid = ?"
        );
        //qDebug() << dish.name;
        pstmt->setString(1, dish.name.toStdString());
        pstmt->setString(2, dish.icon_path.toStdString());
        pstmt->setString(3, dish.description.toStdString());
        pstmt->setDouble(4, dish.base_price);
        pstmt->setDouble(5, dish.price_factor);
        pstmt->setBoolean(6, dish.is_delete);
        pstmt->setString(7, dish.merchant_name.toStdString());
        pstmt->setString(8, dish.uuid.toStdString());
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error updating dish: " << e.what();
        return false;
    }
}

btyGoose::data::Dish btyGoose::DatabaseClient::searchDishByID(const QString& id)
{
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM dishes WHERE uuid = ?"
        );
        pstmt->setString(1, id.toStdString());
        sql::ResultSet* res = pstmt->executeQuery();
        data::Dish dish;
        if (res->next()) {
            dish.uuid = QString::fromStdString(res->getString("uuid"));
            dish.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            dish.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            dish.name = QString::fromStdString(res->getString("name"));
            dish.icon_path = QString::fromStdString(res->getString("icon_path"));
            dish.description = QString::fromStdString(res->getString("description"));
            dish.base_price = res->getDouble("base_price");
            dish.price_factor = res->getDouble("price_factor");
            dish.is_delete = res->getBoolean("is_delete");
        }
        delete res;
        delete pstmt;
        return dish;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error searching dish by ID: " << e.what();
        return data::Dish();  // 返回默认空的 Dish 对象
    }
}

QList<btyGoose::data::Dish> btyGoose::DatabaseClient::getAllDishList()
{
    QMutexLocker locker(&mtx);
    QList<data::Dish> dishList;
    try {
        sql::Statement* stmt = con->createStatement();
        sql::ResultSet* res = stmt->executeQuery("SELECT * FROM dishes");
        while (res->next()) {
            data::Dish dish;
            dish.uuid = QString::fromStdString(res->getString("uuid"));
            dish.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            dish.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            dish.name = QString::fromStdString(res->getString("name"));
            dish.icon_path = QString::fromStdString(res->getString("icon_path"));
            dish.description = QString::fromStdString(res->getString("description"));
            dish.base_price = res->getDouble("base_price");
            dish.price_factor = res->getDouble("price_factor");
            dish.is_delete = res->getBoolean("is_delete");
            if(dish.is_delete == false)
                dishList.append(dish);
        }
        delete res;
        delete stmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error getting all dishes: " << e.what();
    }
    return dishList;
}



QList<btyGoose::data::Dish> btyGoose::DatabaseClient::getDishListByMerchant(const QString& id)
{
    QMutexLocker locker(&mtx);
    QList<data::Dish> dishList;
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM dishes WHERE merchant_id = ?"
        );
        pstmt->setString(1, id.toStdString());
        sql::ResultSet* res = pstmt->executeQuery();
        while (res->next()) {
            data::Dish dish;
            dish.uuid = QString::fromStdString(res->getString("uuid"));
            dish.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            dish.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            dish.name = QString::fromStdString(res->getString("name"));
            dish.icon_path = QString::fromStdString(res->getString("icon_path"));
            dish.description = QString::fromStdString(res->getString("description"));
            dish.base_price = res->getDouble("base_price");
            dish.price_factor = res->getDouble("price_factor");
            dish.is_delete = res->getBoolean("is_delete");
            if(dish.is_delete == false)
                dishList.append(dish);
        }
        delete res;
        delete pstmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error getting dishes by merchant: " << e.what();
    }
    return dishList;
}

bool btyGoose::DatabaseClient::delDishByID(const QString& id)
{
    QMutexLocker locker(&mtx);
    try {
        // 更新 is_delete 字段为 true，表示菜品被删除
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "UPDATE dishes SET is_delete = ? WHERE uuid = ?"
        );
        pstmt->setBoolean(1, true);  // 设置 is_delete 为 true
        pstmt->setString(2, id.toStdString());  // 根据 uuid 找到菜品
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error logically deleting dish by ID: " << e.what();
        return false;
    }
}

//

// 添加订单
bool DatabaseClient::addOrder(const data::Order& order) {
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "INSERT INTO orders (uuid, merchant_id, merchant_name, consumer_id, consumer_name, time, level, pay, status,sum) "
            "VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?,?)"
        );
        pstmt->setString(1, order.uuid.toStdString());
        pstmt->setString(2, order.merchant_id.toStdString());
        pstmt->setString(3, order.merchant_name.toStdString());
        pstmt->setString(4, order.consumer_id.toStdString());
        pstmt->setString(5, order.consumer_name.toStdString());
        pstmt->setString(6, order.time.toStdString());
        pstmt->setInt(7, order.level);
        pstmt->setDouble(8, order.pay);
        pstmt->setInt(9, static_cast<int>(order.status));  // Status 转换为 int
        pstmt->setInt(10, order.sum);  // Status 转换为 int
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error adding order: " << e.what();
        return false;
    }
}

// 更新订单
bool DatabaseClient::updateOrder(const data::Order& order) {
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "UPDATE orders SET merchant_id = ?, merchant_name = ?, consumer_id = ?, consumer_name = ?, time = ?, "
            "level = ?, pay = ?, status = ? WHERE uuid = ?"
        );
        pstmt->setString(1, order.merchant_id.toStdString());
        pstmt->setString(2, order.merchant_name.toStdString());
        pstmt->setString(3, order.consumer_id.toStdString());
        pstmt->setString(4, order.consumer_name.toStdString());
        pstmt->setString(5, order.time.toStdString());
        pstmt->setInt(6, order.level);
        pstmt->setDouble(7, order.pay);
        pstmt->setInt(8, static_cast<int>(order.status));  // Status 转换为 int
        pstmt->setString(9, order.uuid.toStdString());
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error updating order: " << e.what();
        return false;
    }
}

// 更新订单状态
bool DatabaseClient::updateOrder(const QString& id, const data::Order::Status stat) {
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "UPDATE orders SET status = ? WHERE uuid = ?"
        );
        pstmt->setInt(1, static_cast<int>(stat));  // Status 转换为 int
        pstmt->setString(2, id.toStdString());
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error updating order status: " << e.what();
        return false;
    }
}

// 查找订单
data::Order DatabaseClient::searchOrderByID(const QString& order_id) {
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM orders WHERE uuid = ?"
        );
        pstmt->setString(1, order_id.toStdString());
        sql::ResultSet* res = pstmt->executeQuery();
        data::Order order;
        if (res->next()) {
            order.uuid = QString::fromStdString(res->getString("uuid"));
            order.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            order.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            order.consumer_id = QString::fromStdString(res->getString("consumer_id"));
            order.consumer_name = QString::fromStdString(res->getString("consumer_name"));
            order.time = QString::fromStdString(res->getString("time"));
            order.level = static_cast<data::Account::Level>(res->getInt("level"));
            order.pay = res->getDouble("pay");
            order.status = static_cast<data::Order::Status>(res->getInt("status"));
        }
        delete res;
        delete pstmt;
        return order;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error searching order by ID: " << e.what();
        return data::Order();  // 返回默认空的 Order 对象
    }
}

// 获取商家的所有订单
QList<data::Order> DatabaseClient::getOrderListByMerchant(const QString& merchant_id) {
    QMutexLocker locker(&mtx);
    QList<data::Order> orderList;
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM orders WHERE merchant_id = ?"
        );
        pstmt->setString(1, merchant_id.toStdString());
        sql::ResultSet* res = pstmt->executeQuery();
        while (res->next()) {
            data::Order order;
            order.uuid = QString::fromStdString(res->getString("uuid"));
            order.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            order.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            order.consumer_id = QString::fromStdString(res->getString("consumer_id"));
            order.consumer_name = QString::fromStdString(res->getString("consumer_name"));
            order.time = QString::fromStdString(res->getString("time"));
            order.level = static_cast<data::Account::Level>(res->getInt("level"));
            order.pay = res->getDouble("pay");
            order.status = static_cast<data::Order::Status>(res->getInt("status"));
            order.sum = res->getInt("sum");
            orderList.append(order);
        }
        delete res;
        delete pstmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error getting orders by merchant: " << e.what();
    }
    return orderList;
}

QList<data::Order> btyGoose::DatabaseClient::getOrderListByMerchantWaiting(const QString& merchant_id)
{
    QMutexLocker locker(&mtx);
    QList<data::Order> orderList;
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM orders WHERE merchant_id = ? and status = ?"
        );
        pstmt->setString(1, merchant_id.toStdString());
        pstmt->setInt(2, static_cast<int>(data::Order::Status::WAITING));
        qDebug() << merchant_id <<"|"<< static_cast<int>(data::Order::Status::WAITING);
        sql::ResultSet* res = pstmt->executeQuery();
        while (res->next()) {
            data::Order order;
            order.uuid = QString::fromStdString(res->getString("uuid"));
            order.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            order.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            order.consumer_id = QString::fromStdString(res->getString("consumer_id"));
            order.consumer_name = QString::fromStdString(res->getString("consumer_name"));
            order.time = QString::fromStdString(res->getString("time"));
            order.level = static_cast<data::Account::Level>(res->getInt("level"));
            order.pay = res->getDouble("pay");
            order.status = static_cast<data::Order::Status>(res->getInt("status"));
            order.sum = res->getInt("sum");
            orderList.append(order);
        }
        delete res;
        delete pstmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error getting orders by merchant: " << e.what();
    }
    return orderList;
}

QList<data::Order> btyGoose::DatabaseClient::getOrderListByConsumer(const QString& consumer_id)
{
    QMutexLocker locker(&mtx);
    QList<data::Order> orderList;
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM orders WHERE consumer_id = ?"
        );
        pstmt->setString(1, consumer_id.toStdString());
        sql::ResultSet* res = pstmt->executeQuery();
        while (res->next()) {
            data::Order order;
            order.uuid = QString::fromStdString(res->getString("uuid"));
            order.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            order.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            order.consumer_id = QString::fromStdString(res->getString("consumer_id"));
            order.consumer_name = QString::fromStdString(res->getString("consumer_name"));
            order.time = QString::fromStdString(res->getString("time"));
            order.level = static_cast<data::Account::Level>(res->getInt("level"));
            order.pay = res->getDouble("pay");
            order.status = static_cast<data::Order::Status>(res->getInt("status"));
            order.sum = res->getInt("sum");
            orderList.append(order);
        }
        delete res;
        delete pstmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error getting orders by merchant: " << e.what();
    }
    return orderList;
}

QList<data::Order> btyGoose::DatabaseClient::getAllOrderList()
{
    QMutexLocker locker(&mtx);
    QList<data::Order> orderList;
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM orders "
        );
        sql::ResultSet* res = pstmt->executeQuery();;
        while (res->next()) {
            data::Order order;
            order.uuid = QString::fromStdString(res->getString("uuid"));
            order.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            order.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            order.consumer_id = QString::fromStdString(res->getString("consumer_id"));
            order.consumer_name = QString::fromStdString(res->getString("consumer_name"));
            order.time = QString::fromStdString(res->getString("time"));
            order.level = static_cast<data::Account::Level>(res->getInt("level"));
            order.pay = res->getDouble("pay");
            order.status = static_cast<data::Order::Status>(res->getInt("status"));
            order.sum = res->getInt("sum");
            orderList.append(order);
        }
        delete res;
        delete pstmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error getting orders by merchant: " << e.what();
    }
    return orderList;
}

// 删除订单
bool DatabaseClient::delOrderByID(const QString& order_id) {
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "DELETE FROM orders WHERE uuid = ?"
        );
        pstmt->setString(1, order_id.toStdString());
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error deleting order by ID: " << e.what();
        return false;
    }
}



int btyGoose::DatabaseClient::clearOverTimeOrder()
{
    return 0;
}

// 添加订单菜品
bool DatabaseClient::addOrderDishesByID(const QString& order_id, const QList<data::OrderDish>& list) {
    QMutexLocker locker(&mtx);
    try {
        for (const auto& dish : list) {
            sql::PreparedStatement* pstmt = con->prepareStatement(
                "INSERT INTO orderDish (order_id, dish_id, merchant_id, name, dish_price, count) "
                "VALUES (?, ?, ?, ?, ?, ?)"
            );
            pstmt->setString(1, order_id.toStdString());
            pstmt->setString(2, dish.dish_id.toStdString());
            pstmt->setString(3, dish.merchant_id.toStdString());
            pstmt->setString(4, dish.name.toStdString());
            pstmt->setDouble(5, dish.dish_price);
            pstmt->setInt(6, dish.count);
            pstmt->executeUpdate();
            delete pstmt;
        }
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error adding order dishes: " << e.what();
        return false;
    }
}

// 删除订单菜品
bool DatabaseClient::delOrderDishesByID(const QString& order_id) {
    QMutexLocker locker(&mtx);
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "DELETE FROM orderDish WHERE order_id = ?"
        );
        pstmt->setString(1, order_id.toStdString());
        pstmt->executeUpdate();
        delete pstmt;
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error deleting order dishes by order ID: " << e.what();
        return false;
    }
}

// 获取订单菜品列表
QList<data::OrderDish> DatabaseClient::getOrderDishesByID(const QString& order_id) {
    QMutexLocker locker(&mtx);
    QList<data::OrderDish> dishList;
    try {
        sql::PreparedStatement* pstmt = con->prepareStatement(
            "SELECT * FROM orderDish WHERE order_id = ?"
        );
        pstmt->setString(1, order_id.toStdString());
        sql::ResultSet* res = pstmt->executeQuery();

        while (res->next()) {
            data::OrderDish dish;
            dish.order_id = QString::fromStdString(res->getString("order_id"));
            dish.dish_id = QString::fromStdString(res->getString("dish_id"));
            dish.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            dish.name = QString::fromStdString(res->getString("name"));
            dish.dish_price = res->getDouble("dish_price");
            dish.count = res->getInt("count");
            dishList.append(dish);
        }

        delete res;
        delete pstmt;
    }
    catch (sql::SQLException& e) {
        qDebug() << "Error fetching order dishes by order ID: " << e.what();
    }
    return dishList;
}

bool btyGoose::DatabaseClient::addHistory(const data::Order& order)
{
    QMutexLocker locker(&mtx);  // 使用互斥锁保护数据库操作

    try {
        // 插入 SQL 语句
        std::string query = "INSERT INTO history (time, uuid, merchant_id, merchant_name, "
            "consumer_id, consumer_name, level, pay, status,sum) "
            "VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?,?)";

        // 创建准备语句
        std::unique_ptr<sql::PreparedStatement> pstmt(con->prepareStatement(query));

        // 设置参数
        pstmt->setString(1, order.time.toStdString());
        pstmt->setString(2, order.uuid.toStdString());
        pstmt->setString(3, order.merchant_id.toStdString());
        pstmt->setString(4, order.merchant_name.toStdString());
        pstmt->setString(5, order.consumer_id.toStdString());
        pstmt->setString(6, order.consumer_name.toStdString());
        pstmt->setInt(7, static_cast<int>(order.level));
        pstmt->setDouble(8, order.pay);
        pstmt->setInt(9, static_cast<int>(order.status));
        pstmt->setInt(10, order.sum);

        // 执行插入操作
        pstmt->executeUpdate();
        return true;
    }
    catch (sql::SQLException& e) {
        qDebug() << "插入历史记录失败：" << e.what();
        return false;
    }
}

QList<data::Order> btyGoose::DatabaseClient::getAllHistoryList()
{
    QMutexLocker locker(&mtx);  // 使用互斥锁保护数据库操作
    QList<data::Order> orderList;

    try {
        // 查询所有历史记录
        std::unique_ptr<sql::ResultSet> res(stmt->executeQuery("SELECT * FROM history"));

        // 遍历查询结果
        while (res->next()) {
            data::Order order;
            order.time = QString::fromStdString(res->getString("time"));
            order.uuid = QString::fromStdString(res->getString("uuid"));
            order.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            order.merchant_name = QString::fromStdString(res->getString("merchant_name"));
            order.consumer_id = QString::fromStdString(res->getString("consumer_id"));
            order.consumer_name = QString::fromStdString(res->getString("consumer_name"));
            order.level = static_cast<data::Account::Level>(res->getInt("level"));
            order.pay = res->getDouble("pay");
            order.status = static_cast<data::Order::Status>(res->getInt("status"));
            order.sum = res->getInt("sum");

            orderList.append(order);  // 添加到结果列表
        }
    }
    catch (sql::SQLException& e) {
        qDebug() << "查询历史记录失败：" << e.what();
    }

    return orderList;
}

bool DatabaseClient::addHistoryDishesByID(const QString& order_id, const QList<data::OrderDish>& list)
{
    try
    {
        // Start a transaction
        con->setAutoCommit(false);

        // Prepare the SQL query for inserting history dish records
        std::string query = "INSERT INTO historyDish(order_id, dish_id, merchant_id, name, dish_price, count) "
            "VALUES (?, ?, ?, ?, ?, ?)";

        // Prepare the statement
        std::unique_ptr<sql::PreparedStatement> pstmt(con->prepareStatement(query));

        for (const auto& dish : list)
        {
            // Set the parameters for the prepared statement
            pstmt->setString(1, order_id.toStdString());  // order_id
            pstmt->setString(2, dish.dish_id.toStdString());  // dish_id
            pstmt->setString(3, dish.merchant_id.toStdString());  // merchant_id
            pstmt->setString(4, dish.name.toStdString());  // name
            pstmt->setDouble(5, dish.dish_price);  // dish_price
            pstmt->setInt(6, dish.count);  // count

            // Execute the prepared statement
            pstmt->executeUpdate();
        }

        // Commit the transaction
        con->commit();
        return true;
    }
    catch (sql::SQLException& e)
    {
        // Rollback in case of error
        con->rollback();
        qWarning() << "Error adding history dishes: " << e.what();
        return false;
    }
}

QList<data::OrderDish> DatabaseClient::getHistoryDishesByID(const QString& order_id)
{
    QList<data::OrderDish> dishList;

    try
    {
        // Prepare the SQL query to select dishes by order_id
        std::string query = "SELECT dish_id, merchant_id, name, dish_price, count FROM historyDish WHERE order_id = ?";

        // Prepare the statement
        std::unique_ptr<sql::PreparedStatement> pstmt(con->prepareStatement(query));
        pstmt->setString(1, order_id.toStdString());

        // Execute the query
        std::unique_ptr<sql::ResultSet> res(pstmt->executeQuery());

        // Iterate through the result set and populate the dish list
        while (res->next())
        {
            data::OrderDish dish;
            dish.dish_id = QString::fromStdString(res->getString("dish_id"));
            dish.merchant_id = QString::fromStdString(res->getString("merchant_id"));
            dish.name = QString::fromStdString(res->getString("name"));
            dish.dish_price = res->getDouble("dish_price");
            dish.count = res->getInt("count");

            dishList.append(dish);
        }
    }
    catch (sql::SQLException& e)
    {
        qWarning() << "Error retrieving history dishes: " << e.what();
    }

    return dishList;
}
```

## HTTP服务端类
`HTTPServer`负责处理`网络IO`和`控制数据库交互`，所以它**持有**`httplib`中的`httplib::Server`实例和上文`DatabaseClient`的实例

**配置文件** : 将配置信息写在文件`serverConfig.json`中,提供选择`HTTP`监听服务的`主机地址`和`端口号`

> serverConfig.json
```C++
{
    "host": "127.0.0.1",
    "port": 80
}
```

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

> HTTPServer.h
```C++
#pragma once
#include "httplib.h"
#include "DatabaseClient.h"


namespace btyGoose
{

class HTTPServer
{
	class HTTPException
	{
	private:
		QString message;
	public:
		HTTPException(const QString& msg = "a problem")
			:message(msg)
		{}

		QString what()const {
			return message;
		}
	};
public:
	HTTPServer();

	bool loadConfig();
	void saveConfig();

	void initTestAPI();
	void initAccountAPI();
	bool AuthenticateAuthCode(const QString& phone,const QString&auth_code);	//这里留着接入第三方API
	void initConsumerAPI();
	void initMerchantAPI();
	void initAdminAPI();

	bool getPay(double pay);
	//工具函数
	QString DishListToJsonArray(const QList<data::Dish>& dishList);
	QList<data::Dish> DishListFromJsonArray(const QString& jsonString);

	QString OrderListToJsonArray(const QList<data::Order>&orderList);
	QList<data::Order> OrderListFromJsonArray(const QString& jsonString);

	QString OrderDishListToJsonArray(const QList<data::OrderDish>&dishList);


private:
	httplib::Server svr;
	DatabaseClient db;
	QString host = "127.0.0.1";
	int port = 80;
};
}



```

> HTTPServer.cpp
```C++
#include "HTTPServer.h"

btyGoose::HTTPServer::HTTPServer()
{
    loadConfig();
    initTestAPI();
    initAccountAPI();
    initConsumerAPI();
    initMerchantAPI();
    initAdminAPI();
    svr.listen(host.toStdString(), port);
}

bool btyGoose::HTTPServer::loadConfig()
{

    QFile file("serverConfig.json");
    if (file.open(QIODevice::ReadOnly)) {
        QByteArray data = file.readAll();
        QJsonDocument doc = QJsonDocument::fromJson(data);

        if (doc.isObject()) {
            QJsonObject jsonObject = doc.object();
            host = jsonObject["host"].toString();
            port = jsonObject["port"].toInt();
            
            return true;
        }
    }
    //反序列化失败
    saveConfig();
    return false;
}

void btyGoose::HTTPServer::saveConfig()
{
    QFile file("serverConfig.json");
    if (file.open(QIODevice::WriteOnly)) {
        QJsonObject jsonObject;
        jsonObject["host"] = host;
        jsonObject["port"] = port;

        QJsonDocument doc(jsonObject);
        file.write(doc.toJson());
        file.close();
    }
    else
    {
        std::cerr << "server配置文件打开错误";
    }
}

//model

/*
 svr.Post("", [this](const httplib::Request& req, httplib::Response& res) {
        qDebug() << " get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }
        }catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

    });
*/

void btyGoose::HTTPServer::initTestAPI()
{
    svr.Get("/ping", [=](const httplib::Request& req, httplib::Response& res) {
        qDebug() << "/ping";
        res.body = "pong";
        res.set_header("Content-Type", "application/text;charset=UTF-8");
        });
}

void btyGoose::HTTPServer::initAccountAPI()
{
    //用户名登录
    svr.Post("/account/login/username", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/account/login/username get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }
            data::Account record_acc = db.searchAccountByName(jsonObj["name"].toString());
            if (!record_acc.name.isEmpty() && jsonObj["password"].toString() == record_acc.password)
            {
                res.status = 200;
                resJson["message"] = "登录认证成功!";
                resJson["success"] = true;
                resJson["account"] = record_acc.toJson();
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                res.set_header("Content-Type", "application/json;charset=UTF-8");

            }
            else
            {
                res.status = 200;
                resJson["message"] = "登录认证失败!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                res.set_header("Content-Type", "application/json;charset=UTF-8");
            }
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
            qDebug() << e.what();
        }

        LOG() << "login by name done";
    });

    svr.Post("/account/login/phone", [this](const httplib::Request& req, httplib::Response& res) {
        qDebug() << "/account/login/phone get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }
            data::Account record_acc = db.searchAccountByPhone(jsonObj["phone"].toString());
            if (!record_acc.phone.isEmpty() && jsonObj["password"].toString() == record_acc.password)
            {
                res.status = 200;
                resJson["message"] = "登录认证成功!";
                resJson["success"] = true;
                resJson["account"] = record_acc.toJson();
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                res.set_header("Content-Type", "application/json;charset=UTF-8");
            }
            else
            {
                res.status = 200;
                resJson["message"] = "登录认证失败!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                res.set_header("Content-Type", "application/json;charset=UTF-8");
            }
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    //账户注册
    svr.Post("/account/register", [this](const httplib::Request& req, httplib::Response& res) {
        qDebug() << "/account/register get a post!";
        LOG() << "开始注册";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        res.set_header("Content-Type", "application/json;charset=UTF-8");
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            data::Account acc;
            acc.name = jsonObj["name"].toString();
            acc.password = jsonObj["password"].toString();  // 直接使用传入的已加密密码
            acc.phone = jsonObj["phone"].toString();
            acc.nickname = jsonObj["nickname"].toString();
            acc.type = (data::Account::Type)jsonObj["type"].toInt();
            acc.level = data::Account::Level::MEMBER;
            acc.uuid = util::makeId("A");

            if (acc.uuid.isEmpty())
            {
                res.status = 200;
                resJson["message"] = "uuid生成错误!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
            if (!AuthenticateAuthCode(acc.phone, jsonObj["auth_code"].toString()))
            {
                res.status = 200;
                resJson["message"] = "验证码错误!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
            if (!db.searchAccountByName(acc.name).name.isEmpty())
            {
                //账户名重复
                LOG() << "用户名已存在:" << acc.name;
                res.status = 200;
                resJson["message"] = "账户名已存在!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
            LOG() << "用户名查重通过";
            if (!db.searchAccountByPhone(acc.phone).name.isEmpty())
            {
                //电话重复
                res.status = 200;
                LOG() << "电话查重失败";
                resJson["message"] = "手机号已被占用!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
            LOG() << "电话查重通过";

            if (db.searchAccountByID(acc.uuid).name.isEmpty())
            {
                if (!db.addAccount(acc))
                {
                    throw HTTPException("databse add account failed");
                }
                LOG() << "成功向数据库插入新账号";
                resJson["success"] = true;
                resJson["message"] = "账号创建成功";
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
            }
            else
            {
                //uuid生成错误
                res.status = 200;
                resJson["message"] = "uuid生成错误!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    ////冗余
    //svr.Post("/account/consumer/update", [this](const httplib::Request& req, httplib::Response& res) {
    //    qDebug() << "/account/consumer/update get a post!";
    //    std::string jsonStr = req.body;
    //    QString qJsonString = QString::fromStdString(jsonStr);
    //    QJsonObject jsonObj;
    //    // 解析 JSON 字符串
    //    QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
    //    if (doc.isObject()) {
    //        jsonObj = doc.object();
    //    }
    //    else
    //    {
    //        qDebug() << "Invalid Json" << jsonStr;
    //        jsonObj = QJsonObject();
    //    }
    //    QJsonObject resJson;
    //    try {
    //        if (jsonObj.isEmpty())
    //        {
    //            throw HTTPException("Json Serialization failed");
    //        }

    //        QString uuid = jsonObj["uuid"].toString();
    //        double pay = jsonObj["pay"].toDouble();
    //        QString level = jsonObj["level"].toString();
    //        data::Account acc = db.searchAccountByID(uuid);
    //        bool ret = getPay(pay);
    //        if (ret == true)
    //        {
    //            //支付成功
    //            if (level == "VIP") acc.level = data::Account::Level::VIP;
    //            else if (level == "SUVIP") acc.level = data::Account::Level::SUVIP;
    //            else throw HTTPException("未知Level等级" + level);
    //     
    //        }
    //        else
    //        {
    //            QJsonObject resJson;
    //            resJson["success"] = false;
    //            resJson["message"] = "用户升级失败";
    //            QJsonDocument doc(resJson);
    //            res.body = doc.toJson().toStdString();
    //        }
    //        
    //    }
    //    catch (const HTTPException& e)
    //    {
    //        res.status = 500;
    //        resJson["success"] = false;
    //        resJson["message"] = e.what();
    //        QJsonDocument doc(resJson);
    //        res.body = doc.toJson().toStdString();
    //        res.set_header("Content-Type", "application/json;charset=UTF-8");
    //        qDebug() << e.what();
    //    }

    //    });

    svr.Post("/account/update/level", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/account/update/level get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }
            data::Account record_acc = db.searchAccountByID(jsonObj["id"].toString());
            QString level = jsonObj["level"].toString();
            qDebug() << "level change to: " << level;
            if (level == "VIP")
            {
                record_acc.level = data::Account::Level::VIP;
            }            
            else if (level == "SUVIP")
            {
                record_acc.level = data::Account::Level::SUVIP;
            }

            db.updateAccount(record_acc);
            record_acc = db.searchAccountByID(jsonObj["id"].toString());
            resJson["level"] = static_cast<int>(record_acc.level);
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
            qDebug() << e.what();
        }

        LOG() << "login by name done";
        });

    svr.Post("/account/update/nickname", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/account/update/nickname get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }
            data::Account record_acc = db.searchAccountByID(jsonObj["id"].toString());
            QString nickname = jsonObj["nickname"].toString();
            qDebug() << "nickname change to: " << nickname;
            record_acc.nickname = nickname;
            db.updateAccount(record_acc);
            record_acc = db.searchAccountByID(jsonObj["id"].toString());
            resJson["nickname"] = record_acc.nickname;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
            qDebug() << e.what();
        }

        LOG() << "login by name done";
        });
}

bool btyGoose::HTTPServer::AuthenticateAuthCode(const QString& phone, const QString& auth_code)
{
    ///////////////////////////
    // TODO
    // ///////////////////
    //这里就不具体实现了
    if (auth_code == "888888")
    {
        return true;
    }
    else
    {
        return false;
    }
}


void btyGoose::HTTPServer::initConsumerAPI()
{

    svr.Post("/consumer/order/detail/dishlist", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/order/detail/dishlist get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            QList<data::OrderDish> dishList = db.getOrderDishesByID(jsonObj["order_id"].toString());
            LOG() << "order:" << jsonObj["order_id"].toString() << "find dishes" << dishList.size();
            QString dishListJson = OrderDishListToJsonArray(dishList);
            QJsonObject resJson;
            resJson["dish_list"] = dishListJson;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });


    svr.Post("/consumer/dish/list", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/dish/list get a post!";
        QList<data::Dish> dishList = db.getAllDishList();
        QString dishListJson = DishListToJsonArray(dishList);
        QJsonObject resJson;
        resJson["dish_list"] = dishListJson;
        QJsonDocument doc(resJson);
        res.body = doc.toJson().toStdString();
        res.set_header("Content-Type", "application/json;charset=UTF-8");
        });

    svr.Post("/consumer/dish/dishInfo", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/dish/dishInfo get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            QString dish_id = jsonObj["dish_id"].toString();
            data::Dish dish = db.searchDishByID(dish_id);
            resJson["dish"] = dish.toJson();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    //订单生成
    svr.Post("/consumer/order/generate", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/order/generate get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            data::Order order;
            order.uuid = util::makeId("O");
            order.merchant_id = jsonObj["merchant_id"].toString();
            order.merchant_name = jsonObj["merchant_name"].toString();
            order.consumer_id = jsonObj["consumer_id"].toString();
            order.consumer_name = jsonObj["consumer_name"].toString();
            order.level = static_cast<data::Account::Level>(jsonObj["level"].toInt());
            order.pay = jsonObj["pay"].toDouble();
            order.sum = jsonObj["count"].toInt();
            order.time = QString::number(util::getSecTime());
            order.status = data::Order::Status::UNPAYED;

            LOG() << "生成一个order,merchant id:" << order.merchant_id;

            bool ok = db.addOrder(order);
            Q_ASSERT(ok);
            QList<data::OrderDish> dish_list;
            QJsonArray jsonArr = jsonObj["dish_arr"].toArray();
            for (const QJsonValue& value : jsonArr)
            {
                LOG() << "生成一个OrderDish";
                QJsonObject obj = value.toObject();
                data::OrderDish dish;
                dish.order_id = order.uuid;
                dish.merchant_id = order.merchant_id;
                dish.name = obj["dish_name"].toString();
                dish.dish_price = obj["dish_price"].toDouble();
                dish.count = obj["count"].toInt();
                dish.dish_id = obj["dish_id"].toString();
                dish_list.append(dish);
            }
            ok = db.addOrderDishesByID(order.uuid, dish_list);
            Q_ASSERT(ok);

            //构建返回报文
            resJson["order"] = order.toJson();

            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/consumer/order/list", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/order/list get a post!";

        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            QList<data::Order> orderList = db.getOrderListByConsumer(jsonObj["consumer_id"].toString());
            LOG() << "consumer:" << jsonObj["consumer_id"].toString() << "find orders" << orderList.size();
            QString dishListJson = OrderListToJsonArray(orderList);
            QJsonObject resJson;
            resJson["order_list"] = dishListJson;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/consumer/order/pay/confirm", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/order/pay/confirm get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }
            data::Order order = db.searchOrderByID(jsonObj["order_id"].toString());
            order.status = data::Order::Status::WAITING;
            db.updateOrder(order);
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json");
            qDebug() << e.what();
        }

        });

    svr.Post("/consumer/order/cancel", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/consumer/order/cancel get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            data::Order order = db.searchOrderByID(jsonObj["order_id"].toString());
            QString reason;
            if (order.status == data::Order::Status::UNPAYED)
            {
                order.status = data::Order::Status::CANCELED;
                db.updateOrder(order);
                reason = "订单取消成功";
            }
            else if (order.status == data::Order::Status::WAITING)
            {
                reason = "订单无法取消：已支付";
            }
            else
            {
                reason = "订单无法支付,订单状态码:" + QString::number(static_cast<int>(order.status));
            }

            resJson["reason"] = reason;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });
}

void btyGoose::HTTPServer::initMerchantAPI()
{
    svr.Post("/merchant/dish/list", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/dish/list get a post!";

        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            QList<data::Dish> dishList = db.getDishListByMerchant(jsonObj["merchant_id"].toString());
            LOG() << "merchant:" << jsonObj["merchant_id"].toString() << "find dishes" << dishList.size();
            QString dishListJson = DishListToJsonArray(dishList);
            QJsonObject resJson;
            resJson["dish_list"] = dishListJson;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    //菜品注册
    svr.Post("/merchant/dish/register", [this](const httplib::Request& req, httplib::Response& res) {
        qDebug() << "/merchant/dish/register get a post!";
        LOG() << "菜品开始注册";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        res.set_header("Content-Type", "application/json;charset=UTF-8");
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            data::Dish dish;
            dish.uuid = util::makeId("D");
            dish.name = jsonObj["name"].toString();
            dish.merchant_id = jsonObj["merchant_id"].toString();
            dish.merchant_name = jsonObj["merchant_name"].toString();
            dish.icon_path = jsonObj["icon_path"].toString();
            dish.description = jsonObj["introduction"].toString();
            dish.base_price = jsonObj["price"].toDouble();
            dish.price_factor = jsonObj["price_factor"].toDouble();
            dish.is_delete = false;

            if (dish.uuid.isEmpty())
            {
                res.status = 200;
                resJson["message"] = "uuid生成错误!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }

            if (db.searchDishByID(dish.uuid).name.isEmpty())
            {
                if (!db.addDish(dish))
                {
                    throw HTTPException("databse add dish failed");
                }
                LOG() << "成功向数据库插入新菜品"<<dish.name;
                resJson["success"] = true;
                resJson["message"] = "菜品创建成功";
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
            }
            else
            {
                //uuid生成错误
                res.status = 200;
                resJson["message"] = "uuid生成错误!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/merchant/dish/info", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/dish/info get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            QString dish_id = jsonObj["dish_id"].toString();
            data::Dish dish = db.searchDishByID(dish_id);
            resJson["dish"] = dish.toJson();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });



    //菜品注册
    svr.Post("/merchant/dish/update", [this](const httplib::Request& req, httplib::Response& res) {
        qDebug() << "/merchant/dish/update get a post!";
        LOG() << "菜品开始更新";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        res.set_header("Content-Type", "application/json;charset=UTF-8");
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            //qDebug() << jsonObj["dish_id"].toString();
            data::Dish dish = db.searchDishByID(jsonObj["dish_id"].toString());
            qDebug() << dish.uuid;
            dish.name = jsonObj["name"].toString();
            dish.merchant_id = jsonObj["merchant_id"].toString();
            dish.merchant_name = jsonObj["merchant_name"].toString();
            dish.icon_path = jsonObj["icon_path"].toString();
            dish.description = jsonObj["introduction"].toString();
            dish.base_price = jsonObj["price"].toDouble();
            dish.price_factor = jsonObj["price_factor"].toDouble();


            if (!dish.uuid.isEmpty())
            {
                if (!db.updateDish(dish))
                {
                    throw HTTPException("databse add dish failed");
                }
                LOG() << "成功向数据库更新新菜品" << dish.name;
                resJson["success"] = true;
                resJson["message"] = "菜品创建成功";
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
            }
            else
            {
      
                res.status = 200;
                resJson["message"] = "找不到菜品!";
                resJson["success"] = false;
                QJsonDocument doc(resJson);
                res.body = doc.toJson().toStdString();
                return;
            }
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/merchant/dish/del", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/dish/del get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }

            QString dish_id = jsonObj["dish_id"].toString();
            db.delDishByID(dish_id);
            resJson["success"] = true;
            resJson["message"] = "删除成功!";
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/merchant/order/list", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/order/list get a post!";

        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            QList<data::Order> orderList = db.getOrderListByMerchant(jsonObj["merchant_id"].toString());
            //QList<data::Order> orderList = db.getOrderListByMerchantWaiting(jsonObj["merchant_id"].toString());
            LOG() << "merchant:" << jsonObj["merchant_id"].toString() << "find orders" << orderList.size();
            QString dishListJson = OrderListToJsonArray(orderList);
            QJsonObject resJson;
            resJson["order_list"] = dishListJson;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/merchant/order/detail/dishlist", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/order/detail/dishlist get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            QList<data::OrderDish> dishList = db.getOrderDishesByID(jsonObj["order_id"].toString());
            LOG() << "order:" << jsonObj["order_id"].toString() << "find dishes" << dishList.size();
            QString dishListJson = OrderDishListToJsonArray(dishList);
            QJsonObject resJson;
            resJson["dish_list"] = dishListJson;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/merchant/order/accept", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/order/accept get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            data::Order order = db.searchOrderByID(jsonObj["order_id"].toString());
            order.status = data::Order::Status::SUCCESS;//修改状态
            db.updateOrder(order);
            //归档
            db.addHistory(order);
            db.addHistoryDishesByID(order.uuid, db.getOrderDishesByID(order.uuid));

            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });

    svr.Post("/merchant/order/reject", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/merchant/order/reject get a post!";
        std::string jsonStr = req.body;
        QString qJsonString = QString::fromStdString(jsonStr);
        QJsonObject jsonObj;
        // 解析 JSON 字符串
        QJsonDocument doc = QJsonDocument::fromJson(qJsonString.toUtf8());
        if (doc.isObject()) {
            jsonObj = doc.object();
        }
        else
        {
            qDebug() << "Invalid Json" << jsonStr;
            jsonObj = QJsonObject();
        }
        QJsonObject resJson;
        try {
            if (jsonObj.isEmpty())
            {
                throw HTTPException("Json Serialization failed");
            }


            data::Order order = db.searchOrderByID(jsonObj["order_id"].toString());
            order.status = data::Order::Status::REJECTED;//修改状态
            db.addHistory(order);
            db.updateOrder(order);//归档
            db.addHistoryDishesByID(order.uuid, db.getOrderDishesByID(order.uuid));

            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });


}

void btyGoose::HTTPServer::initAdminAPI()
{
    svr.Post("/admin/order/list", [this](const httplib::Request& req, httplib::Response& res) {
        LOG() << "/admin/order/list get a post!";

 
        QJsonObject resJson;
        try {
            QList<data::Order> orderList = db.getAllOrderList();
            LOG() << "admin:" << "find orders" << orderList.size();
            QString dishListJson = OrderListToJsonArray(orderList);
            QJsonObject resJson;
            resJson["order_list"] = dishListJson;
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
        }
        catch (const HTTPException& e)
        {
            res.status = 500;
            resJson["success"] = false;
            resJson["message"] = e.what();
            QJsonDocument doc(resJson);
            res.body = doc.toJson().toStdString();
            res.set_header("Content-Type", "application/json;charset=UTF-8");
            qDebug() << e.what();
        }

        });
}

bool btyGoose::HTTPServer::getPay(double pay)
{
    return false;
}

QString btyGoose::HTTPServer::DishListToJsonArray(const QList<data::Dish>& dishList)
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

QList<btyGoose::data::Dish> btyGoose::HTTPServer::DishListFromJsonArray(const QString& jsonString)
{
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
                dish.loadFromJson(QString(doc.toJson()).toStdString());
                dishList.append(dish);
            }
        }
    }

    return dishList;
}

QString btyGoose::HTTPServer::OrderListToJsonArray(const QList<data::Order>& orderList)
{
    QJsonArray jsonArray;

    // 遍历 orderList，将每个 order 对象的 JSON 添加到 jsonArray 中
    for (const auto& order : orderList)
    {
        // 将每个 order 对象转换为 JSON 字符串并加入到 JSON 数组中
        jsonArray.append(QJsonDocument::fromJson(order.toJson().toUtf8()).object());
    }

    // 将 QJsonArray 转换为 JSON 字符串
    QJsonDocument doc(jsonArray);
    return QString(doc.toJson(QJsonDocument::Compact));
}

QList<btyGoose::data::Order> btyGoose::HTTPServer::OrderListFromJsonArray(const QString& jsonString)
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

QString btyGoose::HTTPServer::OrderDishListToJsonArray(const QList<data::OrderDish>& dishList)
{
    QJsonArray jsonArray;

    // 遍历 orderList，将每个 order 对象的 JSON 添加到 jsonArray 中
    for (const auto& dish : dishList)
    {
        // 将每个 order 对象转换为 JSON 字符串并加入到 JSON 数组中
        jsonArray.append(QJsonDocument::fromJson(dish.toJson().toUtf8()).object());
    }

    // 将 QJsonArray 转换为 JSON 字符串
    QJsonDocument doc(jsonArray);
    return QString(doc.toJson(QJsonDocument::Compact));
}

```

 