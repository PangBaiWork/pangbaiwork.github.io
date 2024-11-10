---
title: STM32固件逆向概述
categories: 
- [ CTF ,逆向工程]
tags:
- 汇编
- Re
- STM32
date: 2024-10-10 14:00:25
---

# 系统架构
stm32基于ARM公司设计的Cortex-M系列芯片
# stm32的存储器映射
作为一块32位芯片，stm32把其拥有的4GB地址空间里分配为几大，
详细分配大概是这样
- 0x00000000 ~ 0x1FFFFFFF (代码块) 512MB
- 0x20000000 ~ 0x3FFFFFFF (SRAM内存) 512MB
- 0x40000000 ~ 0x5FFFFFFF (片上外设) 512MB
- 0x60000000 ~ 0xFFFFFFFF (拓展RAM和一些其他的东西)
对于M3系列芯片代码块里作为可编程部分(给flash的)一般只有0x08000000~0x0807FFFF (512KB)
对于M4系列的则为0x08000000~0x081FFFFF（2MB）
其他M系列也大同小异
