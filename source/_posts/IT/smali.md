title: Smali踩坑纪录
author: PangBai
date: 2021-01-16 13:39:19
tags:
- 安卓
categories:
- 安卓逆向
---
有关于安卓逆向的一篇日记，如何二改＆破解软件？来看看这篇很勉强的技术博客吧。
<!--more-->
关于安卓逆向，大部分的作用都是用于破解软件(一些付费的软件)，另外的就是二改软件(软件汉化，反人类的操作修改)，当然这里面或多或少是涉及到软件提供商的利益问题的，一切利弊由修改者衡量。
这篇文章本人的入坑笔记，因为是小白，一些见解可能不太正确，如果你也有一点见解，可以在本页评论交流。

注: 可能需要一些安卓开发基础知识，可以自行百度
```本文由@PangBai原创，转载请注明原地址```
## 安卓安卓包
### 目录结构
关于安卓安装包，大部分是以```.apk```为后缀，当然，也有.apks和.abb,解压后查看目录大致也相差不大。

apk文件中有许多已编译的文件(字节码形式)，这些我们自然看不懂
如AndroidManifest.xml，classes.dex，resources.arsc
既然看不懂，我们就得找翻译吧，使用工具**Apktool**进行反编译，手机上请用**Apktool_M** （你一定要用MT我也没办法）


以下为反编译后目录内容(也许会出现其他文件夹，但大多与逆向不会有太大关系)

 - assets
 - lib
 - smali
 - res
  - layout
  - color
  - anim
  - drawable
  - raw
  - xml
  

```assets```是一个储存大型资源的目录，可能占apk绝大多的字节大小，```储存其中的资源不会被编译，可以直接修改```，软件运行时会以压缩文件形式解压出来使用。(根据我的经验，html形式文件可以通过ur定位直接访问而其他形式的文件必须需要解压)

```lib```存放已编译的库文件，十分重要，进行逆向时直接忽视它。

```smali```存放反编译出的smali文件(来自classes.dex文件)，是我们进行逆向的主角


```res```存放资源文件，如布局，配色，样式，图片，大多为xml和png形式。
其中```raw```文件夹十分特别，该文件夹下不会被编译(同assets相似，但它还会额外为文件分配一个id)，可以存放任何形式文件。


## Smali

smali是一种类似于汇编语言的语法，可以打包为dex文件运行在Dalvik里（Android虚拟机） 。

### Smali文件
我们在smali文件夹里能看到许多smali文件，它们的是按一定规则进行命名的。


每一个smali文件都对应对应一个java类，如果有一个**activity.java**，那么编译后就有**activity.smali**。我们也许还可以看到另外的**activity\$InnerClass.smali**这样的文件，这说明**activity.java**里面还有一个叫**InnerClass**的内部类。如果出现**activity\$a.smali**或**activity\$b.smali**这种无意义的内部类名称，那就可能是个匿名内部类。
### Smali语法
### 数据类型
- [ ] B---byte 
- [ ] C---char 
- [ ] D---double 
- [ ] F---float 
 - [ ] I---int 
- [ ] J---long 
- [ ] S---short 
- [ ] V---void 
- [ ] Z---boolean 
- [ ] [xxx---array 
- [ ] Lxxx/yyy---object

每种基本数据类型对应一个字母
**[+数据类型**表示一个数组，如[D表示一个double类型数组。
**L+类路径**表示一个对象，如String类型数据使用的是**Ljava/lang/String**

### 类
``` smali
.class public abstract LCar; //abstract 表明为抽象类

.super Ljava/lang/Object;

.source "Car.java"


.implements LIFly;

```
**.class**声明一个类并添加修饰
**.super**声明继承的父类
**.source**声明类来源的文件
**.implements**声明类实现的接口


