---
title: v2.3美鹅外卖平台数据库守护线程更新
date: 2025-03-12 08:57:29
tags:
---
# 更新要点
+ 为`DatabaseClient`配置了一个负责重连的异步守护线程
+ 完成了`DatabaseClient`和`HTTPServer`的解耦合
+ 为每一个数据库客户端接口的异常处理处加入了被动检测连接状态
+ 异步线程循环中加入了计数器，能够定时自动触发连接状态检测
+ 解决了查询订单列表时导致客户端崩溃的问题

# 为什么引入守护线程
原先的数据库连接更多的偏向于能用就行的实验性功能，这导致了一个很严重的问题:

**数据库连接不稳定**，以及**只连接一次数据库，导致断连之后只能通过重启整个容器的方式重新连接**,运维成本过高

因此我们引入守护线程，专门异步地负责数据库的重连。这样就能大大提高数据库连接的健壮性。

# 代码结构解耦合
原先嵌入在`HTTPServer`内的`initDB()`接口暴露了`DatabaseClient`内的`init()`接口参数，导致这两个类的耦合度过高，一旦修改`init()`接口的参数，`initDB`也要跟着修改，这已经是非常明显的因为耦合度过高所造成的问题了，为此要通过修改代码的方式实现二者的解耦合

这里我们采用最简单的方法解耦合:

1. 将原先的`DatabaseClient db`改为智能指针类型`std::shared_ptr<DatabaseClient> db`
2. `HTTPServer`不再负责初始化`db`，而是改为提供接口，由外部传入`db`指针
3. `DatabaseClient`的实例化在`main`函数中完成，然后把指针传给`HTTPServer`单例

改变后的`main()`函数如下

```C++
int main(int argc, char *argv[])
{
    google::ParseCommandLineFlags(&argc,&argv,true);//解析命令行参数
    btyGoose::init_logger(FLAGS_log_release_mode,FLAGS_log_logfile,FLAGS_log_release_level);//初始化日志器
    LOG_DEBUG("全局日志器初始化成功");
    auto db = std::make_shared<btyGoose::DatabaseClient>();
    db->init(FLAGS_db_user,FLAGS_db_password,FLAGS_db_host,FLAGS_db_port,FLAGS_db_database,                         
        std::chrono::seconds(FLAGS_db_reconnect_interval));
    btyGoose::HTTPServer::getInstance()->setDB(db);
    btyGoose::HTTPServer::getInstance()->init(FLAGS_host,FLAGS_port);
    btyGoose::HTTPServer::getInstance()->start();
    return 0;
}
```

# 添加异步守护线程

1. 增加对应的成员变量,*相应地命令行参数也要增加，这里不赘述了*

```C++
private:
	//守护线程相关成员变量
	std::atomic<bool> connected_= false;	//表示连接状态
    std::chrono::seconds reconnect_interval_;//两次重连的最小间隔时间
    std::condition_variable reconnect_cv_;	//重连条件变量
    std::mutex reconnect_mtx_;		//重连锁
    std::thread reconnect_thread_;	//守护线程
    bool stop_reconnect_ = false;	//停止重连的标记

	void reconnectLoop();
	bool checkConnection();
```

2. 实现相关的接口

```C++
//━━━━━━━━━━━━━━ 守护线程相关实现 ━━━━━━━━━━━━━━//

void DatabaseClient::reconnectLoop()
{
    int cnt = 0;//添加一个自动检测连接的计数器
    while(!stop_reconnect_)
    {
        std::unique_lock<std::mutex> lock(reconnect_mtx_);
        reconnect_cv_.wait_for(lock,reconnect_interval_);//最多等待reconnect_interval_秒
        //因为上面有可能是超时等待成功，所以还要再判断一下是否需要重连
        if(!connected_ && !stop_reconnect_)
        {
            LOG_INFO("开始尝试重连");
            try{
                std::unique_lock<std::mutex> db_lock(mtx);//锁住数据库其它操作

                //清理旧连接
                delete stmt;
                delete con;

                //建立新连接
                sql::mysql::MySQL_Driver* driver = sql::mysql::get_driver_instance();
                con = driver->connect("tcp://"+host+":"+port,user,password);
                con->setSchema(database);
                stmt = con->createStatement();
                connected_ = true;

                LOG_INFO("数据库成功重连喵~ (≧∇≦)ﾉ");

            }
            catch(const sql::SQLException e){
                LOG_ERROR("重连失败: {} [错误码: {}]", e.what(), e.getErrorCode());
            }   
        }
        else
        {
            cnt++;
            if(cnt == 50)
            {
                //如果经过50次唤醒，连接状态都没更新，则自动检测一次
                checkConnection();
                cnt = 0;
            }
        }

    }
}

bool DatabaseClient::checkConnection()
{
    std::unique_lock<std::mutex> lock(mtx);
    if(connected_ == false) return false;//如果连接已断开，则不用检查了
    try{
        if(con && con->isValid() && !con->isClosed())
        {
            //执行简单查询验证链接
            std::unique_ptr<sql::Statement> test_stmt(con->createStatement());
            std::unique_ptr<sql::ResultSet> res(test_stmt->executeQuery("SELECT 1"));
            return true;
        }
    }catch (const sql::SQLException&){}

    LOG_WARN("当前数据库连接已丢失");
    connected_ = false;
    return false;
}
```

3. 修改`start()`函数和析构函数

```C++
void DatabaseClient::start()
{
    try {
        sql::mysql::MySQL_Driver *driver = sql::mysql::get_mysql_driver_instance();
        LOG_INFO("开始连接数据库 tcp://{}:{}\n\tuser:{}",host,port,user);
        con = driver->connect("tcp://"+host+":"+port, user, password);
        con->setSchema("BeautyGoose");
        stmt = con->createStatement();
        connected_ = true;
        //启动守护线程
        reconnect_thread_ = std::thread(std::bind(&DatabaseClient::reconnectLoop,this));
        LOG_INFO("守护线程已启动");
    } catch (sql::SQLException &e) {
        LOG_FATAL( "数据库连接失败喵！\n\t错误码: {}\n\t信息:",e.getErrorCode(),e.what());
        exit(-1);
    }
}

DatabaseClient::~DatabaseClient()
{
    stop_reconnect_ = true;
    reconnect_cv_.notify_all();
    if(reconnect_thread_.joinable())    
    {
        reconnect_thread_.join();
    }

    delete stmt;
    delete con;
}
```

# 采用被动检测连接策略
为了防止检测连接状态过于频繁，挤占了正常的业务通信，这里选用了**被动检测的策略**,即出现的`异常`被`catch`后，在里面额外添加对连接状态的检测，也就是调用`checkConnection()`函数

# 附加计数器激发检测策略
除了被动检测，我们额外添加了一个计数器检测的策略，在保持连接状态一定时间后(状态不变的原因可能是这段时间没有服务请求)，主动检测一次连接状态。