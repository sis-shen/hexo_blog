---
title: Linux文件系统
date: 2024-07-26 09:49:27
tags: Linux
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202409230833286.png
---
# 文件系统的组织方式

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-06_14-22-47.png)

这里介绍一下`Block Group`里的分区内容

+ `超级块`(Super Block): **存放文件系统本身的结构信息**。记录的信息主要有：`bolck`(数据块) 和 `inode`(属性块)的总量，
**未使用**的`block`和`inode`的数量，一个`block`和`inode`的大小，最近一次挂载的时间，最近一次写入数据的
时间，最近一次检验磁盘的时间等其他文件系统的相关信息。Super Block的信息被破坏，可以说整个
文件系统结构就被破坏了
+ `GDT`(Group Descriptor Table): 块组描述符，描述块组属性信息。这里不作详细介绍
+ `块位图`(Block Bitmap):位图中的`比特位`记录着Data Block中哪个数据块已经被占用，哪个数据块没有被占用
+ `inode位图`(inode Bitmap):每个`比特位`表示一个inode是否空闲可用。
+ `inode表`(inode Table):每一个`inode`存放了各自文件的`属性`，如文件大小，所有者，最近修改时间等
+ `数据区`：存放文件内容

# Linux系统中文件的增删查改
Linux中，每一个文件都有自己的inode,而每一个inode有自己的inode编号（按分区为单位）

特别的：尽管inode储存了文件的属性，文件名并不属于inode

## 理解目录文件
`目录`也是`文件`!目录也有自己的`inode`和数据块

目录的数据块**储存**了该目录下，文件的**文件名**和文件`inode`的映射关系

## 创建文件的过程

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-06_16-44-21.png)

创建文件主要有以下4个操作

1. `存储属性`:内核先找到一个空闲的`inode`(图中是263467)。**内核**把文件信息记录其中。
2. `存储数据`:该文件需要存储在三个数据块，**内核**找到了三个空闲块:300,600,800。然后将**内核缓冲区**的第一块数据复制到300，下一块复制到600，依次类推
3. `记录分配情况`:文件内容按顺序300,600,800存放。内核在inode上的磁盘分布区记录了上述块列表
4. `添加文件名到目录`:通过写入目录文件，将文件名和inode的建立映射关系(这里是将文件名cab与inode 263467建立联系)

# Linux中的硬链接和软链接
Linux文件系统中特别**重要***的一点是：文件系统使用`inode`管理文件，**而不是**文件名，所以使用`inode`唯一指定文件。也就是说找到了`inode`才是真正找到了`文件`

基于此理论，我们可以更好地理解`软链接`和`硬链接`

## 软链接
使用`ln -s <原文件名> <软链接文件名>`指令可以建立软链接,`-s`参数就是指定创建软链接文件

例如在已有`file.txt`的情况下,使用`ln -s file.txt soft-link`在当前文件夹创建软链接文件`soft-link`,指向文件`file.txt`

然后我们输入`ls -li`查看这两个文件

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-06_21-54-47.png)

可以看到软链接文件和原文件的inode是不同的，且两个文件的硬链接数(第二个框框)都是`1`,说明软链接是一个文件系统意义上的`新文件`

`软链接文件`可类比与`Windows`下的快捷方式

## 硬链接

使用`ln <原文件名> <硬链接文件名>`指令可以建立硬链接。

例如在已有`file.txt`的情况下,使用`ln file.txt hard-link`在当前文件夹创建软链接文件`hard-link`,与文件`file.txt`**等价**

然后我们输入`ls -li`查看这"两个"文件

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-06_22-06-39.png)

可以看到它们的`inode`是完全一样的，那么基于上文文件系统对文件的视角，这"两个"文件完全就是**同一个文件!**,或者说`hard-link`是`file.txt`的`别名`，同时我们也能看到`硬链接引用计数`变成了`2`

在引入了`硬链接引用计数`的概念下，删除目录中的一个文件，只是删除该目录下`文件名`和`inode`的映射关系，并使该文件的`引用计数`减去`1`,当其**归零**时，才会真正释放磁盘上的文件资源

### 目录的硬链接数
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-08-06_22-36-40.png)

对于每一个普通目录（除了根目录），至少有父目录下自己的`文件名`和该目录下的`.`文件`2`个`硬链接`，而在其内部创建的每个`子目录`又会储存一个名为`..`的硬连接，所以普通目录的硬连接数会`>=2`

因此为了防止文件系统结构混乱，操作系统**不允许**用户对目录建立`硬链接`

### 删除链接文件
`rm`不一定能删除链接文件，此时可以用`unlink`命令