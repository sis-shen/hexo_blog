---
title: 设计模式的C++实现(3)——工厂模式与建造者模式
date: 2024-09-24 17:19:21
tags: 设计模式
---

> **模式名称**: 工厂模式-Factory
> **类型**: 创建型
> **问题-使用场景**: 当创建不同对象的过程过于复杂，或者需要隐藏/封装创建对象的具体过程，或需要创建同一家族的产品（对象）时
> **解决方案**：将创建对象的过程封装到工厂类内，并提供设置创建不同对象的接口
> **效果**： 简化了用户操作，使创建对象，创建多个不同的对象更为简单易用，提高了代码的封装性。但是代码复杂性大大提高，类封装的层次更多了，可扩展性会有所欠缺。

> **模式名称**: 建造者模式-Builder
> **类型**: 创建型
> **问题-使用场景**: 当创建不同对象的过程过于复杂，或者需要隐藏/封装创建对象的具体过程，或需要创建同一家族的产品（对象）时
> **解决方案**：将创建对象的过程封装到Builder内，并提供设置创建不同对象的接口，以及提供不同的Builder派生类,最后提供一个`指挥者类`统一指挥对象的创建
> **效果**： **可扩展性强**的同时，简化了用户操作，使创建对象，创建多个不同的对象更为简单易用，提高了代码的封装性。但是代码复杂性大大提高，类封装的层次更多了。

⼯⼚模式是⼀种创建型设计模式， 它提供了⼀种创建对象的最佳⽅式。在⼯⼚模式中，我们创建对象时不会对上层暴露创建逻辑，⽽是通过使⽤⼀个共同结构来指向新创建的对象，以此实现创建-使⽤的分离。

工厂模式可分为以下三种：

+ 简单工厂模式
+ 工厂方法模式
+ 抽象工厂模式

接下来我们具体介绍这三种工厂模式

# 简单工厂模式
简单⼯⼚模式实现由⼀个`⼯⼚对象`通过`类型决定`创建出来指定产品类的实例。假设有个⼯⼚能⽣产出⽔果，当客⼾需要产品的时候明确告知⼯⼚⽣产哪类⽔果，⼯⼚需要接收⽤⼾提供的类别信息，当新增产品的时候，`⼯⼚内部去添加`新产品的⽣产⽅式

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409251006985.png)

+ **优点**:简单粗暴，直观易懂。使⽤⼀个⼯⼚⽣产同⼀等级结构下的任意产品
+ **缺点**:
  + 所有东西⽣产在⼀起，产品太多会导致代码量庞⼤
  + 开闭原则遵循(开放拓展，关闭修改)的不是太好，要新增产品就必须修改⼯⼚⽅法。

```C++
class Fruit {//水果的抽象类
public:
	Fruit() {}
	virtual void show() = 0;
};

class Apple : public Fruit {//苹果类
public:
	Apple() {}
	virtual void show() {
		std::cout << "我是⼀个苹果" << std::endl;
	}
};
class Banana : public Fruit {//香蕉类
public:
	Banana() {}
	virtual void show() {
		std::cout << "我是⼀个⾹蕉" << std::endl;
	}
};
class FruitFactory {//水果工厂类,能生茶哪些水果由名字和内部代码决定
public:
	//当前代码只能生产苹果或香蕉,扩展性差
	static std::shared_ptr<Fruit> create(const std::string& name) {
		if (name == "苹果") {
			return std::make_shared<Apple>();
		}
		else if (name == "⾹蕉") {
			return std::make_shared<Banana>();
		}
		return std::shared_ptr<Fruit>();
	}
};
int main()
{
    //fruit有使用多态
	std::shared_ptr<Fruit> fruit = FruitFactory::create("苹果");
	fruit->show();
	fruit = FruitFactory::create("⾹蕉");
	fruit->show();
	return 0;
}

```

这个模式的结构和管理产品对象的⽅式⼗分简单， 但是它的**扩展性⾮常差**，当我们需要新增产品的时候，就需要去修改⼯⼚类新增⼀个类型的产品创建逻辑，违背了开闭原则。

# 工厂方法模式
在简单⼯⼚模式下新增多个⼯⼚，多个产品，**每个产品对应⼀个⼯⼚**。假设现在有A、B 两种产品，则开两个⼯⼚，⼯⼚ A 负责⽣产产品 A，⼯⼚ B 负责⽣产产品 B，⽤⼾只知道产品的⼯⼚名，⽽不知道具体的产品信息，⼯⼚不需要再接收客⼾的产品类别，⽽只负责⽣产产品。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409251031391.png)

+ **优点**:
  + **减轻了⼯⼚类的负担**，将某类产品的⽣产交给指定的⼯⼚来进⾏
  + **开闭原则遵循较好**，添加新产品只需要新增产品的⼯⼚即可，不需要修改原先的⼯⼚类
+ **缺点**： 缺点：对于某种可以形成⼀组产品族的情况处理较为复杂,需要**创建⼤量的⼯⼚类**

```C++
#include <iostream>
#include <memory>
#include <string>

class Fruit {
public:
	Fruit() {}
	virtual void show() = 0;
};
class Apple : public Fruit {
public:
	Apple() {}
	virtual void show() {
		std::cout << "我是⼀个苹果" << std::endl;
	}
private:
	std::string _color;
};
class Banana : public Fruit {
public:
	Banana() {}
	virtual void show() {
		std::cout << "我是⼀个⾹蕉" << std::endl;
	}
};
class FruitFactory {
public:
	virtual std::shared_ptr<Fruit> create() = 0;
};
//手写一个苹果工厂类，继承父类
class AppleFactory : public FruitFactory {
public:
	virtual std::shared_ptr<Fruit> create() {
		return std::make_shared<Apple>();
	}
};
//手写一个香蕉工厂类，继承父类
class BananaFactory : public FruitFactory {
public:
	virtual std::shared_ptr<Fruit> create() {
		return std::make_shared<Banana>();
	}
};
int main()
{
	//factory有使用多态
	std::shared_ptr<FruitFactory> factory(new AppleFactory());
	//fruit有使用多态
	std::shared_ptr<Fruit> fruit;
	fruit = factory->create();
	fruit->show();
	factory.reset(new BananaFactory());
	fruit = factory->create();
	fruit->show();
	return 0;
}
```

# 抽象工厂模式
⼯⼚⽅法模式通过引⼊⼯⼚等级结构，解决了简单⼯⼚模式中⼯⼚类职责太重的问题，但由于⼯⼚⽅法模式中的**每个⼯⼚只⽣产⼀类产品**，可能会导致系统中存在 **⼤量的⼯⼚类**，势必会增加系统的开销。此时，我们可以考虑将⼀些**相关的产品组成⼀个产品族**（位于不同产品等级结构中功能相关联的产品组成的家族），由同⼀个⼯⼚来统⼀⽣产，这就是抽象⼯⼚模式的基本思想。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409251106237.png)

+ **优点**:采用了分层结构
+ **缺点**:违背了开闭原则，可扩展性差，扩展新产品线时需要对原代码有大量修改

```C++
#include <iostream>
#include <string>
#include <memory>

class Fruit {
public:
	Fruit() {}
	virtual void show() = 0;
};
class Apple : public Fruit {
public:
	Apple() {}
	virtual void show() {
		std::cout << "我是⼀个苹果" << std::endl;
	}
private:
	std::string _color;
};
class Banana : public Fruit {
public:
	Banana() {}
	virtual void show() {
		std::cout << "我是⼀个⾹蕉" << std::endl;
	}
};
class Animal {
public:
	virtual void voice() = 0;
};
class Lamp : public Animal {
public:
	void voice() { std::cout << "咩咩咩\n"; }
};
class Dog : public Animal {
public:
	void voice() { std::cout << "汪汪汪\n"; }
};
class Factory {
public:
	virtual std::shared_ptr<Fruit> getFruit(const std::string& name) = 0;
	virtual std::shared_ptr<Animal> getAnimal(const std::string& name) = 0;
};
class FruitFactory : public Factory {
public:
	virtual std::shared_ptr<Animal> getAnimal(const std::string& name) {
		return std::shared_ptr<Animal>();
	}
	virtual std::shared_ptr<Fruit> getFruit(const std::string& name) {
		if (name == "苹果") {
			return std::make_shared<Apple>();
		}
		else if (name == "⾹蕉") {
			return std::make_shared<Banana>();
		}
		return std::shared_ptr<Fruit>();
	}
};
class AnimalFactory : public Factory {

public:
	virtual std::shared_ptr<Fruit> getFruit(const std::string& name) {
		return std::shared_ptr<Fruit>();
	}
	virtual std::shared_ptr<Animal> getAnimal(const std::string& name) {
		if (name == "⼩⽺") {
			return std::make_shared<Lamp>();
		}
		else if (name == "⼩狗") {
			return std::make_shared<Dog>();
		}
		return std::shared_ptr<Animal>();
	}
};
class FactoryProducer {
public:
	static std::shared_ptr<Factory> getFactory(const std::string& name) {
		if (name == "动物") {
			return std::make_shared<AnimalFactory>();
		}
		else {
			return std::make_shared<FruitFactory>();
		}
	}
};
int main()
{
	std::shared_ptr<Factory> fruit_factory = FactoryProducer::getFactory("⽔果");
	std::shared_ptr<Fruit> fruit = fruit_factory->getFruit("苹果");
	fruit->show();
	fruit = fruit_factory->getFruit("⾹蕉");
	fruit->show();
	std::shared_ptr<Factory> animal_factory = FactoryProducer::getFactory("动物");
	std::shared_ptr<Animal> animal = animal_factory->getAnimal("⼩⽺");
	animal->voice();
	animal = animal_factory->getAnimal("⼩狗");
	animal->voice();
	return 0;
}
```
抽象⼯⼚模式适⽤于⽣产多个⼯⼚系列产品衍⽣的设计模式，增加新的产品等级结构复杂，需要对原有系统进⾏较⼤的修改，甚⾄需要修改抽象层代码，**违背了“开闭原则”**。

不过比较有意思的是具体的工厂（例如水果工厂）又可以采用不同的设计模式，不一定是简单工厂模式

# 建造者模式
建造者模式是⼀种创建型设计模式， 使⽤多个简单的对象**⼀步⼀步构建**成⼀个复杂的对象，能够将⼀个复杂的对象的构建与它的表⽰分离，提供⼀种创建对象的最佳⽅式。主要⽤于解决对象的构建过于复杂的问题。

建造者模式基于四个核心实现:

+ 抽象产品类
+ 具体产品类： 一个具体的产品对象类
+ 抽象`Builder类`:实现抽象接口，构建各个部件
+ 指挥者`Director类`:统一组件过程，提供给调用者使用，通过指挥者来构造产品

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409260755865.png)

+ `优点`:
  + 具体产品和具体的建造者都继承自抽象类
  + 由统一的指挥者类指挥构建,不同的产品类有统一的实例化方式
  + 可扩展性强,仅需实现不同的具体产品类和具体的建造者类

```C++
#include <iostream>
#include <memory>
/*抽象电脑类*/
class Computer {
public:
	using ptr = std::shared_ptr<Computer>;
	Computer() {}
	void setBoard(const std::string& board) { _board = board; }
	void setDisplay(const std::string& display) { _display = display; }
	virtual void setOs() = 0;
	std::string toString() {
		std::string computer = "Computer:{\n";
		computer += "\tboard=" + _board + ",\n";
		computer += "\tdisplay=" + _display + ",\n";
		computer += "\tOs=" + _os + ",\n";
		computer += "}\n";
		return computer;
	}
protected:
	std::string _board;
	std::string _display;
	std::string _os;
};
/*具体产品类*/
class MacBook : public Computer {
public:
	using ptr = std::shared_ptr<MacBook>;
	MacBook() {}
	virtual void setOs() {
		_os = "Max Os X12";
	}
};
/*抽象建造者类：包含创建⼀个产品对象的各个部件的抽象接⼝*/
class Builder {
public:
	using ptr = std::shared_ptr<Builder>;
	virtual void buildBoard(const std::string& board) = 0;
	virtual void buildDisplay(const std::string& display) = 0;
	virtual void buildOs() = 0;
	virtual Computer::ptr build() = 0;
};
/*具体产品的具体建造者类：实现抽象接⼝，构建和组装各个部件*/
class MacBookBuilder : public Builder {
public:
	using ptr = std::shared_ptr<MacBookBuilder>;
	MacBookBuilder() : _computer(new MacBook()) {}
	virtual void buildBoard(const std::string& board) {
		_computer->setBoard(board);
	}
	virtual void buildDisplay(const std::string& display) {
		_computer->setDisplay(display);
	}
	virtual void buildOs() {
		_computer->setOs();
	}
	virtual Computer::ptr build() {
		return _computer;
	}
private:
	Computer::ptr _computer;
};
/*指挥者类，提供给调⽤者使⽤，通过指挥者来构造复杂产品*/
class Director {
public:
	Director(Builder* builder) :_builder(builder) {}
	void construct(const std::string& board, const std::string& display) {
		_builder->buildBoard(board);
		_builder->buildDisplay(display);
		_builder->buildOs();
	}
private:
	Builder::ptr _builder;
};
int main()
{
	Builder* buidler = new MacBookBuilder();
	std::unique_ptr<Director> pd(new Director(buidler));
	pd->construct("英特尔主板", "VOC显⽰器");
	Computer::ptr computer = buidler->build();
	std::cout << computer->toString();
	return 0;
}
```

# 小结
很明显，各个工厂模式和最后的建造者模式各有优劣：

+ 简单的场景用简单工厂就很好
+ 需要形成产品族时，抽象工厂模式的结构就很清晰
+ 需要很强的扩展性时，建造者模式就很符合要求

所以各种设计模式都应当学习，理解其最适用的应用场景