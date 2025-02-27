---
title: MySQL Connector/C++常见接口/类介绍
date: 2024-10-22 15:02:10
tags:
---
当我们有用C++程序连接`MySQL`数据库并执行`SQL`语句时，往往要用到相关的库，这里使用的是`MySQL`官方提供的`Connector/C++`库，为了更好地使用库里的内容，我们先来熟悉一下里面常用的接口和类

# MySQL准备
为了便于测试，我们专门创建一个用于测试的用户和数据库，并给予相关权限

```mysql
create user 'conn'@'localhost' identified by '12345678';

create database testDB;

grant all privileges on testDB.* to 'conn'@'localhost';
```
# 认识接口/类
我们将逐个介绍如下类及其相关接口

+ `sql::SQLString`
+ `sql::mysql::MySQL_Driver`    --*`<mysql_driver.h>`*
  + `sql::mysql::get_mysql_driver_instance()`
+ `sql::Connection`   --*`<mysql_connection.h>`*
+ `sql::Statement`    --*`<cppconn/statement.h>`*
+ `sql::ResultSet`    --*`<cppconn/resultset.h>`*
+ `sql::SQLException` --*`<cppconn/exception.h>`*

## sql::SQLString
从名字就能看出，这是`SQL`语言所用的字符串类。主要用于对`SQL`语句的字符串操作。我们来看看它的特点

### 构造函数的支持
```C++
//访问限定符是博客作者手动加的，与源码的位置和数量略有差异
public:
    SQLString() {}//默认构造函数
    //拷贝构造
    SQLString(const SQLString & other) : realStr(other.realStr) {}
    //std::string构造
    SQLString(const std::string & other) : realStr(other) {}
    //原生字符串构造
    SQLString(const char other[]) : realStr(other) {}
    //指定长度构造
    SQLString(const char * s, size_t n) : realStr(s, n) {}

private:
    std::string realStr;
```
可以看到`SQLString`的构造已经支持了常用的字符串结构,尤其是`std::string`和`C语言字符串`

接下来我们看看它的其它比较特别的接口

## 隐式类型转换重载
```C++
public:
    operator const std::string &() const
    {
      return realStr;
    }
```
这种类型的重载函数可能比较少见，但是它出现于`C++98`，且在后续版本没有改变，都是支持的。

它的效果是为自定义类型提供了自定义的隐式类型转换。有了上述代码的重载后，`sql::SQLString`就支持了以下写法

```C++
sql::SQLString sql_str("这原本是一条SQLString类型的字符串！");
std::String std_string = sql_str;//sql_str发生隐式类型转换，转换成const std::string类型,由`realStr`构造
```

### 字符串比较函数
首先是严格比较的函数
```C++
public:
    int compare(const SQLString& str) const
    {
      return realStr.compare(str.realStr);
    }

    int compare(const char * s) const
    {
      return realStr.compare(s);
    }

    int compare(size_t pos1, size_t n1, const char * s) const
    {
      return realStr.compare(pos1, n1, s);
    }
```
这三个比较函数会对两个字符串进行**严格的大小比较**,唯一的区别就是传参不同，**后续统一以`SQLString`为例介绍字符串相关的操作，但是要知道它也支持另外两种类型的字符串**

接下来介绍不严格比较（忽略大小写）
```C++
public:
    int caseCompare(const SQLString &s) const
    {
      std::string tmp(realStr), str(s);
      std::transform(tmp.begin(), tmp.end(), tmp.begin(), ::tolower);
      std::transform(str.begin(), str.end(), str.begin(), ::tolower);
      return tmp.compare(str);
    }
```
可以看到这个函数拷贝了两个字符串，并全都统一转换成了小写，这样就可以忽略大小写进行比较了

### 重载流插入
```C++
namespace std
{
  // operator << for SQLString output
  inline ostream & operator << (ostream & os, const sql::SQLString & str )
  {
    return os << str.asStdString();
  }
}
```
可以看到它直接支持流插入。

## sql::mysql::MySQL_Driver
来自头文件`<mysql_driver.h>`

这是一种`MySQL 驱动程序类`，它主要用于创建连接对象来`连接数据库`并返回`sql::Connection*`,在代码上就是实例化出了一个对象并返回它的指针。

至于获取driver实例，可以通过命名空间内提供的全局函数获取: `sql::mysql::get_mysql_driver_instance();`。这个函数会返回一个`sql::mysql::MySQL_Driver *`指针

示例如下

```C++
sql::mysql::MySQL_Driver *driver;//声明指针
driver = sql::mysql::get_mysql_driver_instance();//获取实例
```

主要用途:这个对象提供了一个`connect()`方法来创建连接对象并连接到数据库,参数要先后提供`ip地址`,`用户名`,`用户密码`

`sql::Connection * connect(const sql::SQLString& hostName, const sql::SQLString& userName, const sql::SQLString& password);`

创建连接
```C++
std::unique_ptr<sql::Connection> con(driver->connect("tcp://127.0.0.1:3306", "conn", "12345678"));
```

## sql::Connection
`sql::Connection`这个类用于表示与数据库的连接，提供了多种方法来执行 SQL 查询、管理事务、以及处理与数据库交互的各种操作。以下是关于 sql::Connection 的一些关键点

### 基本功能
+ **连接管理**: `sql::Connection` 负责管理与数据库的连接，包括建立和关闭连接。
+ **执行查询**: 通过该类，可以执行 `SQL语句`（如 `SELECT`、`INSERT`、`UPDATE` 和 `DELETE`）。
+ **事务控制**: 支持事务处理，可以**开始、提交和回滚事务**。

### 主要成员函数
+ `setSchema`: **设置数据库**，传入名称即可,无返回值
+ `reconnect`: 用于以相同的连接参数重连
+ `createStatement`: 创建一个用于执行 SQL 语句的 `sql::Statement` 对象。并返回一个`sql::Statement*`指针
+ `prepareStatement`: 创建一个用于执行**预处理** SQL 语句的 `sql::PreparedStatement` 对象。并返回一个`sql::PreparedStatement*`指针
+ `setAutoCommit`: 设置**是否自动提交事务**。传入一个`bool`类型的参数
+ `commit`: 提交当前事务的更改。
+ `setSavepoint`: 创建存档点。会产生一个`sql::Savepoint`对象并返回一个`sql::Savepoint*`指针
  + 有两种重载，可以不传参，也可以传字符串作为存档点的名称
+ `rollback`: 回滚当前事务的更改。
  + 有两种重载，可以不传参，也可以传入`sql::Savepoint*`指针,回滚到存档点
+ `close`: 关闭与数据库的连接。

这里只使演示一下`createStatement`获取`sql::Statement*`用于后文使用

```C++
std::unique_ptr<sql::Statement> stmt(con->createStatement());
```
## sql::Statement
**注：必须包含头文件`#include <cppconn/statement.h>`**

主要用于执行静态 `SQL` 语句,通过调用成员函数,并传入一个字符串

主要方法如下

+ `execute`: 返回值为`bool`类型，可以执行任意类型的 `SQL` 语句，如 `SELECT`、`INSERT`、`UPDATE` 和 `DELETE`。如果执行的是查询语句，则返回 `true`；如果执行的是更新语句，则返回 `false`。
+ `executeQuery`: 专门用于**执行查询语句**（如 `SELECT`），并返回一个 `sql::ResultSet` 对象的指针。
+ `executeUpdate`: 专门用于执行更新语句（如 `INSERT`、`UPDATE` 和 `DELETE`），并返回受影响的行数。返回值为`int`类型
+ `getConnection`: 返回创建这个 `sql::Statement` 的 `sql::Connection` 对象。
+ `close`: 关闭 `sql::Statement`，**释放其占用的资源**。

这里写一个获取查询结果的示例代码
```C++
std::unique_ptr<sql::Statement> stmt(con->createStatement());
std::unique_ptr<sql::ResultSet> res(stmt->executeQuery("SELECT VERSION()"));
```

## sql::ResultSet
它用于处理 SQL 查询返回的结果集，行为上可以认为它是个`单向迭代器`,一次只指向查询结果的**某一行**

主要接口有:

+ `next`: 
  + `bool next()`
  + 将游标从当前位置向前移动一行。如果新的当前行有效，则返回 `true`；如果没有更多的行，则返回 `false`。
+ `getString`: 
  + `SQLString getString(uint32_t columnIndex) const`  *每一列从`1`开始*
  + `SQLString getString(const sql::SQLString& columnLabel) const`
  + *下面的两种get方法均有两种重载*
  + 获取当前行指定列的 std::string 值。
+ `getInt`: 获取当前行指定列的 int 值。 *每一列从`1`开始*
+ `getDouble`: 获取当前行指定列的 double 值。 *每一列从`1`开始*
+ `getRow`
  + `size_t getRow() const`
  + 获取行号，从`1`开始
+ `getMetaData`: 
  + `ResultSetMetaData * getMetaData() const`
  + 返回一个 `sql::ResultSetMetaData` 对象指针，包含结果集的元数据信息。
+ `close`: 关闭 sql::ResultSet，释放其占用的资源

如果要遍历所有行，只需要像使用迭代器一样使用`sql::ResultSet`即可。示例如下:
```C++
while (res->next()) {
        //输出每一行的行号
        std::cout<<res->getRow()<<std::endl;
    }
```

这里再简单介绍一下`getMetaData`获得的`ResultSetMetaData`对象提供了哪些接口，可以获得哪些信息

**注意**⚠️,不能用智能指针储存`sql:ResultSetMetaData*`指针,因为它的析构函数是`portected`的，由别的对象自动释放，而智能指针没有权限自动释放。

它的源码及本文作者注释如下:
```C++
namespace sql
{

class ResultSetMetaData
{
public:
  enum
  {
    columnNoNulls,
    columnNullable,
    columnNullableUnknown
  };

  virtual SQLString getCatalogName(unsigned int column) = 0;

  virtual unsigned int getColumnCount() = 0;//获取列总数

  virtual unsigned int getColumnDisplaySize(unsigned int column) = 0;//获取指定列的最大字符跨度

  virtual SQLString getColumnLabel(unsigned int column) = 0;//获取列标签

  virtual SQLString getColumnName(unsigned int column) = 0;//获取列名称

  virtual int getColumnType(unsigned int column) = 0;//获取列类型

  virtual SQLString getColumnTypeName(unsigned int column) = 0;//获取列类型名称

  virtual SQLString getColumnCharset(unsigned int columnIndex) = 0;//获取列字符集

  virtual SQLString getColumnCollation(unsigned int columnIndex) = 0;//获取列字符集排列规则

  virtual unsigned int getPrecision(unsigned int column) = 0;//获取指定列的精度

  virtual unsigned int getScale(unsigned int column) = 0;//获取指定列的小树位数

  virtual SQLString getSchemaName(unsigned int column) = 0;//获取数据库名称

  virtual SQLString getTableName(unsigned int column) = 0;//获取表名称

  virtual bool isAutoIncrement(unsigned int column) = 0;//获取指定列是否有自动增加

  virtual bool isCaseSensitive(unsigned int column) = 0;//获取指定列是否大小写敏感

  virtual bool isCurrency(unsigned int column) = 0;//判断指定类是否为货币类型

  virtual bool isDefinitelyWritable(unsigned int column) = 0;//判断指定列在操作时是否一定会成功

  virtual int isNullable(unsigned int column) = 0;//是否可为空

  virtual bool isNumeric(unsigned int column) = 0;//判断是否为数值类型

  virtual bool isReadOnly(unsigned int column) = 0;//是否只读

  virtual bool isSearchable(unsigned int column) = 0;//是否可供查询，即在where中使用

  virtual bool isSigned(unsigned int column) = 0;//是否有符号

  virtual bool isWritable(unsigned int column) = 0;//是否可写入

  virtual bool isZerofill(unsigned int column) = 0;//是否有前导零填充

protected:
  virtual ~ResultSetMetaData() {}
};
}
```

## sql::SQLException
用于表示在执行 `SQL` 操作时发生的异常。这个类继承自标准 C++ 异常类`std::runtime_error`，因此它可以被 `try-catch` 块捕获并处理

### 常用方法

+ `getMessage`: 返回异常的详细描述信息。
+ `getSQLState`: 返回 `SQL` 标准的状态码（通常是一个五字符的字符串）。
+ `getErrorCode`: 返回数据库特有的错误代码。

下面是一个简单示例
```C++
#include <iostream>
#include <mysql_driver.h>
#include <mysql_connection.h>
#include <cppconn/statement.h>
#include <cppconn/resultset.h>
#include <cppconn/exception.h>

int main() {
    try {
        sql::mysql::MySQL_Driver* driver = sql::mysql::get_mysql_driver_instance();
        std::unique_ptr<sql::Connection> conn(driver->connect("tcp://127.0.0.1:3306", "user", "password"));
        
        std::unique_ptr<sql::Statement> stmt(conn->createStatement());
        std::unique_ptr<sql::ResultSet> res(stmt->executeQuery("SELECT * FROM non_existent_table"));
        
        while (res->next()) {
            // Process the result set
        }
    } catch (sql::SQLException& e) {
        std::cerr << "SQL Exception: " << e.what() << std::endl;
        std::cerr << "SQL State: " << e.getSQLState() << std::endl;
        std::cerr << "Error Code: " << e.getErrorCode() << std::endl;
    } catch (std::exception& e) {
        std::cerr << "Standard Exception: " << e.what() << std::endl;
    } catch (...) {
        std::cerr << "Unknown exception occurred!" << std::endl;
    }

    return 0;
}
```