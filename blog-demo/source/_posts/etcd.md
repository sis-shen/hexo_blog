---
title: etcd服务发现功能使用探究
date: 2025-04-17 09:09:17
tags:
---
# 简介

# 安装

## 在宿主机上安装服务

```shell
sudo apt update -y
# 安装
sudo apt-get install etcd 
# 启动
sudo systemctl start etcd
# 配置开机自启
sudo systemctl enable etcd

```

## 在Docker中运行etcd容器

```shell
# 从官方镜像仓库拉取
git pull gcr.io/etcd-development/etcd:v3.5.19
# 从国内镜像源拉取
git pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/gcr.io/etcd-development/etcd:v3.5.19
```