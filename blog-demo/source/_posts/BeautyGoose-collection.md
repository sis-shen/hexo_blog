---
title: C++项目】美鹅外卖-总集介绍博客
date: 2025-01-15 21:36:41
tags:
---

# 项目介绍
为客户提供一个C/S架构的外卖平台，使用Qt框架和cpp-httplib 等模块实现其业务功能。能够服务消费者，商家和管理员的使用需求，目前支持的平台为Windows平台。

使用的技术点有:

+ QT界面框架
+ QT Json处理模块
+ cpp-httplib库
+ MySQL connector/c++

## 运行环境
*当前版本v1.0*

客户端: windows 11

服务端: windows 11
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

### 用户注册功能
用户可以使用唯一的***账号名和手机号**和进行账号注册

### 用户登录功能
用户可以使用**账号名或手机号**进行账号登录

### 商家菜品注册功能
商家可以使用菜品注册功能来注册新的菜品

### 商家菜品查看功能
商家可以看到自己的所有菜品列表，也能够看到某个菜品的详情

### 商家菜品修改功能
商家可以修改菜品的信息，也可以删除菜品

### 商家订单查看功能
商家能够看到自己的订单列表，也可以看到某个订单的详情

### 商家订单处理功能
商家能够选择接单或拒单

### 消费者查看菜品功能
消费者可以看到所有菜品列表，也可以看到某一个菜品的详情

### 消费者购物车添加功能
消费者能够将菜品添加到对应商家的购物车中

### 消费者查看购物车功能
消费者能够看到所有商家的购物车列表

### 消费者订单生成功能
消费者能够将指定的购物车生成为订单

### 消费者第三方支付功能
消费者能够使用第三方支付订单

### 消费者订单支付功能
消费者能够支付订单并支付完成的订单被商家看到

### 消费者提升优惠等级功能
消费者能够看到自己的优惠等级，并通过购买VIP的方式提升优惠等级

### 管理员日志查看功能
使用管理员账户登录后，可以看到所有的销售日志

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

### 数据库文件和连接账户
#### 建数据库
```sql
create database BeautyGoose;
use BeautyGoose;
```

#### 创建专门的连接用用户
```sql
set global validate_password_policy = 'LOW';
create user 'btyGooseUser'@'%' identified by 'BueatyGoose';
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

# 项目设计

# 客户端详细设计

## 版本信息
**当前版本**: v0.9测试版

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
[戳我🔗去github的release页面获取当前版本的源码](https://github.com/sis-shen/BeautyGoose/releases/tag/v0.9)

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
    this->setFixedSize(1600,900);
    this->setWindowIcon(QIcon(QPixmap(":/qsrc/icon.png")));
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

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

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
#include <QLabel>
#include <QMessageBox>
#include <QRegularExpression>
#include <QRegularExpressionValidator>
RegisterWidget::RegisterWidget(QWidget *parent)
    : QWidget{parent}
{
    this->setFixedSize(1600,900);
    QGridLayout* layout = new QGridLayout;

    int m = 150;
    layout->setContentsMargins(m,m,m,m);
    //第一行
    QLabel* label = new QLabel;
    label->setText("账户类型");
    layout->addWidget(label,0,0);
    consumerRadio = new QRadioButton;
    consumerRadio->setChecked(true);
    layout->addWidget(consumerRadio,0,1);

    label = new QLabel ;
    label->setText("我是消费者");
    layout->addWidget(label,0,2);
    merchantRadio = new QRadioButton;
    layout->addWidget(merchantRadio,0,3);
    label = new QLabel;
    label->setText("我是商家");
    layout->addWidget(label,0,4);

    //第二三四五六七八行
    label = new QLabel("用户名");
    layout->addWidget(label,1,0);
    nameInput = new QTextEdit;
    nameInput->setFixedHeight(50);
    layout->addWidget(nameInput,1,1,1,4);
    label = new QLabel("昵称");
    layout->addWidget(label,2,0);
    nicknameInput = new QTextEdit;
        nicknameInput->setFixedHeight(50);
    layout->addWidget(nicknameInput,2,1,1,4);
    label = new QLabel("密码");
    layout->addWidget(label,3,0);
    passwordInput = new QLineEdit;
        passwordInput->setFixedHeight(50);
    passwordInput->setEchoMode(QLineEdit::Password);
    layout->addWidget(passwordInput,3,1,1,4);
    label = new QLabel("重复密码");
    layout->addWidget(label,4,0);
    passwordRepeatInput = new QLineEdit;
        passwordRepeatInput->setFixedHeight(50);
        passwordRepeatInput->setEchoMode(QLineEdit::Password);
    layout->addWidget(passwordRepeatInput,4,1,1,4);
    label = new QLabel("手机号");
    layout->addWidget(label,5,0);
    phoneInput = new QTextEdit;
        phoneInput->setFixedHeight(50);
    layout->addWidget(phoneInput,5,1,1,4);
    label = new QLabel("短信验证码");
    layout->addWidget(label,6,0);
    auth_codeInput = new QTextEdit;
        auth_codeInput->setFixedHeight(50);
    auth_codeInput->setFixedWidth(100);
    layout->addWidget(auth_codeInput,6,1);

    getCodeBtn = new QPushButton("获取验证码");
    getCodeBtn->setFixedHeight(50);
    layout->addWidget(getCodeBtn,6,2);

    registerBtn = new QPushButton("注册");
    registerBtn->setFixedHeight(50);
    layout->addWidget(registerBtn,7,1,1,4);

    toNameLoginBtn = new QPushButton("转到用户名登录");
    layout->addWidget(toNameLoginBtn,8,2,1,2);
    toPhoneLoginBtn = new QPushButton("转到手机号登录");
    layout->addWidget(toPhoneLoginBtn,8,4,1,2);


    this->setLayout(layout);

    connect(registerBtn,&QPushButton::clicked,this,&RegisterWidget::registerSlot);
    connect(getCodeBtn,&QPushButton::clicked,this,&RegisterWidget::getCodeSlot);
    connect(toNameLoginBtn,&QPushButton::clicked,this,&RegisterWidget::toNameLoginSignal);
    connect(toPhoneLoginBtn,&QPushButton::clicked,this,&RegisterWidget::toPhoneLoginSignal);
}

bool RegisterWidget::check()
{
    QString name = nameInput->toPlainText();
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
    QString phone = phoneInput->toPlainText();
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
    QString code = auth_codeInput->toPlainText();
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
    input.name = nameInput->toPlainText();
    QString nickname = nicknameInput->toPlainText();
    if(nickname.isEmpty())
    {
        input.nickname = input.name;
    }
    else
    {
        input.nickname = nickname;
    }
    input.password = passwordInput->text();
    input.phone = phoneInput->toPlainText();
    input.auth_code = auth_codeInput->toPlainText();

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

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501201352391.png)

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

        QGridLayout* mainLayout = new QGridLayout;
        QWidget* mainW = new QWidget;
        this->setLayout(mainLayout);
        mainLayout->addWidget(mainW);
        mainW->setStyleSheet("QWidget { border: 2px solid black;}");
        viewUserInfo = new QPushButton("查看个人信息");
        viewUserInfo->setFixedSize(100,100);
        viewDishList = new QPushButton("查看菜品列表");
        viewCartList = new QPushButton("查看购物车列表");
        viewOrderList = new QPushButton("查看订单列表");
        viewVIP = new QPushButton("VIP");

        QGridLayout* layout = new QGridLayout;
        mainW->setLayout(layout);
        layout->addWidget(viewUserInfo,0,0);
        layout->addWidget(viewDishList,1,0);
        layout->addWidget(viewCartList,2,0);
        layout->addWidget(viewOrderList,3,0);
        layout->addWidget(viewVIP,4,0);

        connect(viewUserInfo,&QPushButton::clicked,this,&ConsumerNavWidget::toUserInfoSlot);
        connect(viewDishList,&QPushButton::clicked,this,&ConsumerNavWidget::toDishListSlot);
        connect(viewCartList,&QPushButton::clicked,this,&ConsumerNavWidget::toCartListSlot);
        connect(viewOrderList,&QPushButton::clicked,this,&ConsumerNavWidget::toOrderListSlot);
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
        viewUserInfo = new QPushButton("查看个人信息");
        viewUserInfo->setFixedSize(100,100);
        viewDishList = new QPushButton("查看菜品列表");
        viewOrderList = new QPushButton("查看订单列表");
        dishRegister = new QPushButton("注册菜品");

        QGridLayout* layout = new QGridLayout;
        mainW->setLayout(layout);
        layout->addWidget(viewUserInfo,0,0);
        layout->addWidget(viewDishList,1,0);
        layout->addWidget(viewOrderList,2,0);
        layout->addWidget(dishRegister,3,0);

        connect(viewUserInfo,&QPushButton::clicked,this,&MerchantNavWidget::toUserInfoSlot);
        connect(viewDishList,&QPushButton::clicked,this,&MerchantNavWidget::toDishListSlot);
        connect(viewOrderList,&QPushButton::clicked,this,&MerchantNavWidget::toOrderListSlot);
        connect(dishRegister,&QPushButton::clicked,this,&MerchantNavWidget::toDishRegisterWindowSlot);

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

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

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
#include "Nav.h"

#include "CoreData.h"
#include "DataCenterCoreData.h"

namespace dishList
{
struct DishItem:public QWidget
{
    Q_OBJECT;
public:
    using ptr = QSharedPointer<DishItem>;
    QString dish_id;
    QPushButton* dish_name;
    QLabel* single_picture;
    QLabel* single_price;


    DishItem(const btyGoose::data::Dish& dish)
        :dish_id(dish.uuid)
    {
        QGridLayout* layout = new QGridLayout;
        this->setLayout(layout);
        single_picture = new QLabel("图片");
        dish_name = new QPushButton(dish.name);
        QString price = "单价:";
        price += QString::number(dish.price_factor*dish.base_price);
        price += "元";
        single_price = new QLabel(price);

        single_picture->setFixedWidth(100);
        dish_name->setFixedWidth(100);
        single_price->setFixedWidth(100);


        QSpacerItem* spacer = new QSpacerItem(100,50);

        layout->addWidget(single_picture,0,0);
        layout->addWidget(dish_name,0,1);
        layout->addWidget(single_price,0,2);

        layout->addItem(spacer,0,3);


        connect(dish_name,&QPushButton::clicked,this,&DishItem::dishNameBtnSlot);
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
        merchant_name = new QLabel(lst[0].merchant_name);
        merchant_id = lst[0].merchant_id;
        //默认加三个
        // qDebug()<<"开始往缓冲区插入菜品";
        for(auto dish:lst)
        {
            list.push_back(DishItem::ptr(new DishItem(dish)));
        }
        // qDebug()<<"插入完成";

        QGridLayout* layout = new QGridLayout;
        this->setLayout(layout);
        layout->addWidget(merchant_name,0,0);

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
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202501201451370.png)

## 消费者菜品详情窗口实现
虽然我们把它称为窗口，但实际上我们使用的是`<QDialog>`类

可能是出于开发失误，或者有更多的考量，具体的界面实现还是继承了`QWidget`类封装在`ConsumerDishDetailWidget`中，而对窗口的实现，则是另外封装`ConsumerDishDetailWindow`类并持有`ConsumerDishDetailWidget`指针的方式来实现窗口。

可能是为了使窗口能够更方便地加入动态性，对于所有的窗口，`MainWidget`都存有它们的指针。

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

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
    this->setLayout(mwl);
    mwl->addWidget(MW);
    QGridLayout* layout = new QGridLayout;
    MW->setLayout(layout);
    MW->setStyleSheet("QWidget { border: 2px solid black; }");
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
    QGridLayout* ulayout = new QGridLayout;
    upW->setLayout(ulayout);
    ulayout->addWidget(ulW,0,0);
    ulayout->addWidget(urW,0,1);
    //左上角初始化
    ulW->setFixedWidth(200);
    QGridLayout* lLayout = new QGridLayout;
    ulW->setLayout(lLayout);
    //TODO 此处应改为用QICON
    iconTMP = new QLabel("图片");
    iconTMP->setFixedSize(200,200);

    QLabel* price = new QLabel("价格：" + QString::number(_dish->base_price*_dish->price_factor)+"元");
    price->setFixedSize(100,50);

    lLayout->addWidget(iconTMP,0,0);
    lLayout->addWidget(price,1,0);

    //右上角初始化
    QGridLayout* rLayout = new QGridLayout;
    urW->setLayout(rLayout);
    QLabel* name = new QLabel(_dish->name);
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

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

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

*阅读提示：若代码展开后过长，点击右侧目录可以跳转*

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

https://github.com/sis-shen/BeautyGoose/releases/tag/v0.9
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

