---
title: 【功能更新v1.4】同步&异步⽇志系统Sink类主题更新,新增数据库落地，及重大bug修复
date: 2024-11-11 10:38:21
tags:
---
本次版本更新我们采用`Sink类主题更新`,实现数据库落地，按小时按天滚动输出,网络输出,以及标准输出按等级染色

# 重大bug修复
修改前的代码
```C++
void error(const char*file,size_t line,const char*fmt,...)
{
    if(shouldLog(LogLevel::Level::ERROR) == false)
        return;

    va_list al;
    va_start(al,fmt);//依据fmt从内存中提取可变参数列表
    log(LogLevel::Level::WARN,file,line,fmt,al);//日志输出
    va_end(al);//结束可变参数列表
}
```
可以看到里面的`ERROR`被错误地写成了`WARN`,导致输出`error`等级的日志时会错误输出`WARN`。因为这一bug涉及到项目的核心功能，所以判定为重大bug。**这一重大bug导致前面的版本全都作废**

# 数据库落地

这里我们使用部署在云服务器上的`MySQL`数据库服务来提供远程数据库存储服务

## 数据库准备
特别注意，这里是项目之外的配置，使用数据库落地功能前请自行配置数据库，并**将对应的连接信息改写进入配置文件**

我们来创建一个数据库和对应的`MySQL`用户进行相关的数据库操作

```SQL
create database MDPLS;

set global validate_password_policy = 'LOW'; 
create user 'logSink'@'%' identified by 'strongPassword';

grant ALL PRIVILEGES on MDPLS.* to 'logSink'@'%';
```

然后我们再准备一张表

这里为了保证能够排序数据库信息，所以设计上强制第一列为时间戳

```SQL
use MDPLS;

create table log(
    log_timestamp datetime not null,
    log_time varchar(24),
    log_level varchar(10),
    thread_id varchar(24),
    log_name varchar(24),
    file_name varchar(32),
    line_number varchar(24),
    log_msg text 
);
```

## 引入用户配置模块
在先前的博客中我们已经开发了迷你组件[MySQL登录用户管理和持久化组件](https://www.supdriver.top/2024/11/05/miniMod-MySQLUserConfig/)，接下来我们引入一下它, *这里直接把`.hpp`文件复制粘贴进项目了*

> DBUserConfig.hpp
```C++
#pragma once
#include <jsoncpp/json/json.h>
#include <string>
#include <fstream>
#include <sstream>

class DBUserConfig
{
public:
    DBUserConfig(const std::string &file = "./userConfig",
                 const std::string &user = "root",
                 const std::string &password = "",
                 const std::string &database = "",
                 const std::string &ip = "127.0.0.1",
                 size_t port = 3306)
        : _file(file), _user(user),_password(password), _database(database), _ip(ip), _port(port)
    {
    }

    //实现setter
    void setFile(const std::string &file) { _file = file; }
    void setUser(const std::string &user) { _user = user; }
    void setPassword(const std::string & password){ _password = password; }
    void setDatabase(const std::string &database) { _database = database; }
    void setIp(const std::string &ip) { _ip = ip; }
    void setPort(size_t port) { _port = port; }

    std::string Address()
    {
        std::string ret = _ip;
        ret += ":";
        ret += std::to_string(_port);
        return ret;
    }
    //实现getter
    std::string User() { return _user; }
    std::string Password() {return _password; }
    std::string DataBase() { return _database; }
    std::string Ip() { return _ip; }
    size_t Port() { return _port; }

    //实现基于文件的持久化
    void writeFile()
    {
        Json::Value root;

        root["user"] = _user;
        root["database"] = _database;
        root["ip"] = _ip;
        root["port"] = _port;
        root["password"] = _password;

        Json::StyledWriter w;
        std::ofstream ofs(_file,std::ios::out);

        ofs<< w.write(root);
        ofs.close();
    }

    bool readFile()
    {
        std::ifstream ifs(_file,std::ios::in);
        if(!ifs.is_open())return false;//打开文件
        std::string str;
        std::stringstream buf;
        buf<<ifs.rdbuf();
        str = buf.str();
        ifs.close();

        Json::Value root;
        Json::Reader reader;
        bool ret = reader.parse(str,root);
        if(!ret)
        {
            return false;//解析字符串失败
        }
        _user = root["user"].asString();
        _database = root["database"].asString();
        _ip = root["ip"].asString();
        _port = root["port"].asUInt();
        _password = root["password"].asString();
        return true;
    }


private:
    std::string _file;
    std::string _user;
    std::string _password;
    std::string _database;
    std::string _ip;
    size_t _port;
};
```
### 通过单例模式自动检测配置文件
我们希望在服务启动时能够自动读取配置文件，可是**万一配置文件不存在呢**?所以我们需要实现**自动**检测配置文件的功能。这里我们可以使用一个**全局的静态对象**的构造函数来自动执行相关的检测函数。这里我们正好可以使用`饿汉模式`来实现`单例模式`来实现这个功能

这里就不再多层封装了，我们选择直接改写原本的代码组件

*下面是发生变化的部分*
```C++
class DBUserConfig
{
public:
    using ptr = std::shared_ptr<DBUserConfig>;
private:
    DBUserConfig(const std::string &file = "./userConfig",
                 const std::string &user = "root",
                 const std::string &password = "",
                 const std::string &database = "",
                 const std::string &ip = "127.0.0.1",
                 size_t port = 3306)
        : _file(file), _user(user),_password(password), _database(database), _ip(ip), _port(port)
    {
        //试图读取配置文件
        bool isExist = suplog::util::file::exists(file);
        if(isExist == false)
        {
            writeFile();
            throw suplog::LogException("未检测到配置文件，现在已自动生成");
        }
        else 
        {
            readFile();//检测到配置文件，读取内容
        }
    }

    DBUserConfig(const DBUserConfig&) = delete;
    DBUserConfig operator=(const DBUserConfig&) = delete;

    static DBUserConfig* _singleIntance;
public:
    DBUserConfig* getInstance()//获取单例
    {
        if(_singleIntance == nullptr)
            throw suplog::LogException("DBUserConfig初始化失败");
        return _singleIntance;
    }

}
```

### 修改makefile
先别急着编译，因为我们将用到`jsoncpp`和`mysql connector/c++`两个第三方库，所以在编译选项中要引入第三方库

```makefile
mycmd:main.cpp
	g++ -o $@ $^ -std=c++11 -ljsoncpp -lmysqlcppconn

.PHONY:clean
clean:
	rm -rf mycmd testdir
```

### 生成userconfig配置文件
接下来我们就可以尝试让软件自动生成配置文件了。我们可以让`main.cpp`直接包含头文件，也可以让`sink.hpp`包含头文件，而`main.cpp`间接包含。总之在引入头文件后我们执行编译命令，然后运行程序

```SHELL
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ make
g++ -o mycmd main.cpp -std=c++11 -ljsoncpp -lmysqlcppconn
supdriver@ALi-cloud-Linux-2-2G:~/codes/Asynchrinous-Logging-System$ ./mycmd 
terminate called after throwing an instance of 'suplog::LogException'
  what():  未检测到配置文件，现在已自动生成
Aborted (core dumped)
```

执行了以上操作后我们会发现程序中断了，并且项目文件夹里生成了一个`userConfig`配置文件

```json
{
   "database" : "",
   "ip" : "127.0.0.1",
   "password" : "",
   "port" : 3306,
   "user" : "root"
}
```
我们来把它改成该项目要用到的配置

```json
{
   "database" : "MDPLS",
   "ip" : "47.99.48.121",
   "password" : "strongPassword",
   "port" : 3306,
   "user" : "logSink"
}
```

这样我们就成功引入了用户配置模块

## 实现DBFormatter类
从名字上也能看出，`DBFormatter`类继承自`Formatter`，以达到对原项目作出的改动最小的情况下，增加支持数据库落地的功能。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411121645067.webp)

不同于`Formatter`需要使用格式串来生成`日志文本`，`DBFormatter`重写了`format`函数，将原本格式串的功能改成指定`SQL`语句中`valuse()`括号内的内容及顺序，，正常情况下产生的`SQL语句`会交给`DatabaseSink`类执行落地，但如果错误地交给其它落地方向，也能正常输出。~~尽管观感很糟糕~~

但是`DatabaseSink`类明显不支持`Formatter`产生的日志文本，所以**需要做合法性检查**。这个功能我们交给`DatabaseSink`类与数据库的交互来实现。接下来我们从代码上实现`DBFormatter`

```C++
class DBFormatter:public Formatter
{
public:
    using ptr = std::shared_ptr<DBFormatter>;
    //需要传入values后面括号内的内容，与数据库的表相对应
    DBFormatter(const std::string&tableName = "",const std::string& pattern = "%p%t%c%f%l%m")
    :_tableName(tableName)
    ,Formatter(pattern)
    {
        if(tableName.empty())
        {
            throw LogException("DBFormatter: 表名不能为空");
        }
    }

    virtual std::string format(const LogMsg& msg) override
    {
        std::stringstream ss;//创建string流

        ss<<"insert into "<<_tableName<<" values(";
        //默认插入时间
        ss<<"FROM_UNIXTIME(";
        ss<<std::to_string(msg._ctime);
        ss<<")";
        //按格式串生成日志信息,全都用单引号包裹
        for(auto& it: _items)
        {
            //添加逗号间隔和单引号包裹
            ss<<",\'";
            it->format(ss,msg);
            ss<<"\'";
        }
        ss<<");";
        return ss.str();
    }

private:
    std::string _tableName;
};
```

## 实现DatabaseSink类

我们简单地设计一下`DatabaseSink`类

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411201603898.webp)

因为涉及到网络通信，所以我们采用异步通信的方式向数据库插入数据。至于具体的实现方式则是采用`多线程`和`消息队列`。

同时为了防止消息队列过长造成**内存溢出**,我们使用生产者消费者模型，通过`条件变量`来控制生产者向消息队列存放消息的数量。

因此原本的`log`接口就担任生产者的身份，加锁后首先判断消息队列的大小，如果大于等于最大值，就使用条件变量开始等待消费者发送唤醒信号。当完成一次插入操作时就向消费者发送唤醒信号让它取出数据。

同时`子线程`的入口函数就作为消费者，在收到信号后就从消息队列取出消息，然后解锁，开始异步执行数据库交互行为,而这一部分正是时间消耗的大头。

```C++
    class DatabaseSink:public LogSink
    {
        static const int QUEUE_MAX_SIZE = 1024;//默认设置消息队列的最大长度
    public:
        using ptr = std::shared_ptr<DatabaseSink>;
        DatabaseSink()
        :_running(true)
        ,_t(&DatabaseSink::consumer,this)
        {}

        //生产者
        void log(const char*data,size_t len)override
        {
            std::unique_lock<std::mutex>lock(_mutex);//加锁
            if(_msgQueue.size() >= QUEUE_MAX_SIZE)
            {
                _push_con.wait(lock,[&]{
                    return _msgQueue.size()<DatabaseSink::QUEUE_MAX_SIZE;
                });
            }
            //DEBUG
            std::cout<<"插入数据"<<std::string(data,len)<<std::endl;
            _msgQueue.push(std::string(data,len));
            lock.unlock();
            _pop_con.notify_all();
        }

        static void consumer(void* arg)
        {
            DatabaseSink* ds = (DatabaseSink*)arg;

            DBUserConfig::ptr user(DBUserConfig::getInstance());

            sql::mysql::MySQL_Driver* driver = sql::mysql::get_driver_instance();
            sql::Connection* conn = driver->connect(user->Address(),user->User(),user->Password());
            conn->setSchema(user->DataBase());
            sql::Statement* stm = conn->createStatement();//创建statement用于执行SQL语句

            while(true)
            {
                std::string sql_str;
                {
                    std::unique_lock<std::mutex> lock(ds->_mutex);
                    //这里顺序不能乱， 必须先等待，然后做判断
                    if(ds->_running)
                        ds->_pop_con.wait(lock,[&]{
                            return !ds->_msgQueue.empty() || !ds->_running;
                        });
                    if(ds->_running == false && ds->_msgQueue.empty())
                    {
                        // std::cout<<"子线程退出\n";
                        exit(0);//线程退出
                    }
                    sql_str = ds->_msgQueue.front();
                    ds->_msgQueue.pop();
                    ds->_push_con.notify_all();
                    //其次，这里尽量不要对lock进行unlock，用花括号先定一个作用域就行
                }
                //执行mysql语句
                stm->execute(sql_str);
            }
        }

        ~DatabaseSink()
        {
            //关闭连接
            _running = false;
            _pop_con.notify_all();
            _push_con.notify_all();
            _t.join();
        }

    private:
        std::mutex _mutex;
        std::condition_variable _push_con;
        std::condition_variable _pop_con;
        std::atomic<bool> _running;
        std::queue<std::string> _msgQueue;
        std::thread _t;
    };
```

> 各种各样的BUG
```markdown
这个DatabaseSink类最的最初版各种出bug抛异常,这大概目前本次项目最大范围的bug了,导致作者进行了一些列DEBUG测试，特此记录

# DBFormatter 功能测试
因为这一个类强依赖于`DBFormatter`产生的`SQL`语句，所以验证其正确性很重要。这里我选择:
1. 单独生成一个`DBFormatter`实例 
2. 让它生成`SQL`语句 
3. 然后打印到标准输出
4. 复制到mysql客户端中执行
5. 查看mysql返回情况

实际上初版的`DBFormatter`没有在`SQL`语句中添加`FROM_UNIXTIME`,导致数据格式错误，数据无法插入数据库。

但DEBUG之后的版本`DBFormatter`添加了函数`FROM_UNIXTIME`,也成功在mysql客户端中执行了生成的`SQL`，所以`DBFormatter`的bug排除了

但是程序依然会抛异常，所以我们要继续DEBUG

# 非代码因素排除
我们优先排除非代码因素导致的bug。比如这里就是数据库配置信息的错误要排除。

为此我单独写了个程序使用相同的数据库信息向数据库的同一张表插入数据。同时这里使用的`SQL`语句就是上面的`DBFormatter`产生的，因此语句本身的正确性可以得到保证

这一部分的程序正常运行并且能插入数据到数据库，所以这部分的问题排除

DEBUG继续到下一部分

# 每一步都添加了标准输出标示
由于多线程程序进一步增加了gdb调试的难度，所以我们优先使用标准错误输出打印每一步的操作来指示程序执行到哪就崩了。

## 1.没有意义的全局变量
在最初的版本中，数据库相关的`sql::mysql::MySQL_Driver`和`sql::Connection`都储存在了成员函数中，在类域中可以认为是全局变量。然而与数据库的交互只有子线程需要，完全可以作为子线程函数的局部变量。

所以无论是否这么做会引起bug,*我也没测出来它是不是贡献了部分bug*，应该把它放在子线程函数中。

最后我把它放在了循环体内，离执行`SQL`语句最近的地方,~~我后面会付出代价~~

## 2.对`std::unique_lock<std::mutex>lock(_mutex);`的错误使用
众所周知这是一把只存在于`作用域范围内的互斥锁`,出了作用域就会自动调用析构函数解锁。然而它也提供了`unlock`成员函数。也许需要配合`lock`使用吧，然而我只在从消息队列取出数据后直接使用了`unlock`,而前面只有`lock`的声明，并没有使用`lock`。总之这一系列糟糕的操作一开始没有导致崩溃。然而它影响到了后面的`MySQL`相关的代码。在我把`conn->setSchema(user->DataBase());`改成了执行SQL语句`use MDPLS`后，程序便会在执行插入数据的`SQL`时抛异常。导致一次`SQL`都无法执行

最后把`unlock`那句删了，额外给`lock`所需工作的代码段包了层大括号指明作用域，让`lock`能在作用域内自动上锁解锁,之后便神奇地可以执行一句`SQL`插入操作了。

但**只能执行一句两句**,然后就会报段地址错误

## 3.循环内浪费资源
显然连接`MySQL`需要消耗`网络资源`、`文件资源`和`内存资源`。而这个库的`sql::Connection`和`sql::Statement`是可以复用的，根本没必要每次循环都创建新的实例浪费资源。在相关的代码移动至循环体外面后，不仅仅大大减少了资源浪费，而且程序终于支持执行多条`SQL`语句，不再报错了。

## 4.压力测试
注意我们还没有删除之前DEBUG用的标准错误输出语句

所以我们在执行压力测试时，能够看到它正在将哪些数据送入消息队列，正在取出哪些数据，以及执行完了哪条`SQL`

执行的压力测试代码如下

> int main()
> {
>    suplog::GlobalLoggerBuilder::ptr glb(new suplog::GlobalLoggerBuilder);
>    glb->buildFormatter(std::make_shared<suplog::DBFormatter>("log"));
>    glb->buildLoggerName("DBLogger");
>    glb->buildLoggerType(suplog::Logger::Type::LOGGER_SYNC);
>    glb->buildSink<suplog::DatabaseSink>();
>    suplog::Logger::ptr dbLogger= glb->build();
>    dbLogger->debug("测试Debug日志");
>
>    for(int i = 0;i<100000;++i)
>    {
>        std::string msg = "测试Debug日志"+ std::to_string(i);
>        std::cout<<"即将输出"<<msg<<endl;
>        dbLogger->debug(msg.c_str());
>    }
>
>    cout<<"主程序退出"<<endl;
>    return 0;
> }

最后可以从终端看到消息队列的最大值保持了`1024`，没有发生消息队列过大导致内存移除，同时确实在异步执行`SQL`语句，以及这一落地方向也支持了10万条日志信息连续输出

# 删除DEBUG用的标准错误输出
完成DEBUG后，我们把多余的，有性能消耗的标准错误输出都删除了。
```

## 功能总结
我们提供了`userConfig`文件用于配置数据库相关的配置信息,当`DatabaseSink`配合`DBFormatter`时，日志器便能够正确执行数据库方向的落地。

# 彩色输出

## 输出方法
在C++中，输出彩色字体可以通过控制台的 `ANSI` 转义序列来实现。`ANSI` 转义序列是一种特殊的控制字符，可以用来改变终端的文本颜色和样式

格式如下
```C++
"\033[<code>m"
```
+ `\033`是转义字符
+ `<code>`代指颜色代码,用于指定该语句后面的所有字符按什么颜色输出

常见颜色代码如下

| 代码 | 颜色 |
| --- | --- |
| 30 | 黑色 |
| 31 | 红色 |
| 32 | 绿色 |
| 33 | 黄色 | 
| 34 | 蓝色 |
| 35 | 品红色 |
| 36 | 青色 |
| 37 | 白色 |
| 0 | 重置颜色 |

## 示例
举一个输出红色`DEBUG`日志信息的例子
```C++
std::cout << "[" << "\033[31m" << "DEBUG" << "\033[0m" << "]这是一段日志" << std::endl;
```
在上面的输出中，只有`DEBUG`是**红色**的，剩下的字体颜色则还是默认颜色

## 封装ColorReplace工具类
```C++
//标准输出落地
class StdoutSink:public LogSink
{
public:
    using ptr = std::shared_ptr<StdoutSink>;
    StdoutSink()=default;
    
    void log(const char*data,size_t len) override
    {
        std::cout.write(data,len);
        return;//标记函数结尾
    }
};
```
我们观察标准输出落地的接口，发现我们只能采取通过**替换模式串**的方式来给模式串上色，同时为了提高代码的封装性和可复用性，我们选择把**指定模式串替换为指定颜色**的功能抽象出来，封装在一个工具类的静态成员函数中

我们依然把它放在`util.hpp`的`suplog::util`命名空间内

```C++
namespace suplog{
    namespace util{
        class ColorReplace
        {
        public:
            enum Color
            {
                Black,
                Red,
                Green,
                Yellow,
                Blue,
                Magenta, // 品红色
                Cyan,    // 青色
                White,
                None // 无颜色
            };

            static std::string colorReplace(const std::string &str, const std::string &pattern, Color color)
            {
                size_t prev = 0;
                size_t pos = 0;
                std::string ret;
                while (pos < str.size() && pos != std::string::npos)
                {
                    //依次遍历
                    prev = pos;
                    pos = str.find(pattern, pos);
                    ret += str.substr(prev, pos - prev);//逐段拼接
                    if (pos != std::string::npos)
                    {
                        ret += colorToStr(color);
                        ret += pattern;
                        ret += colorToStr(Color::None);
                        pos += pattern.size();
                    }
                    else
                    {
                        break;
                    }
                }

                return ret;
            }
            
            //颜色枚举转成字符串
            static inline std::string colorToStr(Color color)
            {
                switch (color)
                {
                case Black:
                    return "\033[30m";
                    break;
                case Red:
                    return "\033[31m";
                    break;
                case Green:
                    return "\033[32m";
                    break;
                case Yellow:
                    return "\033[33m";
                    break;
                case Blue:
                    return "\033[34m";
                    break;
                case Magenta:
                    return "\033[35m";
                    break;
                case Cyan:
                    return "\033[36m";
                    break;
                case White:
                    return "\033[37m";
                    break;
                case Color::None:
                    return "\033[0m";
                    break;

                default:
                    return "\033[0m";
                }
            }
        };
    }
}
```
不过出于函数接口要尽量简单的需求，这个工具函数一次只能替换一种格式串，所以我们还要在`StdoutSink`类内定制一个`模式串-颜色`映射表,不过这里就简化处理了,直接封装一个成员函数，每种等级都替换一遍，然后返回处理后的字符串

修改后的`StdoutSink`的代码如下
```C++
    //标准输出落地
    class StdoutSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<StdoutSink>;
        StdoutSink()=default;

        void log(const char*data,size_t len) override
        {
            std::cout<<replaceAllColor(data,len);
            return;//标记函数结尾
        }

        static inline std::string replaceAllColor(const char*data,size_t len)
        {
            std::string str(data,len);
            str = util::ColorReplace::colorReplace(str,"DEBUG",util::ColorReplace::Color::Green);
            str = util::ColorReplace::colorReplace(str,"INFO",util::ColorReplace::Color::Blue);
            str = util::ColorReplace::colorReplace(str,"WARN",util::ColorReplace::Color::Yellow);
            str = util::ColorReplace::colorReplace(str,"ERROR",util::ColorReplace::Color::Magenta);
            str = util::ColorReplace::colorReplace(str,"FATAL",util::ColorReplace::Color::Red);

            return str;
        }
    };
```

然后我们简单地测试一下功能

```C++
//彩色字体输出测试
int main()
{
    auto root = suplog::rootLogger();
    //根日志器默认为标准输出落地
    root->debug("测试");
    root->info("测试");
    root->warn("测试");
    root->error("测试");
    root->fatal("测试");
    return 0;
}
```
输出如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411202045283.webp)

可以看到它很好地完成了功能

# RollSink重写
因为我们准备实现`按天`,`按小时`**滚动文件输出**，所以为了提高代码的复用性和体现类的层次结构，我们把**滚动文件输出**抽象到共同抽象父类`RollSink`中，而原本的按文件大小的类我们把它改名为`RollSizeSink`并作为`RollSink`的派生类。
因此我们设计的类变成了如下图:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411210837586.webp)

然后我们把它们转化成代码

```C++
    class RollSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<RollSink>;

        RollSink(const std::string&basename)
        :_basename(basename)
        {
            //创建目录
            util::file::create_directory(util::file::path(_basename));
        }

        virtual void log(const char* data,size_t len) = 0;
    
    protected:
        virtual void initLogFile() = 0;

        virtual std::string createFilename() = 0;

    protected:
        std::string _basename;//基础文件名
        std::ofstream _ofs;//输出用的文件输出流
    };

    class RollHourSink:public RollSink
    {
    public:
        RollHourSink(const std::string&basename)
        :RollSink(basename){}

    public:
        void log(const char*data,size_t len)override 
        {
            initLogFile();//初始化输出文件和文件输出流
            _ofs.write(data,len);//写入数据
            if(_ofs.good() == false)
            {
                throw LogException("RollSizeSink: 日志文件输出失败！");
            }
        }

    private:
        void initLogFile()override
        {
            std::string name = createFilename();
            if(_ofs.is_open() == false || util::file::exists(name))
            {
                _ofs.close();//关闭原有文件流
                _ofs.open(name,std::ios::binary | std::ios::app);
                if(_ofs.is_open() == false)
                {
                    throw LogException("RollHourSink: init LogFile failed");
                }
            }
            return;
        }

        std::string createFilename()override
        {
            time_t t = time(NULL);
            struct tm lt;
            localtime_r(&t,&lt);
            std::stringstream ss;
            //命名只精确到小时
            ss<<_basename;
            ss<< lt.tm_year + 1900;
            ss <<lt.tm_mon + 1;
            ss << lt.tm_mday;
            ss << lt.tm_hour;
            ss << ".log";
            return ss.str();
        }
        
    };

    class RollDaySink:public RollSink
    {
    public:
        RollDaySink(const std::string&basename)
        :RollSink(basename){}

    public:
        void log(const char*data,size_t len)override 
        {
            initLogFile();//初始化输出文件和文件输出流
            _ofs.write(data,len);//写入数据
            if(_ofs.good() == false)
            {
                throw LogException("RollSizeSink: 日志文件输出失败！");
            }
        }

    private:
        void initLogFile()override
        {
            std::string name = createFilename();
            if(_ofs.is_open() == false || util::file::exists(name))
            {
                _ofs.close();//关闭原有文件流
                _ofs.open(name,std::ios::binary | std::ios::app);
                if(_ofs.is_open() == false)
                {
                    throw LogException("RollDaySink: init LogFile failed");
                }
            }
            return;
        }

        std::string createFilename()override
        {
            time_t t = time(NULL);
            struct tm lt;
            localtime_r(&t,&lt);
            std::stringstream ss;
            //命名只精确到天
            ss<<_basename;
            ss<< lt.tm_year + 1900;
            ss <<lt.tm_mon + 1;
            ss << lt.tm_mday;
            ss << ".log";
            return ss.str();
        }
        
    };

    //按文件大小滚动文件落地类
    class RollSizeSink:public RollSink
    {
    public:
        using ptr = std::shared_ptr<RollSizeSink>;
        //实际文件名 = bsename + 可变部分
        RollSizeSink(const std::string& basename,size_t max_size)
        :RollSink(basename),_max_fsize(max_size),_cur_fsize(0)
        {}

        void log(const char*data,size_t len) override 
        {
            initLogFile();//初始化输出文件和文件输出流
            _ofs.write(data,len);//写入数据
            if(_ofs.good() == false)
            {
                throw LogException("RollSizeSink: 日志文件输出失败！");
            }
            _cur_fsize += len;//更新文件大小
            return;
        }

    private:
        void initLogFile() override
        {
            //如果满足条件，触发创建新文件的条件
            if(_ofs.is_open() == false || _cur_fsize >=_max_fsize)
            {
                _ofs.close();//先关闭原有文件流
                if(_cur_fsize >=_max_fsize) sleep(1);//防止同一秒有过多日志消息要打印，导致打开同一个日志文件
                std::string name = createFilename();
                _ofs.open(name,std::ios::binary | std::ios::app);
                if(_ofs.is_open() == false)
                {
                    throw LogException("RollSizeSink: init LogFile failed");
                }
                _cur_fsize = 0;//创建新文件后，重置文件大小
                return;
            }
            return;
        }
        
        //封装产生文件名的函数
        std::string createFilename()override
        {
            //这里采用的格式为
            // basename + 年月日 + .log
            time_t t = time(NULL);
            struct tm lt;
            localtime_r(&t,&lt);
            std::stringstream ss;
            //获取详细时间信息
            ss << _basename;
            ss<< lt.tm_year + 1900;
            ss <<lt.tm_mon + 1;
            ss << lt.tm_mday;
            ss << lt.tm_hour;
            ss << lt.tm_min;
            ss << lt.tm_sec;
            ss << ".log";
            return ss.str();
        }
    private:
        size_t _max_fsize;//文件最大容量
        size_t _cur_fsize;//当前输出文件的使用量
    };
```