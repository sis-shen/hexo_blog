---
title:  高级IO】④Reactor事件驱动模型探究
date: 2025-02-27 13:43:03
tags:
---
Reactor模型是相对与传统的阻塞式I/O模型提出来的，**节约线程资源**和**提升业务处理并发量**的IO模型，它主要基于`I/O多路复用`配合`线程池`的思想来实现。不过实际上Reactor模式依然是一种同步I/O模型

# 对比参照-传统阻塞式I/O模型

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502271409759.webp)

可以看到当并发规模增大时，这一模型有着显著的性能问题:

1. **线程资源消耗大**: 每一个客户端链接都需要唯一的线程维护链接和处理业务，开销迅速上升
2. **线程利用率低**: 当出现**连接数很高**而**实时活跃连接数**较低时，大部分线程处于阻塞状态，浪费了大部分线程的处理性能。

# Reactor模型的提出
基于传统阻塞式I/O的两大痛点，Reactor模型使用了两大思想对应解决:

1. `I/O多路复用`: 多个连接由同一个对象同时监听，当事件就绪时输出所有就绪的链接（在C++中则是文件描述符）。这样维护链接的任务就可以统一交给实现I/O多路复用的对象中，而不用分散给每一个线程。
2. `线程池`: 当有任务就绪时，则将其分配给线程池中的线程，而但任务完成后，就将空闲的线程还给线程池，这样可以解决线程利用率低的问题。当然相应地它也引出了动态扩容和负载均衡等进一步优化性能的问题。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202502271440763.webp)

## Reactor 模式的组成元素

1. Reactor 反应器
   1. 持有Selector对象
   2. 调用Selector接口,负责监听&分发事件（如连接、读写）
2. Selector 多路复用器
   1. 调用系统接口实现多路复用
3. Handler 事件处理器
   1. 负责解码和业务处理

## 常见线程模型
根据Reactor数量和线程的数量可以分为三种常见模型

1. 单Reactor单线程模型
   +  所有操作在同一个线程完成
2. 单Reactor多线程模型
   + Reactor主线程处理I/O，业务逻辑丢给线程池
   + **Reactor依旧是性能瓶颈** ，且存在读事件与连接事件竞争的问题
3. 主从Reactor多线程模型
   + 主Reactor只处理连接accept，子Reactor负责I/O读写 

# Reactor模型实现
接下来我们使用C++来实现这三种常见的模型

## 单Reactor单线程模型

我们通过三个文件实现

+ EventHandler.hpp  实现事件处理器
+ Selector.hpp      实现I/O多路转接
+ Reactor.hpp       实现Reactor模式

> EventHandler.hpp
```C++
#include <unistd.h>
#include <fcntl.h>
#include <iostream>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include "Selector.hpp"
class EventHandler
{
public:
    virtual void hanle_event(int fd) = 0;
    virtual ~EventHandler() = default;

    Selector* selector;
};

//一次业务把读写都完成了
class EchoHandler:public EventHandler
{
public:
    void hanle_event(int fd)override
    {
        char buffer[1024] = {0};
        int ret = read(fd,buffer,1024-1);
        if(ret == 0)
        {
            //连接关闭
            close(fd);
            selector->Del(fd);
        }
        else if (ret < 0)
        {
            //连接错误
            perror("read fd");
            close(fd);
            selector->Del(fd);
        }

        printf("Get a msg@ %s",buffer);
        //再写回
        write(fd,buffer,ret);
    }

};
```

> Selector.hpp
```C++
#pragma once
#include <sys/epoll.h>
#include <unistd.h>
#include <vector>
#include <iostream>

//poll实现Selector
class Selector
{
public:
    Selector(){
        _epoll_fd = epoll_create(10);
    }   

    ~Selector(){
        close(_epoll_fd);
    }

    bool Add(const int& sock)
    {
        int fd = sock;
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

    bool Del(int fd)
    {
        printf("[Epoll Del] fd = %d\n",fd);
        int ret = epoll_ctl(_epoll_fd,EPOLL_CTL_DEL,fd,NULL);
        if(ret < 0)
        {
            perror("epoll_ctl DEL");
            return false;
        }
        return true;
    }

    bool Wait(std::vector<int>*output)
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
            output->push_back(events[i].data.fd);
        }
        return true;
    }

private:
    int _epoll_fd;
};


```

> Reactor.hpp
```C++
#include "EventHalder.hpp"
#include <memory.h>
#include <memory>
class ReactorServer
{
public:
    ReactorServer(uint16_t port,std::shared_ptr<EventHandler>handler = std::make_shared<EchoHandler>()):_port(port),_handler(handler)
    {
        _listen_sock = socket(AF_INET,SOCK_STREAM,0);
        _handler->selector = &selector;
    }

    void start()
    {
        Bind();
        Listen();
        selector.Add(_listen_sock);

        for(;;)
        {
            std::vector<int>fds;
            selector.Wait(&fds);//获取就绪事件
            Dispatcher(fds);//处理事件
        }
    }

    ~ReactorServer()
    {
        close(_listen_sock);
    }

protected:
    void Dispatcher(const std::vector<int>&fds)
    {
        for(const auto& fd:fds)
        {
            if(fd == _listen_sock)
            {
                Acceptor();
            }
            else
            {
                _handler->hanle_event(fd);//单线程就不搞读写分离了
            }
        }
    }

    void Acceptor()
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int newfd = accept(_listen_sock,(struct sockaddr*)&peer,&len);
        if(newfd < 0)
        {
            printf("[Fatal]accept error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
        //不关心客户端信息
        // char ipstr[64] = {0};
        // inet_ntop(AF_INET,&peer.sin_addr,ipstr,sizeof(ipstr));
        selector.Add(newfd);
    }

    void Bind()
    {
        struct sockaddr_in local;
        memset(&local,0,sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);
        local.sin_addr.s_addr = INADDR_ANY;//自动分配IP
        if(bind(_listen_sock,(const struct sockaddr*)&local,sizeof(local)) < 0)
        {
            printf("[Fatal]bind error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

    void Listen(int backlog = 10)
    {
        if(listen(_listen_sock,backlog) < 0)
        {
            printf("[Fatal]listen error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }



protected:
    int _listen_sock;
    uint16_t _port;
    std::shared_ptr<EventHandler> _handler;
    Selector selector;
};
```

## 单Reactor多线程模型
我们来在第一个模型的基础上修改出第二个模型

因为新增了多线程的特性,我们有如下工作要做:

1. `Selector`加锁保护各个接口
2. `EventHandler`功能扩充
   1. 读写处理分离
   2. 新增写操作的缓冲区
   3. 新增针对每个`fd`的锁
   4. 新增针对锁图的互斥锁
3. `Reactor`微调: 将监听套接字改为非阻塞等
4. `ThreadPool`新增 
   1. 维护任务队列
   2. 分配线程处理任务

> Selector.hpp 
```C++
#pragma once
#include <sys/epoll.h>
#include <unistd.h>
#include <vector>
#include <iostream>
#include <mutex>

//poll实现Selector
class Selector
{
public:
    Selector(){
        _epoll_fd = epoll_create(10);
    }   

    ~Selector(){
        close(_epoll_fd);
    }

    bool Add(const int& sock)
    {
        std::unique_lock<std::mutex> lock(_mtx);
        int fd = sock;
        printf("[Epoll Add] fd = %d\n",fd);
        epoll_event ev;
        ev.data.fd = fd;
        ev.events = EPOLLIN | EPOLLET;
        int ret = epoll_ctl(_epoll_fd,EPOLL_CTL_ADD,fd,&ev);
        if(ret < 0)
        {
            perror("epoll_ctl ADD");
            return false;
        }
        return true;
    }

    bool Del(int fd)
    {
        std::unique_lock<std::mutex> lock(_mtx);
        printf("[Epoll Del] fd = %d\n",fd);
        int ret = epoll_ctl(_epoll_fd,EPOLL_CTL_DEL,fd,NULL);
        if(ret < 0)
        {
            perror("epoll_ctl DEL");
            return false;
        }
        return true;
    }

    bool Mod(int fd,struct epoll_event&ev)
    {
        std::unique_lock<std::mutex> lock(_mtx);
        printf("[Epoll Mod] fd = %d\n",fd);
        int ret = epoll_ctl(_epoll_fd,EPOLL_CTL_DEL,fd,&ev);
        if(ret < 0)
        {
            perror("epoll_ctl MOD");
            return false;
        }
        return true;
    }

    bool Wait(std::vector<struct epoll_event>*output)
    {
        std::unique_lock<std::mutex> lock(_mtx);
        output->clear();
        struct epoll_event events[1000] = {0};
        int nfds = epoll_wait(_epoll_fd,events,(sizeof(events) / sizeof(events[0])),-1);
        if(nfds < 0)
        {
            perror("epoll_wait");
            return false;
        }
        for(int i = 0;i<nfds;++i)
        {
            output->push_back(events[i]);
        }
        return true;
    }

private:
    int _epoll_fd;
    std::mutex _mtx;
};

```

> EventHalder.hpp 
```C++
#pragma once

#include <unistd.h>
#include <fcntl.h>
#include <iostream>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unordered_map>
#include <string>
#include <sys/epoll.h>
#include <shared_mutex>
#include "Selector.hpp"
class EventHandler
{
public:
    virtual void handle_read(int fd) = 0;
    virtual void handle_write(int fd) = 0;
    virtual ~EventHandler() = default;

    std::unordered_map<int,std::string> _buffer_hash;
    std::unordered_map<int,std::shared_mutex> _fd_mutex;
    Selector* selector;
};

class EchoHandler:public EventHandler
{
public:
    void handle_read(int fd)override
    {
        std::unique_lock<std::shared_mutex> lock(getFdMutex(fd));//写锁
        char buffer[1024] = {0};
        int ret = read(fd,buffer,1024-1);
        if(ret == 0)
        {
            //连接关闭
            close(fd);
            selector->Del(fd);
        }
        else if (ret < 0)
        {
            //连接错误
            perror("read fd");
            close(fd);
            selector->Del(fd);
        }
        buffer[ret] = '\0';
        printf("Get a msg@ %s",buffer);
        //再写回
        ssize_t n = send(fd,buffer,sizeof(buffer),MSG_DONTWAIT);//非阻塞发送
        if(n == -1)
        {
            //如果发送失败
            if(errno == EAGAIN || errno == EWOULDBLOCK)
            {
                //发送缓冲区已满，注册 EPOLLOUT等待可写事件
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events= EPOLLIN | EPOLLOUT | EPOLLET;
                selector->Mod(fd,ev);//注册侦听写事件就绪
                std::string& buf_str = _buffer_hash[fd];
                buf_str.append(buffer);//写入缓冲区
            }
            else 
            {
                //无法处理则关闭连接
                close(fd);
                eraseFdMutex(fd);
                selector->Del(fd);
            }
        }
    }

    void handle_write(int fd)override
    {
        std::unique_lock<std::shared_mutex> lock(getFdMutex(fd));//写锁
        auto&str = _buffer_hash[fd];
        ssize_t n = send(fd,str.c_str(),str.size(),MSG_DONTWAIT);//非阻塞发送
        if(n == -1)
        {
            //如果发送失败
            if(errno == EAGAIN || errno == EWOULDBLOCK)
            {
                //什么也不做，等待下次就绪
                //发送缓冲区已满，注册 EPOLLOUT等待可写事件
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events= EPOLLIN | EPOLLOUT | EPOLLET;
                selector->Mod(fd,ev);//注册侦听写事件就绪
                return;
            }
            else 
            {
                //无法处理则关闭连接
                close(fd);
                eraseFdMutex(fd);
                selector->Del(fd);
            }
        }

        if (n > 0) {
            str.erase(0, n); // 删除已发送部分
            if (str.empty()) {
                _buffer_hash.erase(fd);
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events= EPOLLIN | EPOLLET;
                selector->Mod(fd,ev);
                // 取消 EPOLLOUT 监听
            } else {
                // 保留未发送数据，等待下次写入
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events = EPOLLIN | EPOLLOUT | EPOLLET;
                selector->Mod(fd, ev);
            }
        }
        
    }

    std::shared_mutex& getFdMutex(int fd)
    {
        std::unique_lock<std::mutex> lock(_map_mutex);//使对锁表的查询互斥
        return _fd_mutex[fd];//若锁不存在，则会自动构造
    }

    void eraseFdMutex(int fd)
    {
        std::unique_lock<std::mutex> lock(_map_mutex);//使对锁表的查询互斥
        _fd_mutex.erase(fd);
    }
private:
    std::mutex _map_mutex;
};
```

> Reactor.hpp 
```C++
#pragma once

#include <memory.h>
#include <memory>
#include "ThreadPool.hpp"
class ReactorServer
{
public:
    ReactorServer(uint16_t port,std::shared_ptr<EventHandler>handler = std::make_shared<EchoHandler>())
    :_port(port),_selector(),_pool(handler)
    {
        _listen_sock = socket(AF_INET,SOCK_STREAM | SOCK_NONBLOCK,0);
        _pool.setSelector(&_selector);
    }

    void start()
    {
        Bind();
        Listen();
        _selector.Add(_listen_sock);

        for(;;)
        {
            std::vector<struct epoll_event>events;
            _selector.Wait(&events);//获取就绪事件
            Dispatcher(events);//处理事件
        }
    }

    ~ReactorServer()
    {
        close(_listen_sock);
    }

protected:
    void Dispatcher(const std::vector<struct epoll_event>&events)
    {
        printf("get events: %lu\n",events.size());
        for(const auto& event:events)
        {
            if(event.data.fd == _listen_sock)
            {
                Acceptor();
            }
            else
            {
                //读写分离
                if(event.events & EPOLLIN)
                {
                    //创建写事件任务
                    _pool.addTask(event.data.fd,reactor::Task::READ);
                }
                if(event.events & EPOLLOUT)
                {
                    _pool.addTask(event.data.fd,reactor::Task::SEND);
                }
            }
        }
    }

    void Acceptor()
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int newfd = accept(_listen_sock,(struct sockaddr*)&peer,&len);
        if(newfd < 0)
        {
            printf("[Fatal]accept error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
        //不关心客户端信息
        // char ipstr[64] = {0};
        // inet_ntop(AF_INET,&peer.sin_addr,ipstr,sizeof(ipstr));
        // 设置非阻塞标志
        // int flags = fcntl(newfd, F_GETFL, 0);
        // fcntl(newfd, F_SETFL, flags | O_NONBLOCK);//设置为非阻塞
        _selector.Add(newfd);
    }

    void Bind()
    {
        int opt = 1;
        setsockopt(_listen_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
        struct sockaddr_in local;
        memset(&local,0,sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);
        local.sin_addr.s_addr = INADDR_ANY;//自动分配IP
        if(bind(_listen_sock,(const struct sockaddr*)&local,sizeof(local)) < 0)
        {
            printf("[Fatal]bind error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

    void Listen(int backlog = 10)
    {
        if(listen(_listen_sock,backlog) < 0)
        {
            printf("[Fatal]listen error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

protected:
    int _listen_sock;
    uint16_t _port;
    Selector _selector;
    reactor::ThreadPool _pool;
};
```

> ThreadPool.hpp
```C++
#pragma once

#include <thread>
#include <condition_variable>
#include <queue>
#include <vector>
#include <functional>
#include <mutex>
#include <atomic>
#include "EventHandler.hpp"
namespace reactor
{
struct Task
{
    enum Type
    {
        READ,
        SEND
    };
    int fd;
    Type type;
};

class ThreadPool
{
public:

    ThreadPool(std::shared_ptr<EventHandler> handler,int capacity=20,int n = 5)
    :_isRunning(true),_handler(handler),_capacity(capacity)
    {
        for(int i = 0;i<n;++i)
        {
            _pool.emplace_back(std::thread(std::bind(&ThreadPool::worker,this)));
        }
    }

    void worker()
    {
        for(;;)
        {
            if(_isRunning == false)return;
            Task task;
            {
                std::unique_lock lock(_mtx);
                if(_isRunning == false && _tasks.empty())
                    return;

                _pop_cond.wait(lock,[&]{
                    return !_tasks.empty() || !_isRunning;
                });
                if(_tasks.empty())return;
                task = _tasks.front();
                _tasks.pop();
                _push_cond.notify_all();//通知生产者可以生产了


            }
            if(task.type == Task::READ)
            {
                _handler->handle_read(task.fd);
            }
            else
            {
                _handler->handle_write(task.fd);
            }
        }
        
    }


    void addTask(int fd,Task::Type type)
    {
        if(_isRunning == false)return;
        std::unique_lock<std::mutex> lock(_mtx);
        _push_cond.wait(lock,[&]{
            return _tasks.size() < _capacity;
        });//防止任务堆积过多

        _tasks.push({fd,type});
        _pop_cond.notify_all();
    }

    ~ThreadPool()
    {
        _isRunning = false;//关闭运行
        _push_cond.notify_all();//唤醒所有进程
        _pop_cond.notify_all();
        for(int i = 0;i<_pool.size();++i)
        {
            _pool[i].join();
        }
    }

    void setSelector(Selector*s)
    {
        _handler->selector = s;
    }

private:
    std::vector<std::thread> _pool;
    std::shared_ptr<EventHandler> _handler;
    int _capacity;
    std::queue<Task> _tasks;
    std::mutex _mtx;
    std::condition_variable _push_cond;
    std::condition_variable _pop_cond;
    std::atomic<bool> _isRunning;
};
}
```

## 主从Reactor多线程模型

1. 主线程职责
   + 唯一负责 accept() 新连接通过 epoll 监听 _listen_sock 的 EPOLLIN 事件收到新连接后通过**Round-Robin**或**负载均衡**分发给子线程

2. 子线程职责
   + 每个子线程有自己的 epoll 实例只处理已建立连接的读写事件不参与监听套接字的管理

### 重要改动 
1. `Selector`对象传递改成使用接口参数传递指针,使之更加灵活
2. `Acceptor`从`Disatcher`中分离出来，专门由主线程调用
3. 新增`subReactorLoop`子线程入口函数，专门处理客户端的IO请求

> Selector.hpp
```C++
#pragma once

#include <memory.h>
#include <memory>
#include "ThreadPool.hpp"
class ReactorServer
{
public:
    ReactorServer(uint16_t port,int sub_count = 5,std::shared_ptr<EventHandler>handler = std::make_shared<EchoHandler>())
    :_port(port),_selector(),_pool(handler),_sub_count(sub_count),_sub_selectors(sub_count)
    {
        _listen_sock = socket(AF_INET,SOCK_STREAM | SOCK_NONBLOCK,0);
    }

    void start()
    {
        //启动从Reactor线程
        for(int i = 0;i<_sub_count;++i)
        {
            printf("从Reactor: %d 启动\n",i);
            _sub_reactors.emplace_back(std::bind(&ReactorServer::subReactorLoop,this,i));
        }
        printf("从Reactor启动成功！\n");

        Bind();
        Listen();
        _selector.Add(_listen_sock);

        printf("开始监听连接\n");
        // int cnt = 0;
        for(;;)
        {
            // cnt++;
            // printf("loop: %d\n",cnt);
            std::vector<struct epoll_event>events;
            _selector.Wait(&events);//获取就绪事件
            if(!events.empty())
                Acceptor();
        }
    }

    ~ReactorServer()
    {
        close(_listen_sock);
        for(auto& t:_sub_reactors)
        {
            t.join();
        }
    }

    void subReactorLoop(int sub_num)
    {
        Selector& _sub_selector = _sub_selectors[sub_num];

        for(;;)
        {
            std::vector<struct epoll_event>events;
            {
                _sub_selector.Wait(&events);
                Dispatcher(events,&_sub_selector);
            }
        }
    }

protected:
    void Dispatcher(const std::vector<struct epoll_event>&events,Selector* selector)
    {
        // printf("get events: %lu\n",events.size());
        for(const auto& event:events)
        {
            //读写分离
            if(event.events & EPOLLIN)
            {
                //创建读事件任务
                _pool.addTask(event.data.fd,reactor::Task::READ,selector);
            }
            if(event.events & EPOLLOUT)
            {
                _pool.addTask(event.data.fd,reactor::Task::SEND,selector);
            }
        }
    }

    void Acceptor()
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int newfd = accept(_listen_sock,(struct sockaddr*)&peer,&len);
        if(newfd < 0)
        {
            printf("[Fatal]accept error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
        int pos = getSubPos(newfd);
        printf("[sub_selector:%d]",pos);
        _sub_selectors[pos].Add(newfd);
    }

    int getSubPos(int fd)
    {
        //简单整个哈希函数
        return fd%_sub_count;
    }

    void Bind()
    {
        int opt = 1;
        setsockopt(_listen_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
        struct sockaddr_in local;
        memset(&local,0,sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);
        local.sin_addr.s_addr = INADDR_ANY;//自动分配IP
        if(bind(_listen_sock,(const struct sockaddr*)&local,sizeof(local)) < 0)
        {
            printf("[Fatal]bind error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

    void Listen(int backlog = 10)
    {
        if(listen(_listen_sock,backlog) < 0)
        {
            printf("[Fatal]listen error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

protected: 
    int _listen_sock;
    uint16_t _port;
    Selector _selector;
    std::vector<std::thread> _sub_reactors;
    std::vector<Selector> _sub_selectors;
    int _sub_count;
    reactor::ThreadPool _pool;
};
```

> EventHalder.hpp
```C++
#pragma once

#include <unistd.h>
#include <fcntl.h>
#include <iostream>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unordered_map>
#include <string>
#include <sys/epoll.h>
#include <shared_mutex>
#include "Selector.hpp"
class EventHandler
{
public:
    virtual void handle_read(int fd,Selector* selector) = 0;
    virtual void handle_write(int fd,Selector* selector) = 0;
    virtual ~EventHandler() = default;

    std::unordered_map<int,std::string> _buffer_hash;
    std::unordered_map<int,std::shared_mutex> _fd_mutex;
};

class EchoHandler:public EventHandler
{
public:
    void handle_read(int fd,Selector* selector)override
    {
        std::unique_lock<std::shared_mutex> lock(getFdMutex(fd));//写锁
        char buffer[1024] = {0};
        int ret = read(fd,buffer,1024-1);
        if(ret == 0)
        {
            //连接关闭
            close(fd);
            selector->Del(fd);
        }
        else if (ret < 0)
        {
            //连接错误
            perror("read fd");
            close(fd);
            selector->Del(fd);
        }
        buffer[ret] = '\0';
        printf("Get a msg@ %s",buffer);
        //再写回
        ssize_t n = send(fd,buffer,sizeof(buffer),MSG_DONTWAIT);//非阻塞发送
        if(n == -1)
        {
            //如果发送失败
            if(errno == EAGAIN || errno == EWOULDBLOCK)
            {
                //发送缓冲区已满，注册 EPOLLOUT等待可写事件
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events= EPOLLIN | EPOLLOUT | EPOLLET;
                selector->Mod(fd,ev);//注册侦听写事件就绪
                std::string& buf_str = _buffer_hash[fd];
                buf_str.append(buffer);//写入缓冲区
            }
            else 
            {
                //无法处理则关闭连接
                close(fd);
                eraseFdMutex(fd);
                selector->Del(fd);
            }
        }
    }

    void handle_write(int fd,Selector* selector)override
    {
        std::unique_lock<std::shared_mutex> lock(getFdMutex(fd));//写锁
        auto&str = _buffer_hash[fd];
        ssize_t n = send(fd,str.c_str(),str.size(),MSG_DONTWAIT);//非阻塞发送
        if(n == -1)
        {
            //如果发送失败
            if(errno == EAGAIN || errno == EWOULDBLOCK)
            {
                //什么也不做，等待下次就绪
                //发送缓冲区已满，注册 EPOLLOUT等待可写事件
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events= EPOLLIN | EPOLLOUT | EPOLLET;
                selector->Mod(fd,ev);//注册侦听写事件就绪
                return;
            }
            else 
            {
                //无法处理则关闭连接
                close(fd);
                eraseFdMutex(fd);
                selector->Del(fd);
            }
        }

        if (n > 0) {
            str.erase(0, n); // 删除已发送部分
            if (str.empty()) {
                _buffer_hash.erase(fd);
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events= EPOLLIN | EPOLLET;
                selector->Mod(fd,ev);
                // 取消 EPOLLOUT 监听
            } else {
                // 保留未发送数据，等待下次写入
                struct epoll_event ev;
                ev.data.fd = fd;
                ev.events = EPOLLIN | EPOLLOUT | EPOLLET;
                selector->Mod(fd, ev);
            }
        }
        
    }

    std::shared_mutex& getFdMutex(int fd)
    {
        std::unique_lock<std::mutex> lock(_map_mutex);//使对锁表的查询互斥
        return _fd_mutex[fd];//若锁不存在，则会自动构造
    }

    void eraseFdMutex(int fd)
    {
        std::unique_lock<std::mutex> lock(_map_mutex);//使对锁表的查询互斥
        _fd_mutex.erase(fd);
    }
private:
    std::mutex _map_mutex;
};
```

> ThreadPool.hpp
```C++
#pragma once

#include <thread>
#include <condition_variable>
#include <queue>
#include <vector>
#include <functional>
#include <mutex>
#include <atomic>
#include "EventHandler.hpp"
namespace reactor
{
struct Task
{
    enum Type
    {
        READ,
        SEND
    };
    int fd;
    Type type;
    Selector* selector;
};

class ThreadPool
{
public:

    ThreadPool(std::shared_ptr<EventHandler> handler,int capacity=20,int n = 5)
    :_isRunning(true),_handler(handler),_capacity(capacity)
    {
        for(int i = 0;i<n;++i)
        {
            _pool.emplace_back(std::thread(std::bind(&ThreadPool::worker,this)));
        }
    }

    void worker()
    {
        for(;;)
        {
            if(_isRunning == false)return;
            Task task;
            {
                std::unique_lock lock(_mtx);
                if(_isRunning == false && _tasks.empty())
                    return;

                _pop_cond.wait(lock,[&]{
                    return !_tasks.empty() || !_isRunning;
                });
                if(_tasks.empty())return;
                task = _tasks.front();
                _tasks.pop();
                _push_cond.notify_all();//通知生产者可以生产了
            }
            if(task.type == Task::READ)
            {
                _handler->handle_read(task.fd,task.selector);
            }
            else
            {
                _handler->handle_write(task.fd,task.selector);
            }
        }
        
    }


    void addTask(int fd,Task::Type type,Selector* selector)
    {
        if(_isRunning == false)return;
        std::unique_lock<std::mutex> lock(_mtx);
        _push_cond.wait(lock,[&]{
            return _tasks.size() < _capacity;
        });//防止任务堆积过多

        _tasks.push({fd,type,selector});
        _pop_cond.notify_all();
    }

    ~ThreadPool()
    {
        _isRunning = false;//关闭运行
        _push_cond.notify_all();//唤醒所有进程
        _pop_cond.notify_all();
        for(int i = 0;i<_pool.size();++i)
        {
            _pool[i].join();
        }
    }

private:
    std::vector<std::thread> _pool;
    std::shared_ptr<EventHandler> _handler;
    int _capacity;
    std::queue<Task> _tasks;
    std::mutex _mtx;
    std::condition_variable _push_cond;
    std::condition_variable _pop_cond;
    std::atomic<bool> _isRunning;
};
}
```

> ReactorServer.hpp
```C++
#pragma once

#include <memory.h>
#include <memory>
#include "ThreadPool.hpp"
class ReactorServer
{
public:
    ReactorServer(uint16_t port,int sub_count = 5,std::shared_ptr<EventHandler>handler = std::make_shared<EchoHandler>())
    :_port(port),_selector(),_pool(handler),_sub_count(sub_count),_sub_selectors(sub_count)
    {
        _listen_sock = socket(AF_INET,SOCK_STREAM | SOCK_NONBLOCK,0);
    }

    void start()
    {
        //启动从Reactor线程
        for(int i = 0;i<_sub_count;++i)
        {
            printf("从Reactor: %d 启动\n",i);
            _sub_reactors.emplace_back(std::bind(&ReactorServer::subReactorLoop,this,i));
        }
        printf("从Reactor启动成功！\n");

        Bind();
        Listen();
        _selector.Add(_listen_sock);

        printf("开始监听连接\n");
        // int cnt = 0;
        for(;;)
        {
            // cnt++;
            // printf("loop: %d\n",cnt);
            std::vector<struct epoll_event>events;
            _selector.Wait(&events);//获取就绪事件
            if(!events.empty())
                Acceptor();
        }
    }

    ~ReactorServer()
    {
        close(_listen_sock);
        for(auto& t:_sub_reactors)
        {
            t.join();
        }
    }

    void subReactorLoop(int sub_num)
    {
        Selector& _sub_selector = _sub_selectors[sub_num];

        for(;;)
        {
            std::vector<struct epoll_event>events;
            {
                _sub_selector.Wait(&events);
                Dispatcher(events,&_sub_selector);
            }
        }
    }

protected:
    void Dispatcher(const std::vector<struct epoll_event>&events,Selector* selector)
    {
        // printf("get events: %lu\n",events.size());
        for(const auto& event:events)
        {
            //读写分离
            if(event.events & EPOLLIN)
            {
                //创建读事件任务
                _pool.addTask(event.data.fd,reactor::Task::READ,selector);
            }
            if(event.events & EPOLLOUT)
            {
                _pool.addTask(event.data.fd,reactor::Task::SEND,selector);
            }
        }
    }

    void Acceptor()
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int newfd = accept(_listen_sock,(struct sockaddr*)&peer,&len);
        if(newfd < 0)
        {
            printf("[Fatal]accept error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
        int pos = getSubPos(newfd);
        printf("[sub_selector:%d]",pos);
        _sub_selectors[pos].Add(newfd);
    }

    int getSubPos(int fd)
    {
        //简单整个哈希函数
        return fd%_sub_count;
    }

    void Bind()
    {
        int opt = 1;
        setsockopt(_listen_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
        struct sockaddr_in local;
        memset(&local,0,sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);
        local.sin_addr.s_addr = INADDR_ANY;//自动分配IP
        if(bind(_listen_sock,(const struct sockaddr*)&local,sizeof(local)) < 0)
        {
            printf("[Fatal]bind error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

    void Listen(int backlog = 10)
    {
        if(listen(_listen_sock,backlog) < 0)
        {
            printf("[Fatal]listen error,errno: %d error string:%s\n",errno,strerror(errno));
            exit(-1);
        }
    }

protected: 
    int _listen_sock;
    uint16_t _port;
    Selector _selector;
    std::vector<std::thread> _sub_reactors;
    std::vector<Selector> _sub_selectors;
    int _sub_count;
    reactor::ThreadPool _pool;
};
```