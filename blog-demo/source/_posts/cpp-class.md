---
title: 从构建一个Date类入门C++类与对象
date: 2023-12-07 07:42:59
tags: C++ 类和对象
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Date.jpg
---
# 类的定义 #
```C++
class Date
{
public:
    void Init(int year = 1,int month = 1,int day = 1)
    {
        _year = year;
        _month = month;
        _day = day;
    }

    void Print()
    {
        cout << _year << ":" << _month << ":" << _day << endl;
    }
private:
    int _year;
    int _month;
    int _day;
};
```
## 抽象数据类型(类) #
通过如上代码，我们就在源代码中通过`class`声明了一个抽象数据类型`Date`,简称`类`，那么封装一个类有什么好处呢？
好处是类把相关的操作分为**两类**:

- 类的**设计者**:负责考虑类的具体实现，提供类的接口，成员变量等
- 类的**使用者**:只关心类**提供**了哪些功能，而不关心具体实现，从而简化思路
  
以上面的`Date`类为例
>对设计者
- 要考虑实现`Date`,就需要声明**成员变量**`_year` `_month` `_day`,以及声明及实现**成员函数**`Init`和`Print`
  
>对使用者
- 只需知道可以调用`Date`的**成员函数**`Init`和`Print`,以及知道它们的用处即可

## 实例化 -- 将类真正投入使用 ##
类也可以用于声明变量，例如`Date d`就声明了一个变量`d`,但由于是由`类`声明的,我们将这一过程称为`实例化`,其中`Date`这样的抽象数据类型称为`类`，像`d`这样的变量称为`对象`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-04-16_11-19-21.png)

实例化后的对象拥有**私有**的`成员变量`和整个类**公有**的`成员函数`,接下来对`对象`的操作都是对`成员变量`和`成员函数`的操作

## 访问成员函数/变量

### 在`类的内部`
对于类的成员函数，除了显式声明的`函数参数`外,还有**隐式**传入的`this`指针，这是个**默认`非const`修饰**的,指向调用该成员函数的**对象的指针**,编译器可以通过这个指针访问该对象的`成员变量`和`成员函数`。


而我们作为类的**设计者**，既然语法都**隐式**地传入`this`指针了，自然也可以**隐式**地调用`成员`,即**直接写**变量名/函数名调用

*当然，手动显式调用this指针也是可以的*

>以`Date`为例
```C++
//该函数声明在Date类中，成员变量见文章开头
void TestPrint()
{
    _year = 2024;//隐式调用this访问成员变量
    this->_month = 4;//显式调用this
    _day = 1;
    Print();//隐式调用this来调用成员函数Print()
    this->Print();//显式调用this，效果与上一句相同
}
```
但由于`const`修饰的`对象`传出的是`const`修饰的`this`指针，普通的`this`形参无法接收。
那么如何让`成员函数`传入`const`修饰的`this`指针,来使`const`修饰的`对象`有成员函数可调用呢？

语法规定，在函数的参数列表(*圆括号后面*)紧跟一个`const`可使函数传入`const`修饰的`this`指针

*这种函数称为常量成员函数*

>*举个例子*
```C++
//示例代码
//该函数声明在类中
void constPrint() const
{
    //....
}
```

### 在`类的外部`
和C语言的结构体一样，访问对象内的成员有两种方式

- *对象名* + `.` + 成员名 ： 用`.`操作符访问对应成员
- *对象的指针* + `->` + 成员名 : 用`->`操作符访问指针**指向对象**的对应成员

>以`Date`实例化一个`d`为例

```C++
// class Date
// {
//     ....
// };

void test1()
{
    Date d;
    Date* pd = &d;
    d.Init(2024,4,1);// . 操作符调用Init成员函数来初始化对象
    pd->Print();//->操作符调用Print成员函数来打印内容
}

void test2()
{
    Date d;
    d._year = 2024;//试图访问成员变量_year,但是访问权限冲突
}
```
```C++
//test1输出
2024:4:1
//test2输出
报错，无输出，因为访问权限冲突
```

代码如上，`test1`运行的很好，但`test2`报错了，原因在于`test2`作为**非成员函数**访问了访问限定符`private`控制的成员`_year`,权限冲突，就会报错。

由此，C++类和对象还有一个重要概念需要强调--**访问控制**

## 访问权限控制与封装 ##
使用类和对象编程的一大优点就是类可以`封装`代码，让使用者只能使用公有的接口和成员变量，而对内部的具体实现不可见，来提高类的`易用性`和`安全性`

所以C++语法提供了三种`访问说明符`(*access specifiers*)

- public: 该说明符之后的成员在整个程序内可被访问
- private: 之后的成员仅可被该类的的类域里（*如成员函数*）访问
- protected: 一般同`private`,主要特点体现在类的继承，这里**不作讨论**

### 作用范围 #

某一`访问说明符`的作用范围开始于它的`冒号`,终止于下一个`访问说明符`或`类的结尾`,而`类的开始`到第一个`访问说明符`前的访问权限取决于**声明**类的`关键字`,分类如下

- `class`默认为`private`权限
- `struct`默认为`public`权限

> 图例如下
> 
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-04-18_14-15-40.png)


实际上`class`和`struct`除了默认权限不一样，基本**没有差别**。

所以为了防止误读，**提高可读性**,**不建议**在`默认区`写代码,而是保证每段语句前都有合适的`访问限定符`

### 封装 #
作为类的设计者，一个类按`访问权限`可以分为两个区

- `public`: 将提供给`使用者`的`接口（函数）`和`成员变量`声明在此，用于外部调用接口和修改非私有的成员函数
- `private`: 用于存放**受保护**的`成员变量`和`成员函数`,防止外部使用者*意外*或*恶意*调用或修改,造成类的内部结构被破坏等**安全问题**,所以特别重要的成员变量，和不希望被外部调用的函数声明在这里

>例如在声明`Date`类时，我们将`Init`和`Print`接口提供给使用者，用`public`控制；`_year`等成员变量不希望被外部随意修改，就用`private`控制


# 进阶 #
学完以上内容，不过是会写个高级点的`结构体`而已，要写一个完整的类，还需要学习更多的语法知识

## 构造函数 #
像本篇的`Date`类那样显式地调用`Init`函数来初始化是非常挫的，既然语言本身的`内置类型`可以在声明的时候初始化，那么类的设计者设计出来的类也应当提供`初始化的接口`,而支持这一功能的接口便是`构造函数`

按语法规定，`构造函数`的函数名必须是`类名`,**没有返回值**，`const`修饰的成员变量必须位于`初始化列表`,其它则可省略。*关于初始化列表，稍后详细解释*

>以`Date`类为例
```C++
class Date
{
public:
    //一个普通的构造函数
    Date(int year,int month,int day)
    {
        //进入括号时成员变量已经声明，且未初始化
        _year = year;//这是一个赋值操作，而不是初始化
        _month = month;
        _day = day;
    }

private:
    int _year;
    int _month;
    int _day;
}

int main()
{
    Date d(2024,4,1);//使用构造函数声明了一个d对象
    return 0;
}

```

以上的构造函数基本能用了,但还有两个问题

- 构造函数没有初始化成员变量，而是采用赋值操作,无法初始化`const`修饰的成员变量
- 使用`Date d`是会报错的，因为没有提供`默认构造函数`

对于**第一个问题**，就要引入`初始化列表`这一概念，让初始化函数直接拥有`初始化成员变量`的功能

初始化列表位于构造函数的参数列表之后，花括号之前，以`:`开头，用`,`分隔成员变量

>以`Date`为例
```C++
Date(int year,int month,int day):_year(year) , _month(month) , _day(day) {}
//或者换个书写格式（二者完全等价）
Date(int year,int month,int day):_year(year) 
, _month(month) 
, _day(day) 
{}
```
通过这样`初始化列表`,便能在声明对象时，**直接**初始化成员变量

对于**问题二**，我们开启另一个个小专题

### 构造函数的重载和缺省参数 #
没错，构造函数和函数一样，也是能`重载`和给参数传`缺省值`的

*也就是说我们能写好几个构造函数*

下面特别说明几个**特殊**的构造函数

#### 默认构造函数 #
原则上对于每一个类，都应该提供**有且仅有一个**默认构造函数（*多个`默认构造函数`会报错!*）

而要**声明**默认构造函数，只需声明`无参数`构造函数，或者`全缺省参数`构造函数即可

>以`Date`为例
```C++
Date():_year(2024),_month(4),_day(1){}
//====分割线=======
//或者全缺省，两个函数不能同时声明
Date(int year = 2024,int month = 4,int day = 1): _year(year),_month(month),_day(day){}

```

以上就是两种默认构造函数的声明形式

#### 拷贝构造函数 #
有时候我们会希望用**现有的**的对象去初始化一个**新**对象,此时对应的构造函数就称为`拷贝构造（函数）`

`拷贝构造`的声明方式为`构造函数`+参数类型为`类本身的引用传参`,不加`&`的话就会死递归报错,有无`const`皆可，但由于是实现`拷贝功能`，一般是加`const`的

>以`Date`为例
```C++
Date(const Date& d):_year(d._year),_month(d._month),_day(d._day){}

//使用示例
Date d(2024,4,1);
Date copy1(d);//调用方式一
Date copy2 = d;//调用方式二,此时不会调用operator=()

copy1 = copy2;//这种并不会调用拷贝构造,而是调用operator=()

```

#### 使用模板的类的函数缺省值 #
有时我们在使用类模板来设计类时，需要给`模版类`类型的形参提供一个缺省值，有些人可能会写个`0`,但是其实是**错的**，正确的做法是传一个`临时变量`

但此时要求`模板参数中的类`有可用的`默认构造函数`和`拷贝构造`用于调用

>以链表节点`Node`为例
```C++
template<class value_type>
struct Node
{
    value_type _val;
    Node<value_type>* _next;

    Node(const value_type& val = value_type()):_val(val),_next(nullptr){}
}

//以用本文的Date实例化为例
Node<Date> node;
//通过输出会发现node中的val已经调用了默认构造函数
node.val.TestPrint();
```

## 析构函数 #
对于声明在`栈区`或`静态区`的成员函数，程序完全可以自动销毁，
但如果`成员变量`有指向在`堆区`声明的某段`内存块`,在该如果只是仍由程序自动
销毁这个指针，那么那段`内存块`就会一直处于**未释放**的状态，也就是造成内存泄漏，
也就是说此时编译器自动生成的`析构函数`已经不能满足需求,编译器并不知道如何处理声明在`堆区`上的数据,
这部分操作应由类的设计者来规划

所以我们应当**显式**地声明一个合理的`析构函数`

`析构函数`的函数名也是由语法规定的，为`~`+`类名`,并且**不能**声明形参

>以一个`指针类`为例
```C++
class Ptr
{
public:
    Ptr()//构造函数
    { 
        _ptr = new int(1); 
    }

    ~Ptr()//析构函数
    {
        delete _ptr;//手动delte堆区上的数据
        _ptr = nullptr;
    }

private:
    int* _ptr;
}
```

## 类的重载操作符 #
C++语法提供了重载操作符的函数，而由于`this`指针的存在，在类的`内部声明`重载操作符函数会稍有不同

-对于一元操作符，`[]`,`->`之类的重载，不再需要`显式传参`
-对于二元操作符，`+`,`>`之类只需要传`右操作数`

>以`Date`类为例
```C++
//在类的内部,重构一个 ==
public:
    bool operator==(const Date& date) const
    {
        return _year == date._year
            && _month == date._month
            && _day == date._day;
    }
```

# 小结
至此，C++类和对象已基本入门，再进阶的`迭代器`,`继承`,`虚继承`等将单独出博客。

