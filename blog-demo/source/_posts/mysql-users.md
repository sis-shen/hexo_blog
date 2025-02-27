---
title: MySQL用户管理
date: 2024-10-22 08:07:29
tags:
---
# 为什么有用户管理 
类比`Linux`中只有`root`用户过于危险，因为root能随意地增删查改计算机上的任意文件

`MySQL`只有`root`用户登录也是十分危险的，因为`root`有增删查改**所有数据库和表**的权限，数据安全得不到保障，所以`MySQL`有自己的用户管理系统

# 用户管理的使用

## 用户信息
显然用户信息是需要组织和管理的，所以按照`先描述，再组织`的原则，我们来思考一下用户信息在逻辑上是什么样的管理思路

### 描述用户
只需略微思考一下在`MySQL`中描述一个对象怎么样最方便准确，很明显，就是它自带的`表结构`。

实际上也确实是这样，`MySQL`用了一个在`mysql`数据库中的`user`表用来描述所有的用户

```mysql client
mysql> use mysql
Database changed
mysql> select host,user,authentication_string from user;
+-----------+------------------+------------------------------------------------------------------------+
| host      | user             | authentication_string                                                  |
+-----------+------------------+------------------------------------------------------------------------+
| %         | stocer           | $A$005$KKu{+%S+?WLyONB^/UZBeHZsE.i2wvJl.wpcoThot.3TydQQeUXmS1fiWoAV4   |
| localhost | debian-sys-maint | $A$005$lTTz`zNMG&^%%^=Okj/cVfbYBZw3XgJ1Fl3CxH/MRwfNhOa/CE6HHJaag1      |
| localhost | mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | root             | *B8B414B84C3D3937BE77A1BA84B2DF8DA83AA442                              |
+-----------+------------------+------------------------------------------------------------------------+
7 rows in set (0.00 sec)

mysql> desc user;
+--------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Field                    | Type                              | Null | Key | Default               | Extra |
+--------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Host                     | char(255)                         | NO   | PRI |                       |       |
| User                     | char(32)                          | NO   | PRI |                       |       |
| Select_priv              | enum('N','Y')                     | NO   |     | N                     |       |
......
......
| Password_reuse_time      | smallint unsigned                 | YES  |     | NULL                  |       |
| Password_require_current | enum('N','Y')                     | YES  |     | NULL                  |       |
| User_attributes          | json                              | YES  |     | NULL                  |       |
+--------------------------+-----------------------------------+------+-----+-----------------------+-------+
51 rows in set (0.00 sec)
```

通过上述指令，可以看到有51个`column`来描述一个具体用户。其中`host`、`user`、`authentication_string`是最常查询的。（大小写不区分）

+ `host`：表示数据库能够从哪个ip登录，`localhost`只能从本机登录，`%`则是任意主机
+ `user`: 用户名
+ `authentication_string`: 用户密码**单向加密**后形成的字符串，用于登录时匹配

### 组织用户
既然我们能在表中描述用户了，那么该怎么组织维护用户系统呢？

很明显，得益于表结构，对用户的`增删查改`操作转化为了对`user表`的`增删查改`操作。

不过我们也看到了，一个用户表有多达`51`个属性，`删查改`似乎还能做到，那用`insert into`增加条目时难道得一个一个输进去吗？很明显这太麻烦了

所幸`MySQL`人性化地提供了`MySQL指令`用于方便的维护用户表，当然底层还是有`MySQL`自动地对用户表进行增删查改

## 创建用户
语法格式如下（**使用时替换中文，保留引号**,*密码可为空*）
```mysql
create user '用户名'@'登陆主机/ip' identified by '密码';
```

### 样例
```mysql
mysql> create user 'tester'@'localhost' identified by '12354678';
Query OK, 0 rows affected (0.06 sec)

mysql> select user,host,authentication_string from user;
+------------------+-----------+------------------------------------------------------------------------+
| user             | host      | authentication_string                                                  |
+------------------+-----------+------------------------------------------------------------------------+
| stocer           | %         | $A$005$KKu{+%S+?WLyONB^/UZBeHZsE.i2wvJl.wpcoThot.3TydQQeUXmS1fiWoAV4 |
| debian-sys-maint | localhost | $A$005$lTTz`zNMG&^%%^=Okj/cVfbYBZw3XgJ1Fl3CxH/MRwfNhOa/CE6HHJaag1 |
| mysql.infoschema | localhost | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| mysql.session    | localhost | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| mysql.sys        | localhost | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| root             | localhost | *B8B414B84C3D3937BE77A1BA84B2DF8DA83AA442                              |
| tester           | localhost | $A$005$0A_"#,1%*ws+nj^/pEbdS/7ZobqwWK1XRiiozLvuUT/FPxzEwB3zCjBKaK2 |
+------------------+-----------+------------------------------------------------------------------------+
8 rows in set (0.00 sec)
```
可以看到在结果的最后一行多出了一个用户`tester`

### 密码级别问题
可能实际在设置密码的时候，因为mysql本身的认证等级比较高，一些简单的密码无法设置，会爆出如下报错：

```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

不用担心，`MySQL`对新用户的密码强度限制也是可以配置的。

查看当前配置:`show variables like 'validate_password%';`

#### 没有配置
可能有人和我一样，返回了空集合
```mysql
mysql> show variables like 'validate_password%';
Empty set (0.01 sec)
```
此时，可以检查一下 `validate_password` 插件是否已启用

+ 语句为`show plugins`或`SHOW PLUGINS`

如果找不到`validate_password`插件，那它就是未启用的状态，此时我们执行如下语句

+ `install plugin validate_password soname 'validate_password.so';` 安装插件
+ `show plugins` 再查看一次插件状态

如果看到`validate_password`并且是`ACTIVE`状态时，就可以继续配置密码策略了

#### 配置密码策略
我们再查看一次当前配置

查看当前配置:`show variables like 'validate_password%';`

```mysql
mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | ON     |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
```
字段解释：
| Variable_name | Value | 描述 |
| -------------- | ------| --- |
|`validate_password_check_user_name`| `ON`/`OFF` | **是否检查密码中是否包含用户的用户名**。设置为 `ON` 时，密码不得包含用户的名称。 |
| `validate_password_dictionary_file` | <文件路径> | **用于指定密码字典文件的路径**。如果设置了这个值，密码将与字典中的词汇进行比较，以防止使用常见密码。|
| `validate_password_length`| >=1 | **设置密码的最小长度**。此值指定用户创建密码时，密码**至少**需要包含多少个字符。没有最大长度限制，但是建议在`8~128`之间|
| `validate_password_mixed_case_count` | >=0 | **指定密码中至少需要包含一个大写字母和一个小写字母的数量**。设置为 1 表示密码必须至少包含一个大写字母和一个小写字母。|
| `validate_password_number_count` | >=0 | **指定密码中至少需要包含一个数字的数量**。设置为 1 表示密码必须包含至少一个数字。|
| `validate_password_policy` | `LOW`、`MEDIUM` 或 `STRONG` | 设置密码复杂性策略。值为 `LOW`、`MEDIUM` 或 `STRONG`，指示密码的复杂性要求。等级越低，**被忽略**的密码要求越多 |
| `validate_password_special_char_count` | >=0 | **指定密码中至少需要包含一个特殊字符的数量**。设置为 1 表示密码必须包含至少一个特殊字符（如 `!@#$%^&*` 等）。 |

对于三种密码策略的预设，介绍如下：

1. `LOW`
   + 要求：
     + 只需要密码的长度达到最低要求（默认是 8 个字符）。
     + *其余要求的设置会被忽略*
   + 限制：没有其他复杂性要求，即可以使用任意字符组合（包括字母、数字和特殊字符）。
   + 适用场景：适合对安全性要求不高的应用场景。
2. `MEDIUM`
   + 要求：除了满足最低长度要求外，密码还需要包含：
     + 至少一个大写字母
     + 至少一个小写字母
     + 至少一个数字
     + 至少一个特殊字符（如 `!@#$%^&*`）
     + 以上均为默认设置，均可配置，*其余要求的设置会被忽略*
适用场景：适合对安全性有中等要求的应用场景。
1. `STRONG`
   + 要求：需要更高的密码复杂性，必须满足以下条件：
     + 至少 8 个字符的长度（可通过设置 validate_password_length 自定义）。
     + 至少一个大写字母
     + 至少一个小写字母
     + 至少一个数字
     + 至少一个特殊字符
     + 不能包含用户名或其反转的形式
     + 不能包含容易猜测的密码（如字典单词）
     + *以上均为默认设置，均可配置*
适用场景：适合对安全性要求较高的应用场景，尤其是在生产环境中。

解释完成后，我们来着手配置策略,可选的代码如下

```mysql
set global validate_password_policy = 'LOW';    -- 低级别策略
set global validate_password_policy = 'MEDIUM'; -- 中级别策略
set global validate_password_policy = 'STRONG';  -- 高级别策略

set global validate_password_length = 10;         -- 设置最小密码长度
set global validate_password_mixed_case_count = 1; -- 设置至少要有一个大写字母和一个小写字母
set global validate_password_number_count = 1;     -- 设置至少要有一个数字
set global validate_password_special_char_count = 1; -- 设置至少要有一个特殊字符
```

就比如我们现在把密码策略设置成`LOW`

```mysql
mysql> set global validate_password_policy = 'LOW'; 
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_check_user_name    | ON    |
| validate_password_dictionary_file    |       |
| validate_password_length             | 8     |
| validate_password_mixed_case_count   | 1     |
| validate_password_number_count       | 1     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 1     |
+--------------------------------------+-------+
7 rows in set (0.03 sec)
```

**持久化操作**：

要使这些设置在 MySQL 重启后依然有效，需要在 MySQL 配置文件中添加以下内容：
```
[mysqld]
validate_password_policy = MEDIUM
validate_password_length = 10
validate_password_mixed_case_count = 1
validate_password_number_count = 1
validate_password_special_char_count = 1
```

## 删除用户
使用如下语句删除用户

```mysql
drop user '用户名'@'主机名'
```
但如果不加后面的`@'主机名'`,它的缺省值为`@'%'`，可能会出现`误删`，`用户不存在`等问题

> 失败样例
```mysql
mysql> select user,host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| stocer           | %         |
| supdriver        | %         |
| debian-sys-maint | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
| tester           | localhost |
+------------------+-----------+
8 rows in set (0.00 sec)

mysql> drop user 'tester';
ERROR 1396 (HY000): Operation DROP USER failed for 'tester'@'%'
```
## 修改用户密码
+ 自己修改自己的密码
```MySQL
set password=password('新的密码');
```
+ root用户修改其它用户的密码
```MySQL
set password for '用户名'@'主机名'=password('新的密码')；
```
## 用户对数据库的权限管理
就好比Linux文件系统中文件有`读写可执行`三种权限，数据库也是有诸多权限的,如下是主要的权限列表

1. 数据库级权限
   + `ALL PRIVILEGES`: 授予所有权限。
   + `CREATE`: 创建新数据库和表。
   + `DROP`: 删除数据库和表。
   + `INDEX`: 创建和删除索引。
   + `ALTER`: 修改数据库和表的结构。
2. 表级权限
   + `SELECT`: 读取表中的数据。
   + `INSERT`: 向表中插入数据。
   + `UPDATE`: 修改表中的数据。
   + `DELETE`: 从表中删除数据。
   + `EXECUTE`: 执行存储过程。
3. 列级权限
   + `GRANT OPTION`: 允许用户将自己拥有的权限授予其他用户。
   + `CREATE TEMPORARY TABLES`: 创建临时表。
4. 其他权限
   + `FILE`: 读取和写入文件。
   + `PROCESS`: 查看其他用户的线程。
   + `SHUTDOWN`: 关闭服务器。
   + `RELOAD`: 重新加载权限、日志和其他配置。
   + `SUPER`: 执行某些管理操作，如杀死线程、设置全局系统变量等。
   + `REPLICATION SLAVE`: 复制从服务器的权限。
   + `REPLICATION CLIENT`: 查看复制状态。
   + `EVENT`: 创建和删除事件。
5. 特殊权限
   + `LOCK TABLES`: 锁定表。
   + `CREATE ROUTINE`: 创建存储过程和函数。
   + `ALTER ROUTINE`: 修改存储过程和函数。
   + `CREATE VIEW`: 创建视图。
   + `SHOW VIEW`: 查看视图的定义。
   + `CREATE TRIGGER`: 创建触发器。
   + `SHOW DATABASES`: 查看数据库列表。

### 授予用户权限
刚创建的用户默认没有任何权限，还需要手动授予

```mysql
grant 权限列表 on 库.对象名 to '用户名'@'登陆位置' [identified by '密码']
```

+ `权限列表`,多个权限用`,`分开
  + 特别的，`ALL PRIVILEGES`会直接授予所有权限
+ `库.对象名`指定权限的作用范围在哪个数据的哪个对象上(对象有`表`，`视图`，`存储过程`等)
  + `*.*`表示所有数据库的所有对象
  + `库.*`表示指定数据库下的所有对象
+ `identified by`可选。 如果用户存在，赋予权限的**同时修改密码**,如果该用户不存在，就是**创建用户**

如果发现新授予的权限没有生效，可以刷新一下权限

```mysql
flush privileges;
```

### 回收权限
使用`revoke`即可
```mysql
revoke 权限列表 on 库.对象名 from '用户名'@'登陆位置'；
```

# 小结
以上便是用户管理的全部内容了,这一篇是mysql相关开发的铺垫，要认真看完哦