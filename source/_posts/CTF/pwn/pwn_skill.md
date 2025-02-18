---
title: PWN相关的奇淫技巧
categories: CTF
date: 2023-12-23 12:19:19
tags:
- 汇编
---
技巧。。学个pwn吧
<!--more-->
## 启动shell的几种方式
 - [x] /bin/sh
 - [x] $0
## sendline与send
sendline比send在数据末尾加一个换行符，在read进行输入操作的时候要格外主要send和sendline的使用时机。
当read读取IO的字节数量为40个时，我们用sendline输入了40个字符串，但是末尾残留一个换行符，可能会导致下一次read直接读取这个缓冲中的换行符导致输入数据错误