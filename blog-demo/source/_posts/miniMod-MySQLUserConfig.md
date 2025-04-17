---
title: 【迷你组件】MySQL登录用户管理和持久化组件
date: 2024-11-05 17:40:53
tags:
---
当要在代码中连接数据库时，往往需要登录用户的信息，而实际使用时登录用户常常变化，硬编码到代码中，改起来十分麻烦。所以使用`Json`技术将我们封装好的用户管理类`序列化`并储存到文件中，既能够`持久化`，还实现了`软编码`，使用户仅需方便地修改文件内容就能修改用户信息了。

# 用户配置类设计

## 第三方库
这里用到了两个第三方库

+ `-ljsoncpp` Json的第三方库
+ `-lmysqlcppconn` 数据库的第三方库

编译指令如下 

```shell
g++ -o $@ $^ -std=c++11 -ljsoncpp -lmysqlcppconn
```

## 代码实现
作为`MySQL`用户配置，至少要支持储存如下信息

+ 用户名
+ 密码
+ 数据库
+ 登录ip
+ 登录端口号

然后核心功能如下

+ 成员变量的访问和设置
+ 将配置写入文件
+ 从文件读入配置

根据如上要求，我们设计的核心代码如下

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

    void writeFile();//将配置写入文件

    bool readFile();//将文件读入内存

private:
    std::string _file;//选择配置文件的存储位置
    std::string _user;
    std::string _password;
    std::string _database;
    std::string _ip;
    size_t _port;
};
```

关于成员变量的设置和访问的代码如下

```C++
    //实现setter
    void setFile(const std::string &file) { _file = file; }
    void setUser(const std::string &user) { _user = user; }
    void setPassword(const std::string & password){ _password = password; }
    void setDatabase(const std::string &database) { _database = database; }
    void setIp(const std::string &ip) { _ip = ip; }
    void setPort(size_t port) { _port = port; }

        //实现getter
    std::string User() { return _user; }
    std::string Password() {return _password; }
    std::string DataBase() { return _database; }
    std::string Ip() { return _ip; }
    size_t Port() { return _port; }
```

除此之外，我们再实现额外的三个函数

+ `Address`返回地址(ip+端口号),函数内自动实现字符串拼接
+ `writeFile`实现打开配置文件并写入
+ `readFile`实现读取配置文件并获取配置

```C++
    std::string Address()
    {
        std::string ret = _ip;
        ret += ":";
        ret += std::to_string(_port);
        return ret;
    }

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

```

特别的，在上面的代码中，我们用到了`stringstream`和`jsoncpp`来实现相关的操作

`jsoncpp`的`Json::Value`类可以用来描述一个类(更多的是类似于结构体),通过`key-value`的方式把成员变量按需存储和读取，完成`DBUserConfig`类的序列化和反序列化。

`stringstream`则是使用了一个小技巧来直接读取文件的全部内容。当然这主要用于配置文件这种比较小的文件，对于非常大的文件，可能需要逐行或按块读取文件，以避免内存问题。

> 一点读取文件的小技巧
```C++
    std::ifstream ifs(_file,std::ios::in);
    if(!ifs.is_open())return false;//打开文件
    std::string str;
    std::stringstream buf;//创建stringstream
    buf<<ifs.rdbuf();//直接读取文件缓冲区的全部内容到stringstream内
    str = buf.str();//转换成字符串
    ifs.close();
```

整个文件的全部代码如下

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

# 代码测试和连接测试
我们来写个`main.cpp`和`makefile`

> main.cpp
```C++
#include "DBUserConfig.hpp"
#include <iostream>
#include <memory>
#include <mysql_driver.h>
#include <mysql_connection.h>
#include <cppconn/statement.h>
#include <cppconn/resultset.h>

using namespace std;

int main()
{
    DBUserConfig dbu;//创建用户配置类
    bool ret = dbu.readFile();//试图读取配置文件
    if(!ret)
    {
        cout<<"start write\n";
        dbu.setFile("./userConfig");
        dbu.setIp("127.0.0.1");
        dbu.setPort(3306);
        dbu.setUser("conn");
        dbu.setDatabase("testDB");
        dbu.setPassword("12345678");
    }
    
    sql::mysql::MySQL_Driver* driver;//创建mysql驱动程序
    driver = sql::mysql::get_mysql_driver_instance();//获取实例

    //连接数据库
    std::unique_ptr<sql::Connection> conn(driver->connect(dbu.Address(),dbu.User(),dbu.Password()));

    cout<<conn->getClientInfo()<<endl;//输出连接状态

    dbu.writeFile();//将配置写入文件
    return 0;
}
```

> makefile
```makefile
mycmd:main.cpp
	g++ -o $@ $^ -std=c++11 -ljsoncpp -lmysqlcppconn

.PHONY:clean
clean:
	rm -f mycmd
```

接下来我们来编译执行并测试代码

```shell
make

./mycmd
```

*结果如下*

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060841440.webp)

可以看到，当读不到配置文件时，自动写入了用户配置信息，并成功连接上了数据库

紧接着我们再运行一次

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202411060845137.webp)

可以看到直接读取了配置文件并成功连接上了数据库

# End
以上就是一个用户配置类的小组件开发。

