---
title: 设计模式的C++实现(5)——原型模式
date: 2024-10-29 11:07:41
tags:
---

> **模式名称**: 原型模式-Prototype
> **类型**: 创建型模式
> **问题-使用场景**: 当创建不同对象的过程过于复杂，或者需要隐藏/封装创建对象的具体过程，或者**组织项目需要创建过多的子类，类的数目需要优化减少时**，亦或者需要**动态类**时，可以由**原型对象**来执行类的功能
> **解决方案**： 用原型实例指定创建对象的种类，并通过**拷贝**这些原型来创建新的对象
> **效果**： 相比其它模式，用户只需考虑怎么检索到所需要的`原型对象`来拷贝出新的对象给自己用，而不用考虑如何构造。这样的设计能简化用户操作，且能极大地增加扩展性。

# 概念抽象

原型模式旨在通过使用不同的**原型对象**来克隆获取不同的实例，而不是声明许多派生类，再通过派生类来实例化出不同的对象。这样减少类总数的设计方式，可以很好地简化整个项目的**类的结构设计**,毕竟类总数越多，要维护的类关系就越复杂,理解成本就越高。

原型模式抽象出的参与者有如下三种
1. `Client`:负责找到指定原型，并调用其克隆接口，克隆出一个对象
2. `Prototype`:抽象类，声明一个抽象接口
3. `ConcretePrototype`:实现具体的克隆操作

三者的类图关系如下：

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410301057940.png)

# 具体使用
我们来举一个具体一点的例子来体现原型模式的思想。

接下来来我们设计一个乐谱编辑-绘制系统，让它能够动态地注册音符、添加音符到乐谱，绘制乐谱。于是我们在类的设计上有决策

1. `MusicNoteDrawer`类负责注册原型，管理原型和绘制乐谱
2. `MusicNote`抽象类规定音符派生类的接口，尤其是`Clone`接口
3. `音符派生类`，可以每一个音符都设计一个派生类，但我们也可以使用原型的思想，设计一个更为复杂的`WholeNote`类和`HalfNote`类，让它们在实例化时调整不同的参数来表示不同的音符。这样我们就把众多音符派生类压缩成了两大类，而不同的音符对象依然能够正确表示

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410301010763.png)

类图设计如上，按照上面的设计，我们编写如下C++代码示例

```C++
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>
using namespace std;

class MusicNote
{
public:
	virtual MusicNote* clone() = 0;//规定提供克隆接口

	virtual void draw() = 0;//规定提供绘制自身的接口
};

class WholeNote:public MusicNote//继承
{
public:
	WholeNote(int level) :_level(level) {}

	WholeNote* clone()override
	{
		return new WholeNote(_level);
	}

	void draw()override
	{
		cout << "WholeNote(" << _level << ") ";
	}

private:
	int _level;
};

class HalfNote :public MusicNote//继承
{
public:
	HalfNote(int level) :_level(level) {}

	HalfNote* clone()override
	{
		return new HalfNote(_level);
	}

	void draw()override
	{
		cout << "HalfNote(" << _level << ") ";
	}

private:
	int _level;
};


class MusicNoteDrawer
{
public:
	MusicNoteDrawer():_index() {}

	bool registerNote(const string& name, int level)//提供动态注册功能
	{
		string fullName = name + to_string(level);
		if (_index.count(fullName))return false;//已存在

		if (name == "WholeNote")
		{
			_index[fullName] = new WholeNote(level);
			return true;
		}
		else if (name == "HalfNote")
		{
			_index[fullName] = new HalfNote(level);
			return true;
		}
		else
		{
			cout << "unkown Note :" << fullName << endl;
			return false;
		}
	}

	//为乐谱添加音符
	bool addNote(const string& name, int level)
	{
		string fullName = name + to_string(level);
		if (_index.count(fullName))
		{
			_music.push_back(_index[fullName]->clone());//调用克隆接口
			return true;
		}
		else
		{
			cout << "unkown Note :" << fullName << endl;
			return false;
		}
	}

	//绘制乐谱
	void draw()
	{
		for (auto ptr : _music)
		{
			ptr->draw();
		}
		cout << endl;
	}

	~MusicNoteDrawer()
	{
		for (auto& p : _index)
		{
			delete p.second;
			p.second = nullptr;
		}
		for (auto ptr : _music)
		{
			delete ptr;
			ptr = nullptr;
		}
	}
private:
	vector<MusicNote*>_music;//储存乐谱
	unordered_map<string, MusicNote*> _index;//实现索引原型
};


int main()
{
	MusicNoteDrawer* mnd = new MusicNoteDrawer;

	for (int i = 0; i < 10; ++i)
	{
		//注册各种音符
		mnd->registerNote("WholeNote", i);
		mnd->registerNote("HalfNote", i);
	}
    //制作乐谱
	mnd->addNote("HalfNote", 1);
	mnd->addNote("HalfNote", 1);
	mnd->addNote("WholeNote", 5);
	mnd->addNote("WholeNote", 5);
	mnd->addNote("WholeNote", 6);
	mnd->addNote("WholeNote", 6);
	mnd->addNote("HalfNote", 5);

	mnd->draw();//绘制

	return 0;
}
```

上述代码中，当我们想要添加`MusicNote`派生类时，不需要从类示例化出对象，而是找到对于的原型调用它的`clone()`接口来获取对象，这就是原型模式的工作方式

实际上，我们还可以把类的数量再压缩一下，把`WholeNote`和`HalfNote`合并到同一个更复杂的类中,类图如下:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202410301134316.png)

# 功能总结

## 适用性
+ 当一个系统应该**独立于**它的产品创建、构成和表示时，要使用`Prototype`模式
+ 需要的原型是要再运行时刻动态指定的，使用`Prototype`模式十分合适
+ 需要避免创建一个与产品类层次平行的工厂类层次时，也可以用此种模式
+ 当一个类的示例限定只能从几个人为指定的状态组合中选择一种时，建立相应数目的原型，并只用它们的克隆接口，可能比每次使用合适的状态手工实例化该类要更方便一些。

## 优点

+ `Prototype`有许多和`Abstract Factory`和`Builder`一样的效果：它对客户隐藏了具体的产品类，因此减少了客户需要知道的参数的数目。
+ `运行时动态增加和删除产品`：`Prototype`运行客户注册原型实例，就可以将新的具体产品类并入系统（这个原型执行了类的功能）。这使得它比别的创建型模式更为灵活，提供了更多的运行时的动态性
+ `将原型作为一个动态类来使用`： 通过在运行时动态地指定原型对象的一些参数，我们就可以获得一个动态变化的原型来执行动态类的功能，而不是在编译前声明更多的类
+ `提高对象的复用性`：许多应用由部件和子部件来创建复杂对象。为用户提供注册这样复杂对象的原型，可以提高其复用性，减少用户创建复杂对象的代价。比如在电路设计编辑器中，一个电路由多个电子器件相互连接构成，编辑器应当为用户提供注册用户自定义的电路对象原型的接口，并且让用户能够方便地复用 
+ `减少子类的构造` `Factory Method`经常产生一个与产品类层次平行的`Creator`类层次。`Prototype`模式是的你克隆一个原型而不是请求一个工厂方法去产生一个新的对象。因此更不不需要`Creator`类层次。这对于像`C++`这样不将类作为一级类对象的语言受益颇多
+ `用类动态配置应用`：`Prototype`模式允许动态将类(原型对象执行类的功能)装载到应用中,在C++这样的语言中，`Prototype`是利用这种功能的关键

## 缺点
+ `Prototype`的主要缺陷是每一个`Prototype`子类都必须实现`Clone操作`，有时这可能很困难。例如，当对象内部包括一些不支持拷贝或有循环引用的对象时，实现克隆可能会很困难
+ `派生类复杂化和聚合化`,`ConcretePrototype`作为`Prototype`的派生类，要使实例化出的原型能够执行类的功能，往往需要将许多属性和接口聚合在同一个类中，这将使这个类的复杂度大大提升

# 使用要点

## 原型管理器 Prototype Manager
当一个系统中原型数目不固定时，或者说当向客户开放了创建和销毁原型的接口时，需要保持一个可用原型的注册表。因为尽管客户不会自己来管理原型，但是会在注册表中储存和检索原型。客户在克隆一个原型前需要向注册表请求该原型，我们可以形象地把这个注册表称为原型管理器

## 克隆操作时的 深拷贝 与 浅拷贝 问题
一旦涉及到对象的拷贝时，就绕不开深拷贝与浅拷贝问题。克隆出的对象究竟和原型共享某些资源呢，还是深度拷贝一份，这都是需要好好考虑的问题

## 维护Clone接口统一性
有时客户可能想要在克隆原型时传入参数来进行初始化，但是如果通过`Clone`接口传参，这将会破坏克隆接口的统一性。如果想要提供传参功能，应当提供额外的接口，例如`Initialize`接口
