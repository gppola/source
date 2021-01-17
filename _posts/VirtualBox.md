---
title: Mac通过Mosh连接虚拟机
date: 2021-01-17 22:38:46
tags: Linux
---

## VirtualBox 
下载Ubuntu Server并安装
配置shared folder,映射本机文件
headless启动ubuntu server实例

## Mosh 连接
Mac安装mosh客户端

Ubuntu安装mosh
```
apt-get update
apt-get install -y mosh
mosh username@ip
```
mosh不会像ssh那样过一会儿断开，推荐使用mosh