---
title: 通过设计list类深入理解iterator迭代器
date: 2024-04-26 16:10:11
tags: C++ 类和对象
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Date.jpg
---

前置博客:**[从构建一个Date类入门C++类与对象](https://www.supdriver.top/2023/12/07/cpp-class/ '点击跳转')**

>下面先迅速地搓一个`list`类
```C++
template <class T>//先用模板创建一个节点类
struct ListNode
{
	T _val;
	ListNode<T>* _next;
	ListNode<T>* _prev;

	//提供全缺省的默认构造函数
	ListNode(const T& val = T()) :_val(val), _next(nullptr), _prev(nullptr) {}
};

//用ListNode构造list类

template <class T>
class list
{
	typedef ListNode<T> Node;//用typedef简化代码
public:
	list()//默认构造函数
	{
		_head = new Node;
		//维护两个指针
		_head->_next = _head;
		_head->_prev = _head;
	}

	void push_front(const T& val)//头插
	{
		Node* newnode = new Node(val);
		Node* next = _head->_next;//额外的指针，简化代码

		_head->_next = newnode;
		newnode->_prev = _head;
		newnode->_next = next;
		next->_prev = newnode;
	}

	void pop_front()//尾插
	{
		if (empty()) return;

		Node* cur = _head->_next;
		
		_head->_next = cur->_next;
		cur->_next->_prev = _head;
		delete cur;
	}

	bool empty() //判空
	{
		return _head->_next == _head;
	}

	//剩余代码自行补全
    //......
private:
	Node* _head;
};

```

# list的迭代器 #

不同于`vector`底层的数据在内存中**连续**存储,可以用`原生指针`充当迭代器,例如`typedef T* iterator`

`list`的底层是链表,在内存中**分散***存储，是**不能**用`原生指针`来**连续**访问的,所以为了解决这一复杂问题，
需要自己写一个`iterator`类

## 普通迭代器 #

```C++
template<class T>//迭代器也得用模板
struct __list_iterator
{
	typedef __list_iterator<T> Self;//简化代码
	typedef ListNode<T> Node;
	Node* _node;

	__list_iterator(Node* node) :_node(node) {}

	Self& operator++()//重载operator++
	{
		_node = _node->_next;
		return *this;
	}

	bool operator!=(const Self& it) const //重载!==,比较操作符记得加const
	{
		return _node != it._node;
	}

	T& operator*()//重载 *
	{
		return _node->_val;
	}
};

```
*`list`类中添加如下代码*
```C++
public:
	typedef __list_iterator<T> iterator;

    iterator begin()
	{
		return iterator(_head->_next);
	}

	iterator end()
	{
		return iterator(_head);
	}
```

通过如上修改,`list`已经支持`普通迭代器`,并且非`const`修饰的`list`已经支持`范围for`了

>测试代码如下
```C++
int main()
{
	list<int> lst;
	lst.push_front(5);
	lst.push_front(4);
	lst.push_front(3);
	lst.push_front(2);
	lst.push_front(1);

	for (auto e : lst)
	{
		cout << e << " ";
	}
	return 0;
}
```

## const迭代器
`const list`要能提供`const_iterator`，因此我们还要写一个`const_iterator`类...吗？

其实**并不用**，要利用好C++中的`模板语法`来大大提高代码的复用性,尤其像`iterator`和`const_iterator`这种差别不大的类,没必要每个都单独写一段代码

为此我们的`__list_iterator`只需要能用`模板`解决好二者的差异即可。而目前最大的问题是什么？是`operator*()`的返回值问题,一个是返回`T&`,另一个是`const T&`,其他的成员函数则基本没差别,所以不妨扩充一下`模板参数`,添加一个`Ref`类。

>有变化的代码如下
```C++
template<class T,class Ref>//增加一个Ref参数
struct __list_iterator
{
	typedef __list_iterator<T,Ref> Self;//Self这里的原类也要加
	typedef ListNode<T> Node;
	Node* _node;

	__list_iterator(Node* node) :_node(node) {}

	Ref operator*()//直接返回Ref类
	{
		return _node->_val;
	}
};
```
*`list`类也有相应变化*
```C++
public:
	typedef __list_iterator<T,T&> iterator;//普通迭代器
	typedef __list_iterator<T,const T&> const_iterator;//const迭代器

    const_iterator begin() const //针对const指针的
	{
		return const_iterator(_head->_next);
	}

	const_iterator end() const
	{
		return const_iterator(_head);
	}
```

这样通过增加一个`Ref`模板参数,完成了对`iterator`和`const_iterator`的代码级统一(*当然模板实例化出来是不一样的*)

但别忘了迭代器还要提供`->`操作符的重载,而`operator->()`函数要返回不同的指针，所以我们如法炮制，再增加一个`Ptr`模板参数

>有变化的代码如下
```C++
template<class T,class Ref,class Ptr>//增加一个Ptr参数
struct __list_iterator
{
	typedef __list_iterator<T,Ref,Ptr> Self;//Self相应更改

	Ptr operator->()//重载->
	{
		return &(_node->_val);
	}
};

```
*`list`类也有相应变化*
```C++
public:
	typedef __list_iterator<T,T&,T*> iterator;//普通迭代器
	typedef __list_iterator<T,const T&,const T*> const_iterator;//const迭代器
```

至此，`list`和`__list_iterator`的基本功能已基本完成，本篇的重点`__list_iterator`主要解决了两点问题

-为了应对`list`的迭代器的复杂性，单独为其构建一个`__list_iterator`类，并提供一系列的操作符重载
-为了提高代码的**复用性**,仅用一个`__list_iterator`类来`typedef`普通迭代器和`const`迭代器,我们增加了模板参数,最终模板变为`template<class T, class Ref, class Ptr>`

## 用普通迭代器构造const迭代器
有时候我们需要用普通迭代器构造`const`迭代器,于是可以给`__list_iterator`提供一个比较有意思的`构造函数`,
可以实现时而充当拷贝构造，时而充当满足上述的构造

>代码如下
```C++
typedef __list_iterator<T,Ref,Ptr> Self;//再展示一遍Self的代码，便于下文对比
typedef __list_iterator<T,T&,T*> iterator;//指定普通迭代器，并用typedef简化代码

__list_iterator(iterator it) :_node(it._node) {}

```
-当模板参数为`<T,T&,T*>`时，`Self`和`iterator`相同，上段代码中的构造函数相当于`拷贝构造`

-当模板参数为`<T,const T&,const T*>`时，`Self`和`iterator`不同,`Slef`是`const`迭代器,`iterator`**始终**是普通迭代器，这个构造函数便能用普通迭代器构造`const`迭代器

# 小结
通过构造一个`list`类，我们使用到了更复杂的迭代器，使用了带`3个模板参数`的`__list_iterator`类定义普通迭代器和`const`迭代器，学习了如何利用模板参数提高代码的复用性，如何提供额外的`构造函数`使`__list_iterator`支持用普通迭代器构造`const`迭代器