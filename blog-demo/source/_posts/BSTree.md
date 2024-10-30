---
title: 【数据结构】一步到胃，键值对版二叉搜索树
date: 2024-07-27 08:14:12
tags: 数据结构 二叉树 搜索树
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409230835315.png
---
# 什么是二叉搜索树
二叉搜索树的`定义`是一颗二叉树的所有节点满足:`根的左右孩子存在时，满足 左孩子 < 根 < 右孩子`

`递归定义`则是:
1. `左子树的根` < `根` < `右子树的根`
2. `左子树是二叉搜索树`,`右子树是二叉搜索树`

## 写一个验证二叉搜索树的函数 ##
[Leetecode题目链接🔗](https://leetcode.cn/problems/validate-binary-search-tree/)

# 封装一个二叉树类

## 文件布置
+ `BSTree.h`用于声明和实现`BSTree`类
+ `test.cpp`用于测试

### 头文件
`BSTree.h`
```C++
#include <iostream>
#include <utility>

```
`test.cpp`
```C++
#include "BSTree.h"
```
## 命名空间
```C++
namespace key_value
```
这里使用`key_value`作为命名空间，表示这是键值表示的搜索二叉树

## 定义节点
二叉树的节点用于储存`键值对`和`左右指针`，并提供`默认构造函数`,使用`初始化列表`初始化成员变量

```C++
template<class K,class V>
class BSTNode
{
public:
    BSTNode(const K& key = K(), const V& value = V())
        :_left(nullptr)
        , _right(nullptr)
        , _key(key)
        , _value(value)
    {}
    BSTNode<K, V>* _left;//指向左子树
    BSTNode<K, V>* _right;//指向右子树
    K _key;//储存键
    V _value;//储存值
};
```

## 封装二叉搜索树
`Binary Search Tree`,这里用简称`BSTree`封装

### 基本架构
鉴于该类的接口基本依赖于成员变量，所以先组织好成员变量

作为一颗`二叉树类`,成员变量仅需一个`指向根的指`针即可

再次之前先用`typedef`定义一个`节点类`出来用于简化代码

最后提供一个`默认构造函数`将`_root`初始化为`nullptr`

```C++
	template<class K,class V>
	class BSTree
	{
		typedef BSTNode<K, V> Node;//使用typedef简化代码
	public:
		BSTree() :_root(nullptr) {}//提供默认构造函数
	private:
		Node* _root;//指向根节点的指针作为成员变量
	};
```

### `insert`函数
准备好后第一个接口就是`insert`,用于构建搜索二叉树

这里需要考虑的情况有

+ 空树时的插入
+ 插入的`key`已存在
+ 一般情况下成功的插入

```C++
public:
bool insert(const K& key, const V& value)
{
    if (_root == nullptr)//空树
    {
        _root = new Node(key, value);
        return true;
    }

    Node* cur = _root;
    Node* parent = nullptr;
    while (cur)
    {
        if (key < cur->_key)//key比当前节点小，往左子树走
        {
            parent = cur;
            cur = cur->_left;
        }
        else if (key > cur->_key)//往右子树走
        {
            parent = cur;
            cur = cur->_right;
        }
        else
        {
            return false;//key已存在，插入失败
        }
    }
    //此时cur为nullptr, parent为cur的结点
    if (key < parent->_key)parent->_left = new Node(key, value);
    else parent->_right = new Node(key, value);
    return true;//成功插入
}
```

### `in_order`函数
使用此函数`前序遍历打印`二叉树来验证其满足`搜索树`的性质

这里使用递归打印,所以要借助`_in_order`子函数

```C++
public:
    void in_order()
    {
        _in_order(_root);//传入根结点
        std::cout << std::endl;
    }
protected:
    void _in_order(Node* root)
    {
        if (root == nullptr) return;
        _in_order(root->_left);//先访问左结点
        std::cout << root->_value << " " << std::endl;//再访问根结点
        _in_order(root->_right);//最后访问右结点
    }
```

然后写一段测试代码测试性质

> test.cpp
```C++
#include "BSTree.h"
#include <vector>
using namespace std;
int main()
{
	vector<int> arr({ 2,3,1,5,4,6,8,7 });//准备待插入的键值对
	key_value::BSTree<int, int> bst;
	for (int i = 0; i < arr.size(); ++i)
	{
		bst.insert(arr[i],arr[i]);//这里使键值相同，方便观察
	}

	bst.in_order();//测试

	return 0;
}
```

### `find`函数
可以用`find`函数查找对应`key`的结点。

同时观察可知，控制`cur`和`parent`的移动的代码段和前面的函数很像，所以给`find`函数分出来一个`_find`子函数，并使它返回`pair<Node*,Node*>`，将这两个指针返回利于其它函数对`_find`的回调

同时为了简化代码，继续用`typedef`封装类型

```C++
public:
    typedef std::pair< Node*, Node* > PNN;//简化代码
    bool find(const K& key)
    {
        return _find(key).first != nullptr;//检查是否找到
    }
protected:

    PNN _find(const K& key)//返回PNN用于简化其它接口
    {
        if (_root == nullptr) return {nullptr,nullptr};
        Node* cur = _root;
        Node* parent = nullptr;
        while (cur)
        {
            if (key < cur->_key)//key比当前节点小，往左子树走
            {
                parent = cur;
                cur = cur->_left;
            }
            else if (key > cur->_key)//往右子树走
            {
                parent = cur;
                cur = cur->_right;
            }
            else
            {
                //找到key
                return { cur,parent };
            }
        }
        //没找到key,cur为nullptr
        return { nullptr,parent };
    }
```

#### 重写`insert`函数
```C++
    bool insert(const K& key, const V& value)
    {
        if (_root == nullptr)//空树
        {
            _root = new Node(key, value);
            return true;
        }
        //====修改的部分====
        PNN pnn = _find(key);

        Node* cur = pnn.first;
        Node* parent = pnn.second;

        if (cur != nullptr)//该key已存在，插入失败
        {
            return false;
        }
        //================
        
        //此时cur为nullptr, parent为cur的结点
        if (key < parent->_key)parent->_left = new Node(key, value);
        else parent->_right = new Node(key, value);
        return true;//成功插入
    }
```

### `erase`函数
这里也可以复用`_find`来方便地删除结点

这里要考虑的情况有:
+ 树为空
+ 删除最后一个结点
+ 删除根节点
+ 左子树为空（包括叶子结点）
+ 右子树为空
+ 删除一般的结点

当树有`>=2`个结点，且要删除`非叶子`结点时，要考虑`结点替换`，否则二叉树会断掉，这里一般两种策略，取左子树的最右结点（最大结点），或取右子树的最左结点（最小结点）

```C++
public:
    bool erase(const K& key)
    {
        if (_root == nullptr)return false;//空树无法删除
        if (_root->_key == key && _root->_left == nullptr && _root->_right == nullptr)//删除最后一个结点
        {
            delete _root;
            _root = nullptr;
        }
        PNN pnn = _find(key);
        Node* cur = pnn.first;
        Node* parent = pnn.second;
        if (cur == nullptr) return false;//没找到该结点

        //下面的cur必不为空
        if(cur->_left == nullptr)
        {
            if (cur == _root)//为根节点时要替换根节点
            {
                Node* right = _root->_right;
                delete _root;
                _root = right;
            }
            //没有左子树，则直接移除结点,右子树替代原结点
            if (cur == parent->_left) parent->_left = cur->_right;
            else parent->_right = cur->_right;
            delete cur;
        }
        else if (cur->_right == nullptr)
        {
            if (cur == _root)
            {
                Node* left = _root->_left;
                delete _root;
                _root = left;
            }
            //直接过继左子树
            if (cur == parent->_left)parent->_left = cur->_left;
            else parent->_right = cur->_left;
        }
        else 
        {
            //找到左子树的最右结点
            Node* _cur = cur->_left;
            Node* _parent = cur;
            while (_cur->_right != nullptr)
            {
                _parent = _cur;
                _cur = _cur->_right;
            }
            //移走结点
            if (_cur == _parent->_left) _parent->_left = nullptr;
            else _parent->_right = nullptr;
            //获取cur的左右结点,必须再移走结点后，否则可能出现指向自己的结点
            _cur->_left = cur->_left;
            _cur->_right = cur->_right;
            //代替cur的位置
            if (cur == _root)//删除根结点时
            {
                delete _root;
                _root = _cur;
            }
            else//删除普通结点时
            {
                if (cur == parent->_left) parent->_left = _cur;
                else parent->_right = _cur;
                delete cur;
            }

        }
        return true;
    }
```

### 拷贝构造
利用二叉树的性质，可以再构建个`copy`子函数来递归拷贝

```C++
public:
    BSTree(const BSTree<K>& t)
    {
        _root = Copy(t._root);
    }
protected:
    Node* copy(Node* root)
    {
        if (root == nullptr) return nullptr;//递归出口
        Node* pnode = new Node(root->_key, root->_value);//复制结点
        pnode->_left = copy(root->_left);
        pnode->_right = copy(root->_right);
        return pnode;
    }
```
### 析构函数
这里也用`destroy`子函数来递归地后序遍历依次删除各个结点
```C++
public:
    ~BSTree()
    {
        destroy(_root);
    }
protected:
    void destroy(Node* root)
    {
        if (root == nullptr) return;
        destroy(root->_left);
        destroy(root->_right);
        delete root;
    }
```

至此，一个基本的二叉搜索树已封装完成

实现的功能有

+ 构建二叉搜索树
+ 拷贝复制二叉树
+ 按`key`值查找
+ 按`key`值删除

### 完整代码
```C++
#pragma once
#include <iostream>
#include <utility>

namespace key_value
{
	template<class K,class V>
	class BSTNode
	{
	public:
		BSTNode(const K& key = K(), const V& value = V())
			:_left(nullptr)
			, _right(nullptr)
			, _key(key)
			, _value(value)
		{}
		BSTNode<K, V>* _left;//指向左子树
		BSTNode<K, V>* _right;//指向右子树
		K _key;//储存键
		V _value;//储存值
	};

	template<class K,class V>
	class BSTree
	{
		typedef BSTNode<K, V> Node;//使用typedef简化代码
	public:
		BSTree() :_root(nullptr) {}//提供默认构造函数

		BSTree(const BSTree<K, V>& bst)
		{
			_root = copy(bst._root);
		}

		~BSTree()
		{
			destroy(_root);
		}

		bool insert(const K& key, const V& value)
		{
			if (_root == nullptr)//空树
			{
				_root = new Node(key, value);
				return true;
			}
			//====修改的部分====
			PNN pnn = _find(key);

			Node* cur = pnn.first;
			Node* parent = pnn.second;

			if (cur != nullptr)//该key已存在，插入失败
			{
				return false;
			}
			//================
			
			//此时cur为nullptr, parent为cur的结点
			if (key < parent->_key)parent->_left = new Node(key, value);
			else parent->_right = new Node(key, value);
			return true;//成功插入
		}

		void in_order()
		{
			_in_order(_root);//传入根结点
			std::cout << std::endl;
		}
	protected:
		void _in_order(Node* root)
		{
			if (root == nullptr) return;
			_in_order(root->_left);//先访问左结点
			std::cout << root->_value << " ";//再访问根结点
			_in_order(root->_right);//最后访问右结点
		}


	public:
		typedef std::pair< Node*, Node* > PNN;//简化代码
		bool find(const K& key)
		{
			return _find(key).first != nullptr;//检查是否找到
		}
	protected:
	
		PNN _find(const K& key)//返回PNN用于简化其它接口
		{
			if (_root == nullptr) return {nullptr,nullptr};
			Node* cur = _root;
			Node* parent = nullptr;
			while (cur)
			{
				if (key < cur->_key)//key比当前节点小，往左子树走
				{
					parent = cur;
					cur = cur->_left;
				}
				else if (key > cur->_key)//往右子树走
				{
					parent = cur;
					cur = cur->_right;
				}
				else
				{
					//找到key
					return { cur,parent };
				}
			}
			//没找到key,cur为nullptr
			return { nullptr,parent };
		}
	public:

		bool erase(const K& key)
		{
			if (_root == nullptr)return false;//空树无法删除
			if (_root->_key == key && _root->_left == nullptr && _root->_right == nullptr)//删除最后一个结点
			{
				delete _root;
				_root = nullptr;
			}
			PNN pnn = _find(key);
			Node* cur = pnn.first;
			Node* parent = pnn.second;
			if (cur == nullptr) return false;//没找到该结点

			//下面的cur必不为空
			if(cur->_left == nullptr)
			{
				if (cur == _root)//为根节点时要替换根节点
				{
					Node* right = _root->_right;
					delete _root;
					_root = right;
				}
				//没有左子树，则直接移除结点,右子树替代原结点
				if (cur == parent->_left) parent->_left = cur->_right;
				else parent->_right = cur->_right;
				delete cur;
			}
			else if (cur->_right == nullptr)
			{
				if (cur == _root)
				{
					Node* left = _root->_left;
					delete _root;
					_root = left;
				}
				//直接过继左子树
				if (cur == parent->_left)parent->_left = cur->_left;
				else parent->_right = cur->_left;
			}
			else 
			{
				//找到左子树的最右结点
				Node* _cur = cur->_left;
				Node* _parent = cur;
				while (_cur->_right != nullptr)
				{
					_parent = _cur;
					_cur = _cur->_right;
				}
				//移走结点
				if (_cur == _parent->_left) _parent->_left = nullptr;
				else _parent->_right = nullptr;
				//获取cur的左右结点,必须再移走结点后，否则可能出现指向自己的结点
				_cur->_left = cur->_left;
				_cur->_right = cur->_right;
				//代替cur的位置
				if (cur == _root)//删除根结点时
				{
					delete _root;
					_root = _cur;
				}
				else//删除普通结点时
				{
					if (cur == parent->_left) parent->_left = _cur;
					else parent->_right = _cur;
					delete cur;
				}

			}
			return true;
		}


	protected:
		Node* copy(Node* root)
		{
			if (root == nullptr) return nullptr;//递归出口
			Node* pnode = new Node(root->_key, root->_value);//复制结点
			pnode->_left = copy(root->_left);
			pnode->_right = copy(root->_right);
			return pnode;
		}

		void destroy(Node* root)
		{
			if (root == nullptr) return;
			destroy(root->_left);
			destroy(root->_right);
			delete root;
		}
	private:
		Node* _root;//指向根节点的指针作为成员变量
	};
}
```
