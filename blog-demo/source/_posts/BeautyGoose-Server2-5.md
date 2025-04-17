---
title: v2.5美鹅外卖平台Redis数据缓存更新
date: 2025-04-16 09:35:49
tags:
---

# 更新要点
+ 在容器编排中引入Redis 7.2.4
+ 将Redis服务作为MySQL缓冲层
+ 修复客户端部分bug，并使用Release模式重新编译

# 为什么引入Redis缓存
根据时间局部性原理，访问过的数据，短时间内大概率会被再次访问，由于服务端的业务处理运行在内存中，而数据库的访问数据会发生硬盘IO，存在巨大的速度差异，导致性能瓶颈，所以我们在数据库与业务层中引入数据缓存，提高数据的访问速率。

# 创建Redis容器
为了更方便地使用Redis服务，我们使用Redis的Docker镜像生成一个容器来提供服务。比如我们使用镜像`redis:7.2.4`

```shell
docker pull redis:7.2.4
```
下面是docker-compose.yml的部分代码,**特别的**，我们的业务层容器依赖于redis容器先启动，所以也要修改`server`容器的配置

```yml
services:
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
        condition: service_healthy  # 新增对Redis的依赖
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
        condition: service_healthy  #新增对Redis的依赖
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
```

# 编写Dockerfile搭建环境
因为这里选用了编译安装的方式安装`redis-plus-plus`库，所以`Dockerfile`写起来会较为复杂，我们至少有如下步骤:

1. apt安装hiredis
2. git clone项目到本地
3. cmake构建
4. make编译
5. make install安装

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

# 编写Redis客户端
*我们也可以在开发环境中配置安装redis-plus-plus方便写代码*

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

## 如何存储结构化的数据
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

## 缓存一致性问题
一旦引入缓存，缓存一致性问题就绕不开了。这里采用偏向于简单的解决方法

1. 对于一般的数据，我们会把增删改即时同步到数据库中
2. 对于`OrderList`这样的数据，我们采用额外的集合来标记有效的`key`,当发生增删改时，使`key`失效，对应的每次查询缓存时会先检查`key`是否有效,若失效，则重新从数据库加载，并标记`key`为有效
3. 对于**被序列化的数据**,我们在增删改的时候采用删除`key`的方式进行懒加载，等待到下次查询时发现缓存不命中，自动从数据库加载打破缓存

## 缓存淘汰策略
部分较大的数据被设置了存活时间，过期自动淘汰，它们有:

1. `dishListJsonByMerchant`
2. `orderListById`

