---
title: 手撕AVL树
date: 2024-08-12 10:55:47
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409230910780.png
---
# AVL树的概念

## 二叉搜索树的不足
二叉搜索树虽可以缩短查找的效率，但如果数据有序或接近有序二叉搜索树将**退化**为`单支树`，查找元素相当于在顺序表中搜索元素，效率低下。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/lajibstree2.png)

## AVL树的提出

两位俄罗斯的数学家*G.M.Adelson-Velskii*和*E.M.Landis*在1962年发明了一种解决上述问题的方法：
**当向二叉搜索树中插入新结点后，如果能保证每个结点的左右子树高度之差的绝对值不超过1(需要对树中的结点进行调整)**,即构建一颗`绝对的平衡搜索二叉树`，即可降低树的高度，从而减少平均搜索长度。

**定义**: 一棵AVL树或者是空树，或者是具有以下性质的二`叉搜索树`：
+ 它的左右子树都是AVL树
+ 左右子树高度之差(简称`平衡因子`)的绝对值不超过1(-1/0/1)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/aaaAVL80.png)

如果一棵二叉搜索树是高度平衡的，它就是AVL树。如果它有n个结点，其高度可保持在`O(log_2 n)`,搜索的时间复杂度`O(log_2 n)`

# 封装AVL树

## 头文件
```C++
#include <utility>
#include <cassert>
```

## 封装AVL树的节点
+ 这里采用`键值(KV)`类型的二叉树节点,使其泛用性更高
+ 并增加指向父节点的指针构造三叉链表，来**简化**对二叉树的调整，但代价是**维护成本**变高(多了一个指针要维护)。
+ 引入`平衡因子`:
  + `-1`表示左子树比右子树高`1`层  *左>右*
  + `0`表示左右子树等高           *左=右*
  + `1`表示右子树比左子树高`1`层   *左<右*

```C++
#include <utility>
#include <iostream>
#include <cassert>

template <class K,class V>
struct AVLTreeNode
{
	typedef AVLTreeNode<K, V> Node;//使用typedef简化代码
	Node* _left;                   
	Node* _right;
	Node* _parent;
	std::pair<K, V> _kv; //节点储存的键值对
	int _bf;// ballance factor 平衡因子

	//构造函数，但不提供无参的默认构造函数
	AVLTreeNode(const std::pair<K, V>& kv)
		:_left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _kv(kv)
		, _bf(0)
	{}
};
```

## 规定AVLTree类的框架
首先确定成员变量,这里仅用`_root`指向二叉树，具体的维护由`成员函数`完成

代码简化方面，使用
```C++
typedef AVLTreeNode<K, V> Node;
typedef std::pair<K,V> PKV;
```
简化代码

然后是准备实现的`成员函数`

**公共接口**
+ `Insert`插入节点
+ `Inorder`前序遍历打印节点，用于debug
+ `IsBalance`检测是否平衡
+ `Height`获取子树高度

**私有接口**
+ `RotateL`向左旋转子树，用于维护平衡
+ `RotateR`向右旋转子树，用于维护平衡
+ `RotateLR`向左再向右旋转子树，用于维护平衡
+ `RotateRL`向右再向左旋转子树，用于维护平衡

实现AVL树的核心就是实现这四个**旋转操作**,并再在`insert`中回调。所以`Insert`函数我们放到最后再实现

## 默认构造函数
```C++
template<class K, class V>
class AVLTree
{
	typedef AVLTreeNode<K, V> Node;
	typedef std::pair<K, V> PKV;
public:
	AVLTree() :_root(nullptr) {}//默认构造函数

private:
	Node* _root;
};
```

## Inorder函数
公共的`Inorder`接口为无参函数,内部调用另外定义的`_Inorder`子函数

```C++
public:
	AVLTree() :_root(nullptr) {}//默认构造函数

	void Inorder()
	{
		_Inorder(_root);
		std::cout<<"nullptr" << std::endl;
	}

protected:
	void _Inorder(Node* root)
	{
		if (root == nullptr) return;
		_Inorder(root->_left);
		std::cout << root->_kv.second << "->";
		_Inorder(root->_right);
	}
```

## Height函数
同上，使用`_Height`子函数

```C++
public:
	int Height()
	{
		return _Height(_root);
	}
private:
	int _Height(Node* root)
	{
		if (root == nullptr) return 0;

		int LHeight = _Height(root->_left);
		int RHeight = _Height(root->_right);
		if (LHeight > RHeight) return LHeight + 1;
		else return RHeight + 1; //将左右子树相等的情况合并在这里
	}
```

## IsBalance函数
同上，使用`_Balance`子函数

这里要较为严格，**可靠**地地判定平衡,而平衡因子由我们自己维护，作为判定的依据，并**不可靠**，相比之下,计算左右子树的高度差**更可靠**,所以这里回调`_Height`函数

```C++
public:
	bool IsBalance()
	{
		return _IsBalance(_root);
	}
private:
	bool _IsBalance(Node* root)
	{
		if (root == nullptr) return true;
		bool left = _IsBalance(root->_left);
		bool right = _IsBalance(root-> - _right);

		if (left == false || right == false) return false;//先判定左右子树

		int LHeight = _Height(root->_left);
		int RHeight = _Height(root->_right);

		if (LHeight - RHeight >= 2 || LHeight - RHeight <= -2)return false;//最后判定根
		else return true;
	}
```

## Insert函数
`Insert`函数主要按步骤实现以下功能

1. `插入节点`:先按照二叉搜索树的规则将节点插入到AVL树中
2. `维护平衡`:新节点插入后，AVL树的平衡性可能会遭到破坏，此时就需要更新平衡因子，并检测是否破坏了AVL树

### 插入节点
具体插入方式与`平衡搜索二叉树`基本相同,但是还要额外维护`_parent`指针
```C++
	bool Insert(const PKV& kv)
	{
		Cmp cmp;//仿函数实例化一个对象
		if (_root == nullptr)
		{
			_root = new Node(kv);
			return true;
		}
		Node* cur = _root;
		Node* parent = nullptr;
		//寻找插入位置
		while (cur)
		{
			if (cur->_kv.first == kv.first)
			{
				//该键下的节点已存在，发生冲突
				return false;
			}
			else if (cmp(cur->_kv.first, kv.first))//根比节点"大"
			{
				parent = cur;
				cur = cur->_left;
			}
			else //根比节点小
			{
				parent = cur;
				cur = cur->_right;
			}
		}
		
		//开始插入
		Node* node = new Node(kv);

		if (cmp(parent->_kv.first, kv.first))//应该插入左边
		{
			parent->_left = node;
			node->_parent = parent;
			parent->_bf -= 1;//_bf越小，左子树越高
		}
		else //插入右子树
		{
			parent->_right = node;
			node->_parent = parent;
			parent->_bf += 1;//_bf越大，右子树越高
		}
		//插入完成，准备开始维护平衡
		//.....
		
	}
```
### 维护平衡
插入节点的所有父级子树的高度都有可能受到其影响，所以要**一路向上递归**维护，直至某棵子树的高度不变,或者到达根节点

下面我们一一枚举所有情况,并逐一解决

+ 父节点`_bf == 0`
  + 右子树高度增加: `_bf = 1`,且父节点高度增加，需继续**向上调整**
  + 左子树高度增加: `_bf = -1`,且父节点高度增加，需继续**向上调整**
+ 父节点`_bf == -1`
  + 右子树高度增加: `_bf = 0`,父节点高度不变，停止调整
  + 左子树高度增加: `_bf = -2`,平衡被打破,需要**旋转调整**, *下文详细讨论*
+ 父节点`_bf == 1`
  + 右子树高度增加: `_bf = 2`,平衡被打破,需要**旋转调整**, *下文详细讨论*
  + 左子树高度增加: `_bf = 0`,父节点高度不变，停止调整

可以看到，需要调整调整的情况有两大方向，接下来详细介绍调整方法

#### `RotateR `:`_bf == -2`且子树的左子树超高

根据约定，`_bf == -2`表示左子树比右子树高`2`，所以我们要先通过向右旋转操作减少左子树高度

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409021916938.png)

如图所示,通过向右旋转，成功地降低了左子树的高度，而根据`左子树`的平衡因子，又可以分为以下两种情况

+ `-1`:旋转后平衡性得到维护，但整棵树的高度由`h+2`降低到了`h-1`,总高度`-1`,与原本的高度`+1`相抵消,停止调整
+ `0`:非法情况,不可能左右子树同时超高
+ `1`:不存在的情况，因为破坏了左子树超高的前提

以上分析可知，一次向右旋转已经可以解决**两种**情况了，所以我们着手实现`RotateR`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RotateR2.gif)

如上图所示，我们要对树里的父子关系作出调整，以此改变高度。

然后除了图中的内容，我们还要维护一系列`_bf`,指针之类的内容

```C++
private:
	void RotateR(Node* parent)
	{
		if (parent == nullptr) return;
		Node* child = parent->_left;

		//维护树的结构
		parent->_left = child->_right;//过继子树的右子树
		child->_right = parent;//重构树结构
		Node* grandParent = parent->_parent;//维护祖父节点
		child->_parent = grandParent;//维护child
		parent->_parent = child;
		if (grandParent)//如果不是根节点
		{
			//维护根节点的父节点的子树
			if (parent == grandParent->_left) grandParent->_left = child;
			else grandParent->_right = child;
		}
		
		//维护平衡因子
		if (child->_bf == -1)
		{
			child->_bf = 0;
			parent->_bf = 0;
		}
		else if (child->_bf == 0)
		{
			assert(false);//不存在的情况
		}
		else
		{
			assert(false);//需要多次旋转时不应该调用该函数
		}

		if (parent == _root)
		{
			_root = child;//更新根节点
		}
	}
```

#### `RotateL`:`_bf == 2`且子树的右子树超高
根据约定，`_bf == -2`表示右子树比左子树高`2`，所以我们要先通过向左旋转操作减少右子树高度

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409030810533.png)

如图所示,通过向左旋转，成功地降低了右子树的高度，而根据`右子树`的平衡因子，又可以分为以下三种情况

+ `-1`:旋转后平衡性得到维护，但整棵树的高度由`h+2`降低到了`h-1`,总高度`-1`,需继续**向上调整**
+ `0`:非法情况,不可能左右子树同时超高
+ `1`:不存在的情况，因为破坏了子树的右子树超高的前提

以上分析可知，一次向左旋转已经可以解决**两种**情况了，所以我们先实现`RotateL`
和`RotateL`一样，我们要对树里的父子关系作出调整，以此改变高度。

然后除了图中的内容，我们还要维护一系列`_bf`,指针之类的内容

```C++
private:
	void RotateL(Node* parent)
	{
		if (parent == nullptr) return;
		Node* child = parent->_right;
		//维护树结构
		parent->_right = child->_left;//过继子树
		child->_left = parent;//重构树结构
		Node* grandParent = parent->_parent;//维护祖父节点
		child->_parent = grandParent;//维护_parent
		parent->_parent = child;//维护_parent
		if (grandParent)//如果不是根节点
		{
			//维护祖父节点的子树
			if (parent == grandParent->_left) grandParent->_left = child;
			else grandParent->_right = child;
		}

		//维护平衡因子
		if (child->_bf == 1)
		{
			child->_bf = 0;
			parent->_bf = 0;
		}
		else if (child->_bf == 0)
		{
			assert(false);//不存在的情况
		}
		else
		{
			assert(false);//需要多次旋转时不应该调用该函数
		}
		if (parent == _root)
		{
			_root = child;//更新根节点
		}
	}
```
#### `RotateRL`:`_bf == -2`且子树的右子树超高

当出现该情况时，单纯的向右旋转并不顶用,效果如下图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RotateR_err.png)

所以说我们要先调整左子树的平衡因子(左子树向左旋转)，再最后进行向右旋转

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/ROtateLR_re.png)

如上图，通过两次旋转，完成了整体的平衡因子的维护

**高度变化**：整棵树的高度由`h+3`变成了`h+2`,高度减少了`1`,与原本的高度`+1`向抵消，停止调整

```C++
private:
	void RotateLR(Node* parent)
	{
		if (parent == nullptr) return;//错误情况
		Node* Lchild = parent->_left;
		if (Lchild == nullptr) return;//错误情况
		Node* Rchild = Lchild->_right;

		//先旋转左子树

		//维护树结构
		Lchild->_right = Rchild->_left;
		Rchild->_left = Lchild;
		Lchild->_parent = Rchild;
		parent->_left = Rchild;
		Rchild->_parent = parent;

		//维护_bf
		if (Lchild->_bf == 1)
		{
			Lchild->_bf = -1;
		}
		else
		{
			//_bf == -1 或 0都是非法的
			assert(false);
		}

		//再向右旋转
		Node* child = parent->_left;
		
		//维护树结构
		parent->_left = child->_right;
		child->_right = parent;

		Node* grandParent = parent->_parent;
		child->_parent = grandParent;
		parent->_parent = child;

		if (grandParent)//不是根节点
		{
			if (parent == grandParent->_left)grandParent->_left = child;
			else grandParent->_right = child;
		}

		//维护_bf
		parent->_bf = -1;
		child->_bf = 0;
		
		if (parent == _root)
		{
			_root = child;//更新根节点
		}
	}
```

#### `RotateLR`:`bf == 2`且子树的左子树超高
具体情况与上一个接口相同，一次旋转无法完成任务，需要先向右旋转右子树调节平衡因子,再向左旋转，完成平衡的维护

**高度变化**：整棵树的高度由`h+3`变成了`h+2`,高度减少了`1`


```C++
private:
	void RotateRL(Node* parent)
	{
		//先向右旋转子树
		Node* Rchild = parent->_right;
		Node* Lchild = Rchild->_left;

		//维护树结构
		Rchild->_left = Lchild->_right;
		Lchild->_right = Rchild;
		Rchild->_parent = Lchild;
		Lchild->_parent = parent;
		parent->_right = Lchild;

		//维护_bf
		if (Rchild->_bf == -1)
		{
			Rchild->_bf = 0;
		}
		else
		{
			assert(false);//非法的情况
		}

		//再向左旋转
		Node* child = parent->_right;

		parent->_right = child->_left;//过继左子树
		child->_left = parent;
		parent->_parent = child;
		Node* grandParent = parent->_parent;
		child->_parent = grandParent;
		
		if (grandParent)//如果不是根节点
		{
			if (parent == grandParent->_left)grandParent->_left = child;
			else grandParent->_right = child;
		}

		//维护_bf
		parent->_bf = -1;
		child->_bf = 0;

		if (parent == _root)
		{
			_root = child;//更新根节点
		}
	}
```

#### 完成Insert中调整二叉树的代码

```C++
public:
	bool Insert(const PKV& kv)
	{
		Cmp cmp;//仿函数实例化一个对象
		if (_root == nullptr)
		{
			_root = new Node(kv);
			return true;
		}
		Node* cur = _root;
		Node* parent = nullptr;
		//寻找插入位置
		while (cur)
		{
			if (cur->_kv.first == kv.first)
			{
				//该键下的节点已存在，发生冲突
				return false;
			}
			else if (cmp(cur->_kv.first, kv.first))//根比节点"大"
			{
				parent = cur;
				cur = cur->_left;
			}
			else //根比节点小
			{
				parent = cur;
				cur = cur->_right;
			}
		}

		//开始插入
		Node* node = new Node(kv);
		if (cmp(parent->_kv.first, kv.first))//应该插入左边
		{
			parent->_left = node;
			node->_parent = parent;
			parent->_bf -= 1;//_bf越小，左子树越高
		}
		else //插入右子树
		{
			parent->_right = node;
			node->_parent = parent;
			parent->_bf += 1;//_bf越大，右子树越高
		}
		//插入完成，准备开始维护平衡
		while (true)
		{
			if (parent->_bf == 0)break;
			else if (parent->_bf == 1 || parent->_bf == -1)
			{
				//子树高度增加
				node = parent;
				parent = parent->_parent;
				if (parent == nullptr)break;//走到根节点了
				if (node == parent->_left)
				{
					parent->_bf -= 1;
				}
				else if (node == parent->_right)
				{
					parent->_bf += 1;
				}
				else
				{
					assert(false);//非法情况
				}
			}
			else if (parent->_bf == -2)
			{
				if (parent->_left->_bf == 0)
				{
					assert(false);//非法情况,不可能左右子树同时超高
				}
				else if (parent->_left->_bf == -1)
				{
					//子树的左子树超高
					RotateR(parent);
					break;
				}
				else if (parent->_left->_bf == 1)
				{
					//子树的右子树超高
					RotateLR(parent);
				}
			}
			else if (parent->_bf == 2)
			{
				if (parent->_right->_bf == 0)
				{
					assert(false);//非法情况,不可能左右子树同时超高
				}
				else if (parent->_right->_bf == 1)
				{
					//子树的右子树超高
					RotateL(parent);
					break;
				}
				else if (parent->_right->_bf == -1)
				{
					//子树的左子树超高
					RotateRL(parent);
					break;
				}
			}
			else
			{
				assert(false);//非法情况
			}
		}
		return true;
	}
```

# 测试
我们来写个简单的`main`函数测试一下我们封装的`AVL树`

> main.cpp
```C++
#include "AVLTree.hpp"
#include <iostream>
using namespace std;
int main()
{
	AVLTree<int,int> avt;//实例化一个对象
	for (int i = 0; i < 100; ++i)//准备100个数据
	{
		avt.Insert({ i,i });//插入数据
	}
	cout <<"Height " << avt.Height() << endl;   //检查高度
	cout << "IsBalance " << avt.IsBalance() << endl; //检测平衡
	avt.Inorder();//检测搜索树的特性
	return 0;
}
```
我们来看看结果

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409030818252.png)

可以看到,`100`个数据下，而且还是按顺序插入，`AVL树`的高度只有`7`，很好得提高了搜索效率

# AVL树的性能
AVL树是一棵绝**对平衡的二叉搜索树**，其要求每个节点的左右子树高度差的绝对值都不超过1，这样可以保证查询时高效的时间复杂度，即$log_2 (N)$。但是如果要对AVL树做一些结构修改的操作，性能非常低下，比如：插入时要维护其绝对平衡，旋转的次数比较多，更差的是在删除时，有可能一直要让旋转持续到根的位置。因此：如果需要一种查询高效且有序的数据结构，而且数据的个数为静态的(即不会改变)，可以考虑AVL树，**但一个结构经常修改，就不太适合。**

# 改造成 红黑树
目前的`AVL树`只是实现了最基本的功能，增删查改只支持了`增`,剩下的内容可自行完善，或者跟随作者的脚步，将`AVL树`改成`红黑树`,增加其修改的性能,并进一步完善增删查改的功能，以及封装`迭代器`等，最后将`红黑树`封装成我们自己的`set`类和`map`类

[戳我前往红黑树篇🔗](https://www.supdriver.top/2024/09/03/RBTree/)