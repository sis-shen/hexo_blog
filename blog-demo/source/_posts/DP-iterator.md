---
title: 设计模式的C++实现(4)——迭代器Iterator
date: 2024-10-15 20:00:25
tags:
---

> **模式名称**: 迭代器-Iterator
> **类型**: 行为模式
> **问题-使用场景**: 提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示 
> **解决方案**： 将对列表（对象集合）的访问和遍历从具体对象中分离出来，并放入一个迭代器对象中，由它负责实现访问和遍历功能
> **效果**： 提供了统一的遍历成员的方法，降低了用户的使用难度，提高了代码的封装性和可扩展性。代价是增加了代码复杂性，增加了更多的类，提高了代码的设计难度

*注：本文的迭代器风格更偏向`STL库`中的迭代器，而不是《设计模式》中的抽象迭代器*

**灵感来源**：某种意义上将，`Iterator`要做的事就是模仿数组中的`指针`,指针可以前后移动，*方便地*遍历数组，还可以用指针访问数组元素。我们希望把这种指针特性也用在其它数据结构上(**特指组织管理多个对象的聚合体**)，但可惜的是原生指针的这种*方便*依赖于数组的地址是连续的。因此为了实现遍历和访问功能，我们需要把这些功能封装在`Iterator`类中，让它实例化出的对象来行使`数组指针`一样的行为。

# 类层次设计
我们先来设计一个抽象迭代器类来规定迭代器应有哪些行为：

```C++
template<typename T, typename Ptr, typename Ref>
class __base_iterator
{
public:
	__base_iterator() = default;

	__base_iterator(const __base_iterator<T, T*, T&>& it){};//要支持普通迭代器构造const迭代器

	virtual __base_iterator<T, Ptr, Ref>& operator++() = 0;//向前遍历

	//不是所有容器都支持反向迭代器，所以这里根据具体情况声明函数
	//__base_iterator<T>& operator--() = 0; 

	virtual Ptr operator->() = 0;//重载->操作，使其部分实现指针功能

	virtual Ref operator*() = 0;//重载解引用，部分实现指针功能

	virtual bool operator!=(const __base_iterator<T,Ptr,Ref>&)const = 0;//应当支持迭代器的不等比较
};

template<typename ValueType>
struct ListNode
{
	ListNode(const ValueType& val = ValueType()) :_val(val), _next(nullptr), _prev(nullptr) {}
	ValueType _val;
	ListNode<ValueType>* _next;
	ListNode<ValueType>* _prev;
};
```
接下来我们逐步解释一下为什么要这么设计。

## 三个模板参数
*为什么要用三个模板参数，一个不行吗，看起来三个更复杂了？*

因为迭代器分`普通迭代器`和`const迭代器`，二者权限不同，在重载`operator->()`和`operator*()`时的返回值类型不同，把两种迭代器分开来声明，代码冗余太多了。为了使代码简洁，并提高代码复用性，这里选择了使用三个模板类的方式，大大提高了代码复用性。

## 操作符重载
我们规定了哪些操作符需要重载，来更好地模仿链表指针

+ `operator++()`重载前置++，模仿数组指针的遍历功能
+ `operator->()`重载->,模仿指针的访问成员功能
+ `operator*()`重载*,模仿指针的访问内存功能

### 代码示例

先准备一个勉强能用的链表类（没有提供迭代器）

```C++
#include <iostream>

template<typename ValueType>
struct ListNode
{
	ListNode(const ValueType& val = ValueType()) :_val(val), _next(nullptr), _prev(nullptr) {}
	ValueType _val;
	ListNode<ValueType>* _next;
	ListNode<ValueType>* _prev;
};

template<typename Type>
class List
{
	typedef ListNode<Type> Node;//简化代码
public:
	List() :_head(new Node) { _head->_next = _head; _head->_prev = _head; }

	void push_front(const Type& val)
	{
		Node* newnode = new Node(val);
		Node* prev = _head;
		Node* next = _head->_next;

		prev->_next = newnode;
		next->_prev = newnode;

		newnode->_prev = prev;
		newnode->_next = next;
	}

	void push_back(const Type& val)
	{
		Node* newnode = new Node(val);
		Node* next = _head;
		Node* prev = _head->_prev;

		prev->_next = newnode;
		next->_prev = newnode;

		newnode->_prev = prev;
		newnode->_next = next;
	}

	void inorder()
	{
		Node* cur = _head->_next;
		while (cur != _head)
		{
			std::cout << cur->_val << "->";
			cur = cur->_next;
		}
		std::cout << "_head\n";
	}
private:
	Node* _head;
};
```

然后我们来继承一下`__base_iterator`抽象类并实现相关接口

```C++
template<typename T, typename Ptr, typename Ref>
class __base_iterator
{
public:
	__base_iterator() = default;

	__base_iterator(const __base_iterator<T, T*, T&>& it){};//要支持普通迭代器构造const迭代器

	virtual __base_iterator<T, Ptr, Ref>& operator++() = 0;//向前遍历

	//不是所有容器都支持反向迭代器，所以这里根据具体情况声明函数
	//__base_iterator<T>& operator--() = 0; 

	virtual Ptr operator->() = 0;//重载->操作，使其部分实现指针功能

	virtual Ref operator*() = 0;//重载解引用，部分实现指针功能

	virtual bool operator!=(const __base_iterator<T,Ptr,Ref>&)const = 0;//应当支持迭代器的不等比较
};

template<typename ValueType>
struct ListNode
{
	ListNode(const ValueType& val = ValueType()) :_val(val), _next(nullptr), _prev(nullptr) {}
	ValueType _val;
	ListNode<ValueType>* _next;
	ListNode<ValueType>* _prev;
};


template<typename T,typename Ptr,typename Ref>
class __list_iterator :public __base_iterator<T,Ptr,Ref>
{
	typedef __list_iterator<T, Ptr, Ref> Self;
	typedef __list_iterator<T, T*, T&> iterator;
public:
	__list_iterator(ListNode<T>* ptr) :_ptr(ptr) {}

	__list_iterator(const iterator& it) :_ptr(it._ptr) {}

	__base_iterator<T, Ptr, Ref>& operator++() override
	{
		_ptr = _ptr->_next;
		return *this;
	}

	Ptr operator->() override
	{
		return &(_ptr->_val);
	}

	Ref operator*() override
	{
		return _ptr->_val;
	}

	bool operator!=(const __base_iterator<T,Ptr,Ref>&it)const 
	{
		const Self* pit = dynamic_cast<const Self*>(&it);//指针的(父类子类间)动态转换
		if (pit)
		{
			return _ptr != pit->_ptr;
		}
		return true;//类型不匹配,认为不相等
	}

private:
	ListNode<T>* _ptr;
};

```
**实现难点分析**:
+ `operator++()`使用了在多态中，父类引用作为返回值时，函数的重载可以返回子类的引用
+ `operator!=()`中因为父类对接口的规定，传参使用了父类的引用，所以要用`dynamic_cast`做动态的指针转换。

以上就是对`List`类的迭代器的封装，很明显这是一种`外部迭代器`：**由客户控制迭代**。所以为了能够让客户操作，`List`类应当提供相关的接口。

不同于书本中的简单工厂设计模式，这里参照的是`STL`的接口标准，即提供`begin()` `end()` `begin() const` `end() const`创建`iterator`或`const iterator`对象

```C++
template<typename Type>
class List
{
	typedef ListNode<Type> Node;//简化代码

public:
	typedef __list_iterator<Type, Type*, Type&> iterator;
	typedef __list_iterator<Type, const Type*, const Type&> const_iterator;
public:
	List() :_head(new Node) { _head->_next = _head; _head->_prev = _head; }

	iterator begin()
	{
		return _head->_next;
	}

	iterator end()
	{
		return _head;
	}

	const_iterator begin() const
	{
		return _head->_next;
	}

	const_iterator end() const
	{
		return _head;
	}

	//..........下面代码略
};
```

# 功能总结与拓展

## 小结
经过上面的初步设计，我们的迭代器已经能支持如下功能了：

+ 支持按程序员自定义的方式遍历一个`聚合体`
+ 简化了`聚合体`的接口，`聚合体`本身不需要提供逐步遍历或自动遍历的接口了
+ 在**同一个**聚合体上，可以同时进行多个遍历。*实例化多个迭代器对象即可*

## 拓展
作为一种设计模式，上面的功能还不能算是一种完整的设计模式，`Iterator`的可扩展性和功能不应该仅局限于简单的遍历

### 由谁控制迭代
关于这个问题，迭代器分为了两种
+ `外部迭代器`：由客户来控制迭代
+ `内部迭代器`：由迭代器自身来控制迭代

**外部迭代器比内部迭代器更灵活，对客户来说也更易于控制**。尽管`C++11`以后的`lambda表达式`支持了`匿名函数`和`闭包`，内部迭代器可能比书本中描述的要好用一些，内部迭代器的弱点依然显著。*为数不多的好处之一就是它们已经定义好了迭代逻辑，客户用起来比较简单*

### 由谁定义遍历/迭代算法
实际上迭代器不一定是唯一定义遍历算法的地方。

可以由**聚合体本身定义遍历算法**，由迭代器负责调用算法和`储存迭代`状态。这样的迭代器我们称其为`指针Cursor`，因为它仅仅用于指示当前位置。

当然，由**迭代器实现遍历算法**才是最常用的。这样可以提供很大的灵活性。比如在相同的聚合体上提供不同的遍历算法，或者在不同的聚合体上重用相同的算法。**但代价是**,迭代器可能需要访问聚合体的私有变量，这在一定程度上破坏了封装性

### 迭代器的健壮性和迭代器失效问题
在使用迭代器遍历时，**插入或是删除操作都是十分危险的**，因为这很可能使迭代器`两次访问同一个元素`或`漏掉某一个元素`，这就造成了所谓的`迭代器失效问题`。

一种简单的解决方法是在开始遍历时，拷贝一整个聚合体用于安全遍历，但这样实在是太浪费资源了

或者参照`STL`的做法，让聚合体提供的插入或删除的接口能够**返回新的迭代器对象**,这样旧的迭代器失效了也没关系，可以继续使用更新后的**新的迭代器**继续遍历了。比如`std::vector`中的`erase()`函数的返回值描述如下

> ### Return Value
> An iterator pointing to the new location of the element that followed the last element erased by the function call. This is the container end if the operation erased the last element in the sequence.
> 返回一个指向被删除元素的后继的迭代器。如果被删除的是最后一个迭代器，则返回表示结尾的迭代器

### 多种迭代器
迭代器也不一定只有一种，可以按不同的遍历方式，声明并实现不同的迭代器，最经典的就是`普通迭代器`与`const 迭代器`,`正向迭代器`与`反向迭代器`。它们都是不同的类，会实例化出不同的迭代器对象。在这里对类的继承体系并不关心

### 使用多态的迭代器
就像上面的示例代码，实际的迭代器类都继承自一个抽象父类`__base_iterator`，都继承了父类的抽象接口，显然这已经满足了多态的使用条件。

然而**使用多态是有代价的**,客户可能需要通过`工厂方法设计模式`获取迭代器对象，而且还需要客户`手动释放迭代器对象`，然而客户常常会漏掉或忘记对象的释放，尤其是`堆区上的对象`。当然，现在有`shared_ptr`智能指针来减少这种情况的出现。

### 空迭代器
一个空的迭代器不能实现遍历功能，但是它可以用来标识一次遍历的结尾。这种功能在单向链表，树的遍历就尤为有用，当遍历到`nullptr`时，返回一个空迭代器即可

# 应用
+ C++中的STL库中的容器都支持迭代器，使其能够方便地遍历
+ MySQL提供的C语言接口库中提供的`mysql_fetch_row()`函数，每一次调用都会获取下一行，行为就和迭代器很像
+ 还有更多聚合体可以使用迭代器设计模式，比如可视化控件类的遍历等