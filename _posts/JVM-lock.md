---
title: JVM lock
date: 2020-12-10 17:57:43
tags: JVM
---



## 对象存储布局

- markword: 8 bytes

- class pointer: 4 bytes (java 运行默认参数 开启 compress class pointer

  64 地址8字节， 压缩为4字节)

- instance data

- padding对齐