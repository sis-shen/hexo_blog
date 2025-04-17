---
title: 设计模式的C++实现(7)——组合模式Composite
date: 2024-12-10 16:40:39
tags:
---

> **模式名称**: 组合模式
> **类型**: 结构型
> **问题-使用场景**: 可用于构建对象树这样的部分-整体层次结构，使用户对单个对象和组合对象的使用具有一致性
> **解决方案**: 使用`递归组合`的方式构建类
> **效果**: 使用户对单个对象和组合对象的使用具有一致性

# 样例引入

如下图，有过QT开发经验的朋友能够看出来，这是QT组件管理里的对象树，它是一种管理组件的数据结构，同时它也很好地体现了`组合模式`在实际应用中的作用。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412112143060.webp)

# 实现方式

我们可以通过`继承`和`聚合`配合使用的方式实现组合模式，就以模拟实现上图的`QWidget`为例，我们来设计一个自己`SWidget`使之能够达到类似的效果

我们设计的类图如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412121006981.webp)

可以看到，在类图中，`SObject`和`SWidget`既是**继承关系**，又有**组合关系**,这一结构特点使`SWidget`之间能够构成对象树，而`SLabel`也是`SObject`的子类，但由于没有聚合关系，所以`SLabel`在对象树中仅能作为"叶子节点"存在。

代码实现如下，我们成功构建了一颗三层的对象树。

```C++
#include <iostream>
#include <vector>

using namespace std;

class SObject
{
public:
	SObject(SObject* parent = nullptr)
		:_parent(parent)
	{}

	void setParent(SObject* parent)
	{
		_parent = parent;
	}

	virtual void showParent()
	{
		cout << "SOBject : parent " << _parent<< endl;
	}
public:
	SObject* _parent;
};

class SWidget: public SObject
{
public:
	SWidget(SObject* parent = nullptr)
		:SObject(parent)
	{}

	virtual void showParent()
	{
		cout << "SWidget : parent" << _parent << endl;
	}

	void addChild(SObject* child)
	{
		_childs.push_back(child);
	}
public:
	vector<SObject*>_childs;
};

class SLabel:public SObject
{
public:
	SLabel(SObject* parent = nullptr)
		:SObject(parent)
	{}

	virtual void showParent()
	{
		cout << "SLabel : parent" << _parent << endl;
	}
};

int main()
{
	SWidget* mainW = new SWidget;
	SWidget* subW = new SWidget(mainW);
	mainW->addChild(subW);

	SWidget* leafW1 = new SWidget(subW);
	SWidget* leafW2 = new SWidget(subW);
	subW->addChild(leafW1);
	subW->addChild(leafW2);

	SLabel* label = new SLabel(mainW);
	mainW->addChild(label);

	mainW->showParent();
	subW->showParent();
	label->showParent();

	leafW1->showParent();
	leafW2->showParent();
	return 0;
}
```
输出结果如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412121101496.webp)

# 抽象模型
根据上面的样例引入，我们可以抽象出`组合模式`的类图模型

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412121112730.webp)

根据此类图，我们常见的对象结构可以如下

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202412121118442.webp)

# 参与者
在这一设计模式下，参与者有:

+ Compoment
  + 所有组件的公共父类，使之构成继承关系，提供多态特性
  + 可发挥规定子类接口等继承关系中父类的功能
+ Leaf 
  + 在对象树中作为叶子节点对象
+ Composite
  + 作为容器储存子部件
  + 对象树节点的重要组成部分
+ Client
  + 对整颗对象树的操作者

# 适用性
根据先前的样例和抽象，我们已经可以总结出`组合模式`的适用情况

+ 希望对象能够组成 部分-整体层次结构 
+ 希望统一单个对象和容器/组合对象能够有较高的统一性

# 优缺点
组合模式能带来许多好处

+ 组合模式里可以嵌套组合模式，即相对于`基本对象`的`组合对象`也可以用于**被组合**
+ **简化客户代码** 因为客户可以一致地所使用组合和单个对象。
+ **高扩展性**，新定义的`Composite`类和`Leaf`类能够自动地并入已有的架构
+ **使设计更一般化** 各个组件的通用性会很高

但更一般化的设计也会有些缺陷

+ **难以限制组合中的组件** ,因为多态的缘故，在使用`Composite`时，无法依赖`类型检查系统`限制`Composite`里哪些组件能放，哪些不能。必须付出额外的努力才行

# 实现要点
我们在实现`Composite`时压迫考虑以下几个问题

+ `显式的父部件引用` 正如同例子里的，我们很自然地使用了`父部件的引用`，即里面的`parent`指针，当然也可以是其它形式。这样的结构组成的树与一般的存储子类的树可能在边的方向上会有所差别。同时，父部件的引用也支持`Chain of Responsibility`模式。
+ `共享组件` 共享组件可以减少内存消耗。所谓的共享是指在对象树中，同一个组件被多个父部件共享（根据边的方向是子部件指向父部件，准确来说是一个子部件有多个父部件）。然而这种一个子部件有多个父部件的结构中，如果有请求需要由子部件向父部件传递时，会出现多义性问题。在往后的`Flyweight`模式中我们将解决这个问题
+ 最大化`Compoment`接口	为了使`Composite`类和`Leaf`类的操作有更高的统一性，我们应使`Compoment`类为两种子类定义更多的公共操作。
+ 声明管理子部件的操作	管理子部件用的`Add`和`Move`这种管理子部件的操作，在实际实现中有一个位置问题:在类层次结构的哪一层声明，这将涉及到`安全性`和`透明性`之间的权衡问题。
  + `透明性` :在类层次结构的根部(`Compoment`)定义管理接口的方法具有良好的透明性，因为它可以使用户一致地使用所有组件。但是这会造成安全性问题，因为有些操作对一些类是无意义的，例如对`Leaf`对象调用增加或删除子部件的操作
  + `安全性` :在组合对象`Composite`中定义管理子部件的方法具有良好的安全性。因为这些方法在`Leaf`中是未定义的。但是这牺牲了`透明性`
  + 出于使用这一设计模式的目的和使用场景，我们**更倾向于强调透明性**
+ `是否存储子类指针`：在基类中规定存放子类指针时，对`叶节点`来说会导致空间浪费(开辟了空间却只能存储空指针)。这种做法只有当结构中叶子节点数目相对较少时才值得
+ `子部件排序`：在许多设计中`Composite子部件的顺序`是**严格而重要的**。例如在`语法分析树`结构中使用组合模式时，`Composite子部件的顺序`必须能够正确反映程序结构。
+ `性能优化：高速缓冲贮存`如果需要对组合进行频繁的遍历或查找,可以引入缓冲存储机制
+ `删除Component`:在没有垃圾回收机制的语言中（例如C++），需要注意及时删除声明的对象防止内存泄漏
+ `存储组件的数据结构`：对于一组组件对象，实际存储它们的数据结构可以有多种选择，包括`连接列表`、`数`、`数组`、`哈希表`等

# 常见应用
Composite模式是一种常用的设计模式,许多地方都能体现它的思想

1. 文件系统：文件系统中的目录和文件可以被视为组合对象和个体对象。目录可以包含文件和子目录，而文件则是最基本的个体对象。
2. 图形用户界面：图形用户界面中的容器控件（如窗口、面板等）和基本控件（如按钮、文本框等）可以被视为组合对象和个体对象。
3. XML 和 HTML 文档：XML 和 HTML 文档中的元素可以被视为组合对象和个体对象。元素可以包含其他元素和文本内容。
4. 数据库：数据库中的表和记录可以被视为组合对象和个体对象。表可以包含多个记录，而记录则是最基本的个体对象。

# 相关设计模式
+ `Decorator模式`：常与Composite模式一起使用。当它们一起使用时，它们通常有一个公共的父类
+ `Flyweight模式`：能够让用户共享组件，而**不再能引用**他们的父部件
+ `Iterator模式`：提供遍历所有组件的方式
+ `Visitor模式`： 将本来应该分布在Composite和Leaf类中的操作和行为局部化。