---
title: 高级IO】②poll实现IO多路转接
date: 2025-02-17 19:10:38
tags:
---
以系统提供的函数`poll`为核心实现多路转接，即高级网络IO

# 认识 poll 函数

```C
#include <poll.h>

    int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

特殊的结构体`pollfd`

```C
struct pollfd {
    int   fd;         /* 文件描述符 */
    short events;     /* 请求事件 */
    short revents;    /* 应答事件 */
};
```

## 功能
`poll`也类似于`select`函数，能够同时监听多个文件描述符的**就绪事件**

### 参数解释

+ `fds`是`pollfd`结构体数组的指针/地址，提供待监听文件描述符的列表。其中结构体中的位图提供了每个文件描述符的**各种状态的可能性**
+ `nfds`表示传入数组的长度
+ `timeout`表示`poll`函数的超时时间，单位是毫秒(ms)

其中`events`和`revents`都是位图，可以有如下取值

| 事件          | 描述                                  | 是否可作为输入 | 是否可作为输出     |
| `POLLIN`      | 数据(包括普通数据和优先级数据)可读      |   是          | 是 |
| `POLLRDNORM`  | 普通数据可读                          |   是          | 是 |
| `POLLRDBAND`  | 优先级带数据可读                      |   是          | 是 |
| `POLLPRI`     | 高优先级数据可读，比如TCP带外数据       | 是             | 是 |
| `POLLOUT`     | 数据（包括普通数据和优先数据）可写        | 是            | 是 |
| `POLLWRNORM`  | 普通数据可写                          | 是            | 是 |
| `POLLWRBAND`  | 优先级带数据可写                      | 是            | 是 | 
| `POLLRDHUP`   | TCP连接被对方关闭，或者对方关闭了写操作，它由GNU引入 | 是 | 是 |
| `POLLERR`     | 错误                                  | 否        | 是    |
| `POLLHUP`     | 挂起。比如管道的写端被关闭后，读端描述符上将收到POLLHUP事件 | 否 | 是 |
| `POLLNVAL`    | 文件描述符没有打开                    | 否            | 是 |

### 返回值
+ `<0`,表示出错
+ `=0`,表示等待超时
+ `>0`,表示就绪的文件描述符数量

## poll就绪条件
与select相同
### 读就绪
+ socket内核中, 接收缓冲区中的字节数, **大于等于**低水位标记SO_RCVLOWAT. 此时可以无阻塞的读该文件描述符, 并且返回值大于0;
+ socket TCP通信中, **对端关闭连接**, 此时对该socket读, 则返回0;
+ 监听的socket上有**新的连接请求**;
+ socket上有**未处理的错误**;

### 写就绪
+ socket内核中, 发送缓冲区中的可用字节数(发送缓冲区的空闲位置大小), 大于等于低水位标记
+ `SO_SNDLOWAT`, 此时可以无阻塞的写, 并且返回值大于0;
+ socket的**写操作被关闭**(close或者shutdown). 对一个写操作被关闭的socket进行写操作, 会触发SIGPIPE信号;
+ socket使用`非阻塞connect连接`成功或失败之后;
+ socket上有**未读取的错误**;

## poll 的优点
+ **对象化**维护文件描述符

不同于`select`使用三个位图来表示三个`fdset`的方式，`poll`所使用的`pollfd`结构体将功能集成到了一个`pollfd`对象中，在`poll`服务器中每个`fd`的封装性和独立性更高，更接近面向对象编程的思想，这导致了如下优点:

+ `poll`所需的传参比`select`更简单更方便
+ `poll`使用的是数组而不是位图，没有最大数量限制（当然数量太大会导致性能下降）

## poll 的缺点
难以**高效**处理大量的文件描述符

+ `后续处理效率低`: `poll`函数返回后，用户仍然需要遍历轮询`pollfd`数组来获取就绪的描述符
+ `大量的拷贝消耗`： 每次调用`poll`都需要把大量的`pollfd`结构从**用户态**拷贝到**内核**中
+ `线性下降的效率`: 同时连接的大量客户端在一时刻可能只有很少的处于就绪状态, 因此随着监视的描述符数量的增长, 其效
率也会线性下降

# 代码实践
实际上`poll服务器`和`select服务器`差别不是很大，仅仅只是`核心函数`和`要维护的数组`发生了变化

## 文件结构

1. `Socket.h` 基于先前封装过的[`Socket`类](https://www.supdriver.top/2024/10/08/LinuxSocket/?highlight=socket)，但修改了日志输出方式
2. `main.cpp`，启动服务的源文件
3. `selectServer.hpp`，服务器的实现文件
4. `makefile` 项目构建文件。

## 框架函数设计
```C++
class PollServer
{
public:
    PollServer(const uint16_t port)
    :_port(port)
    {}

    ~PollServer(){close(_listen_sock.Fd());};

        //初始化函数
        void init();
        //负责调用监听链接请求 或 接收消息
        void dispatcher();
        //负责接收所有就绪的连接请求
        void acceptor();
        //负责接收指定fd的信息
        void recver(struct pollfd& fd,int pos);
        //启动函数
        void start();

private:
    Sock _listen_sock;
    std::vector<struct pollfd> _fd_array;
    uint16_t _port;
};
```
可以看到我们这次使用了`vector容器`管理待监听的文件描述符，因为在`poll服务器`中文件描述符数组的动态性更强，使用`vector`更加适用于动态维护

## makefile 实现
```makefile
pollserver:main.cpp
	g++ -o $@ $^ -std=c++11

.PHONY:clean
clean:
	rm -f pollserver
```

## 完整代码实现
 > PollServer.hpp
```C++
#pragma once

#include <sys/fcntl.h>
#include <unistd.h>
#include <iostream>
#include <string>
#include <poll.h>
#include <string.h>
#include <vector>
#include "Socket.hpp"

static const uint16_t defaultport = 25565;
static const uint16_t fd_num_max = 8;
static struct pollfd default_fd = {-1,0,0};

class PollServer
{
public:
    PollServer(const uint16_t port)
    :_port(port)
    {}

    ~PollServer(){close(_listen_sock.Fd());};

    //初始化函数
    void init()
    {
        _listen_sock.Socket();
        _listen_sock.Bind(_port);
        _listen_sock.Listen();
    }
    //负责调用监听链接请求 或 接收消息
    void dispatcher()
    {
        std::vector<int> del_list;
        int size = _fd_array.size();
        for(int i = 0;i<size;++i)
        {
            struct pollfd fd = _fd_array[i];
            if(fd.fd == default_fd.fd)
            {
                continue;
            }
            if(fd.fd == _listen_sock.Fd() == fd.revents == POLLIN)
            {
                acceptor();
            }
            else if (fd.revents == POLLIN)
            {
                printf("recver\n");
                recver(fd,i,del_list);
            }
        }
        for(int i = del_list.size()-1;i>=0;--i)
        {
            //从后向前删
            printf("已删除连接fd : %d \n",_fd_array[i].fd);
            _fd_array.erase(_fd_array.begin() + del_list[i]);
        }
    }
    //负责接收所有就绪的连接请求
    void acceptor()
    {
        std::string client_ip;
        uint16_t client_port;
        int sock = _listen_sock.Accept(&client_ip,&client_port);
        if(sock < 0)
            return;
        printf("[Info]accept success,[%s,%d]\n",client_ip.c_str(),client_port);
        
        _fd_array[0].revents = 0;//重置状态
        if(_fd_array.size() < fd_num_max)
        {
            _fd_array.push_back({sock,POLLIN,0});
        }
        else 
        {
            printf("[Warning] 连接数超载，已丢弃本次连接 fd :%d",sock);
            close(sock);
        }
    }
    //负责接收指定fd的信息
    void recver(struct pollfd& fd,int pos,std::vector<int>& del_list)
    {
        char buffer[1024] = {0};
        int n = read(fd.fd,buffer,1024-1);
        if(n>0)
        {
            fd.revents = 0;//重置状态
            std::cout<<"poll get msgs#" << buffer << std::endl;
        }
        else if(n == 0)
        {
            //连接断开
            printf("[INFO] client quit,close fd : %d",fd.fd);
            close(fd.fd);
            del_list.push_back(pos);
        }
        else 
        {
            printf("[Warning] read error, fd: %d",fd.fd);
            close(fd.fd);
            del_list.push_back(pos);
        }
    }
    //启动函数
    void start()
    {
        int listenfd = _listen_sock.Fd();
        _fd_array.push_back({listenfd,POLLIN,0});
        int cnt = 0;
        for(;;)
        {
            //DEBUG
            for(auto& fd : _fd_array)
            {
                printf("%d ",fd.fd);
            }printf("\n");
            int ret = poll(_fd_array.data(),_fd_array.size(),1000);
            if(ret < 0)
            {
                //发生错误
                perror("poll");
                continue;
            }
            else if (ret == 0)
            {
                printf("[INFO] poll time out %d\n",cnt++);
                continue;
            }
            else
            {
                dispatcher();
                continue;
            }
        }
    }

private:
    Sock _listen_sock;
    std::vector<struct pollfd> _fd_array;
    uint16_t _port;
};
```

> main.cpp
```C++
#include "PollServer.hpp"

int main()
{
    PollServer svr(8818);
    svr.init();
    svr.start();
    return 0;
}
```