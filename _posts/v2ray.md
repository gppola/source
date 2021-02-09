---
title: 搭梯子
date: 2021-02-08 21:57:44
tags: Linux
---

## Server
服务器 google cloud n1-standard-1（1 个 vCPU，3.75 GB 内存）
区域
asia-east2-a (hong kong)
50美金/月左右
```
bash <(curl -s -L https://git.io/v2ray.sh)

port: 37307

# firewalld放行端口（适用于CentOS7/8）
firewall-cmd --permanent --add-port=37307/tcp # 37307改成你配置文件中的端口号
firewall-cmd --reload


ss -ntlp | grep v2ray 命令可以查看v2ray是否正在运行

生成url链接导入客户端
```

## Mac
url 导入 v2ray-core即可
![](/images/v2ray/mac.png)

## Chrome
![](/images/v2ray/omega1.png)
![](/images/v2ray/omega1.png)


## iPhone

Apple Store 购买 shadowrocket $2.99
扫描server生成的url链接的二维码课直接导入