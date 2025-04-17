---
title: 高级IO】①select实现IO多路转接
date: 2024-12-16 08:12:58
tags:
---

以系统提供的函数`select`为核心实现多路转接，即高级网络IO

# 认识select函数
```C
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds,
    fd_set *exceptfds, struct timeval *timeout);
```

## 功能

`select`系统调用能够同时监测**多个文件描述符**的状态变化,这个系统调用是阻塞式的，退出阻塞等待的条件是被监视的文件描述符中有一个或多个发生了变化。

## 参数解释
+ `nfds`: 文件描述符数组长度，值为**最大**的文件描述符**值**+1。*因为文件描述符从0开始*
+ `readfds`: 本质上是`位图`，表示待监视的**可读**文件描述符的集合，返回时**标记**发生变化的fd
+ `writefds`: 本质上是`位图`，表示待监视的**可写**文件描述符集合,返回时**标记**发生变化的fd
+ `exset`: 本质上是`位图`,表示待监视的**异常**文件描述符的集合,返回时**标记**发生变化的fd
+ `timeout`: 用于设置`select()`的等待时间

> struct timeval
```C
struct timeval
{
#ifdef __USE_TIME_BITS64
  __time64_t tv_sec;		/* Seconds.  */
  __suseconds64_t tv_usec;	/* Microseconds.  */
#else
  __time_t tv_sec;		/* Seconds.秒  */
  __suseconds_t tv_usec;	/* Microseconds. 毫秒 */
#endif
};
```

`struct timeval`是一个储存时间的结构体，它提供了两个成员变量用于分别存储**秒**和**毫秒**,因此该参数有三种传参类型

1. `NULL`,表示禁用`select`的超时功能，若没有文件描述符发生变化，将`永久阻塞`
2. `{0,0}`,表示阻塞时间为`0`,即不等待也不会阻塞，仅用于检测文件描述符状态变化
3. `非零`,若在给定的时间内无文件描述符的变化，就会发生超时返回

> fd_set
```C
/* fd_set for select and pselect.  */
typedef struct
  {
    /* XPG4.2 requires this member name.  Otherwise avoid the name
       from the global namespace.  */
#ifdef __USE_XOPEN
    __fd_mask fds_bits[__FD_SETSIZE / __NFDBITS];
# define __FDS_BITS(set) ((set)->fds_bits)
#else
    __fd_mask __fds_bits[__FD_SETSIZE / __NFDBITS];
# define __FDS_BITS(set) ((set)->__fds_bits)
#endif
  } fd_set;
```

虽然在形式上看`fd_set`内部存储了一个数组，但本质来讲就是一个`位图`,用相应位置的`比特位`的`0/1`状态表示该文件描述符是否在集合中

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412160917773.webp)

同时`select.h`也提供了一系列接口来帮助操作这些封装在`fd_set`里的`位图`

把它们按C接口的风格表示如下.*实际上它们都是封装过的宏函数*
```C
void FD_CLR(int fd, fd_set *set); // 用来清除描述词组set中相关fd 的位
int FD_ISSET(int fd, fd_set *set); // 用来测试描述词组set中相关fd 的位是否为真
void FD_SET(int fd, fd_set *set); // 用来设置描述词组set中相关fd的位
void FD_ZERO(fd_set *set); // 用来清除描述词组set的全部位
```

## 返回值分析
+ `返回非零`: 执行成功，返回文件描述符状态改变的**个数**
+ `返回0`: 代表超时返回，没有检测到状态改变
+ `返回-1`: 发生错误，错误的原因会设置在errno,此时输出型参数`readfds`,`writefds`,`exceptfds`,`timeout`的值变得不可预测

`errno`对应的错误为：
+ `EADF` 文件描述词为无效的或该文件已关闭
+ `EINTR` 此调用被信号所中断
+ `EINVAL` 参数n 为负值。
+ `ENOMEM` 核心内存不足


## select就绪条件

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

## 特点/使用要点
+ 可监控的文件描述符**个数**取决于`sizeof(fd_set)`
+ 需要**额外**的数据结构`array`保存放到`select`监控集中的`fd`,原因如下
  + 传入的`fd_set`是输出型参数，返回后待监控的发生变化的`fd`对应的标志位会置`1`,而未发生变化的`fd`对应的标志位会清空，导致原本传入的参数信息丢失，所以需要额外存储

## 缺点
由此我们可以总结出`select`的一些缺点

1. 每次调用前都要设置待监测的`fd`集合
2. 每次调用`select`都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
3. 同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
4. `select`支持的文件描述符数量太小


# 代码实践

## 文件结构
1. `Socket.h` 基于先前封装过的[`Socket`类](https://www.supdriver.top/2024/10/08/LinuxSocket/?highlight=socket)，但修改了日志输出方式
2. `main.cpp`，启动服务的源文件
3. `selectServer.hpp`，服务器的实现文件
4. `makefile` 项目构建文件。


## Socket.hpp

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
//TODO
const int backlog = 10;

class Sock
{
public:
    Sock(){}
    ~Sock(){}
public:
    void Socket()
    {
        _sockfd = socket(AF_INET,SOCK_STREAM,0);
        if(_sockfd < 0)
        {
            printf("[Fatal]socket error,errno: %d error string:%s",errno,strerror(errno));
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
            printf("[Fatal]bind error,errno: %d error string:%s",errno,strerror(errno));
            exit(BIND_ERR);
        }   
    }

    void Listen()
    {
        if(listen(_sockfd,backlog) < 0)
        {
            printf("[Fatal]listen error,errno: %d error string:%s",errno,strerror(errno));
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
            printf("[Fatal]accept error,errno: %d error string:%s",errno,strerror(errno));
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
    void Close()
    {
        close(_sockfd);
    }

    int Fd()
    {
        return _sockfd;
    }
private:
    int _sockfd;
};
```

## makefile 实现
```makefile
selectServer:main.cpp
	g++ -o $@ $^ -std=c++11

.PHONY:clean
clean:
	rm -f selectServer
```

## selectServer.hpp 实现

### 引入头文件和定义全局变量

```C++
#pragma once

#include <sys/select.h> //提供select函数
#include <fcntl.h>      //提供文件控制
#include <iostream>     //提供输入输出流
#include "Socket.hpp"   
#include <string>       

static const uint16_t defualt_port = 8976;
static const uint16_t fd_num_max = sizeof(fd_set)*8;
const int defualt_fd = -1;

```

## 实现 SelectServer类

### 框架函数设计

```C++
class SelectServer
{
public:
    SelectServer(uint16_t port = default_port)
    : _port(port)
    {
        for (int i = 0; i < fd_num_max; ++i)//初始化
        {
            _fd_array[i] = default_fd;
        }
    }

    ~SelectServer() { _listen_sock.Close(); }

    //初始化函数
    void init();
    //负责调用监听链接请求 或 接收消息
    void dispatcher(fd_set & rfds);
    //负责接收所有就绪的连接请求
    void acceptor();
    //负责接收指定fd的信息
    void recver(int fd,int pos);
    //启动函数
    void start();

private:
    Sock _listen_sock;
    uint16_t _port;
    int _fd_array[fd_num_max];
}
```                                      
从成员变量也能看出，在`select`服务器中，我们需要重点维护`_listen_sock`和`_fd_array`两个变量

+ `_listen_sock`负责监听处理网络连接请求
+ `_fd_array`负责储存处于被监控状态下的所有`fd`

### 完整代码实现
把每个接口实现之后，一个简单的`select`服务器就完成了

```C++
#pragma once

#include <fcntl.h>
#include <iostream>
#include <sys/select.h>
#include "Socket.hpp"
#include <sys/time.h>
#include <string>

using std::cerr;
using std::cout;
using std::endl;

extern Log log;

static const uint16_t defaultport = 25565;
static const uint16_t fd_num_max = sizeof(fd_set) * 8;
const int defaultfd = -1;

class SelectServer
{
public:
    SelectServer(uint16_t port = defaultport)
        : _port(port)
    {
        for (int i = 0; i < fd_num_max; ++i)
        {
            _fd_array[i] = defaultfd;
        }
    }

    ~SelectServer() { _listensock.Close(); }

    void Init()
    {
        _listensock.Socket();
        _listensock.Bind(_port);
        _listensock.Listen();
    }

    void Dispatcher(fd_set &rfds)
    {
        for (int i = 0; i < fd_num_max; ++i)
        {
            int fd = _fd_array[i];
            if (fd == defaultfd)
            {
                continue;
            }
            else if (FD_ISSET(fd, &rfds))
            {
                if (fd == _listensock.Fd())
                {
                    Acceptor();
                }
                else
                {
                    Recver(fd, i);
                }
            }
        }
    }

    void Acceptor()
    {
        // 链接事件就绪
        std::string clientip;
        uint16_t clientport = 0;
        int sock = _listensock.Accept(&clientip, &clientport);
        if (sock < 0)
            return;
        log(Info, "accept success, [%s:%d]", clientip.c_str(), clientport);

        int pos = fd_num_max;
        for (int i = 0; i < fd_num_max; ++i)
        {
            if (_fd_array[i] == defaultfd)
            {
                pos = i;
                break;
            }
        }
        if (pos < fd_num_max)
        {
            _fd_array[pos] = sock;
        }
        else
        {
            log(Warning, "连接数超载，已丢弃本次链接 fd: %d", sock);
            close(sock);
        }
    }

    void Recver(int fd, int i)
    {
        char buffer[1024] = {0};
        int n = read(fd, buffer, 1024 - 1);
        if (n > 0)
        {
            cout << "get a msg@ " << buffer << endl;
        }
        else if (n == 0)
        {
            log(Info, "client quit,close fd:%d", fd);
            close(fd);
            _fd_array[i] = defaultfd;
        }
        else
        {
            log(Warning, "read error ,fd:%d", fd);
            close(fd);
            _fd_array[i] = defaultfd;
        }
    }

    void Start()
    {

        int listenfd = _listensock.Fd();
        _fd_array[0] = _listensock.Fd();

        for (;;)
        {
            fd_set rfds;
            FD_ZERO(&rfds);
            struct timeval timeout = {1, 0};

            int maxfd = _fd_array[0];
            for (int i = 0; i < fd_num_max; ++i)
            {
                if (_fd_array[i] == defaultfd)
                    continue;
                ;
                FD_SET(_fd_array[i], &rfds);
                if (maxfd < _fd_array[i])
                {
                    maxfd = _fd_array[i];
                }
            }

            int n = select(maxfd + 1, &rfds, nullptr, nullptr, &timeout);
            switch (n)
            {
            case 0:
                cout << "time out timeout:" << timeout.tv_sec << "." << timeout.tv_usec << endl;
                break;
            case -1:
                cerr << "select err" << endl;
                break;

            default:
                log(Info, "get a new link");
                Dispatcher(rfds);
                break;
            }
        }
    }

private:
    Sock _listensock;
    uint16_t _port;
    int _fd_array[fd_num_max];
};
```

> main.cpp
```C++
#include "selectServer.hpp"
#include <memory>

int main()
{
    std::unique_ptr<SelectServer> svr(new SelectServer());
    svr->init();
    svr->start();
    return 0;
}
```