---
title: Cpp售货机类模拟实现
date: 2024-09-24 10:35:39
tags:
---
这是一篇复习类的博客,主要综合运用以下知识：

+ 格式化输入输出及打印菜单
+ 类的封装
+ 类的组合
+ 子函数概念
+ 共享指针
+ lambda表达式

# 头文件
为了简便，就把类的声明和实现写在同一个`VendingMachine.h`文件里了

```C++
#pragma once
#include <string>
#include <vector>
#include <unordered_map>
#include <iostream>
#include <algorithm>
#include <windows.h>
using namespace std; //展开std命名空间
```

这里一次性给出所用的头文件，后文便不再添加了

# 封装商品类
售货机当然要管理商品啦，那怎么管理呢？依然是**先描述，再组织**

怎么描述？把它封装成商品类

```C++
struct Item
{
public:
	Item(const string& name,double price,int cnt)
		:_name(name),_price(price),_cnt(cnt)
	{}
public:
	string _name;
	double _price;
	int _cnt;
};
```

怎么组织？用`vecotr`储存，用`unordered_map`建立以`name`为键，`下标`为值的索引，统一管理所有商品

# 封装VendingMachine类

## 基本框架

```C++
class VendingMachine
{
public:
	VendingMachine(int capacity = 100) :_amount(0), _sale(0),_capacity(capacity) {}

private:
	vector<Item> _GoodsList; //储存商品列表
	unordered_map<string, int> _index; //储存名字指向索引的哈希表

	double _amount;//货物总价值
	double _sale;//总销售额
	int _capacity;//容量
};
```

## 实现增加货物和销售货物
这里就要使用子函数的概念了。将实际增加货物的功能和IO功能进行解耦合，封装在两个函数中。`_AddItem`负责实现增加货物，`AddItem`负责`IO处理`和回调`_AddItem`
```C++
public:
	bool AddItem()
	{
		Info();
		string name;
		double price;
		int num;
		cout << "请先后输入 [商品名称] [单价] [数量]" << endl;
		cin >> name >> price >> num;

		return _AddItem(name, price, num);
	}

private:
	bool _AddItem(const string& name, double price,int num)
	{
		if (_index.count(name))
		{
			//更新商品数量
			Item& item = _GoodsList[_index[name]];
			if (item._price != price)
			{
				cout << "商品添加失败，价格与记录不符: " << price << endl;
				return false;
			}
			item._cnt += num;
			_amount += price * num;
		}
		else
		{
			if (price <= 0)
			{
				cout << "非法单价: " << price << endl;
				return false;
			}
			if (num <= 0)
			{
				cout << "非法数量: " << num <<  endl;
				return false;
			}
			if (_capacity < num)
			{
				printf("容量不足， 容量: %d ，添加的数量: %d\n", _capacity, num);
				return false;
			}
			int sz = _GoodsList.size();
			_index[name] = sz;
			_GoodsList.push_back(Item(name, price, num));
			_amount += price * num;
			_capacity -= num;

			printf("商品添加成功，商品名: %s,单价: %.2lf,本次添加的数量: %d\n", name.c_str(), price, num);
			return true;
		}
	}
```

销售货物也是同理
```C++
public:
	bool SellItem()
	{
		string name;
		int num;
		cout << "请输入 [商品名] [数量]" << endl;
		cin >> name >> num;
		return _SellItem(name, num);
	}

private:
	bool _SellItem(const string& name,int num)
	{
		if (num <= 0)
		{
			cout << "非法的售卖数量: " << num << endl;
			return false;
		}

		if (!_index.count(name))
		{
			cout << "不存在的商品: " << name << endl;
			return false;
		}

		Item& item = _GoodsList[_index[name]];
		if (item._cnt < num)
		{
			cout << "交易失败，剩余库存不足！剩余: " << item._cnt << " 预期售卖: " << num << endl;
			return false;
		}

		item._cnt -= num;
		double sum = num * item._price;
		_amount -= sum;
		_capacity += num;
		_sale += sum;
		printf("交易成功！商品名: %s,单价: %.2lf,交易数量: %d,本次交易额: %.2lf\n", name.c_str(), item._price, num, sum);
	}
```

## 实现多种查询功能

```C++
public:
	void InfoItem()
	{
		string name;
		cout << "请输入待查询的商品名称" << endl;
		_InfoItem(name);
	}

private:
	void _InfoItem(const string& name)
	{
		if (!_index.count(name))
		{
			cout << "商品不存在" << endl;
			return;
		}
		Item& item = _GoodsList[_index[name]];
		printf("商品名: %s,单价: %.2lf,剩余量: %d", name.c_str(), item._price, item._cnt);
	}


```
下面要根据不同需求排序数组，这里我们就要用`sort`函数配合`lambda表达式`方便地使用不同的排序规则进行排序
```C++
public:
	void ShowItems()
	{
		cout << "默认顺序" << endl;
		_ShowItems(_GoodsList);
	}

	void ShowItemsByName(bool reverse_arr=false)//默认升序
	{
		if (reverse_arr)
		{
			cout << "按名字降序" << endl;
		}
		else cout << "按名字升序" << endl;
		auto copy = _GoodsList;
		sort(copy.begin(), copy.end(), [&](const Item& item1, const Item& item2) {
			if (item1._name <= item2._name)return true;
			else return false;
			});
		if (reverse_arr)
		{
			reverse(copy.begin(), copy.end());
		}
		_ShowItems(copy);
	}

	void ShowItemsByPrice(bool reverse_arr = false)//默认升序
	{
		if (reverse_arr)cout << "按单价降序" << endl;
		else cout << "按价格升序" << endl;
		auto copy = _GoodsList;
		sort(copy.begin(), copy.end(), [&](const Item& item1, const Item& item2) {
			if (item1._price <= item2._price)return true;
			else return false;
			});
		if (reverse_arr)
		{
			reverse(copy.begin(), copy.end());
		}
		_ShowItems(copy);
	}

	void ShowItemsByCNT(bool reverse_arr = false)//默认升序
	{
		if (reverse_arr) cout << "按剩余量降序" << endl;
		else cout << "按剩余量升序" << endl;
		auto copy = _GoodsList;
		sort(copy.begin(), copy.end(), [&](const Item& item1, const Item& item2) {
			if (item1._cnt <= item2._cnt)return true;
			else return false;
			});
		if (reverse_arr)
		{
			reverse(copy.begin(), copy.end());
		}
		_ShowItems(copy);
	}

private:
	void _ShowItems(const vector<Item>& goodslist)
	{
		printf("%-10s %-10s %-4s\n", "商品名", "单价", "剩余数量");
		for (auto item : goodslist)
		{
			printf("%-10s %-10.2lf %-4d\n", item._name.c_str(), item._price, item._cnt);
		}
		printf("\n");
	}
public:

	void Info()
	{
		printf("商品数量: %u ,机器内商品总价值 %.2lf,总销售额: %.2lf,剩余容量: %d\n",
			_GoodsList.size(), _amount, _sale, _capacity);
	}
```

## 实现自动初始化
写一个函数自动初始化参数，方便调试
```C++
	void AutoInit()
	{
		if (_GoodsList.size() != 0)
		{
			cout << "不能重复初始化!!!" << endl;
			return;
		}
		_AddItem("小苹果", 5, 10);
		_AddItem("大香蕉", 2.5, 2);
		_AddItem("香蕉君", 30, 5);
		_AddItem("野兽先辈", 1145.14, 15);

	}
```

## 实现菜单和运行功能
```C++
public:
	void Run()
	{

		int cmd = 1;
		do
		{
			PrintMenu();
			cout << "请输入操作码" << endl;
			cin >> cmd;
			switch (cmd)
			{
			case 0:
				break;
			case 1:
				AutoInit(); break;
			case 2:
				AddItem(); break;
			case 3:
				ShowItems(); SellItem(); break;
			case 4:
				Info(); break;
			case 5:
				ShowItems(); break;
			case 6:
				ShowItemsByName(); break;
			case 7:
				ShowItemsByName(true); break;
			case 8:
				ShowItemsByPrice(); break;
			case 9:
				ShowItemsByPrice(true); break;
			case 10:
				ShowItemsByCNT(); break;
			case 11:
				ShowItemsByCNT(true); break;
			default:
				cout << "未知操作码" << endl;
				break;
			}

			cout << "输入任意按键继续..." << endl;
			getchar();
			getchar();

		} while (cmd);
	}
private:
	void PrintMenu()
	{
		ClearScreen();
		printf("===========菜单=============\n");
		printf("0.退出 \n1.自动初始化 \n2.添加商品 \n3.购买商品 \n4.显示售货机信息  \n");

		cout << "5.默认顺序显示商品列表\n";
		cout << "6.按名称升序显示商品列表\n";
		cout << "7.按名称降序显示商品列表\n";
		cout << "8.按单价升序显示商品列表\n";
		cout << "9.按单价降序显示商品列表\n";
		cout << "10.按剩余量升序显示商品列表\n";
		cout << "11.按剩余量降序显示商品列表\n";

		printf("=============================\n\n");
	}

	void ClearScreen()
	{
		system("cls");
	}
```

# 编写main函数
把`main`函数放在`main.cpp`里

其中我们要用到`shared_ptr`实现共享指针，内部的原理是对同一个指针提供引用计数，当引用计数要归零时才释放资源，达到自动内存管理的目的

```C++
#include "VendingMachine.h"

int main()
{
	shared_ptr<VendingMachine> pvm(new VendingMachine());

	pvm->Run();
	return 0;
}
```

# 小结
啪的一下很快啊，一个模拟售货机就做出来了。项目很简单，拿来复习，写起来还是很心情愉悦的