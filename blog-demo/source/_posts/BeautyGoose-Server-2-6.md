---
title: v2.6美鹅外卖平台Web框架替换更新
date: 2025-04-20 19:52:48
tags:
---

# Drogon简介
Drogon是一个基于C++17/20的Http应用框架,专门用于构建高效的Web应用和服务端程序
+ 网络层使用基于epoll(macOS/FreeBSD下是kqueue)的非阻塞IO框架，提供高并发、高性能的网络IO。
## 安装

```shell
#安装依赖（Ubuntu示例）
sudo apt install -y gcc g++ cmake libssl-dev libjsoncpp-dev uuid-dev zlib1g-dev
#克隆仓库
git clone https://github.com/drogonframework/drogon.git
cd drogon
git submodule update --init
#编译安装（约5-10分钟）
mkdir build
cd build
cmake ..
make -j$(nproc)
sudo make install
```



# 创建控制器类
```shell
# 当前位置:BeautyGoose/Server/server/src
mkdir controllers && cd controllers
drogon_ctl create controller -h btyGoose::TestCtrl
drogon_ctl create controller -h btyGoose::AccountCtrl
drogon_ctl create controller -h btyGoose::ConsumerCtrl
drogon_ctl create controller -h btyGoose::MerchantCtrl
drogon_ctl create controller -h btyGoose::AdminCtrl

```

# 引入CMake
由于项目越发复杂，再手动编写makefile过于繁琐，因此开始使用CMkae进行项目构建,并且配合`makefile`进行快速重新构建项目

```cmake
cmake_minimum_required(VERSION 3.12)
project(server)

# C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_BUILD_TYPE "Release")  
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -flto")  # 链接时优化
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)  # IPO优化

# 包含目录设置
include_directories(
    /usr/include  # 显式添加常见路径
    /usr/local/include    # 第三方库常见安装位置
)

# 查找依赖库
find_package(Threads REQUIRED)
# 查找Drogon依赖
find_package(Drogon REQUIRED COMPOMENTS jsoncpp)

set(CONTROLLERS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/controllers")
message(STATUS "控制器路径是: ${CONTROLLERS_DIR}")

# 添加可执行文件
add_executable(
    server
    DatabaseClient.cpp 
    logger.cpp 
    main.cpp 
    RedisClient.cpp 
    # ${CONTROLLERS_DIR}/btyGoose_TestCtrl.cc
)

target_link_libraries(server
    PRIVATE 
    mysqlcppconn
    Threads::Threads 
    gflags
    fmt 
    redis++
    hiredis
    Drogon::Drogon 
    ${JSONCPP_LIBRARIES}
    ${OPENSSL_LIBRARIES}
)


```

# 解决宏定义冲突问题
原本的日志的宏定义与`trantor`中的宏定义冲突，所以需要修改原本的宏定义,加上前缀`SUP_`防止冲突

# 解决共享Redis和MySQL等连接资源的问题
我们将原本存在`HTTPServer`类中的资源转移到新的头文件`SharedResources.h`中

```C++
#pragma once
#include "../DatabaseClient.hpp"
#include "../RedisClient.hpp"
#include "../CoreData.hpp"
#include <drogon/HttpController.h>
#include "../logger.hpp"

namespace btyGoose
{
    //SharedResources
    class SR
    {
    private:
        SR(){};
        SR(const SR&) =delete;
        SR operator=(const SR&) = delete;
    public:
        static SR& ins()
        {
            static SR sr;
            return sr;
        }

        void setDB(std::shared_ptr<DatabaseClient>& ptr)
        {
            // LOG_TRACE("HTTPServer: 开始初始化db");
            _db = ptr;
        }

        void setRedis(std::shared_ptr<RedisClient>&ptr)
        {
            _redis = ptr;
        }

        bool checkRedisOrderListExist(const data::Order &order)
        {
            bool has_consumer = _redis->hasOrderListByUserId(order.consumer_id);
            bool has_merchant = _redis->hasOrderListByUserId(order.merchant_id);

            if (has_consumer && has_merchant)
                return true;
            if (!has_consumer)
            {
                auto order_list = _db->getOrderListByConsumer(order.consumer_id);
                _redis->setOrderList(order_list);
                _redis->setOrderListByIdDone(order.consumer_id);
            }
            if (!has_merchant)
            {
                auto dish_list = _db->getOrderListByMerchant(order.merchant_id);
                _redis->setOrderList(dish_list);
                _redis->setOrderListByIdDone(order.merchant_id);
            }
            return false;
        }

        const std::shared_ptr<DatabaseClient> db(){return _db;}
        const std::shared_ptr<RedisClient> redis(){return _redis;}
    private:
        std::shared_ptr<DatabaseClient> _db;
	    std::shared_ptr<RedisClient> _redis;
    };
}
```

# 在各个头文件内实现路径绑定和函数声明

btyGoose_TestCtrl.h
```C++
#pragma once

#include <drogon/HttpController.h>
#include <drogon/HttpResponse.h>
#include "SharedResources.h"

using namespace drogon;

namespace btyGoose
{
class TestCtrl : public drogon::HttpController<TestCtrl>
{
  public:
    METHOD_LIST_BEGIN
    ADD_METHOD_TO(TestCtrl::ping,"/ping",Get);
    ADD_METHOD_TO(TestCtrl::flushAll,"/flush/redis",Get);
    // ADD_METHOD_TO(TestCtrl::your_method_name, "/absolute/path/{1}/{2}/list", Get); // path is /absolute/path/{arg1}/{arg2}/list

    METHOD_LIST_END
    void ping(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback)
    {
      SUP_LOG_DEBUG("收到请求，methid ={}, path = {}","Get",req->getPath());
      auto resp = drogon::HttpResponse::newHttpResponse();
      resp->setStatusCode(k200OK);
      resp->setContentTypeCode(CT_TEXT_HTML);
      resp->setBody("pong");
      callback(resp);    
    }
    void flushAll(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback)
    {
      SUP_LOG_DEBUG("收到请求，methid ={}, path = {}","Get",req->getPath());
      std::string password = req->getOptionalParameter<std::string>("password").value_or("");

      auto resp = drogon::HttpResponse::newHttpResponse();
      if(password == "IDOCONFIRM")
      {
          SR::ins().redis()->flushall();
          resp->setBody("flush done");
      }
      else
      {
        resp->setBody("密码错误");
      }
      resp->setStatusCode(k200OK);
      resp->setContentTypeCode(CT_TEXT_HTML);
      SR::ins().redis()->flushall();
      callback(resp);    
    }
};
}

```

btyGoose_MerchantCtrl.h
```C++
#pragma once

#include <drogon/HttpController.h>
#include "SharedResources.h"

using namespace drogon;

namespace btyGoose
{
  class MerchantCtrl : public drogon::HttpController<MerchantCtrl>
  {
  public:
    MerchantCtrl()
    {
      redis = SR::ins().redis();
      db = SR::ins().db();
    }
    METHOD_LIST_BEGIN
    ADD_METHOD_TO(MerchantCtrl::merchantDishInfoDebug,"/debug/merchant/dish/info",Post);
    ADD_METHOD_TO(MerchantCtrl::merchantDishList,"/merchant/dish/list",Post);
    ADD_METHOD_TO(MerchantCtrl::merchantDishRegister,"/merchant/dish/register",Post);
    ADD_METHOD_TO(MerchantCtrl::merchantDishInfo,"/merchant/dish/info",Post);
    ADD_METHOD_TO(MerchantCtrl::merhcnatDishUpdate,"/merchant/dish/update",Post);
    ADD_METHOD_TO(MerchantCtrl::merhcnatDishDel,"/merchant/dish/del",Post);
    ADD_METHOD_TO(MerchantCtrl::merhcnatOrderList,"/merchant/order/list",Post);
    ADD_METHOD_TO(MerchantCtrl::merhcnatOrderDetailDishList,"/merchant/order/detail/dishlist",Post);
    ADD_METHOD_TO(MerchantCtrl::merhcnatOrderAccept,"/merchant/order/accept",Post);
    ADD_METHOD_TO(MerchantCtrl::merchantOrderReject,"/merchant/order/reject",Post);

    METHOD_LIST_END
    void merchantDishInfoDebug(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);

    void merchantDishList(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merchantDishRegister(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merchantDishInfo(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merhcnatDishUpdate(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merhcnatDishDel(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merhcnatOrderList(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merhcnatOrderDetailDishList(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merhcnatOrderAccept(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void merchantOrderReject(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);


  private:
    std::shared_ptr<RedisClient> redis;
    std::shared_ptr<DatabaseClient> db;
  };
}

```

btyGoose_ConsumerCtrl.h
```C++
#pragma once

#include <drogon/HttpController.h>
#include "SharedResources.h"
using namespace drogon;

namespace btyGoose
{
class ConsumerCtrl : public drogon::HttpController<ConsumerCtrl>
{
  public:
  ConsumerCtrl()
  {
    redis = SR::ins().redis();
    db = SR::ins().db();
  }

  public:
    METHOD_LIST_BEGIN
    ADD_METHOD_TO(ConsumerCtrl::consumerOrderDetailDishlist,"/consumer/order/detail/dishlist",Post);
    ADD_METHOD_TO(ConsumerCtrl::consumerDishList,"/consumer/dish/list",Post);
    ADD_METHOD_TO(ConsumerCtrl::consumerDishDishInfo,"/consumer/dish/dishInfo",Post);
    ADD_METHOD_TO(ConsumerCtrl::consumerOrderGenerate,"/consumer/order/generate",Post);
    ADD_METHOD_TO(ConsumerCtrl::consumerOrderList,"/consumer/order/list",Post);
    ADD_METHOD_TO(ConsumerCtrl::consumerOrderPayConfirm,"/consumer/order/pay/confirm",Post);
    ADD_METHOD_TO(ConsumerCtrl::consumerOrderCancel,"/consumer/order/cancel",Post);
    METHOD_LIST_END
    void consumerOrderDetailDishlist(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void consumerDishList(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void consumerDishDishInfo(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void consumerOrderGenerate(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void consumerOrderList(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void consumerOrderPayConfirm(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void consumerOrderCancel(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);

    private:
    std::shared_ptr<RedisClient> redis;
    std::shared_ptr<DatabaseClient> db;
};
}

```

btyGoose_AccountCtrl.h
```C++
#pragma once

#include <drogon/HttpController.h>
#include "SharedResources.h"

using namespace drogon;

namespace btyGoose
{
class AccountCtrl : public drogon::HttpController<AccountCtrl>
{
  public:
  AccountCtrl()
  {
    redis = SR::ins().redis();
    db = SR::ins().db();
  }
  public:
    METHOD_LIST_BEGIN
    ADD_METHOD_TO(AccountCtrl::accountLoginUsername,"/account/login/username",Post);
    ADD_METHOD_TO(AccountCtrl::accountLoginPhone,"/account/login/phone",Post);
    ADD_METHOD_TO(AccountCtrl::accountRegister,"/account/register",Post);
    ADD_METHOD_TO(AccountCtrl::accountUpdateLevel,"/account/update/level",Post);
    ADD_METHOD_TO(AccountCtrl::accountUpdateNickname,"/account/update/nickname",Post);

    METHOD_LIST_END
    void accountLoginUsername(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void accountLoginPhone(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void accountRegister(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void accountUpdateLevel(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);
    void accountUpdateNickname(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback);

    bool AuthenticateAuthCode(const string& phone, const string& auth_code);
  
    private:
    std::shared_ptr<RedisClient> redis;
    std::shared_ptr<DatabaseClient> db;
};
}

```
# 重写业务函数代码
因为使用了不同的网络框架，所以业务代码也需要重写以适配新的函数接口。最大的变化就是因为使用异步框架的缘故，返回报文需要手动调用`callback`回调函数。

例子如下:
```C++
void ConsumerCtrl::consumerOrderDetailDishlist(const HttpRequestPtr &req, std::function<void(const HttpResponsePtr &)> &&callback)
{
    SUP_LOG_DEBUG("收到HTTP请求, method={},path={}","Post",req->getPath());

    std::string jsonStr = req->getBody().data();
    Json::Value jsonObj,resJson;
    Json::Reader reader;

    if (!reader.parse(jsonStr, jsonObj)) {
        SUP_LOG_ERROR("[消费者获取订单菜品列表]无效的JSON格式:{}",jsonStr);
        resJson["success"] = false;
        resJson["message"] = "无效的JSON格式";
        auto resp = drogon::HttpResponse::newHttpJsonResponse(resJson);
        resp->setStatusCode(k400BadRequest);
        resp->setContentTypeCode(CT_APPLICATION_JSON);
        callback(resp);
        return;
    }

    if (!jsonObj.isMember("order_id"))
    {
        SUP_LOG_ERROR("[消费者获取订单菜品列表] 不存在字段: order_id, 报文:{}", jsonStr);
        resJson["success"] = false;
        resJson["message"] = "无效的JSON格式";
        auto resp = drogon::HttpResponse::newHttpJsonResponse(resJson);
        resp->setStatusCode(k400BadRequest);
        resp->setContentTypeCode(CT_APPLICATION_JSON);
        callback(resp);
        return;
    }

    std::string order_id = jsonObj["order_id"].asString();
    std::string dish_list_json = redis->getOrderDishListJson(order_id);
    if(dish_list_json.empty())
    {
        std::vector<data::OrderDish> dishList = db->getOrderDishesByID(order_id);
        std::string dishListJson = json::OrderDishListToJsonArray(dishList);
        resJson["dish_list"] = dishListJson;
        redis->setOrderDishListJson(order_id,dishListJson);

    }
    else 
    {
        resJson["dish_list"] = dish_list_json;
    }
    SUP_LOG_INFO("[消费者获取订单菜品列表]成功,order_id:{}",order_id);
    auto resp = drogon::HttpResponse::newHttpJsonResponse(resJson);
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_APPLICATION_JSON);
    callback(resp);
}
```