---
title: v2.5美鹅外卖平台延迟瓶颈测试
date: 2025-04-17 16:56:55
tags:
---

# 计时器类
为了测量延迟，我们需要使用计时功能，为此我们把相关的功能封装进计时器类内，简化使用

这段代码位于`logger.hpp`中
```C++
class ScopedTimer
{
public:
    ScopedTimer(const std::string&name)
    :_name(name),_start(std::chrono::high_resolution_clock::now())
    {
        _prev = _start;
    }

    int64_t staged()
    {
        auto stage = std::chrono::high_resolution_clock::now();
        auto ret = stage - _prev;
        _prev = stage;
        return (std::chrono::duration_cast<std::chrono::microseconds>(ret)).count();
    }
        
    ~ScopedTimer() {
        auto end = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end-_start);
        LOG_DEBUG("计时器 {} 存活时间: {} μs",_name,duration.count());
    }

private:
    std::string _name;
    std::chrono::time_point<std::chrono::high_resolution_clock> _start;
    std::chrono::time_point<std::chrono::high_resolution_clock> _prev;
};
```

# 检查无缓存时的延迟分布
我们先把中间层的缓存清理掉，强行使缓存不命中，前往数据库去查询。

```shell
[default-logger][08:41:39][8][info    ][HTTPServer.cpp:75] 收到HTTP请求, method=GET,path=/flush/redis
[default-logger][08:41:50][9][debug   ][HTTPServer.cpp:68] 收到HTTP请求, method=GET,path=/ping
/ping
[default-logger][08:41:55][10][debug   ][HTTPServer.cpp:888] 收到HTTP请求, method=POST,path=/debug/merchant/dish/info
[default-logger][08:41:55][10][trace   ][HTTPServer.cpp:899] Json解析完成,    耗时 1168μs
[default-logger][08:41:55][10][trace   ][HTTPServer.cpp:914] 完成Redis的get查询,  耗时 1068μs
[default-logger][08:41:55][10][trace   ][DatabaseClient.cpp:355] API数据库接口加锁耗时: 0μs
[default-logger][08:41:55][10][debug   ][DatabaseClient.cpp:356] 数据库查询菜品, ID:D561d2f6d155a
[default-logger][08:41:56][10][trace   ][DatabaseClient.cpp:362] API生成SQL语句，耗时 29678μs
[default-logger][08:41:56][10][trace   ][DatabaseClient.cpp:364] API执行SQL语句，耗时 10400μs
[default-logger][08:41:56][10][debug   ][logger.hpp:40] 计时器 数据库接口内部 存活时间: 40230 μs
[default-logger][08:41:56][10][trace   ][HTTPServer.cpp:920] 完成数据库查询数据，耗时 40254μs
[default-logger][08:41:56][10][trace   ][HTTPServer.cpp:926] 完成Redis设置缓存，耗时 561μs
[default-logger][08:41:56][10][trace   ][HTTPServer.cpp:931] 完成Json序列化和报文写入,耗时 1007μs
[default-logger][08:41:56][10][info    ][HTTPServer.cpp:933] [商家查询菜品详情]成功
[default-logger][08:41:56][10][debug   ][logger.hpp:40] 计时器 菜品详情查询 存活时间: 44240 μs
```

然后我们再一次访问，触发缓存命中，再查看日志信息

```shell
[default-logger][08:57:52][10][debug   ][HTTPServer.cpp:888] 收到HTTP请求, method=POST,path=/debug/merchant/dish/info
[default-logger][08:57:52][10][trace   ][HTTPServer.cpp:899] Json解析完成,    耗时 83μs
[default-logger][08:57:52][10][trace   ][HTTPServer.cpp:914] 完成Redis的get查询,  耗时 2819μs
[default-logger][08:57:52][10][trace   ][HTTPServer.cpp:931] 完成Json序列化和报文写入,耗时 143μs
[default-logger][08:57:52][10][info    ][HTTPServer.cpp:933] [商家查询菜品详情]成功
[default-logger][08:57:52][10][debug   ][logger.hpp:40] 计时器 菜品详情查询 存活时间: 3149 μs
```

+ 可以看到，引入`Redis`缓存后，延迟得到了大大的缩减，
+ 再看数据库查询的情况，可以发现生成SQL语句的过程居然比执行SQL语句还慢！这是非常大的性能瓶颈了

## 针对性优化
因为目前的数据库连接选项是默认的，还没有启用优化选项，所以针对生成SQL语句过慢的问题，我们修改数据库客户端的启动代码

```C++
    sql::ConnectOptionsMap ops;

    ops["hostName"] = "tcp://"+host+":"+port;
    ops["userName"] = user;
    ops["password"] = password;

    ops["useServerPrepStmts"] = true;
    ops["cachePrepStmts"] = true;
    ops["prepStmtCacheSize"] = 25;  // 默认25个
    con = driver->connect(ops);
```

此时延迟问题也得到了大大缩小

```shell
[default-logger][09:31:08][10][debug   ][HTTPServer.cpp:888] 收到HTTP请求, method=POST,path=/debug/merchant/dish/info
[default-logger][09:31:08][10][trace   ][HTTPServer.cpp:899] Json解析完成,    耗时 48μs
[default-logger][09:31:08][10][trace   ][HTTPServer.cpp:914] 完成Redis的get查询,  耗时 357μs
[default-logger][09:31:08][10][trace   ][DatabaseClient.cpp:364] API数据库接口加锁耗时: 0μs
[default-logger][09:31:08][10][debug   ][DatabaseClient.cpp:365] 数据库查询菜品, ID:D561d2f6d155a
[default-logger][09:31:08][10][trace   ][DatabaseClient.cpp:371] API生成SQL语句，耗时 4816μs
[default-logger][09:31:08][10][trace   ][DatabaseClient.cpp:373] API执行SQL语句，耗时 18426μs
[default-logger][09:31:08][10][trace   ][DatabaseClient.cpp:385] API完成对象初始化,耗时102μs
[default-logger][09:31:08][10][debug   ][logger.hpp:40] 计时器 数据库接口内部 存活时间: 23360 μs
[default-logger][09:31:08][10][trace   ][HTTPServer.cpp:920] 完成数据库查询数据，耗时 23376μs
[default-logger][09:31:08][10][trace   ][HTTPServer.cpp:926] 完成Redis设置缓存，耗时 671μs
[default-logger][09:31:08][10][trace   ][HTTPServer.cpp:931] 完成Json序列化和报文写入,耗时 129μs
[default-logger][09:31:08][10][info    ][HTTPServer.cpp:933] [商家查询菜品详情]成功
[default-logger][09:31:08][10][debug   ][logger.hpp:40] 计时器 菜品详情查询 存活时间: 24964 μs
```

## Json序列化耗时过长

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504181029879.webp)

# 网络IO成为最大瓶颈

```shell
wrk -t2 -c100 -d30s -s dishInfo.lua http://127.0.0.1:30001/debug/merchant/dish/info --latency
Running 30s test @ http://127.0.0.1:30001/debug/merchant/dish/info
  2 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   134.57ms  161.39ms   1.41s    95.23%
    Req/Sec   461.73     85.57   767.00     75.67%
  Latency Distribution
     50%  104.34ms
     75%  155.32ms
     90%  205.12ms
     99%    1.02s 
  27598 requests in 30.03s, 15.32MB read
Requests/sec:    918.91
Transfer/sec:    522.27KB
```

可以看到，网络API的延迟达到几百毫秒。而实际上查看业务代码的计时器，我们来把日志重定向到文件中并进行分析:

## 分析应用服务器日志

重定向:
```shell
sudo docker logs -f server-server-1 >> container.log 2>&1 &
```
然后压力测试

```shell
wrk -t2 -c100 -d30s -s dishInfo.lua http://127.0.0.1:30001/debug/merchant/dish/info --latency
```
接着获取`container.log`并写一个python程序分析（代码略）

获取到如下输出图标和信息

```shell
========================================
📊 日志分析报告 (2025-04-20 10:43)
========================================

📌 总查询次数: 25292
🔍 平均Redis耗时: 2.76 ms
🐢 慢查询数量(>10ms): 3372
⏱️ 最慢查询: 69.00 ms

🧵 线程性能分析:
  Thread 10: 平均Redis耗时 2.74 ms
  Thread 9: 平均Redis耗时 2.77 ms
  Thread 11: 平均Redis耗时 2.76 ms
  Thread 8: 平均Redis耗时 2.78 ms
```

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504201049398.png)

可以看到实际上业务代码最多只有几十毫秒的延迟，也就是说基本确定网络IO成为了最大瓶颈,接下来我们进一步定位

## 查看nginx日志

但是网络IO发生在多处地方，我们要定位就是反向代理服务器是瓶颈，还是应用服务器的网络IO成为瓶颈，为此我们需要查看nginx的代理业务的日志

把nginx的配置文件修改了一下，启用了日志文件
```conf
log_format main_ext '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'urt="$upstream_response_time" uht="$upstream_header_time" '
                    'ups="$upstream_status" cs=$connection cs=$connection_requests '
                    'lb=$upstream_addr';

log_format error_ext '$remote_addr - [$time_local] '
                    'req_err: $status, upstream_err: $upstream_status '
                    'req_time=$request_time, upstream_time=$upstream_response_time '
                    'client: $http_user_agent '
                    'req_body: "$request_body"';

upstream backend{
    server server:80 weight=1;
    server server2:80 weight=1;

    # 开启健康检查日志
    keepalive 16;
}

server {
    listen 80;
    # access_log off;

    # 访问日志（JSON格式便于ELK分析）
    access_log /var/log/nginx/access.log main_ext buffer=32k flush=5s;
    
    # 错误日志（记录4xx/5xx）
    error_log /var/log/nginx/error.log warn;

    # 调试日志（临时使用）
    # error_log /var/log/nginx/debug.log debug;

    location / {
        proxy_pass http://backend;
        proxy_connect_timeout 3s;
        proxy_read_timeout 10s;

        # 增强的代理头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Request-Start "t=${msec}";

        # 启用详细日志
        access_log /var/log/nginx/proxy_access.log main_ext;
        log_subrequest on;
        
        # 错误响应单独记录
        error_log /var/log/nginx/proxy_error.log warn;
    }

    # 健康检查端点
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
    # 日志轮转配置
    location /logs {
        alias /var/log/nginx;
        autoindex on;
        access_log off;
    }
}

```

然后跑了一次压力测试后，分析日志文件查看上游服务器的最大响应时间,找到了部分响应时间大于一秒的记录：

```
172.28.0.1 - - [18/Apr/2025:08:34:46 +0000] "POST /debug/merchant/dish/info HTTP/1.1" 200 427 "-" "-" "-" rt=1.366 uct="1.025" urt="1.365" uht="1.365" ups="200" cs=47 cs=1 lb=172.28.0.5:80
```
以及生成的部分图表:
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202504201057630.png)

由此可以最终定位得到，部分高延迟请求来自于应用服务器,，所以我们要针对`cpp-httplib`网络框架进行一些参数调优。（没错，目前用的还是默认参数）

非常可惜，即使经过了参数配置，问题依然没有得到解决，因此我们决定更换网络框架。

经过对比，我们选择使用开源项目`Drogon`，期望使用更高效的网络框架能缓解延迟问题