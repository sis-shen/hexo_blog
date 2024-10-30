---
title: 【代码整理】同步&异步⽇志系统的代码复盘
date: 2024-10-09 11:13:18
tags:
---

在项目的[介绍博客🔗](https://www.supdriver.top/2024/09/24/muiltiDesignPatternsLogSystem/)中，我们着重于代码功能的实现，
而对项目代码整体的结构和风格欠缺重视，使其**可读性有所欠缺**,**代码规范性也有所不足**。所以在这篇博客中，我们将按上篇博客编写代码的顺序，
再一次回顾写好的代码，并着重于：

+ 调整代码结构
+ 增加必要注释，删除冗余注释
+ 添加必要的小括号，突出优先级
+ 将代码的简便写法改写回易读的写法

# 实用类

+ 增加功能简介
+ 增加注释
+ 增加空行
```C++
#pragma once

#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <ctime>
#include <cassert>
#include <sys/stat.h>

//提供各种实用类，内部封装了实用接口

namespace suplog{
    namespace util{
        class date
        {
        public:
            //获取当前时间戳--秒级
            static size_t now(){return (size_t)time(nullptr);}
        };

        class file{
        public:
            static bool exists(const std::string&name)
            {
                //struct stat配合stat函数作为输出型参数
                struct stat st;
                //stat获取文件的状态，若成功获取，就会返回0，可用于判断文件是否存在,stat函数是系统调用接口
                return stat(name.c_str(),&st) == 0;
            }

            static std::string path(const std::string &name)
            {
                //规定找不到就返回".""
                if(name.empty()) return ".";

                size_t pos = name.find_last_of("/\\");//找到最后一个"/"或"\\"
                return name.substr(0,pos+1);//取得目录的路径
            }

            static void create_directory(const std::string &path)
            {
                if(path.empty()) return;//路径为空
                if(exists(path)) return;//路径不存在-使用系统调用接口

                //逐级创建目录
                size_t pos,index = 0;//pos标记所要创建的目录，index用于找"/"
                while(index<path.size())
                {
                    //找到index及以后的"/"
                    pos = path.find_first_of("/\\",index);
                    if(pos == std::string::npos)
                    {
                        //创建最后一层目录
                        mkdir(path.c_str(),0755);
                        //函数出口
                        return;
                    }
                    //二者重叠时，跳到下一次循环
                    if(pos ==index) {index = pos+1;continue;}

                    //准备创建目录，准备路径
                    std::string subdir = path.substr(0,pos);
                    //文件夹已存在，不用创建，跳到下一段即可
                    if(subdir == "." || subdir == "..")
                        {index = pos + 1;continue;}
                    if(exists(subdir))//理由同上
                        {index = pos + 1;continue;}
                    //创建目录文件，权限为0755
                    mkdir(subdir.c_str(),0755);
                    index = pos + 1;
                }
            }
        };
    }
}
```

# 日志等级类

+ 提供功能描述
+ 增加注释

```C++
#pragma once

//提供日志等级服务
//提供日志等级枚举
//提供日志等级转字符串

namespace suplog{
    class LogLevel
    {
    public:
        //枚举所有的日志等级
        enum class Level
        {
            DEBUG = 1,
            INFO,
            WARN,
            ERROR,
            FATAL,
            OFF
        };

        //提供日志等级转换成字符串的静态成员函数
        static const char* toString(LogLevel::Level lv)
        {
            switch(lv)
            {
                //在内部定义一个宏函数,把变量名name变成字符串
                //宏函数能较少函数调用，提高运行效率
                #define TOSTRING(name) #name
                case LogLevel::Level::DEBUG: return TOSTRING(DEBUG);//直接返回，不加用break
                case LogLevel::Level::INFO: return TOSTRING(INFO);
                case LogLevel::Level::WARN: return TOSTRING(WARN);
                case LogLevel::Level::ERROR: return TOSTRING(ERROR);
                case LogLevel::Level::FATAL: return TOSTRING(FATAL);
                case LogLevel::Level::OFF: return TOSTRING(OFF);
                #undef TOSTRING //取消宏函数
                default: return "UNKOWN";
            }

            //提供默认返回值
            return "UNKNOWN";//防止编译报错
        }

    };
}
```

# 日志消息类

+ 增加功能描述
+ 增加注释

## 注释勘误
+ 对`shared_ptr`的注释有误，不是全局唯一，是全局引用计数的智能指针

```C++
#pragma once

#include "util.hpp"
#include "level.hpp"
#include <thread>//c++多线程库
#include <memory>
#include <string>

//实现LogMsg类，将日志消息封装起来，用于组织管理
//且自动生成时间戳和tid，简化用户操作

namespace suplog{
    struct LogMsg
    {
        //声明一个全局引用计数的智能指针
        using ptr = std::shared_ptr<LogMsg>;

        std::string _name;  //日志器名称
        std::string _file;  //文件名
        size_t _line;       //行号
        std::string _payload;//日志消息
        size_t _ctime;      //时间-时间戳
        std::thread::id _tid;//线程id
        LogLevel::Level _level;//日志等级
        //默认构造函数
        LogMsg(){}
        //构造函数
        LogMsg(
            std::string name,
            std::string file,
            size_t line,
            std::string payload,
            LogLevel::Level level
        ):
        _name(name),
        _file(file),
        _payload(payload),
        _level(level),
        _line(line),
        _ctime(util::date::now()),
        _tid(std::this_thread::get_id())
        {}//默认构造函数结尾

    };
}
```

# 日志输出格式化类

+ 修改每个`FormatItem`
  + 增加注释对功能的描述
  + 增加`override`标记为虚函数重写
  + 增加`return`强调成员函数的函数身份
+ 增加对`Foramtter`类的更详细注释
+ 调整私有成员到类的最后，把共有成员提前

```C++
#pragma once

#include "util.hpp"
#include "level.hpp"
#include "message.hpp"
#include <memory>
#include <vector>
#include <tuple> //提供一种存储一组不同类型变量的容器

//提供日志信息嵌入格式化字符串的类实现
//FormatItem抽象类

namespace suplog{

    /*=========FormatItem及其派生类===========*/
    class FormatItem
    {
    public:
        using ptr = std::shared_ptr<FormatItem>;
        virtual ~FormatItem(){}
        //规定统一实现format接口
        virtual void format(std::ostream& os,const LogMsg& msg) = 0;
    };

    //日志消息输出
    class MsgFormatItem:public FormatItem
    {
    public:
        MsgFormatItem(const std::string& str = ""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<msg._payload;
            return;//没有实际意义，只是标记函数结束
        }
    };

    //日志等级输出
    class LevelFormatItem:public FormatItem{
    public:
        LevelFormatItem(const std::string& str = ""){};
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<LogLevel::toString(msg._level);
            return;
        }
    };

    //日志器名称输出
    class NameFormatItem:public FormatItem{
    public:
        NameFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<msg._name;
            return;
        }
    };

    //线程td输出
    class ThreadFormatItem:public FormatItem{
    public:
        ThreadFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<msg._tid;
            return;
        }
    };

    //格式化时间输出
    class TimeFormatItem:public FormatItem{
    public://共有接口写在最前面
        TimeFormatItem(const std::string& format="%H:%M:%S"):_format(format)
        {
            if(format.empty()) _format = {"%H:%M:%S"};//这里要获取格式串
        }
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            time_t t = msg._ctime;
            struct tm lt;
            localtime_r(&t,&lt);//从时间戳t中提取时间信息到结构体lt中
            char tmp[128];
            strftime(tmp,127,_format.c_str(),&lt);//格式化日期信息到字符串
            os<<tmp;//最后输出
            return;
        }
    private://私有接口写在后面
        std::string _format;
    };

    //日志调用者的文件名输出
    class CFileFormatItem:public FormatItem{
    public:
        CFileFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<msg._file;
            return;
        }
    };

    //日志调用行号输出
    class CLineFormatItem:public FormatItem{
    public:
        CLineFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<msg._line;
            return;
        }
    };

    //Tab输出
    class TabFormatItem:public FormatItem{
    public:
        TabFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<'\t';
            return;
        }
    };        

    //换行符输出
    class NLineFormatItem:public FormatItem{
    public:
        NLineFormatItem(const std::string& str=""){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<'\n';
            return;
        }
    };

    //其它日志信息输出
    class OtherFormatItem:public FormatItem{
    public://公共接口写在前面
        OtherFormatItem(const std::string &str=""):_str(str){}
        virtual void format(std::ostream& os,const LogMsg& msg) override
        {
            os<<_str;
            return;
        }
    private:
        std::string _str;
    };

    /*====End==Of==FormatItem及其派生类===========*/

    class Formatter
    {
    public:
        using ptr = std::shared_ptr<Formatter>;//内置一个指针类
        /*
            %d ⽇期
            %T 缩进
            %t 线程id
            %p ⽇志级别
            %c ⽇志器名称
            %f ⽂件名
            %l ⾏号
            %m ⽇志消息
            %n 换⾏
            */
        
        //使用格式串初始化,规则如上
        Formatter(const std::string& pattern = "[%d{%H:%M:%S}][%t][%p][%c][%f:%l] %m%n")
        :_pattern(pattern)
        {
            assert(parsePattern());//调用成员函数，后面声明
        }

        //简单的查看pattern的接口
        const std::string pattern() { return _pattern; }
        //最后实现，但是是最常用的接口，故写在前面
        //使用LogMsg对象生成日志字符串
        std::string format(const LogMsg& msg)
        {
            std::stringstream ss;//创建string流
            //按格式串生成日志信息
            for(auto& it: _items)
            {
                it->format(ss,msg);
            }
            return ss.str();
        }

        //创建FormatItem(给调用者使用多态)
        FormatItem::ptr createItem(const std::string& fc,const std::string& subfmt)
        {
            //根据传入的format character选择创建的FormatItem类

            if(fc == "m") return FormatItem::ptr(new MsgFormatItem(subfmt));
            if(fc == "p") return FormatItem::ptr(new LevelFormatItem(subfmt));
            if(fc == "c") return FormatItem::ptr(new NameFormatItem(subfmt));
            if(fc == "t") return FormatItem::ptr(new ThreadFormatItem(subfmt));
            if(fc == "n") return FormatItem::ptr(new NLineFormatItem(subfmt));
            if(fc == "d") return FormatItem::ptr(new TimeFormatItem(subfmt));
            if(fc == "f") return FormatItem::ptr(new CFileFormatItem(subfmt));
            if(fc == "l") return FormatItem::ptr(new CLineFormatItem(subfmt));
            if(fc == "T") return FormatItem::ptr(new TabFormatItem(subfmt));
            return FormatItem::ptr();//未知fc
        }
        //pattern解析

        //每个要素分为三部分：
        // 格式化字符 : %d %T %p...
        // 对应的输出⼦格式 ： {%H:%M:%S}
        // 对应数据的类型 ： 0-表⽰原始字符串，也就是⾮格式化字符，1-表⽰格式化数据类型

        //默认格式"[%d{%H:%M:%S}][%t][%p][%c][%f:%l] %m%n"
        bool parsePattern()
        {
            //储存分割好的字符串,是否为原始字符串由int指示
            std::vector<std::tuple<std::string,std::string,int>> arry;//储存分析后的原素材
            std::string format_key;//存放%之后的格式化字符
            std::string format_val;//存放格式化字符串后边{}中的子格式化字符串
            std::string string_row;//存放原始的非格式化字符串
            bool sub_format_error = false;
            int pos = 0;
            while(pos < _pattern.size())//开始扫描格式串
            {
                //储存原始字符
                if(_pattern[pos] != '%')
                {
                    string_row.push_back(_pattern[pos++]);
                    continue;//跳过循环的后半部分,创建尽可能长的原始字符串
                }
                if(pos+1<_pattern.size() && _pattern[pos+1] == '%')
                {
                    //规定%%格式化为原生的'%'字符
                    string_row.push_back('%');
                    pos+=2;
                    continue;//跳过循环的后半部分,创建尽可能长的原始字符串
                }
                if(string_row.empty() == false)//原生字符串不为空
                {
                    //准备好了一段素材
                    arry.push_back(std::make_tuple(string_row,"",0));
                    //清空容器
                    string_row.clear();
                }

                //当前位置指向%字符
                pos++;//pos指向格式化字符位置
                if(pos<_pattern.size() && isalpha(_pattern[pos]))//判断是不是字符
                {
                    //储存fc字符
                    format_key = _pattern[pos];
                }
                else 
                {
                    std::cout<< &_pattern[pos-1]<<"位置附近格式错误！\n";
                    return false;
                }
                //下一步使pos指向格式化字符的下个位置，判断是否包含有子格式,例如在%d{%Y-%m-%d}中
                pos++;
                if(pos+1 < _pattern.size() && _pattern[pos] == '{')
                {
                    sub_format_error = true;
                    pos++;//pos指向花括号下一个字符
                    while (pos<_pattern.size())
                    {
                        if(_pattern[pos] == '}')
                        {
                            //循环出口
                            sub_format_error = false;
                            pos++;
                            break;
                        }
                        //在循环中时,存储子格式
                        format_val.push_back(_pattern[pos++]);
                    }
                    if(sub_format_error)
                    {
                        std::cout<<"{}对应出错\n";
                        return false;
                    }
                }
                //储存格式字符创建的原素材
                arry.push_back(std::make_tuple(format_key,format_val,1));
                format_key.clear();
                format_val.clear();
            }//结束循环

            //继续处理未清空的容器/缓存
            if(string_row.empty() == false)//原生字符串不为空
                arry.push_back(std::make_tuple(string_row,"",0));
            if(format_key.empty() == false)//格式化字符不为空，注，上下顺序不能换,要和循环内一致
                arry.push_back(std::make_tuple(format_key,format_val,1));

            //开始产生item列表
            if(_items.empty() == false)_items.clear();//清理_items
            for(auto& it:arry)
            {
                if(std::get<2>(it) == 0)//获取第三个元素
                {
                    //生成非格式原始字符串
                    FormatItem::ptr fi(new OtherFormatItem(std::get<0>(it)));
                    _items.push_back(fi);
                }
                else 
                {
                    FormatItem::ptr fi = createItem(std::get<0>(it),
                                                    std::get<1>(it));
                    if(fi.get() == nullptr)
                    {
                        std::cout<<"没有对应的格式化字符串： %"
                                 <<std::get<0>(it)
                                 <<std::endl;
                        return false;
                    }
                    _items.push_back(fi);
                }
            }//完成拼接
            return true;
        }

    private:
        std::string _pattern;//储存格式串
        std::vector<FormatItem::ptr> _items;//产生格式化字符串的生成器列表
    };

}
```

# 日志落地类
+ 增加注释 

```C++
#pragma once

#include "util.hpp"
#include "message.hpp"
#include "formatter.hpp"
#include <unistd.h>
#include <iostream>
#include <memory>
#include <mutex>

namespace suplog{
    //声明抽象父类LogSink
    class LogSink
    {
    public:
        using ptr = std::shared_ptr<LogSink>;
        LogSink(){}
        virtual ~LogSink() =default;
        virtual void log(const char*data,size_t len) = 0;//声明统一的log接口
    };

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

    //文件落地类
    class FileSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<FileSink>;
        FileSink(const std::string& filename):_filename(filename)
        {
            //创建目录
            util::file::create_directory(util::file::path(filename));
            _ofs.open(_filename,std::ios::binary|std::ios::app);//二进制方式写入
            //打开文件失败就assert,待改成抛异常
            assert(_ofs.is_open());
        }

        //提供给外界获取文件名
        const std::string &file(){ return _filename; }

        void log(const char*data,size_t len) override 
        {
            //写入数据
            _ofs.write((const char*)data,len);
            if(_ofs.good() == false)
            {
                //文件流状态异常
                std::cout<<"日志输出文件失败!\n";
            }
            return;
        }
    private:
        std::string _filename;//文件路径
        std::ofstream _ofs;//文件输出流
    };

    //滚动文件落地类
    class RollSink:public LogSink
    {
    public:
        using ptr = std::shared_ptr<RollSink>;
        //实际文件名 = bsename + 可变部分
        RollSink(const std::string& basename,size_t max_size)
        :_basename(basename),_max_fsize(max_size),_cur_fsize(0)
        {
            //创建目录
            util::file::create_directory(util::file::path(_basename));
        }

        void log(const char*data,size_t len) override 
        {
            initLogFile();//初始化输出文件和文件输出流
            _ofs.write(data,len);//写入数据
            if(_ofs.good() == false)
            {
                std::cout<<"日志输出文件失败!\n";
            }
            _cur_fsize += len;//更新文件大小
            return;
        }

    private:
        void initLogFile(){
            //如果满足条件，触发创建新文件的条件
            if(_ofs.is_open() == false || _cur_fsize >=_max_fsize)
            {
                _ofs.close();//先关闭原有文件流
                if(_cur_fsize >=_max_fsize) sleep(1);//防止同一秒有过多日志消息要打印，导致打开同一个日志文件
                std::string name = createFilename();
                _ofs.open(name,std::ios::binary | std::ios::app);
                assert(_ofs.is_open());
                _cur_fsize = 0;//创建新文件后，重置文件大小
                return;
            }
            return;
        }
        
        //封装产生文件名的函数
        std::string createFilename()
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
        std::string _basename;//基础文件名
        std::ofstream _ofs;//输出用的文件输出流
        size_t _max_fsize;//文件最大容量
        size_t _cur_fsize;//当前输出文件的使用量
    };

    //创建简单工厂
    class SinkFactory{
    public:
        //可变参数列表里储存了构造函数所需的所有参数
        template<typename SinkType,typename ...Args>
        static LogSink::ptr create(Args&&...args){
            //返回shared_ptr,传的是构造函数参数列表
            return std::make_shared<SinkType>(std::forward<Args>(args)...);
        }
    };
}
```

# 日志类

+ 为更多接口添加功能描述
+ 使用注释为代码分块
+ 调整接口函数的括号位置，使格式统一
+ 为`void`函数添加`return`，标记函数结尾

## 补丁
内部的`Build`类未提供清空`_sinks`的接口,遂添加`clearSink`接口

```C++
protected:
    //清空落地类列表
    void clearSink(){
        _sinks.clear();
        return;
    }
```

## 修改后的代码
```C++
#include "util.hpp"
#include "level.hpp"
#include "message.hpp"
#include "formatter.hpp"
#include "sink.hpp"
#include "looper.hpp"//不认识？没关系
                    //这个头文件后面才实现,给异步日志器的实现用的
#include <vector>
#include <list>
#include <atomic>
#include <unordered_map>
#include <cstdarg>
#include <type_traits>

namespace suplog{
    class Logger{
    public:
        //类内枚举日志器的类型
        enum class Type{
            LOGGER_SYNC = 0,
            LOGGER_ASYNC
        };
        using ptr = std::shared_ptr<Logger>;

        Logger(const std::string& name,
            Formatter::ptr formatter,
            std::vector<LogSink::ptr> &sinks,
            LogLevel::Level level = LogLevel::Level::DEBUG
        ):_name(name),_level(level),_formatter(formatter),
        _sinks(sinks.begin(),sinks.end())
        {}

        std::string loggerName(){ return _name; }//获取日志器名称
        LogLevel::Level loggerLevel(){ return _level; }//获取日志器等级

        //使用C语言风格的不定参数输出日志
        //接口log位于protected访问限定符下
        //=========start========
        //各个接口的主要差别在于日志等级不同
        void debug(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::DEBUG) == false)//如果等级不满足日志器最低要求
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::DEBUG,file,line,fmt,al);//日志输出,al向函数传递可变参数列表
            va_end(al);//结束可变参数列表
        }

        void info(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::INFO) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::INFO,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void warn(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::WARN) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::WARN,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void error(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::ERROR) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::WARN,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        void fatal(const char*file,size_t line,const char*fmt,...)
        {
            if(shouldLog(LogLevel::Level::FATAL) == false)
                return;

            va_list al;
            va_start(al,fmt);//依据fmt从内存中提取可变参数列表
            log(LogLevel::Level::FATAL,file,line,fmt,al);//日志输出
            va_end(al);//结束可变参数列表
        }

        //===============end===============

    public:
        //在内部声明一个建造者抽象类
        class Builder
        {
        public:
            using ptr = std::shared_ptr<Builder>;

            Builder():_level(LogLevel::Level::DEBUG),
                _logger_type(Logger::Type::LOGGER_SYNC)
            {}
            
            //默认支持以下接口
            //======start======//

            //设置日志器名称
            void buildLoggerName(const std::string& name){ 
                _logger_name = name;
                return;
            }

            //设置日志器等级
            void buildLoggerLevel(LogLevel::Level level){ 
                _level = level;
                return;
            }

            //设置日志器类型
            void buildLoggerType(Logger::Type type){
                _logger_type = type;
                return;
            }

            //设置日志器格式串
            void buidFormatter(const Formatter::ptr& formatter){
                _formatter = formatter;
                return;
            }

            //设置日志器格式串-函数重载
            void buidFormatter(const std::string& formatStr){
                auto formatter = std::make_shared<suplog::Formatter>(formatStr);
                _formatter = formatter;
                return;
            }

            //C++风格不定参数
            //增加落地类
            template<typename SinkType,typename ...Args>
            void buildSink(Args &&...args){
                auto sink = SinkFactory::create<SinkType>(std::forward<Args>(args)...);
                _sinks.push_back(sink);
                return;
            }

            //清空落地类列表
            void clearSink(){
                _sinks.clear();
                return;
            }
            //======end======//

            //声明抽象接口build，具体实现交给派生类
            virtual Logger::ptr build() = 0;
        protected:
            Logger::Type _logger_type;
            std::string _logger_name;
            LogLevel::Level _level;
            Formatter::ptr _formatter;
            std::vector<LogSink::ptr> _sinks;
        };//=====Build类声明定义结束=======

    //回到Logger类
    protected:
    //根据日志等级，判断是否应该日志输出
    bool shouldLog(LogLevel::Level level){ return level >= _level; }

    void log(LogLevel::Level level,const char*file,
        size_t line,const char*fmt,va_list al)
    {
        char *buf;//可以不初始化
        std::string msg;
        //将格式化串和可变参数列表生成字符串，并存入buf指向的内存
        int len = vasprintf(&buf,fmt,al);//自动在堆区申请内存
        if(len < 0)
            msg = "格式化日志消息失败!!";
        else 
        {
            msg.assign(buf,len);//转存到msg对象中
            free(buf);//释放空间
        }

        //LogMsg(name, file, line, payload, level)
        LogMsg logmsg(_name,file,line,std::move(msg),level);//使用了拷贝构造
        std::string str;
        //使用logmsg获取输出字符串/输出任务
        str = _formatter->format(logmsg);
        logIt(std::move(str));//真正开始执行输出任务。如何调用落地类由派生类具体实现
    }

    //交给派生类实现
    virtual void logIt(const std::string &msg) = 0;

    protected:
        std::mutex _mutex;//为多线程环境提前准互斥量
        std::string _name;
        Formatter::ptr _formatter;
        std::atomic<LogLevel::Level> _level;//使用atomic使_level的修改操作原子化
        std::vector<LogSink::ptr> _sinks; 
    };
```

## 同步日志器类
+ 增加注释和调整函数结构

```C++
//同步日志器
    class SyncLogger:public Logger
    {
    public:
        using ptr = std::shared_ptr<SyncLogger>;

        SyncLogger(const std::string& name,
            Formatter::ptr formatter,
            std::vector<LogSink::ptr>&sinks,
            LogLevel::Level level = LogLevel::Level::DEBUG)
            :Logger(name,formatter,sinks,level)
            {
                std::cout << LogLevel::toString(level)<<"同步日志器创建成功...\n";
            }

        private:
            virtual void logIt(const std::string& msg_str)override
            {
                 // lock 的析构函数在离开作用域时自动释放互斥锁
                std::unique_lock<std::mutex> lock(_mutex);
                if(_sinks.empty()) { return; }//没有落地方向

                //每个落地方向都输出一次
                for(auto &it:_sinks)
                    it->log(msg_str.c_str(),msg_str.size());

                return;
            }
    };
```

## 本地日志器建造者类
+ 增加注释

```C++
//本地日志器建造者
    class LocalLoggerBuilder:public Logger::Builder
    {
    public:
        virtual Logger::ptr build() override
        {
            //检测名称是否存在
            if(_logger_name.empty())
            {
                std::cout<<"日志器名称不能为空！！";
                abort();
            }
            //检测格式串
            if(_formatter.get() == nullptr){
                std::cout<<"当前日志器： "<<_logger_name;
                std::cout<<" 未检测到⽇志格式,默认设置为: ";
                std::cout<<" %d{%H:%M:%S}%T%t%T[%p]%T[%c]%T%f:%l%T%m%n\n";
                _formatter = std::make_shared<Formatter>();
            }
            //检测是否存在落地方向
            if(_sinks.empty())
            {
                std::cout<<"当前日志器: "<<_logger_name<<"问检测到落地方向，默认为标准输出!\n";
                _sinks.push_back(std::make_shared<StdoutSink>());
            }

            Logger::ptr lp;//使用多态
            if(_logger_type == Logger::Type::LOGGER_ASYNC)
            {
                lp = std::make_shared<AsyncLogger>(_logger_name,_formatter,_sinks,_level);
            }
            else 
            {
                lp = std::make_shared<SyncLogger>(_logger_name,_formatter,_sinks,_level);
            }
            return lp;
        }
    };
```

# 双缓冲区异步任务处理器 

## Buffer类
+ 增加注释
+ 调整函数结构

```C++
#include <iostream>
#include <string>
#include <vector>
#include <thread>
#include <mutex>
#include <atomic>
#include <condition_variable>
#include <functional>
#include <cassert>

namespace suplog
{

// 设置Buffer的一些基本参数
#define BUFFER_DEFUALT_SIZE (1*1024*1024)      //缓冲区默认大小
#define BUFFER_INCREMENT_SIZE (1*1024*1024)    //缓冲区默认扩展大小
#define BUFFER_THRESHOLD_SIZE (10*1024*1024)   //缓冲区最大值

class Buffer{
public:
    Buffer():_reader_idx(0),_writer_idx(0),_v(BUFFER_DEFUALT_SIZE){}

    bool empty(){ return _reader_idx == _writer_idx; }        //判空
    size_t readAbleSize(){ return _writer_idx - _reader_idx; }//是否可读
    size_t writeAbleSize(){ return _v.size() - _writer_idx; } //是否可写
    void reset(){ _reader_idx = _writer_idx = 0; }  //重置读写指针
    //交换缓冲区
    void swap(Buffer& buf)
    {
        _v.swap(buf._v);
        std::swap(_reader_idx,buf._reader_idx);
        std::swap(_writer_idx,buf._writer_idx);
    }
    
    //向缓冲区推送数据
    void push(const char*data,size_t len)
    {
        ensureEnoughSpace(len);
        assert(len<=writeAbleSize());
        std::copy(data,data+len,&_v[_writer_idx]);
        _writer_idx+=len;
    }

    //缓冲区删除定长数据
    void pop(size_t len)
    {
        _reader_idx +=len;
        assert(_reader_idx <=_writer_idx);
    }

    const char* begin() 
    {
        //获取起始地址 
        return &_v[_reader_idx]; 
    }

protected:
    void ensureEnoughSpace(size_t len)
    {
        //如果写的下,则退出
        if(len <=writeAbleSize())return;

        size_t new_capacity;
        //每次增加1M的大小
        if(_v.size() < BUFFER_THRESHOLD_SIZE)
            new_capacity = _v.size()*2+len;
        else 
            new_capacity = _v.size() + BUFFER_INCREMENT_SIZE + len;

        _v.resize(new_capacity);
        return;
    }

private:
    size_t _reader_idx;
    size_t _writer_idx;
    std::vector<char> _v;
};

}
```

## AsyncLooper 类
+ 增加注释
+ 修改变量名`al`为`alooper`，防止与之前出现过的变量发生含义混淆，影响代码理解

```C++
#pragma once

#include "util.hpp"
#include "buffer.hpp"
#include <vector>
#include <thread>
#include <mutex>
#include <atomic>
#include <condition_variable>
#include <functional>

namespace suplog
{
    class AsyncLooper
    {
    public:
        using ptr = std::shared_ptr<AsyncLooper>;
        using Functor = std::function<void(Buffer &buffer)>;

        AsyncLooper(const Functor&cb)
        : _running(true),
        _looper_callback(cb),
        _thread(std::thread(&AsyncLooper::worker_loop,this))//在构造函数中启动线程
        {}

        ~AsyncLooper(){ stop(); }

        void stop()
        {
            _running =false;
            _pop_cond.notify_all();//将所有线程的条件变量等待唤醒
            _thread.join();//接收子进程
        }

        //推送任务
        void push(const std::string&msg)
        {
            if(_running == false) return;

            std::unique_lock<std::mutex> lock(_mutex);//加锁

            //等待条件
            _push_cond.wait(lock,[&]{
                return _tasks_push.writeAbleSize() >= msg.size();
            });//防止消息过大

            _tasks_push.push(msg.c_str(),msg.size());//完成消息任务推送

            _pop_cond.notify_all();//唤醒所有消费者
        }

    private:
        //子线程的入口函数
        static void worker_loop(void* arg)
        {
            //多线程执行函数得靠arg传递this指针
            AsyncLooper* alooper = (AsyncLooper* )arg;
            while(1)
            {
                 // lock 的析构函数在离开作用域时(完成一趟循环)自动释放互斥锁
                std::unique_lock<std::mutex> lock(alooper->_mutex);
                //线程出口,为空或关闭时退出循环
                if(alooper->_running == false && alooper->_tasks_push.empty())
                    return;
                
                //生产者缓冲不为空或者停止运行时才会被唤醒
                alooper->_pop_cond.wait(lock,[&]{
                    return !alooper->_tasks_push.empty() || !alooper->_running;
                });
                alooper->_tasks_push.swap(alooper->_tasks_pop);//交换缓冲区

                alooper->_push_cond.notify_all();//唤醒所有生产者缓冲区
                alooper->_looper_callback(alooper->_tasks_pop);//调用回调函数，输出消费者缓冲区
                //输出完成，清空缓冲区
                alooper->_tasks_pop.reset();
            }
        }
    
    private:
        Functor _looper_callback;//输出缓冲区内容的回调函数
    private:
        std::mutex _mutex;//互斥锁
        std::atomic<bool> _running;//能够原子地赋值状态指示
        std::condition_variable _push_cond;//生产者条件变量
        std::condition_variable _pop_cond;//消费者条件变量
        Buffer _tasks_push;//双缓冲区
        Buffer _tasks_pop;
        std::thread _thread;//主线程管理子线程类用的
    };
}
```

## 异步日志器
+ 增加注释
+ 增加`return`

```C++
//异步日志器
class AsyncLogger:public Logger
{
public:
    using ptr = std::shared_ptr<AsyncLogger>;

    AsyncLogger(const std::string& name,
        Formatter::ptr formatter,
        std::vector<LogSink::ptr>&sinks,
        LogLevel::Level level = LogLevel::Level::DEBUG)
        :Logger(name,formatter,sinks,level)
        ,_looper(std::make_shared<AsyncLooper>(
            std::bind(&AsyncLogger::readLog,this,std::placeholders::_1)))
            //传一个this,使包装器里的函数能够是成员函数,this后面的才是包装器指定的参数
        {
            std::cout << LogLevel::toString(level)<<"异步日志器创建成功...\n";
        }

private:
    virtual void logIt(const std::string &msg) override
    {
        _looper->push(msg);//推送消息
        return;
    }

    //_looper所用的回调函数
    void readLog(Buffer& msg)
    {
        if(_sinks.empty()){ return; }//判空

        for(auto &it:_sinks)
        {
            //调用落地功能
            it->log(msg.begin(),msg.readAbleSize());//直接一次性输出所有缓存的日志
        }
        return;
    }
protected:
    AsyncLooper::ptr _looper;
};
```

# 单例日志器管理类
+ 基本没有改动

# 全局日志建造者
+ 基本没有改动

# suplog.hpp
+ 无变化

# 小结
这次整理代码，一方面是增加了其可读性，使其看起来更优雅，更整齐，另一方面也是再一次加深对项目的理解，顺便还能勘误 👆🤓