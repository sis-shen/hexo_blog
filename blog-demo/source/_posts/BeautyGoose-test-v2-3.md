---
title:  C++测试】美鹅外卖v2.3~v2.4版本性能测试
date: 2025-03-10 08:38:57
tags:
---

> 在本版本中，整个服务端的基本功能已经完成，包括稳定性提升和日志标准化输出，因此为了进一步优化服务器的性能，提高其并发性和准确率，我们将使用测试工具`Postman`、`perf`、`perl`和火焰图开源项目进行测试、绘制火焰图和分析性能瓶颈
> 
> [戳我去Postman介绍博客🔗](https://www.supdriver.top/2025/03/13/postman/) \
> [戳我去火焰图绘制介绍博客🔗](https://www.supdriver.top/2025/03/11/perf-perl/)

# v2.3第一次测试

## 测试信息
| key | value |
| --- | ------ |
| 项目版本 | v2.3 |
| 当前架构 | 逻辑上`应用数据分离架构`,物理上单机架构(只有一台云服务器) |
| 服务端信息 | 阿里云服务器`Ubuntu 22.04` , 2核2G,Docker 容器资源分配默认配置 |
| 测试端信息 | Postman客户端 |

## 修改 `docker-compose.yml`
经过初次进入容器尝试使用perf，会发现权限不足，原因是`Docker`对容器的权限做了限制，即使使用`root`用户，获得的权限也是受限的，所以我们要修改`docker-compose.yml`修改容器的启动参数，来获得全部权限。**注：获得权限的参数理应只在测试时使用**

获得权限的方式就是加上`privileged: true`参数即可

展示的部分代码如下
```yml
services:
  server:
    build:
      context: ./server
    image: btygoose:v1.0
    # ======= 仅测试使用  ======
    privileged: true
    # ======== end =========
    ports:
      - 30001:80
    depends_on:
      db:
        condition: service_healthy
    networks:  # 新增网络配置
      - btygoose_net
```

## 修改makefile
为了获取更详细的程序运行细节，我们需要降低编译器的优化程度，因此要修改`makefile`新增一条专门用于调试的编译指令

```makefile
server_g:main.o DatabaseClient.o HTTPServer.o logger.o
	g++ -fno-omit-frame-pointer -O0 -g -o server *.o -std=c++17 $(LDLIBS)
```

## 修改Dockerfile
构建镜像时运行的指令也要发生变化

```Dockerfile
FROM ubuntu:22.04 AS build1
RUN apt update -y && apt install -y g++ \
    make \
    libmysqlcppconn-dev \
    libjsoncpp-dev \
    libboost-system-dev\
    libgflags-dev \ 
    libfmt-dev \
    libspdlog-dev
COPY ./src* /src

WORKDIR /src
# RUN make && mkdir logs
#DEBUG
RUN make server_g && mkdir logs
#end
CMD ["/src/server","--flagfile=server.conf"]
```

## 重新构建镜像

```shell 
# cd 进入BeautyGoose/Server/目录
sudo docker compose build server

```
## 启动并进入容器

```shell
# cd 进入BeautyGoose/Server/目录
# 关闭容器
sudo docker compose down
# 启动容器，因为镜像没有变化，所以不用重新build
sudo docker compose up -d
# 进入容器,注：这边运行的服务端容器名称为server-server-1
sudo docker exec -it server-server-1
```

## 查询进程PID并准备启动采样
### 安装perf
因为是镜像容器，所以**没有sudo指令**
```shell
apt update -y
# 安装 linux-tools 通用包（适配当前运行的内核）
apt install -y linux-tools-generic linux-tools-$(uname -r)
```
### 查询PID
```shell
ps aux | head -n 1  && ps aux |  grep server
```

查询结果如下:
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503141028308.webp)
可以看到我们的目标程序因为是容器的入口指令，所以`PID`理所应当地为`1`

所以采样的指令为`perf record -F 99 -g -e cycles:u -p 1 --call-graph fp`

## 配置Postman开始测试
第一次测试先测试仅读接口`/consumer/dish/dishInfo`

配置测试脚本
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503141042314.webp)

配置Runner
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503141043303.webp)

点击`Run`按钮后，我们回到容器内运行采样代码:

```shell
perf record -F 99 -g -e cycles:u -p 1 --call-graph fp
```

在等待几分钟后，`Postman`测试结束前，我们键盘输入`ctrl+c`停止采样，这样就能获得`perf.data`了

## 开始绘制火焰图
1. 我们先输入`exit`指令退出容器，然后为了方便使用xftp传文件,我们把容器内的`perf.data`拷贝到`/root/data`目录下
2. 为了进入`/root/data`目录，还要切换到root用户
3. `git clone`一个火焰图项目

```shell
sudo docker cp server-server-1:/src/perf.data /root/data/
su root
cd /root/data
git clone https://github.com/brendangregg/FlameGraph.git
```

然后我们依次输入以下代码生成`perf.svg`

```shell
sudo perf script -i perf.data &> perf.unfold 
sudo ./FlameGraph/stackcollapse-perf.pl perf.unfold &> perf.folded 
sudo ./FlameGraph/flamegraph.pl perf.folded > perf.svg
```
执行后目录结构如下:
```shell
root@ALi-cloud-Linux-2-2G:~/data# ll
total 764
drwxr-xr-x  3 root root   4096 Mar 14 11:05 ./
drwx------ 19 root root   4096 Mar 13 22:29 ../
drwxr-xr-x  7 root root   4096 Mar 13 22:50 FlameGraph/
-rw-------  1 root root 318015 Mar 14 10:53 perf.data
-rw-r--r--  1 root root  19706 Mar 14 11:05 perf.folded
-rw-r--r--  1 root root  42047 Mar 14 11:05 perf.svg
-rw-r--r--  1 root root 382113 Mar 14 11:05 perf.unfold
root@ALi-cloud-Linux-2-2G:~/data# 
```

## 从服务器下载文件并在本机上打开
这里使用软件`xftp`连接至服务器并传输文件

然后用浏览器获得如下火焰图(截图):
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503151855164.webp)
## 分析结果
可以看到：
+ 火焰图中没有特别明显的平顶,程序的性能较为平衡

再看一看`Postman`的测试结果:
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503141117625.webp)
可以看到接口延迟不稳定，错误率也时有存在。

经过其它的单独测试，对于错误率的问题，在于`cpp-httplib`对于高并发访问难以保存正确率。

## 恢复生产环境
我们有如下文件要恢复:

1. Dockfile
2. docker-compose.yml

# v2.4负载均衡测试
## 测试信息
| key | value |
| --- | ------ |
| 项目版本 | v2.4 |
| 当前架构 | 逻辑上`应用服务集群架构`,物理上单机架构(只有一台云服务器) |
| 服务端信息 | 阿里云服务器`Ubuntu 22.04` , 2核2G,Docker 容器资源分配默认配置 |
| 测试端信息 | Postman客户端 |
| 负载均衡方式 | Weight-Round-Robin轮询算法,但是权重相等 | 
| 测试接口数量 | 两个查询接口轮询并发 |

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503200907178.webp)

## 测试标准
`平均延迟`达`100ms`左右时的平均`Rps`作为本批次测试的指标

## 测试目标
找出本版本架构下较为合适的应用服务器集群规模

## 应用服务器数量 1
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503201410202.webp)

挂了。。

## 应用服务器数量 2 
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503211137821.webp)

提升很明显，但是也挂了

## 测试结果
原本的目标没有达成，尽管性能提升非常明显，但是应用服务器的稳定性的问题暴露了出来，因此暂时选用和服务器CPU核心数量相同的2个应用服务器，在接下来的迭代中我们将优先解决稳定性问题。