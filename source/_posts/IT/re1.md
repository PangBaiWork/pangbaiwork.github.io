title: 汇编学习第一天
author: PangBai
date: 2023-09-23 10:39:19
tags:
- 汇编语言
categories:
- 逆向
---
计算机CPU相关基础知识
<!--more-->
## 基本知识
 - 所有编译语言都是将可读性描述转化为二进制机械语言进行执行
 通用计算机语言->汇编语言->二进制语言
 - 计算机几乎所有组件都有存储器
 CPU最直接的存储器是寄存器
 
 - CPU可以对所有存储器读写
 CPU接有直接到达各个计算机部件的电路
 CPU可以对存储器进行三类交互
+++danger 三种交互类型
 [1]{.label .info}地址信息 (通过地址总线)
 [2]{.label .info}控制信息 (通过控制总线)
 [3]{.label .info}数据信息 (通过数据总线)
+++
 - 程序执行流程
 硬盘文件->虚拟内存->CPU
 - 存储器芯片
 分为RAM和ROM
+++info 存储器芯片分类
内存->RAM
接口->RAM
bios->ROM
+++
 - 内存地址空间(物理地址)
 逻辑上所有计算机接入设备都是相连的一大块
 CPU可见的是一大块的存储器(包含内存,显卡,网卡...)
 系统为存储器分配地址空间
 [举个栗子]{.label .danger} 主随机存储器地址为0~7FFFH,显存地址为8000H~9FFFH,各ROM地址为A000H~FFFFH
 
## 地址总线与CPU寻址
CPU到内存的总线有很多条(逻辑上),具体数目与CPU有关,一条总线可以传输一位数据(0/1),将所有总线的数据联合起来就组成了地址信息,主流处理器分32位和64位(也叫地址宽度),在地址总线上就对应了CPU能访问到的地址数目2^32和2^64。
[举个栗子]{.label .info}当CPU从地址总线发送数据(11010000...)到内存，内存会定位到这个地址,这个地址的所在的内存单元数据(通常为1byte)也就被寻找到了。
## 数据总线与数据传输
数据总线的宽度决定速度传输速度
8088CPU是8位的，一次可以传输8位数据
8086CPU是16位的，一次可以传输16位数据
## 控制总线
控制总线发送控制信息(通常是读和写)
控制接入电脑的硬件。
## CPU概述
一个典型CPU由```运算器```,```控制器```,```寄存器```组成，这些部件靠CPU内部总线相连。
## 寄存器概述
x86寄存器一般是16位的，可以存放两个字节(等于一个字)，AX,BX,CX,DX存储一般性数据,可以分为两个独立的8位使用，称为通用寄存器
通用寄存器一般为8个，其他寄存器可以有若干
AX等带X后缀的都为16位,而EAX之类为32位
通用寄存器可以分为高位和地位如AH和AL,BH和BL(用于兼容旧系统)

