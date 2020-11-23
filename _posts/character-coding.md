---
title: ASCII,GB2312,Unicode,UTF8,UTF16
date: 2020-11-10 15:03:56
tags:
---

## ASCII (American Standard Code Information Interchange)

`
可见字符 95个
控制字符 33个
`
![](/images/character_coding/ascii.png)

## 中文

### 设计字符集GB2312
分区管理，共计94个区，每个区含94个位，共8836个码位
01-09区收录除汉字外的682个字符t 
10-15区为空白区，没有使用
16-55区收录3755个一级汉字，按拼音排序
56-87区收录3008个二级汉字，按部首/笔画排序
99-94区位空白区，没有使用

### 存储GB2312

eg 侃 5709

0x57+0xA0（大于127，避开ascii码）=0xD9
0x09+0xA0=0xA9

得到侃的GB2312码: `0xD90XA9`

### 扩展GB2312：GBK

新增20000个汉字和符号，不再规定地位大于127

### 扩展GBK：GB18030

新增几千少数民族字符


## Unicode

不同国家有不同的编码，Unicode统一所有的字符编码

一开始unicode使用UCS-2字符集(Universal Character Set)，16位可表示65536个字符
但还是无法表示世界上所有的字符
后来又有了UCS-4字符集，用32位表示一个字符，可表示近43亿个字符
但是空间过大，不能接受

`
Unicode是用0至65535之间的数字来表示所有字符.其中0至127这128个数字表示的字符仍然跟ASCII完全一样.65536是2的16次方.这是第一步.第二步就是怎么把0至65535这些数字转化成01串保存到计算机中.这肯定就有不同的保存方式了.于是出现了UTF(unicode transformation format),有UTF-8,UTF-16.
`

## UTF-8编码（unicode transformation format）

将UCS-4字符集的码位划分为4个区间
![](/images/character_coding/utf8.png)

## UTF-16编码

在Unicode基本多文种平面定义的字符（无论是拉丁字母、汉字或其他文字或符号），一律使用2字节储存。而在辅助平面定义的字符，会以代理对（surrogate pair）的形式，以两个2字节的值来储存。

UTF-16比起UTF-8，好处在于大部分字符都以固定长度的字节（2字节）储存，但UTF-16却无法兼容于ASCII编码。

Java char类型采用UTF-16编码表示一个代码单元, 辅助平面定义的字符需要用两个char表示

```
String s = "hi\uD83E\uDF60";
System.out.println(s);
System.out.println(s.length());
int cpCount = s.codePointCount(0, s.length());
System.out.println(cpCount);
char ch = s.charAt(2);
System.out.println(ch);

hi🭠
4
3
?
```


