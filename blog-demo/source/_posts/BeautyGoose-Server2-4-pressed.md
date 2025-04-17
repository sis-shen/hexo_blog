---
title: v2.4美鹅外卖平台压力测试
date: 2025-03-21 11:23:05
tags:
---

# 日志查阅
在2.4出现后台服务器挂掉的情况后，我立即去查看了容器的日志信息，获得了如下日志:

```shell
[default-logger][06:16:43][9][info    ][HTTPServer.cpp:415] [消费者获取菜品详情]成功,dish id:D7d33e8b47dda
[default-logger][06:16:43][9][trace   ][HTTPServer.cpp:416] {
        "dish" : "{\n    \"base_price\" : 11.0,\n    \"description\" : \"11\",\n    \"icon_path\" : \"https://ts3.cn.mm.bing.net/th?id=ORMS.800de0e05f326302bea15ba2e5ff707d&pid=Wdp&w=268&h=140&qlt=90&c=1&rs=1&dpr=1.5&p=0\",\n    \"is_delete\" : false,\n    \"merchant_id\" : \"Afdaf4ca89c1d\",\n    \"merchant_name\" : \"11\",\n    \"name\" : \"11\",\n    \"price_factor\" : 1.0,\n    \"uuid\" : \"D7d33e8b47dda\"\n}"
}
[default-logger][06:16:43][13][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][13][debug   ][DatabaseClient.cpp:646] 数据库查询消费者订单列表,consumer id: Ae1c45d30139c
[default-logger][06:16:43][13][info    ][HTTPServer.cpp:519] [消费者获取订单列表]成功，总数:1
[default-logger][06:16:43][8][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][8][debug   ][DatabaseClient.cpp:646] 数据库查询消费者订单列表,consumer id: Ae1c45d30139c
[default-logger][06:16:43][8][info    ][HTTPServer.cpp:519] [消费者获取订单列表]成功，总数:1
[default-logger][06:16:43][15][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][15][debug   ][DatabaseClient.cpp:646] 数据库查询消费者订单列表,consumer id: Ae1c45d30139c
[default-logger][06:16:43][14][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][12][debug   ][HTTPServer.cpp:394] 收到HTTP请求, method=POST,path=/consumer/dish/dishInfo
[default-logger][06:16:43][11][debug   ][HTTPServer.cpp:394] 收到HTTP请求, method=POST,path=/consumer/dish/dishInfo
[default-logger][06:16:43][10][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][9][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][13][debug   ][HTTPServer.cpp:496] 收到HTTP请求, method=POST,path=/consumer/order/list
[default-logger][06:16:43][8][debug   ][HTTPServer.cpp:394] 收到HTTP请求, method=POST,path=/consumer/dish/dishInfo
[default-logger][06:16:43][15][error   ][DatabaseClient.cpp:669] 获取用户订单失败喵！
        错误码: 1461
        信息:
```

可以看到，最后数据库客户端抛出了异常并被捕获，然后输出了报错信息，尽管没有显示具体信息，但是给出了错误码`1461`。

经过查询可得，`MySQL Connector/C++`中错误码`1461`的意思是:`字符串过长错误`，尽管我们只执行了查询语句，即`select`语句，但是由于*字符的编码会影响字节长度*，*内存碎片影响字符串长度*等一系列不稳定的问题，仍然会造成字符串长度溢出等问题，因此在这个版本中我们要修改优化对数据库的查询。

# 重新设计表
原本的表为字段设置的长度是刚刚好的，这就导致长度检查十分严格。所以这次重新设计表设置一些冗余长度

+ `uuid`: varchar(13) -> varchar(32)
+ `name`: varchar(10) -> varchar(16)

同时为了弥补之前没有主键的严重问题，我们为每一个表都增加了**自增主键字段**,为什么不用`uuid`作为主键呢？因为`uuid`过大，作为主键会使效率大大下降，远不如使用较小的自增主键+`uuid`新建索引的模式

```mysql
    `id` BIGINT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
```

## 阶段测试
重新生成数据库后又简单测了一下，还是挂了。

