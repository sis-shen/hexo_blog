---
title: 归并排序
date: 2024-06-28 09:13:09
tags: 排序 算法
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/%E5%BD%92%E5%B9%B6%E6%8E%92%E5%BA%8F.png
---

***
时间复杂度: O(nlogn)
空间复杂度: O(n)
稳定性： 稳定
实现语言: C/C++
***

# 原理

## 思想
这里采用的是**分治**的思想，但与`快速排序`相反的是，归并排序采用的是先分治再合并。

已知在有额外空间的情况下，合并两个**有序**数组得到一个新的较长有序数组是很高效的。 所以能不能把一个任意数组分成由左右两个有序数组组成然后合并成有序数组呢？

显然不能，大部分情况并不能分成两个有序数组，但如果在此之前用同样的方法（这里采用递归）事先排序左右两部分呢？大部分情况依然不能，因此这个递归会一直递推下去，最终待排序区间**不断缩小**,到只剩一个或零个元素，此时就可以将其看为有序数组了,也就是说递归在这里停止，可以一路合并有序数组一路回归上去了

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-07-02_21-29-34.png)

## 分治
这里使用左右指针**控制待排序区间**（迭代器也行）,并采用递归的方式形象地完成分治操作

```C++
void _MergeSort(vector<int>& arr,int left,int right,vector<int>&tmp)
{
    //分治
    if(left >= right) return;//递归出口

    int mid = (left+right)/2;
    _MergeSort(arr,left,mid,tmp);//排序左半边
    _MergeSort(arr,mid+1,right,tmp);//排序右半边

    //合并数组
    //...
}

void MergeSort(vector<int>& arr)
{
    vector<int> tmp(arr.size());//用tmp开辟额外空间用于合并数组
    _MergeSort(arr,0,arr.size()-1);
}
```

## 合并有序数组
因为合并两个有序数组**难以原地**完成，所以要借助`tmp`数组提供额外空间。

具体做法就是用两个指针分别从两个数组中挑最小值，然后用第三个指针从左向右填到`tmp`中，最后再拷贝至原数组

```C++
void _MergeSort(vector<int>& arr,int left,int right,vector<int>&tmp)
{
    //分治
    if(left >= right) return;//递归出口

    int mid = (left+right)/2;
    _MergeSort(arr,left,mid,tmp);//排序左半边
    _MergeSort(arr,mid+1,right,tmp);//排序右半边

    //合并数组
    	int cur1 = left;
	int cur2 = mid + 1;
	int cur = left;

	while (cur1 <= mid && cur2 <= right)
	{
		if (arr[cur1] < arr[cur2])
		{
			tmp[cur++] = arr[cur1++];
		}
		else
		{
			tmp[cur++] = arr[cur2++];
		}
	}

	while (cur1 <= left)
		tmp[cur++] = arr[cur1++];
	while (cur2 <= right)
		tmp[cur++] = arr[cur2++];

	for (int i = left; i <= right; i++)
		arr[i] = tmp[i];
}

void MergeSort(vector<int>& arr)
{
    vector<int> tmp(arr.size());//用tmp开辟额外空间用于合并数组
    _MergeSort(arr,0,arr.size()-1);
}
```

以上就是C++版的完整代码,下面再提供`C`语言版的

```C
void _MergeSort(int* arr, int sz, int left, int right, int* tmp)
{
	if (left >= right) return;

	int mid = (left + right) / 2;

	_MergeSort(arr,sz ,left, mid, tmp);
	_MergeSort(arr, sz, mid + 1,right, tmp);
	int cur1 = left, cur2 = mid + 1, cur = left;

	while (cur1 <= mid && cur2 <= right)
	{
		if (arr[cur1] < arr[cur2])
			tmp[cur++] = arr[cur1++];
		else
			tmp[cur++] = arr[cur2++];
	}

	while (cur1 <= mid)
		tmp[cur++] = arr[cur1++];
	while (cur2 <= right)
		tmp[cur++] = arr[cur2++];

	for (int i = left; i <= right; i++)
		arr[i] = tmp[i];
}

void MergeSort(int* arr, int sz)
{
	int* tmp = (int*)malloc(sizeof(int) * sz);
	_MergeSort(arr, sz, 0, sz - 1, tmp);
}
```

# 小结
归并排序的原理乍一看很吓人，好像很高深的样子，但其实多上手练练，多试着独立敲代码就能掌握其精髓了，之后手撕归并排序不要太简单