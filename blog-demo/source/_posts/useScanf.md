---
title: 如何在VS里使用scanf
date: 2023-10-05 22:37:32
tags:
---
# VS里怎么连scanf都用不了？ #
不少刚接触[Visual Studio](https://visualstudio.microsoft.com/zh-hans/free-developer-offers/)的可能发现使用scanf会报错(如下)

![报错图片](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/VS_scanf.jpg)

vs告诉你说`scanf`不安全，然后你会发现vs给你提供了`scanf_s`去代替`scanf`,**但是**,只有vs能编译`scanf_s`,**可移植性**太差了，所以我们要用回`scanf`,所以要怎么不让它报错呢？**可以在源文件开头添加一行宏定义(如下)**

`#define  _CRT_SECURE_NO_WARNINGS 1`

这样就能**关闭报错**了,但请先**别急着走**,每次都要复制粘贴一句宏定义太麻烦了，想**一劳永逸**的请往下看。

# 修改newc++file.cpp来自动添加宏定义 #
**先来看怎么做：**首先搜索找到电脑中叫做`newc++file.cpp`的文件。（这里推荐使用[everything](https://www.voidtools.com/zh-cn/downloads/)）

---

*后半段路径应与图片一致，注意不是快捷方式*

![c++文件的位置](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/newcppfile.jpg)

---

**注意**：由于权限原因，**无法**直接修改此文件

所以先将这个文件复制粘贴到别处，例如桌面，下文用`副本`代称。

用记事本类软件(记事本就行)打开`副本`,在第一行输入上文提到的宏定义代码`#define  _CRT_SECURE_NO_WARNINGS 1`,然后`ctrl+s`保存。

**关闭编辑窗口**，将该`副本`移动到原始文件所在文件夹，弹出如下窗口，然后选择**替换文件**

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-10-08_12-28-18.jpg)

接着弹出下一个提示，点**接续**

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-10-08_12-28-33.jpg)

等待它替换完成，然后**大功告成！**。之后新建的每一个`.c`或`.cpp` 文件都会自带那段宏定义，于是`scanf`从此任君使用。~~当然平时删代码的时候记得别把那段宏定义删了~~

## 原理 #
VS所新建的`.c`和`.cpp`文件都源自于对上文`newc++file.cpp`文件的拷贝，通过修改它就能改变`VS新建文件`的初始内容