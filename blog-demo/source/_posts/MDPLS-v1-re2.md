---
title: 【补丁更新v1.2~1.3】同步&异步⽇志系统的问题优化与异常处理
date: 2024-10-24 22:58:25
tags:
---

# 解决异步日志器黏包问题
由于原本的设计是一股脑直接把日志信息送进了缓冲区，导致输出的时候会产生黏包问题。但是为什么一开始没在意呢？因为标准输出和文件输出都不在意黏包问题，同时输出多条日志是没问题的。

但是！一旦要开始插入数据库，问题就很严重了。日志信息必须一行一行储存。原本的黏包问题会导致日志信息的解析不可解，所以我们需要修改原本的代码使其能够解决黏包问题

## 分隔符
用**特定的分隔符**标记一次日志的头尾是最容易想到的解决方案。

然而日志信息是字符串,**任何字符都有可能出现**,导致找不到特定的分隔符可以安全地分隔日志信息

分隔符只适用于待封装信息的字符在限定范围内时使用。

## 封装报头
*假如我们能获得一段日志的长度就好了*。

这样的愿望可以封装报头实现。在获取一长段数据时，我们规定最前面的是报头，包含第一段报文的信息(在这个项目里只简单的包括长度信息)。这样我们就能先读取信息再读取报文了。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410251626680.png)

那么怎么标定报头的范围呢？

+ 约定使用定长报头
+ 约定使用分隔符

定长报头很好理解，而这里又能用分隔符了是因为报头在一段信息的最前面，最先遇到的**特殊字符**必定是作为分隔符存在的

这里因为没太多信息需要封装进报头，所以我们采用定长报头。

## 代码实现
既然我们选择用定长报头表示储存报文长度，那么我们就得考虑空间利用效率的问题了。直接使用十进制确实挺好，但是密度似乎还是有些低。这里我们选择使用更高的进制来提高空间利用效率。又考虑到易用性和易读性的问题，我们选择了十六进制，因为`STL库`提供了方便的接口进行数制转换

为了提高代码复用性，我们把封装报头相关的函数封装进`suplog::util`里的`header`类域里

> util.hpp
```C++
//v1.2中额外添加
#include <sstream>
#include <iomanip>

namespace suplog{
    namespace util{
        class header
        {
        public:
            static const int HEADER_LEN = 8;
            static std::string addHeader(const std::string&raw_str)
            {
                //生成长度固定为8的十六进制报头
                int len = raw_str.size();
                std::ostringstream oss;
                oss<< std::hex << std::uppercase << std::setw(8) << std::setfill('0') << len;

                return oss.str()+raw_str;
            }

            static std::string delHeader(const std::string&pack_str,int* real_len)
            {
                std::string head = pack_str.substr(0,HEADER_LEN);
                int len = stoi(head,nullptr,16);
                if(real_len != nullptr)
                    *real_len = len;
                return pack_str.substr(8,len);
            }

            //读取报头中的长度
            static int readHeader(const char* str,size_t len)
            {
                if(len < 8) return -1;//越界访问

                std::string num_str(str,HEADER_LEN);
                for(auto ch:num_str)
                {
                    if(!(('0'<=ch && ch<='9') || ('A'<=ch && ch<='F')))
                        return -1;//非法字符
                }
                return stoi(num_str);
            }
        };
    }
}

```

准备好工具后，我们就要着手调整原本的业务流程了

原本的业务流程如下，搞清楚哪些业务流程要改动，有助于帮我定位需要修改的代码位置

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410251702608.png)

期望修改的新业务流程如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410251705691.png)

接下来我们修改一下代码

> AsyncLogger
```C++
private:
    virtual void logIt(const std::string &msg) override
    {
        //v1.2增加报头封装
        std::string header_str = suplog::util::header::addHeader(msg);
        _looper->push(header_str);//推送消息
        return;
    }

    //_looper所用的回调函数
    void readLog(Buffer& msg_line)
    {
        if(_sinks.empty()){ return; }//判空

        int cur = 0;
        while(cur < msg_line.readAbleSize())
        {
            //v1.2增加报头解析和字符串拆分
            int header_len = suplog::util::header::HEADER_LEN;
            int len = suplog::util::header::readHeader(msg_line.begin()+cur,header_len);//读取长度
            cur+=header_len;//跳过报头
            std::string msg(msg_line.begin()+cur,len);//提取报文
            cur+=len;//跳过报文

            for(auto &it:_sinks)
            {
                //调用落地功能
                it->log(msg.c_str(),msg.size());//直接一次性输出所有缓存的日志
            }
        }

        return;
    }
```

经过如上处理，黏包问题便顺利解决了。
# 添加异常处理

这里就简单处理了，我们直接继承一个`STL库`里的`runtime_error`类来封装成自己的异常类

> logexception.hpp
```C++
#include <stdexcept>
#include <string>

namespace suplog
{
    class LogException : public std::runtime_error
    {
    public:
        LogException(const std::string &msg = "") : runtime_error(msg) {}
    };
}
```
类的设计目前就这么简单，如果要添加功能，会在后续的更新中继续修改

接下来是逐个把`assert`替换掉并在合适的地方加入`try...catch`

## sink.hpp

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060924126.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060929336.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060931997.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060932714.webp)

## formatter.hpp

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060935578.webp)

## buffer.hpp

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060954477.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060955175.webp)

## logger.hpp

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411061000902.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411061902822.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411061902868.webp)
