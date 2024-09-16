---
title: 手撕红黑树 并封装map和set
date: 2024-09-03 08:21:33
tags:
---

# 红黑树

虽然`AVL树`作为绝对的平衡搜索二叉树，有着极高的查询效率，但正因为其严格的要求，修改`AVL树`的某个结点时，可能要一路调整到根节点，效率低下。为了解决这一痛点，略微没那么严格的**近似平衡搜索二叉树**，即`红黑树`被提出

## 红黑树的概念
红黑树，是一种二叉搜索树，但在每个结点上`增加一个存储位表示结点的颜色`，可以是`Red`或`Black`。 通过对任何一条从根到叶子的路径上各个结点着色方式的限制，**红黑树确保没有一条路径会比其他路径长出俩倍**，因而是**接近平衡**的。

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RBTree_RBT.png)

## 红黑树的性质
首先与一般的定义不同，在红黑树中将`空指针`(上图为`NIL`)作为`叶子节点`,然后我们来讨论具体的性质

1. 每个结点不是`红色`就是`黑色`
2. 根节点必定是`黑色`的
3. 如果一个结点是`红色`的,则它的两个**孩子结点**是`黑色`的
4. 对于每个结点，该结点到其所有后代叶结点的简单路径上，均包含**相同数目**的`黑色`结点
5. 每个**叶子结点**都是`黑色`的此处的叶子结点指的是`空结点NIL`

**思考：为什么满足上面的性质，红黑树就能保证：其最长路径中节点个数不会超过最短路径节点个数的两倍？**
> 红黑树的性质保证了从根节点到所有叶子结点（空结点）的路径上，**包含相同数量的黑色结点**。这是红黑树平衡性的重要保证，以下是如何保证最长路径不会超过最短路径两倍的具体解释：
> 1. 黑色高度的定义：
>   + 对于红黑树中的每个结点，定义其黑色高度为从该结点到叶子结点的路径上黑色结点的数量。根据性质 4，这个数量在所有路径上都是相同的。
>   + 设定黑色高度为`h`。那么从根到任意叶子结点的路径都包含`h`个黑色结点。
> 2. 红色结点的插入：
>   + 根据性质3，如果一个结点是红色的，则它的两个孩子结点是黑色的。这就意味着每个红色结点必定有两个黑色的孩子。
>   + 因此，从根到叶子结点的路径上，红色结点与黑色结点交替出现，红色结点之间的路径长度会相对较短。
> 3. 路径长度的关系：
>   + 根到一个叶子结点的路径的最长可能路径是最短路径的两倍。这是因为每个红色结点都在路径上增加了一个额外的层级，但它们只增加了一个红色结点的高度，而黑色结点的数量不变。
>   + 如果某条路径上有 `r` 个红色结点，那么该路径的总长度将是 `h + r`，其中 `h` 是黑色结点的数量。因为红色结点的数量 `r` 可以最大等于 `h`，所以最长路径的长度 `h + r`最大为 `2h`，即**最长路径的长度不会超过最短路径的两倍**。


## 红黑树的实现
这里我们仿照`STL30`的源码,再用更简洁易懂的代码形式实现红黑树,*比如节点和迭代器的双层设计，这里就不使用了*

### 红黑树结点
+ 因为要经常访问父结点，所以这里用三叉链表维护树结构
+ 红黑树的节点需要一个变量表示颜色，这里用`typedef`把`bool`类型封装为`Color`类
+ 使用模板来存储泛类型的值

```C++
#pragma once
#include <functional>

typedef bool Color;//只有红黑两种颜色，正好使用bool类型
const Color Red = false;//定义两种颜色
const Color Black = true;

//红黑树结点的定义
template <class ValueType>
struct RBTreeNode
{
	typedef RBTreeNode<ValueType> Node;//简化代码
	Node* _left;
	Node* _right;
	Node* _parent;
	Color _color;

	ValueType _value;
	RBTreeNode(const ValueType& val, Color color)
		:_value(val)
		, _color(color)
		, _left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
	{}
};
```

### 红黑树的基本结构
+ 模板参数:红黑树也是`KV`类型的二叉树，所以模板参数有:
  + `K`:即Key,节点使用`K`类型的参数比较
  + `V`:即ValueType,构造红黑树结点时传入的模板参数
  + `KeyOfValue`:因为节点将`Key`值也存在了节点的值域，所以需要外部传入`仿函数KeyOfValue`将Key值取出
  + `Compare`:允许外部自定义`K类型`的比较规则
+ typedef封装:
  + `typedef RBTreeNode<V> Node;`
+ 私有成员变量:
  + `Compare _cmp`:声明一个仿函数成员变量，整个对象里都能直接调用
  + `KeyOfValue _kof`:用处同上
  + `Node* _header`
+ 默认构造函数：
  + 使用缺省传入`const Compare&`类型定义的`std::greater<K>()`临时对象
  + 创建哨兵位作为头结点，使后续对根结点的调整与一般情况统一
+ 私有函数`GetRoot()` `left()` `right()`
  + 将对根节点的获取封装起来，使代码更直观
  + 简化代码，获取左右子树

至于迭代器的封装，我们在[后文实现](#iterator )

```C++
template<class K,class V,class KeyOfValue,class Compare = std::greater<int> >
class RBTree
{
	typedef RBTreeNode<V> Node;

public:
	RBTree(const Compare& cmp =Compare())
		:_cmp(cmp), _header(nullptr)
	{
		//创建哨兵位头结点，使后面调整树形状不用区分根节点
		_header = new Node(V(), Black);
	}

private:
	Node* GetRoot()
	{
		if (_header->_parent == _header)return nullptr;
		else return _header->_parent;
	}

	Node* SetRoot(Node* root)
	{
		if (root == nullptr)return nullptr;
		root->_parent = _header;
		_header->_parent = root;
	}

	Node* left(Node* root)
	{
		if (root == nullptr)return nullptr;
		return root->_left;
	}

	Node* right(Node* root)
	{
		if (root == nullptr)return nullptr;
		return root->_right;
	}

private:
	Compare _cmp;
	KeyOfValue _kof;
	Node* _header;
};
```

### 红黑树的插入
和`AVL树`一样，红黑树的插入也分为两步`新增结点`和`调整`

#### 新增结点
关于根结点的颜色，前面的[规定性质](#红黑树的性质)里也说了，根节点为黑色，这里在重申一下为什么一定是黑色

> 1. 简化性质的维护：根节点为黑色可以简化在插入和删除操作后维持红黑树性质的过程。特别是在插入节点时，新的节点（通常为红色）可能违反了红黑树的性质。根节点为黑色确保了在树的高度平衡和黑色节点数量保持一致的情况下，维护这些性质更加容易。
> 2. 平衡树的高度：红黑树的性质确保了树的高度约为 ( \log(n) )，其中 ( n ) 是树中节点的数量。通过将根节点设为黑色，红黑树的黑色高度（从根到所有叶子节点的黑色节点数目）得到保证，从而有助于维护树的平衡。
> 3. 避免违反性质：如果根节点允许是红色，可能会导致在某些情况下难以保证红色节点的数量平衡，特别是在连续插入和删除操作之后。将根节点设为黑色可以避免这种复杂情况，从而简化了实现和维护过程。

首先我们先写个`Find`函数防止插入重复数据

```C++
public:
	bool Find(const K& key)
	{
		Node* prev = nullptr;
		Node* cur = GetRoot();
		while (cur)
		{
			if (_cmp(_kof(cur->_value), key))//相等或在左子树
			{
				prev = cur;
				cur = left(cur);
			}
			else
			{
				cur = right(cur);//可能在右子树
			}
		}

		if (prev == nullptr || _cmp(_kof(prev->_value),key))//一直向右走或一直向左走
		{
			return false;
		}
		else return true;//走到头的过程中左转了一次之后一直向右走
	}
```

关于新结点（非根）的颜色,这里规定为`红色`，使新结点对红黑树中`黑树`高度的影响最小

```C++
public:
	Node* proot = GetRoot();
	if (proot == nullptr)//树为空
	{
		Node* newnode = new Node(value, Black);
		newnode->_parent = _header;
		_header->_parent = newnode;
		return true;
	}

	if (Find(_kof(value)))return false;//结点已存在，插入失败
	Node* cur = proot;
	Node* parent = proot->_parent;
	while (cur)
	{
		if (_cmp(_kof(cur->_value), _kof(value)))
		{
			//插在左子树
			parent = cur;
			cur = left(cur);
		}
		else
		{
			//插在右子树
			parent = cur;
			cur = right(cur);
		}
	}
	//开始插入
	Node* newnode = new Node(value, Red);
	if (_cmp(_kof(parent->_value),_kof(value) ))
	{
		//插入右子树
		parent->_left = newnode;
		newnode->_parent = parent;
	}
	else
	{
		parent->_right = newnode;
		newnode->_parent = parent;
	}

	//开始调整
	//.......

	
```
#### 调整红黑树
每次新节点插入后，都要检测红黑树的性质是否造到破坏

因为新节点的默认颜色是红色，因此：如果其双亲节点的颜色是黑色，没有违反红黑树任何性质，则不需要调整；但当新插入节点的双亲节点颜色为红色时，就违反了性质三`不能有连在一起的红色节点`，此时需要对红黑树分情况来讨论:

**约定** **`cur`当前结点,`parent`为父结点,`uncle`为叔叔结点,`gp`为祖父结点**

##### 情况一：直接染色

+ ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RBTREE_1.png)
+ 可以看到这种情况只需染色即可,将`gp`染色为红色，防止整棵树的`黑树高度`改变0
+ 不过当该树为整棵树而不是子树时，根节点(`gp`)在最后还需染色回黑色（这里留到所有调整的最后）
+ 当该树为子树，即`gp`存在父结点时，因为`gp`变为了红色，还需继续向上调整
+ ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RBTREE_1_1.png)

##### 情况二：单次旋转+染色
+ ![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RBTREE_2.png)
+ 图中`uncle`的情况有两种
  + 如果`uncle`节点不存在，则`cur`一定是**新插入节点**，因为如果`cur`不是新插入节点，则`cur`和`parent`一定有一个节点的颜色是黑色，就不满足性质4：每条路径黑色节点个数相同。
  + 如果`uncle`节点存在，则其一定是黑色的，那么`cur`节点原来的颜色一定是黑色的，现在看到其是红色的原因是因为`cur`的子树在**调整的过程中**将`cur`节点的颜色由黑色改成红色。
+ 调整步骤：
  + `parent`为`pg`的左孩子，`cur`为`parent`的左孩子，则进行右单旋转
  + `parent`为`pg`的右孩子，`cur`为`parent`的右孩子，则进行左单旋转
  + `parent`->黑色
  + `pg`->红色

##### 情况三：两次旋转

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/RBTREE_3.png)

+ 如图，一次旋转后变成了**情况二**,所以总共要旋转两次
  + `parent`为`gp`的左孩子，`cur`为`parent`的右孩子，则针对`parent`做左单旋转
  + `pparent`为`gp`的右孩子，`cur`为`parent`的左孩子，则针对`parent`做右单旋转
+ 变成情况二后按情况二处理即可

##### 封装代码
旋转操作的代码量较大，我们选择封装到成员函数中

```C++
private:
	void RotateR(Node* node)
	{
		if (node == nullptr)assert(false);
		Node* cur = node;
		Node* parent = cur->_parent;
		if (parent == nullptr) assert(false);
		Node* gp = parent->_parent;
		if (gp == nullptr) assert(false);

		//调整树结构
		Node* ggp = gp->_parent;
		if (gp == ggp->_left)ggp->_left = parent;
		else ggp->_right = parent;
		parent->_parent = ggp;

		gp->_left = parent->_right;
		if (parent->_right)parent->_right->_parent = gp;
		parent->_right = gp;
		gp->_parent = parent;

		if (gp == GetRoot())
		{
			SetRoot(parent);
		}

		//染色
		parent->_color = Black;
		gp->_color = Red;
	}

	void RotateL(Node* node)
	{
		Node* cur = node;
		if (cur == nullptr)assert(false);
		Node* parent = cur->_parent;
		if (parent == nullptr)assert(false);
		Node* gp = parent->_parent;
		if (gp == nullptr)assert(false);

		//调整树结构
		Node* ggp = gp->_parent;
		if (ggp->_left == gp) ggp->_left = parent;
		else ggp->_right = parent;
		parent->_parent = ggp;
		
		gp->_right = parent->_left;
		if (parent->_left)parent->_left->_parent = gp;
		parent->_left = gp;
		gp->_parent = parent;

		if (gp == GetRoot())
		{
			SetRoot(parent);
		}

		//染色
		parent->_color = Black;
		gp->_color = Red;
	}

	void RotateLR(Node* node)
	{
		Node* cur = node;
		if (cur == nullptr)assert(false);
		Node* parent = cur->_parent;
		if (parent == nullptr)assert(false);
		Node* gp = parent->_parent;
		if (gp == nullptr) assert(false);

		parent->_right = cur->_left;
		if (cur->_left)cur->_left->_parent = parent;
		cur->_left = parent;
		parent->_parent = cur;
		cur->_parent = gp;
		gp->_left = cur;

		RotateR(parent);
	}

	void RotateRL(Node* node)
	{
		Node* cur = node;
		if (cur == nullptr)assert(false);
		Node* parent = cur->_parent;
		if (parent == nullptr)assert(false);
		Node* gp = parent->_parent;
		if (gp == nullptr) assert(false);

		parent->_left = cur->_right;
		if (cur->_right)cur->_right->_parent = parent;
		cur->_right = parent;
		parent->_parent = cur;
		cur->_parent = gp;
		gp->_right = cur;

		RotateL(parent);
	}
```

然后将`Insert`函数补充完整

```C++
public:
	bool Insert(const V& value)
	{
		Node* proot = GetRoot();
		if (proot == nullptr)//树为空
		{
			Node* newnode = new Node(value, Black);
			newnode->_parent = _header;
			_header->_parent = newnode;
			return true;
		}

		if (Find(_kof(value)))return false;//结点已存在，插入失败
		Node* cur = proot;
		Node* parent = proot->_parent;
		while (cur)
		{
			if (_cmp(_kof(cur->_value), _kof(value)))
			{
				//插在左子树
				parent = cur;
				cur = left(cur);
			}
			else
			{
				//插在右子树
				parent = cur;
				cur = right(cur);
			}
		}
		//开始插入
		Node* newnode = new Node(value, Red);
		if (_cmp(_kof(parent->_value),_kof(value) ))
		{
			//插入右子树
			parent->_left = newnode;
			newnode->_parent = parent;
		}
		else
		{
			parent->_right = newnode;
			newnode->_parent = parent;
		}

		cur = newnode;
		//开始调整
		while (parent != _header && parent->_color == Red)
		{
			Node* gp = parent->_parent;
			Node* uncle = nullptr;
			if (parent == gp->_left)uncle = gp->_right;
			else uncle = gp->_left;

			if (uncle && uncle->_color == Red)//情况一
			{
				parent->_color = Black;
				uncle->_color = Black;
				gp->_color = Red;

				cur = gp;
				parent = gp->_parent;
			}
			else if (cur == parent->_left && parent == gp->_left)
			{
				RotateR(cur);
				break;
			}
			else if (cur == parent->_right && parent == gp->_right)
			{
				RotateL(cur);
				break;
			}
			else if (cur == parent->_right && parent == gp->_left)
			{
				RotateLR(cur);
				break;
			}
			else if (cur == parent->_left && parent == gp->_right)
			{
				RotateRL(cur);
				break;
			}
		}

		Node* root = GetRoot();
		root->_color = Black;//修改根节点颜色
		return true;
	}
```

### 红黑树的删除

~~让红黑树和进程一起被清理也不是不行~~

*under construction*

### 红黑树的验证、高度和遍历
下面再补充一些获取红黑树参数的接口

```C++
public:
	void Inorder()
	{
		_Inorder(GetRoot());
		std::cout << "nullptr" << std::endl;
	}

protected:
	void _Inorder(Node* root)
	{
		if (root == nullptr) return;
		_Inorder(root->_left);
		std::cout << root->_value << "->";
		_Inorder(root->_right);
	}
public:
	int Height()
	{
		return _Height(GetRoot());
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

public:
	bool IsValidRBTree()
	{
		Node* root = GetRoot();
		if (root->_color != Black)return false;//违反根节点染色规则
		
		int blackCount = 0;
		Node* cur = root;
		while (cur)
		{
			if (cur->_color == Black) blackCount++;
			cur = cur->_left;
		}

		blackCount++;//cur为nullptr时也算作一个黑节点

		return _IsValidRBTree(root,0,blackCount);
	}

protected:
	bool _IsValidRBTree(Node* root,int cnt,const int blackCount)
	{
		if (root == nullptr)
		{
			cnt++;
			if (cnt == blackCount)return true;
			else return false;//黑色节点数量不相等
		}
		//检测与父结点的染色规则
		if (root->_color == Red && root->_parent->_color == Red)
		{
			return false;
			//不能有连续的红色节点
		}

		if (root->_color == Black) cnt++;
		return _IsValidRBTree(root->_left, cnt, blackCount)
			&& _IsValidRBTree(root->_right, cnt, blackCount);
	}

```

### 封装迭代器

红黑树的迭代器是双向迭代器，所以储存`空指针`的迭代器没有意义，我们在构造函数上就禁掉用空指针构造

#### `拷贝构造`

除了直接构造迭代器，还要提供用`const_iterator`构造`iterator`的拷贝构造，而我们依然使用`template <class Value,class Ref,class Ptr>`作为模板参数，所以要在`iterator`类型需要在类内手动封装,代码如下

```C++
template <class Value,class Ref,class Ptr>
class __rbtree_iterator
{
	typedef __rbtree_iterator<Value, const Value&, const Value*> const_iterator;
	typedef __rbtree_iterator<Value, Ref, Ptr> self;
	typedef RBTreeNode<Value>* linkeType;
private:
	linkType _ptr;
}
```

#### 迭代器的++和--
迭代器的`++`和`--`是按前序遍历的顺序和逆序来访问的，而三叉链表有自己独特的访问方式

**`++`**

+ 操作当前结点时，该结点和该节点的左子树已经完成遍历,前序遍历已经走过 `左子树`->`根`
  + 该节点有右子树时，前序遍历右子树
  + 该节点没有右子树时，该子树完成遍历，回到根节点`root`的父节点`p`再次执行`++`操作(这里可以用递归),下面介绍循环
    + 当根节点`root`为`p`的右结点时，`p`为根节点的子树也完成了遍历，把`p`作为所在子树的`root`，执行上一条操作
    + 当根节点`root`为`p`的左结点时，`p`的左子树完成了遍历，正好下一步访问根节点,即`p`,返回指向`p`的迭代器即可

**`--`**

+ 操作当前结点时，该结点和该节点的右子树已经完成遍历,前序遍历已经走过 `右子树`->`根`
  + 该节点有左子树时，后序遍历右子树
  + 该节点没有左子树时，该子树完成遍历，回到根节点`root`的父节点`p`再次执行`--`操作(这里可以用递归),下面介绍循环
    + 当根节点`root`为`p`的左结点时，`p`为根节点的子树也完成了遍历，把`p`作为所在子树的`root`，执行上一条操作
    + 当根节点`root`为`p`的右结点时，`p`的右子树完成了遍历，正好下一步访问根节点,即`p`,返回指向`p`的迭代器即可

完整代码如下
```C++
template <class Value,class Ref,class Ptr>
class __rbtree_iterator
{
	typedef __rbtree_iterator<Value, const Value&, const Value*> const_iterator;
	typedef __rbtree_iterator<Value, Ref, Ptr> self;
	typedef RBTreeNode<Value>* linkeType;

public:
	__rbtree_iterator(const linkeType& ptr)
		:_ptr(ptr)
	{
		if (_ptr == nullptr)exit(-1);//非法的空迭代器
	}

	__rbtree_iterator(const const_iterator& const_it)
		:_ptr(const_it._ptr)
	{}

	self operator++()
	{
		if (_ptr == nullptr || _ptr->_right == _ptr )
		{
			exit(-1);//end()迭代器不能++,空迭代器不能加减
		}
		if (_ptr->_right)
		{
			_ptr = _ptr->_right;
		}
		else
		{
			linkeType parent = _ptr->_parent;
			while (parent->_right == _ptr)
			{
				_ptr = parent;
				parent = _ptr->_parent;
			}
			_ptr = parent;
		}
		return (*this);
	}

	self operator--()
	{
		if (_ptr == nullptr)
		{
			exit(-1);//非法情况
		}
		else if (_ptr->_left == _ptr)//在头结点
		{
			linkeType right = _ptr->_parent;//走到根节点
			while (right)
			{
				_ptr = right;
				right = _ptr->_right;
			}
		}
		else if (_ptr->_left)
		{
			_ptr = _ptr->_left;
			linkeType right = _ptr->_right;//寻找左子树的最右结点
			while (right)
			{
				_ptr = right;
				right = _ptr->_right;
			}
		}
		else
		{
			linkeType parent = _ptr->_parent;
			while (parent->_left == _ptr)
			{
				_ptr = parent;
				parent = _ptr->_parent;
			}
			_ptr = parent;
		}

		return *this;
	}

	Ref operator*()
	{
		return _ptr->_value;
	}

	Ptr operator->()
	{
		return &(_ptr->_value);
	}

	bool operator!=(const self& it)
	{
		return _ptr != it._ptr;
	}

	bool operator==(const self& it)
	{
		return !operator!=(it);
	}


private:
	linkeType _ptr;
};
```

在`RBTree`类中声明迭代器类并添加接口

```C++
public:
	typedef __rbtree_iterator<V, V&, V*> iterator;
	typedef __rbtree_iterator<V, const V&, const V*>const_iterator;
	public:
	iterator begin()
	{
		return iterator(Most_Left());
	}

	iterator end()
	{
		return iterator(_header);
	}

	const_iterator begin()const
	{
		return const_iterator(Most_Left());
	}

	const_iterator end()const
	{
		return const_iterator(_header);
	}
```

### 修改Insert
封装了迭代器之后，我们就可以让`insert`承担更多的功能了,让它同时返回`bool`和`iterator/const_iterator`

`std::pair<bool,iterator>`

### 新增关于容量的接口
这里我们新增一个私有成员变量`size_t _size`记录红黑树中有效节点个数，此处需要修改一下构造函数和`Insert`函数，这里就不列出来了

```C++
public:
	size_t size()const
	{
		return _size;
	}
	bool empty()const
	{
		return _size == 0;
	}
```

### 新增拷贝构造
有了迭代器，拷贝构造也十分好写了

```C++
	RBTree(const self& rbt)
	{
		for (auto value : rbt)
		{
			Insert(value);
		}
	}
```
