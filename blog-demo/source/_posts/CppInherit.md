---
layout: hexo
title: C++继承
date: 2024-09-02 15:26:20
tags: C++ 继承
---
# 继承的概念及定义

## 继承的概念
`继承`(inheritance)机制是面向对象程序设计使代码可以**复用**的最重要的手段，它允许程序员在保持原有类特性的基础上进行扩展，增加功能，这样产生新的类，称`派生类`。继承呈现了面向对象程序设计的`层次结构`，体现了由简单到复杂的认知过程。以前我们接触的复用都是函数复用，继承是类设计层次的`复用`

下面用一个简单的示例演示一下继承
**示例**
```C++
#include <iostream>
#include <string>
using namespace std;

class Person
{
public:
	void Info()
	{
		printf("name:%s age:%d\n", _name.c_str(), _age);
	}
protected:
	string _name = "supdriver";
	int _age = 24;
};

class Student:public Person
{
public:
	void getID()
	{
		cout << "ID:" << _ID << endl;
	}
private:
	int _ID = 114514;
};


int main()
{
	Student stu;
	stu.Info();
	stu.getID();
	return 0;
}
```

## 继承的使用
当我们准备好一个`基类`后，便可将其用于`派生类`的声明

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Inherit.png)

由图中可以看到，新的概念：`继承方式`，而且很巧的是，它和`访问限定符`的关键字是一一对应的

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409030857308.png)

实际上，它们共同决定了基类成员在派生类内的访问限定符

### 继承基类成员访问方式的变化

| 类成员/继承方式       | `public`继承            | `protected`继承         | `private`继承         |
| --------------------- | ----------------------- | ----------------------- | --------------------- |
| 基类的`public`成员    | 派生类的`public`成员    | 派生类的`protected`成员 | 派生类的`private`成员 |
| 基类的`protected`成员 | 派生类的`protected`成员 | 派生类的`protected`成员 | 派生类的`private`成员 |
| 基类的`private`成员   | 在派生类中`不可见`      | 在派生类中`不可见`      | 在派生类中`不可见`    |

#### 总结
1. `不可见`:基类private成员在派生类中无论以什么方式继承都是不可见的。**这里的不可见是指基类的私有成员还是被继承到了派生类对象中，但是语法上限制派生类对象不管在类里面还是类外面都不能去访问它。**
2. `protected`:基类private成员在派生类中是不能被访问，如果基类成员不想在类外直接被访问，但需要在派生类中能访问，就定义为protected。**可以看出`protected`保护成员限定符是因继承才出现的。**
3. `规律`:实际上面的表格我们进行一下总结会发现，基类的私有成员在子类都是不可见。基类的其他成员在子类的访问方式 == `Min(成员在基类的访问限定符，继承方式)`，其中的大小关系为`public > protected > private`
4. `默认行为`:使用关键字class时默认的继承方式是private，使用struct时默认的继承方式是public，不过**最好显示**的写出继承方式。
5. `扩展性`:在实际运用中一般使用都是public继承，几乎很少使用protetced/private继承，也不提倡使用protetced/private继承，因为protetced/private继承下来的成员都只能在派生类的类里面使用，实际中扩展维护性不强

## 派生类的构造函数
有时基类并不提供无参的默认构造函数,那么如何在派生类的构造函数里`显示地调用基类构造函数`呢?

其实编译器的报错也给出了提示

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/inherit_construct.png)

可以看到，在初始化列表后面出现了报错:`Person不存在默认构造函数`,显然，显示调用构造函数的位置在初始化列表。

那么怎么调用呢？构造函数也是成员函数！除了基类中`private`修饰的构造函数，派生类中皆可调用父类的成员函数，直接在初始化列表中显式调用即可

```C++
#include <iostream>
#include <string>
using namespace std;

class Person
{
public:
	//基类的公有构造函数
	Person(const string& name, const string& gender, int age)
		:_name(name)
		,_gender(gender)
		,_age(age)
	{}
	void Info()
	{
		printf("name:%s age:%d\n", _name.c_str(), _age);
	}
protected:
	string _name;
	string _gender;
	int _age;
};

class Student :public Person
{
public:
	Student(const string& name,const string& gender,int age,int ID)
		:Person(name,gender,age)//显式调用父类的构造函数
		,_ID(ID)
	{}
	void getID()
	{
		cout << "ID:" << _ID << endl;
	}
private:
	int _ID;
};


int main()
{
	Student stu("supdriver", "male", 24, 114514);//调用构造函数
	return 0;
}
```

## 基类和派生类的赋值转化
+ 派生类对象 可以赋值给 `基类的对象` / `基类的指针` / `基类的引用`。这里有个形象的说法叫`切片`或者`切割`。寓意把派生类中父类那部分切来赋值过去。
+ 基类对象**不能**赋值给派生类对象
+ 基类的`指针`或者`引用`可以通过强制类型转换赋值给派生类的指针或者引用。但是必须是**基类的指针是指向派生类对象时才是安全的**。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/slice.png)

下面用一个示例演示一下切片,赋值转换等问题

```C++
#include <iostream>
#include <string>
using namespace std;

class Person
{
public:
	//基类的公有构造函数
	Person(const string& name, const string& gender, int age)
		:_name(name)
		,_gender(gender)
		,_age(age)
	{}
	void Info()
	{
		printf("name:%s age:%d\n", _name.c_str(), _age);
	}
protected:
	string _name;
	string _gender;
	int _age;
};

class Student :public Person
{
public:
	Student(const string& name,const string& gender,int age,int ID)
		:Person(name,gender,age)//显式调用父类的构造函数
		,_ID(ID)
	{}
	void getID()
	{
		cout << "ID:" << _ID << endl;
	}
public:
	int _ID;
};


int main()
{
	Student stu("supdriver", "male", 24, 114514);//调用构造函数

	Person  person(stu);//这里没有显式声明对应的构造函数，而是自动切片
	Person* pp = &stu;//父类指针可以指向派生类
	Person& rp = stu;//父类引用可以指向派生类

	//stu = person;//父类对象不能赋值给派生类

	//基类的指针可以通过强制类型转换赋值给派生类的指针
	pp = &stu;
	Student* pstu = (Student*)pp;//指针地址就是指向派生类的，所以这种情况是安全的，合法的
	pstu->_ID = 233;//这里并没有越界

	pp = &person;
	pstu = (Student*)pp;//这种就是不安全的，可能会越界访问	
	return 0;
}
```

## 继承中的作用域(`隐藏`机制)
1. 在继承体系中`基类`和`派生类`都有独立的作用域(`类域`)
2. 子类和父类中有同名成员，**不指定成员所在类域时**，子类成员将屏蔽父类对同名成员的直接访问，这种情况叫`隐藏`，也叫`重定义`。
3. 需要注意的是如果是成员函数的隐藏，只需要**函数名相**同就构成隐藏。
4. 注意在**实际中**在继承体系里面最好**不要定义同名的成员**。

```C++
#include <iostream>
#include <string>
using namespace std;

class Person
{
public:
	void Info()
	{
		printf("name:%s age:%d\n", _name.c_str(), _age);
	}
protected:
	string _name = "supdriver";
	int _age = 24;
};

class Student:public Person
{
public:
	void Info()//同名函数构成隐藏
	{
		printf("name:%s age:%d\n ID:%d", _name.c_str(), _age,_ID);
	}
private:
	int _ID = 114514;
};


int main()
{
	Student stu;
	stu.Info();//调用Student类内的Info函数，而父类内的被隐藏
	stu.Person::Info();//显式调用Person内的Info函数
	return 0;
}
```

### 隐藏与重载辨析

+ `重载`：函数重载是根据同名函数的`参数类型`和`参数顺序`选择实际调用的函数，选择的动作只能由编译器完成
+ `隐藏`：成员的隐藏是同名成员根据`类域`的`就近原则`由编译器自动调用，但也可以通过指定类域，在代码中选择

## 派生类的默认成员函数
6个默认成员函数，“默认"的意思就是指我们不写，编译器会变我们自动生成一个，那么在派生类
中，这几个成员函数是如何生成的呢?

1. 派生类的构造函数必须调用基类的构造函数初始化基类的那一部分成员。如果基类没有默认的构造函数，则必须在派生类构造函数的初始化列表阶段显示调用。
2. 派生类的拷贝构造函数必须调用基类的拷贝构造完成基类的拷贝初始化。
3. 派生类的operator=必须要调用基类的operator=完成基类的复制。
4. 派生类的析构函数会在被调用完成后自动调用基类的析构函数清理基类成员。因为这样才能保证派生类对象先清理派生类成员再清理基类成员的顺序。
5. 派生类对象初始化先调用基类构造再调派生类构造。
6. 派生类对象析构清理先调用派生类析构再调基类的析构。
7. 因为后续一些场景析构函数需要构成重写，重写的条件之一是函数名相同(这个我们后面会讲解)。那么编译器会对**析构函数名进行特殊处理**，处理成`destrutor()`，所以父类析构函数不加virtual的情况下，子类析构函数和父类析构函数构成隐藏关系。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409031848644.png)

## 继承与友元函数
友元关系不能继承，也就是说基类友元不能访问子类私有和保护成员 

示例如下
```C++
#include <iostream>
#include <string>
using namespace std;

class Student;
class Person
{
public:
	friend void Display(const Person& p, const Student& s);
protected:
	string _name; // 姓名
};
class Student : public Person
{
protected:
	int _stuNum; // 学号
};

void Display(const Person& p, const Student& s)
{
	cout << p._name << endl;
	cout << s._name << endl;//这行代码不会报错，因为_name是继承自Person类的
	cout << s._stuNum << endl;//这行代码会报错，因为函数没有权限访问Student类自己声明的成员
}
void main()
{
	Person p;
	Student s;
	Display(p, s);
}
```

## 继承与静态成员
基类定义了`static静态成员`，则**整个继承体系里面只有一个这样的成员**。无论派生出多少个子类，都**只有一个**static成员实例 。


```C++
#include <iostream>
#include <string>
using namespace std;

class Person
{
public:
	Person() { ++_count; }
protected:
	string _name; // 姓名
public:
	static int _count; // 静态成员变量,统计人的个数。
};
int Person::_count = 0;

class Student : public Person
{
protected:
	int _stuNum; // 学号
};
class Graduate : public Student
{
protected:
	string _seminarCourse; // 研究科目
};
int main()
{
	Student s1;
	Student s2;
	Student s3;
	Graduate s4;
	cout << " 人数 :" << Person::_count << endl;
	Student::_count = 0;
	cout << " 人数 :" << Person::_count << endl;
	return 0;
}
```
结果如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409040748921.png)

## 复杂的菱形继承及菱形虚拟继承

### 单继承

`单继承`：一个子类只有*一个直接父类*时称这个继承关系为`单继承`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/single_inherit.png)

### 多继承

`多继承`：一个子类有**两个或以上直接父类**时称这个继承关系为`多继承`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/multi_inherit.png)

### 菱形继承

`菱形继承`：菱形继承是多继承的一种**特殊情况**。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Diamond_Inherit.png)

`菱形继承的问题`：从下面的对象成员模型构造，可以看出菱形继承有`数据冗余`和`二义性`的问题。
在Assistant的对象中Person成员会有两份。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/IHR_diamond.png)

```C++
class Person
{
public :
 	string _name ; // 姓名
};
class Student : public Person
{
protected :
 	int _num ; //学号
};
class Teacher : public Person
{
protected :
 	int _id ; // 职工编号
};
class Assistant : public Student, public Teacher
{
protected :
 	string _majorCourse ; // 主修课程
};
void Test ()
{
	// 这样会有二义性无法明确知道访问的是哪一个
	Assistant a ;
	a._name = "peter";
	// 需要显示指定访问哪个父类的成员可以解决二义性问题，但是数据冗余问题无法解决
	a.Student::_name = "xxx";
	a.Teacher::_name = "yyy";
}
```

####  虚拟继承 
`虚拟继承`可以解决**菱形继承**的二义性和数据冗余的问题。如上面的继承关系，在Student和Teacher的继承Person时使用`虚拟继承`，即可解决问题。需要注意的是，**虚拟继承不要在其他地方去使用。**

```C++
class Person
{
public :
 	string _name ; // 姓名
};
class Student : virtual public Person //虚拟继承共同父类
{
protected :
 	int _num ; //学号
};
class Teacher : virtual public Person //虚拟继承共同父类
{
protected :
 	int _id ; // 职工编号
};
class Assistant : public Student, public Teacher
{
protected :
 	string _majorCourse ; // 主修课程
};
void Test ()
{
 	Assistant a ;
 	a._name = "peter";//二义性和数据冗余已被解决
}
```

**虚拟继承解决数据冗余和二义性的原理**
下图是菱形虚拟继承的内存对象成员模型：这里可以分析出`Assistant`对象中将`Person`放到的了对象组成的最下面，这个`Person`同时属于`Student`和`Teacher`，那么`Student`和`Teacher`如何去找到公共的`Person`呢？**这里是通过了`Student`和`Teacher`的两个指针，指向的一张表。这两个指针叫`虚基表指针`，这两个表叫`虚基表`。虚基表中存的偏移量。通过偏移量可以找到下面的`Person`**

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/INHERIT_virtual.png)

# 继承的总结和反思

## 尽量不使用多继承，不要设计出菱形继承
C++语法中的`多继承`正是其语法复杂性的体现之一。有了多继承，就存在菱形继承，有了菱形继承就有菱形虚拟继承，底层实现就很复杂。所以**一般不建议设计出多继承，一定不要设计出菱形继承**。否则在复杂度及性能上都有问题。

## 继承和组成辨析
+ public继承是一种`is-a`的关系。也就是说每个派生类对象都是一个基类对象
+ 组合是一种`has-a`的关系。假设B组合了A，每个B对象中都有一个A对象。
+ 为了提高代码的`封装性`和`解耦合`，[优先使用对象组合，而不是类继承🔗](https://www.cnblogs.com/nexiyi/archive/2013/06/16/3138568.html)
+ `白箱复用`：继承允许你根据基类的实现来定义派生类的实现。这种通过生成派生类的复用通常被称为白箱复用(white-box reuse)。术语“白箱”是相对可视性而言：在继承方式中，**基类的内部细节对子类可见** 。继承一定程度**破坏**了基类的封装，基类的改变，对派生类有很大的影响。派生类和基类间的依赖关系很强，**耦合度高**。
+ `黑箱复用`：对象组合是类继承之外的另一种复用选择。新的更复杂的功能可以通过组装或组合对象来获得。对象组合要求被组合的对象具有良好定义的接口。这种复用风格被称为黑箱复用(black-box reuse)，因为对象的内部细节是不可见的。对象只以“黑箱”的形式出现。组合类之间没有很强的依赖关系，耦合度低。优先使用对象组合有助于你保持每个类被封装。
+ 实际尽量多去用组合。组合的耦合度低，代码维护性好。不过继承也有用武之地的，有些关系就适合继承那就用继承，另外要实现多态，也必须要继承。类之间的关系可以用继承，可以用组合，就用组合。