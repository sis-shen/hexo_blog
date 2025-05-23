---
title: 计算机网络复习题-答案-第一章
date: 2025-03-04 10:05:05
tags:
---

来自计算机科学丛书：《计算机网络：自顶向下方法》的每章复习题，我们给出每个题目的文字答案并选择合适的题目配上图解

# 1.1节

1. **R1 "主机"和“端系统”之间有什么不同？列举几种不同类型的端系统。Web服务器是一种端系统吗?**
    + (1)没有区别。在本书中“主机”和“终端系统”是交换使用的。
    + (2)端系统包括PC、工作站、网络服务器、邮件服务器、智能手机
    + (3)显然Web服务器是一种端系统

2. **R2 协议一次常被用于外交关系，维基百科是怎样描述外交协议的？**
    + 来自维基百科：外交协议常用于描述一系列国家来往规则。这些构建完备和经过时间检验的规则可以使国家和人民生活和工作更简单。协议规则以人民准则为基础，其中的一部分已经作为现在分级的地位声明。

3. **R3 标准对于协议为什么重要？**
    + 标准对于协议来说，可以让人们创建可以相互操作的网络系统和产品

# 1.2节
4. **R4 列出6种接入技术。将它们分类为住宅接入，公司接入或广域无线网接入**
    1. 通过`电话线`的拨号调制解调器： 住宅接入
    2. 通过`电话线`的DSL: 住宅或小办公室
    3. 混合光纤同轴电缆: 住宅
    4. 100M交换以太网: 公司接入
    5. 无线网: 住宅接入或公司接入
    6. 3G、4G和5G : 广域无线网

5. **R5 HFC传输速率在用户间是专用的还是共享的？在下行的HFC信道中，可能出现碰撞吗？为什么?**
    + (1)HFC带宽由用户共享。
    + (2)下行通道中，所有的包都是由头端这一个单一源发出的。因此下行信道中没有碰撞

6. **R6 列出你所在城市中可供使用的住宅接入技术。**
   1. 3G、4G和5G
   2. 光线入户
   3. 无线网

7. **R7 以太LAN的传输速率是多少**
    + `10M` `100M` `1G` `10G`

8. **R8 能够运行以太网的一些物理媒介是什么**
    + 双绞线、光线

9. **R9 拨号调制解调器、HFC、DSL和FTTH都用于住宅接入。对于这些技术，给出每种技术的传输速率的范围，并讨论它们的传输速率是共享的还是专用的**
    1. 调制解调器: 最高56K,带宽专用
    2. HFC(混合光线同轴电缆): 下行 100M ~ 1G , 上行10M ~ 100M, 共享带宽
    3. DSL(数字用户线路): 
       1. ADSL: 下行 8~24M ,上行1~3M, 带宽专用
       2. VDSL2: 下行 50~100M,上行10~20M, 带宽专用
    4. FTTH(光纤到户): 100M ~ 10G ,带宽共享/专用

10. **R10 描述今天最为流行的无线因特网接入技术。对它们进行比较和对照**
    1. WiFi。用于无线局域网，无线用户从辐射范围为几十米的基站传数据包。基站连接无线网络，并为无线用户提供无线网服务。
    2. 4G和5G。大范围无线网，此系统通过电信服务商提供的基站，由蜂窝电话通过一个无线设备传输数据。可以提供基站几十千米范围内的无线网络

# 1.3节
11. **R11 假定在发送主机和接收主机之间只有一台`分组交换机`。发送主机和交换机间以及交换机和接收主机间的传输速率分别是`R1`和`R2`。设该交换机采用存储转发分组交换方式，发送一个长度为L的分组的端到端总时延是多少？（忽略排队时延，传播时延和处理时延）



