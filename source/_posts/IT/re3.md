title: 汇编学习-汇编语法
author: PangBai
date: 2023-09-24 12:19:19
tags:
- 汇编语言
categories:
- 逆向
---
汇编语法
<!--more-->
# 基础语法
## 标号
为一个段命名,标号指代了一个地址,如codesg
```nasm 
codesg segment
```
这个段由以下命令定义结束
```nasm
codesg ends
```
## 声明段的类型
段的类型可能是代码也可能是数据,需要加以声明```assume```
```nasm
assume cs:codesg
```
## 几种结束
 - 段结束
```nasm
段名 ends
```
 - 程序结束
```nasm
end
```
 - 程序返回
```nasm
 mov ax, 4c00H
 int 21H
```

## 示例程序
编写1.asm
```nasm
   assume cs:codesg

   codesg segment
     mov ax, 2
     add ax, ax
     add ax, ax
     mov ax, 4c00H
     int 21H
   codesg ends
   
   end
```
执行编译和链接
```shell
masm 1.asm
link 1.obj
```
## 设置程序入口
用一个标号标识,用```end 标号```结束程序入口


```nasm
   assume cs:codesg

   codesg segment
   start:    mov ax, 2
            add ax, ax
            add ax, ax
            mov ax, 4c00H
            int 21H
   codesg ends
   
   end start
```
+++info 小知识
程序被加载到内存后,还会在其前方设置一段内存设置代码(psp)用于与程序通信，运行时并不会从程序直接开始,而是从psp开始
+++
## 数据的存储
对于x86CPU,数据在内存中是小端存储
我们规定关于一个地址编号
0x0001->0xFFFF
前为低地址,后为高地址,是按内存编号的方向区分
对于数据0x1234(十六进制数据,需要16个比特位存储)
需要拆分为两个字节存储
+++infor 注意
在计算机中一个字节的存储可以完整存储，是没有大小端之分的
+++
这个数据需要在两个相邻地址中存储，这就涉及到了谁先谁后的问题。
大端存储: 高位数据0x1200需要存储在低地址,低位数据0x34需要存储在高地址。
小端存储: 低位数据存储在低地址，高位数据存储在高地址。

x86CPU小端存储的数据读入CPU寄存器的时候都会自动反转字节

