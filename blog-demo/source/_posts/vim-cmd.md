---
title: vim基础指令集
date: 2023-12-11 07:41:57
tags: Linux vim
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/terminalpng.png
---
`Vim`是一款文本编辑器,下面介绍在vim界面中的常用指令

**三种模式**:`命令模式(Command Mode)` `插入模式（Insert Mode` `命令行模式（Command-Line Mode）`（这里称命令行模式为`底行模式`）

三者关系如下图

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2023-12-11_10-39-04.png)

# 命令模式
`vim`界面中多摁几次`ESC`就能退出其它模式回到`命令模式`，在这个模式下可以使用一系列vim[快捷键](https://linux.cn/article-8144-1.html)

# 底行模式 #
`tips`:不管目前是什么模式,先狂按`ESC`,回到`命令模式`,然后输入`:`进入`底行模式`,准备开始输命令

`命令组成`
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2023-12-11_12-48-54.png)

+ 保存`:w`->强制保存`!w`
+ 退出`:q`->强制退出`:!q`
+ 保存并退出`:wq`-.强制保存并退出`:!wq`
+ 对比`:vs `+`(源文件路径)`


# 插入模式
在`命令模式`下按键盘`i`进入`插入模式`，执行正常的文本编辑功能