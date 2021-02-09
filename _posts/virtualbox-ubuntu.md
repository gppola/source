---
title: virtualbox ubuntu安装配置
date: 2021-02-09 22:09:53
tags: Linux
---
```
下载 ubuntu-20.04-live-server-amd64.iso
VirtualBox 安装,换成阿里云镜像
http://mirrors.aliyun.com/ubuntu

ssh设置network/NAT port forwarding
TCP 127.0.0.1[Host IP] 2222[Host Port] /[Guest IP] 22[Guest Port]

登录 
ssh -p 2222 username@localhost

免密登录，导入客户端id_rsa.pub到服务器~/.ssh/authorized_keys

共享文件夹
VB选择文件夹映射

sudo mount -t vboxsf share ~/shared
设置自动加载 /etc/fstab 在末尾另起一行，加上
share ~/shared vboxsf rw,gid=100,uid=1000,auto 0 0

命令行控制VB
VBoxManage startvm ubuntu --type headless;
VBoxManage controlvm ubuntu poweroff
```