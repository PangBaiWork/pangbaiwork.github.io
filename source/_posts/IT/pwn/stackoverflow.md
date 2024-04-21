---
title: 栈溢出基础
categories: CTF
tags:
- 汇编
---
我就学学吧,开开眼界也好
<!--more-->
# 重要芝士
PLT表存放跳转相关指令，GOT表存放外部函数（符号）地址
plt在ida是蓝色的，got在ida是粉色的，plt可跳转got表
# 32位调用约定
 - [x] 普通函数调用
 参数按函数签名由右往左入栈
 