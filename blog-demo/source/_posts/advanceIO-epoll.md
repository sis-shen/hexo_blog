---
title: 高级IO】③epoll实现IO多路转接
date: 2025-02-19 21:45:37
tags:
# hidden: true
---
以`epoll`相关接口构成的高级IO

不仅是名称上类似，`epoll`与`poll`的功能也是类似的，但是按`man`手册的说法，`epoll`是为处理大批量句柄而做了改进的`poll`，它几乎具备了之前所介绍的`select`和`poll`构成的高级IO的一切优点，因此它被公认为`Linux2.6`下性能最好的多路`I/O`就绪通知方法
# 认识 epoll 系列接口
`epoll`有`3`个相关的系统调用

来自头文件`<sys/epoll.h>`

## `epoll_create`

该接口用于创建`epoll实例`,它的返回值我们称为`epfd`

```C
int epoll_create(int size);
```

根据`man`手册，自`Linux 2.6.8`之后，参数`size`**将会被忽略，但是必须大于**`0`

## `epoll_ctl`

`epoll`的事件注册函数

```C
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

这一系统用于`增`、`改`、`查`位于`epfd`对应的`epoll`实例内的待监听的文件描述符列表

+ **参数描述**
  + `epfd`: 由先前的`epoll_create`产生的`epfd`
  + `op`: 操作代码，有如下三个可选宏
    + `EPOLL_CTL_ADD`: 注册新的`fd`到`epfd`中
    + `EPOLL_CTL_MOD`: 修改已注册的`fd`的监听事件
    + `EPOLL_CTL_DEL`: 从`epfd`中删除一个`fd`
  + `fd`: 被操作的`fd`
  + `event`: 告诉内核需要监听什么事
+ `struct epoll_event`的结构如下

```C
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events 事件*/
    epoll_data_t data;        /* User data variable 用户数据变量*/
};
```
`events`是位图，可以是以下几个宏的集合

+ EPOLLIN  : 表示对应的文件描述符可以读 (包括对端SOCKET正常关闭); 
+ EPOLLOUT : 表示对应的文件描述符可以写;
+ EPOLLPRI : 表示对应的文件描述符有紧急的数据可读 (这里应该表示有带外数据到来);
+ EPOLLERR : 表示对应的文件描述符发生错误;
+ EPOLLHUP : 表示对应的文件描述符被挂断;
+ EPOLLET  : 将EPOLL设为边缘触发(Edge Triggered)模式, 这是相对于水平触发(Level Triggered)来说的.
+ EPOLLONESHOT：**只监听一次事件**, 当监听完这次事件之后, 如果还需要继续监听这个socket的话, 需要再次把这个socket加入到EPOLL队列里

## epoll_wait

```C
int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);
```
`epoll_wait`用于收集在`epoll`监控的事件中已发生的事件

+ **参数介绍**
  + `epfd`: 指定`epoll实例`
  + `events`: 输出型参数，需要传入一段内存缓冲区，`epoll`会将数组写入该指针指向的地址内
  + `maxevents`: 上个参数表示的数组的最大长度,即`event`的最大数量
  + `timeout`: 超时时间
    + `0`: 立即返回
    + `非0`: 等待timeout毫秒
    + `-1`: 永久阻塞等待
+ **返回值**: 
  + `>0`表示已I/O就绪的文件描述符数目
  + `0`表示超时返回
  + `负数`表示函数失败

# epoll 的优点
+ **接收使用方便**: 虽然将`poll`拆成了三个函数,但实际上使用起来更加方便高效，不需要每次循环都设置关注的文件描述符，也做到了输入输出分离
+ **数据拷贝轻量**: 由于内核中`epoll实例`的存在，只需要在合适的时候调用`EPOLL_CTL_ADD`操作将`fd`拷贝到内核中
+ **事件回调机制**: 避免使用遍历, 而是使用回调函数的方式, 将就绪的文件描述符结构加入到就绪队列中, epoll_wait 返回直接访问就绪队列就知道哪些文件描述符就绪. 这个操作时间复杂度O(1). 即使文件描述符数目很多, 效率也不会受到影响. 
+ **没有数量限制**: 文件描述符数目无上限

# epoll 的工作模式--边缘触发/水平触发

## 两种触发模式的示意图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502211050287.webp)

## 水平触发Level Triggered 工作模式

`epoll`**默认状态**下的工作模式就是水平触发模式

如果一次`epoll_wait`触发后，没有把`socket`缓冲区内的数据读完，第二次调用`epoll_wait`时，`epoll_wait`仍然能被触发并立刻返回通知`socket`读事件就绪

+ 支持阻塞读写和非阻塞读写

## 边缘触发Edge Triggered工作模式

如果我们在第1步将socket添加到epoll描述符的时候使用了EPOLLET标志, epoll进入ET工作模式.

如果一次`epoll_wait`触发后，必须立刻全部处理，如果没有把`socket`缓冲区内的数据读完，第二次调用`epoll_wait`时，`epoll_wait`不会再次出发，不会返回。也就是说在`ET`模式下文件描述符上的事件就绪后，只有一次处理机会

相应的由于`epoll_wait`返回的次数少了很多，`ET`的性能比`LT`的性能更高。例如`Nginx`默认就采用`ET模式`使用`epoll`

+ 仅支持非阻塞的读写

## 对比 ET 和 LT

`ET`能够减少`epoll`出发的次数（即用户态和内核态的切换）, 但代价是代码复杂程度更高了，硬逼着程序员在一次响应就绪过程中把所有数据都处理完。

# epoll 的使用场景
epoll的高性能, 是有一定的特定场景的. 如果场景选择的不适宜, epoll的性能可能适得其反.

对于多连接, 且多连接中只有一部分连接比较活跃时, 比较适合使用epoll. 

例如, 典型的一个需要处理上万个客户端的服务器, 例如各种互联网APP的入口服务器, 这样的服务器就很适合epoll.

如果只是系统内部, 服务器和服务器之间进行通信, 只有少数的几个连接, 这种情况下用epoll就并不合适. 具体要根

据需求和场景特点来决定使用哪种IO模型

# epoll 中的惊群问题
很多朋友都在`Linux`下使用`epoll`编写过`socket`的服务端程序，在多线程环境下可能会遇到`epoll`的`惊群效应`。

在多线程或者多进程环境下，有些人为了提高程序的稳定性，往往会让**多个线程或者多个进程同时**在`epoll_wait`监听的`socket`描述符。当一个新的链接请求进来时，操作系统不知道选派那个线程或者进程处理此事件，则干脆将其中几个线程或者进程给唤醒，而实际上只有其中一个进程或者线程能够成功处理`accept`事件，其他线程都将失败，且`errno`错误码为`EAGAIN`。这种现象称为`惊群效应`，结果是肯定的，惊群效应肯定会带来资源的消耗和性能的影响。

# 代码实践- LT模式epoll服务器

## 完善Sock类
为了完成通信，主要增加的改动是增加了`Recv`接口和`Send`接口

> Sock.hpp
```C++
#pragma once

#include <unistd.h>
#include <iostream>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <cstring>
#include <cstdlib>
#include <string>

enum{
    SOCK_ERR = 2
    ,BIND_ERR
    ,LISTEN_ERR
    ,ACCEPT_ERR
};

class Sock
{
public:
    Sock(){}
    Sock(int fd):_sockfd(fd){}
    ~Sock(){}
public:

    void Socket()
    {
        _sockfd = socket(AF_INET,SOCK_STREAM,0);
        if(_sockfd < 0)
        {
            printf("[Fatal]socket error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(SOCK_ERR);
        }
        int opt = 1;
        setsockopt(_sockfd,SOL_SOCKET,SO_REUSEADDR | SO_REUSEPORT,&opt,sizeof(opt));
    }
    void Bind(uint16_t port)
    {  
        struct sockaddr_in local;
        memset(&local,0,sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(port);
        local.sin_addr.s_addr = INADDR_ANY;
        if(bind(_sockfd,(const struct sockaddr*)&local,sizeof(local)) < 0)
        {
            printf("[Fatal]bind error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(BIND_ERR);
        }   
    }

    void Listen(int backlog = 5)
    {
        if(listen(_sockfd,backlog) < 0)
        {
            printf("[Fatal]listen error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(LISTEN_ERR);
        }
    }
    int Accept(std::string* clientip,uint16_t* clienport)
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int newfd = accept(_sockfd,(struct sockaddr*)&peer,&len);
        if(newfd < 0)
        {
            printf("[Fatal]accept error,errno: %d error string:%s\n",errno,strerror(errno));
            return -1;
        }
        char ipstr[64] = {0};
        inet_ntop(AF_INET,&peer.sin_addr,ipstr,sizeof(ipstr));
        *clientip = ipstr;
        *clienport = ntohs(peer.sin_port);
        return newfd;
    }
    bool Connect(const std::string& ip,uint16_t port)
    {
        struct sockaddr_in peer;
        memset(&peer,0,sizeof(peer));
        peer.sin_family =AF_INET;
        peer.sin_port = htons(port);
        inet_pton(AF_INET,ip.c_str(),&(peer.sin_addr));

        int n = connect(_sockfd,(struct sockaddr*)&peer,sizeof(peer));
        if(n == -1) 
        {
            std::cerr<<"connect to "<<ip<<":"<<port<<std::endl;
            return false;
        }

        return true;

    }

    int Recv(std::string& req)
    {
        char buffer[1024] = {0};
        int ret = read(_sockfd,buffer,1024-1);
        if(ret == 0)
        {
            //连接关闭
            return 0;
        }
        else if(ret < 0)
        {
            //发生错误
            perror("read");
            return ret;
        }
        else
        {
            //正确读取
            req = std::string(buffer);
        }
        return ret;
    }

    int Send(const std::string&res)
    {
        return write(_sockfd,res.c_str(),res.size());
    }

    void Close()
    {
        close(_sockfd);
    }

    int Fd() const
    {
        return _sockfd;
    }
private:
    int _sockfd;
};
```

## 封装Epoll类
为了进一步简化对epoll的调用，我们把它封装到`Epoll`类中，提供如下接口

+ `Epoll()`构造函数，提供对epoll实例的创建
+ `Add()`增加被监听的套接字文件
+ `Del()`从epoll中删除指定文件描述符
+ `Wait()`获取所有就绪的文件描述符--这里使用了输出型参数

> Epoll.hpp
```C++
#pragma once
#include <sys/epoll.h>
// #include <sys/fcntl.h>
#include <unistd.h>
#include <vector>
#include "Sock.hpp"

class Epoll
{
public:
    Epoll(){
        _epoll_fd = epoll_create(10);
    }   

    ~Epoll(){
        close(_epoll_fd);
    }

    bool Add(const Sock& sock)
    {
        int fd = sock.Fd();
        printf("[Epoll Add] fd = %d\n",fd);
        epoll_event ev;
        ev.data.fd = fd;
        ev.events = EPOLLIN;
        int ret = epoll_ctl(_epoll_fd,EPOLL_CTL_ADD,fd,&ev);
        if(ret < 0)
        {
            perror("epoll_ctl ADD");
            return false;
        }
        return true;
    }

    bool Del(const Sock&sock)
    {
        int fd = sock.Fd();
        printf("[Epoll Del] fd = %d\n",fd);
        int ret = epoll_ctl(_epoll_fd,EPOLL_CTL_DEL,fd,NULL);
        if(ret < 0)
        {
            perror("epoll_ctl DEL");
            return false;
        }
        return true;
    }

    bool Wait(std::vector<Sock>*output)
    {
        output->clear();
        epoll_event events[1000] = {0};
        int nfds = epoll_wait(_epoll_fd,events,(sizeof(events) / sizeof(events[0])),-1);
        if(nfds < 0)
        {
            perror("epoll_wait");
            return false;
        }
        for(int i = 0;i<nfds;++i)
        {
            output->push_back(Sock(events[i].data.fd));
        }
        return true;
    }

private:
    int _epoll_fd;
};

```
## 封装EpollServer类

实现思路如下

1. 创建`监听套接字`，绑定接口，然后开始监听
2. 创建`Epoll`对象，并将监听套接字加入`Epoll`对象
3. 进入事件循环，每次循环`Epoll`都执行一次`Wait`，然后处理就绪的连接事件或者读写事件

> EpollServer.hpp
```C++
#include "Epoll.hpp"
#include <string>
#include <functional>

typedef std::function<void(const std::string& req,std::string&res)> Handler;

static Handler default_hander([](const std::string&req,std::string& res){
    res = req;//做一个回显处理
    return;
});

//这是一个只读Server
class EpollServer
{
public:
    EpollServer(uint16_t port = 80,Handler handler = default_hander):_port(port),_handler(handler){}

    void Start()
    {
        //创建监听套接字
        Sock listen_sock;
        listen_sock.Socket();
        listen_sock.Bind(_port);
        listen_sock.Listen(10);
        std::cout<<"服务器在端口:"<< _port << "开放!\n";
        Epoll epoll;//实例化一个epoll对象
        if(!epoll.Add(listen_sock)) return;
        //进入事件循环
        for(;;)
        {
            std::vector<Sock> output;
            if(!epoll.Wait(&output))
            {
                continue;
            }
            for(auto sock : output)
            {
                if(sock.Fd() == listen_sock.Fd())
                {
                    //监听套接字调用Accept
                    std::string client_ip;
                    uint16_t client_port;
                    int fd = listen_sock.Accept(&client_ip,&client_port);
                    epoll.Add(Sock(fd));
                    //因为对客户端具体信息不感兴趣，就丢弃了
                }
                else
                {
                    //如果是用于通信，则进行一次读写
                    std::string req,res;//request 和 response
                    int ret = sock.Recv(req);//获取request
                    if(ret <= 0)
                    {
                        //发生错误,或者关闭连接都要去除
                        epoll.Del(sock);
                        sock.Close();
                        continue;
                    }
                    std::cout<<"Server get msg@"<<req<<"\n";
                    _handler(req,res);//获取响应
                    ret = sock.Send(res);
                    if(ret < 0)
                    {
                        //发生错误,或者关闭连接都要去除
                        epoll.Del(sock);
                        sock.Close();
                        continue;
                    }
                }
            }
        }
    }

private:
    uint16_t _port;
    Handler _handler;
};
```


