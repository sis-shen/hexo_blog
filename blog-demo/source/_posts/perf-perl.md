---
title: 使用perf和perl工具进行C++性能分析和生成火焰图
date: 2025-03-11 19:17:00
tags:
---

# 简介

# 安装

## perf安装
perf指令位于linux-tools通用包内，因此要安装适配当前运行内核的包

```shell
apt update -y
# 安装 linux-tools 通用包（适配当前运行的内核）
apt install -y linux-tools-generic linux-tools-$(uname -r)
```

## 获取火焰图生成工具

```shell
git clone https://github.com/brendangregg/FlameGraph.git
```

# 开始性能分析

## perf采样

perf采样可以有多种方式

1. 运行并采样
```shell
sudo perf record -F 99 -g ./cmd --sleep 60
# 参数说明：
# -F 99    : 每秒采样 99 次（平衡精度与开销）
# -a       : 监控所有 CPU
# -g       : 记录调用栈（stack trace）
# -- sleep : 采样持续时间
```

2. 采样特定pid的进程
```shell
# 获取待采样的进程的pid
ps aux | grep <进程名>
# 根据pid采样进程,假设pid为 <PID>
perf record -F 99 -p 1 -- sleep 300

# 采样多线程程序,这段指令需要手动输入ctrl + C 停止采样
perf record -F 99 -g -e cycles:uk -p 1 
# 生成调用图的版本
perf record -F 99 -g -e cycles:u -p 1 --call-graph fp
```

## 绘制火焰图

```shell
#perf record -g ./cmd sleep10
sudo perf script -i perf.data &> perf.unfold 
sudo ./FlameGraph/stackcollapse-perf.pl perf.unfold &> perf.folded 
sudo ./FlameGraph/flamegraph.pl perf.folded > perf.svg
```

## 样例

被测试的示例代码如下:
```C++
#include <iostream>

void test1()
{
    double j = 2.0;
    for(int i = 0;i<1000000000;++i)
    {
        j += 1.0+2.111;
    }
}

void test2()
{
    for(int i = 0;i<50;++i)
    {
        test1();
    }
}

void test3()
{
    for(int  i = 0;i<10;++i) test1();
    test2();
}

int main()
{
    test3();
    return 0;
}
```

我们把文件下载到本机然后打开:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131711873.webp)