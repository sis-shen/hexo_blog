---
title: v2.4美鹅外卖平台反向代理更新
date: 2025-03-21 11:23:00
tags:
---
# 更新内容
+ 架构上添加代理层，升级为应用服务集群架构
+ 容器编排上，后端服务程序数量提升为两个
+ 采用反向代理负载均衡技术，后端服务器均衡承载负载

# 什么是应用服务集群架构
当单台应用服务器出现了性能瓶颈之后，可以有如下两种扩展解决方案:

1. **垂直扩展/纵向扩展ScaleUp**。通过购买性能更优、价格更⾼的应⽤服务器来应对更多的流量。这种⽅案的优势在于完全不需要对系统软件做任何的调整；但劣势也很明显：硬件性能和价格的增⻓关系是⾮线性的，意味着选择性能2倍的硬件可能需要花费超过4倍的价格，其次硬件性能提升是有明显上限的。
2. **⽔平扩展/横向扩展ScaleOut**。通过调整软件架构，增加应⽤层硬件，将⽤⼾流量分担到不同的应⽤层服务器上，来提升系统的承载能⼒。这种⽅案的优势在于成本相对较低，并且提升的上限空间也很⼤。但劣势是带给系统更多的复杂性，需要技术团队有更丰富的经验

这里我们决定使用水平扩展，调整软件架构的方式提高服务法并发量。

但这需要引⼊⼀个新的组件⸺**负载均衡**：为了解决⽤⼾流量向哪台应⽤服务器分发的问题，需要⼀个专⻔的系统组件做流量分发。

关于流量分发的策略，有以下几种常见的策略
1. **Round-Robin轮询算法**:即⾮常公平地将请求依次分给不同的应⽤服务器
2. **Weight-Round-Robin轮询算法**:引入权重,为不同的服务器（⽐如性能不同）赋予不同的权重（weight）,按权重分配流量
3. **⼀致哈希散列算法**:通过计算⽤⼾的特征值（⽐如IP地址）得到哈希值，根据哈希结果做分发，优点是确保来⾃相同⽤⼾的请求总是被分给指定的服务器。

# 本次具体更新内容

## 选用nginx
这里我们选用了`nginx`作为现成的反向代理服务器，并提供**负载均衡**服务，分发策略使用的是`Weight-Round-Robin轮询算法`，但是目前分配的权重是`1比1`的等权重。

为什么使用`nginx`呢，因为`nginx`作为成熟的产品，性能优异且稳定,单进程达10万+的并发，而且仅需修改配置文件便能完成项目适配，十分简单易用。而且配置选项丰富，方便未来做调优调整

## 引入nginx
当前服务端架构图为：
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503211406436.webp)

### 构建配置文件
我们创建一个nginx文件夹:
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503211419693.webp)

然后在里面新建一个配置文件`sup.conf`并写入如下内容,这样就能执行一个简单的负载均衡功能了。其中`server`和`server2`都使用了`docker`提供的`DNS`服务，所以要保证它们都在同一个`docker`网络环境下

```nginx
upstream backend{
    server server:80 weight=1;
    server server2:80 weight=1;
}

server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://backend;
        proxy_connect_timeout 3s;
        proxy_read_timeout 10s;
    }
}
```

### 构建Dockerfile
我们使用`nginx:1.24.0`官方镜像为基底，替换掉配置文件后重新构建一个自己的nginx镜像

```Dockerfile
FROM nginx:1.24.0
# 先删除默认配置
RUN rm /etc/nginx/conf.d/default.conf
COPY ./sup.conf /etc/nginx/conf.d
CMD ["nginx","-g","daemon off;"]
ENTRYPOINT [ "/docker-entrypoint.sh" ]
```

### 修改dockerfile.yml
对于容器编排，我们主要有以下几步要做

1. 新增一个后端服务器容器`server2`
2. 新增一个自定义nginx容器`web`
3. 后端服务器不再暴露端口，注释端口映射代码
4. `nginx`镜像添加端口映射，暴露容器的80端口

修改后的`docker-compose-yml`如下

```yml
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
    ports:
      - 30006:3306
    networks:  # 新增网络配置
      - btygoose_net

networks:  # 新增网络定义
  btygoose_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16  # 自定义子网
```