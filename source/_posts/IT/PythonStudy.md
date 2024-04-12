---
title: Python学习
categories: 
tags:
- IT
- 编程
---
Python脚本，用来写点有用东西
<!--more-->
## 大学物理，不确定度计算
```python
import math
from statistics import mean
#数据，有几组写几组
v1=[35.95,36.00,35.91,35.90]
#仪器的最小分度值
min=0.1

avg=mean(v1)
print("数据平均值",avg)
sum=0.0
for i in range(len(v1)):
    sum+=(v1[i]-avg)**2
print("A类不确定度: ")
u1=math.sqrt( sum/(len(v1)**2-len(v1))) 
print(u1)

print("B类不确定度: ")
u2=min/math.sqrt(3)
print(u2)
print("总不确定度:  ")
print(math.sqrt(u2**2+u1**2))
```