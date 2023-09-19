title: 干货！安卓手机上kali
date: 2021-7-18 10:24:58
author: PangBai                                        
tags:
- linux
- kail
- 渗透
- Android
categories:
- 在安卓上安装你的kail
cover: http://shp.qpic.cn/collector/1642981619/5d10a393-4056-405f-8cf6-1b0520c3f0b8/0
---
### `在你的任意安卓手机上刷入kali`
在手机上稳定便捷运行linux
<!--more-->
如果你了解过网络安全方面的知识，那你一定会听过```Kali Linux ```的名字

它被广泛使用于网络攻击，不过分的说，多年来，进攻安全系统一直以Kali Linux的系统主导市场。
而`Kali Linux NetHunter`则是一款适用于Android移动设备的Kali Linux渗透测试系统。(但从攻击工具的使用上，ANDRAX似乎比已经过时的它更加好用)

我是一个高中生，没有什么很厉害等想法，也没有想过黑客之类的东西，只想用手机玩玩linux，看着指令行，用着电脑的图形页面很惬意。手机已经root自然不用去搞termux了，我把目光转向chroot。于是安装了`linuxdeploy`，它可以选择不同的linux发行版与架构，非常完美，但我不认为它足够稳定。使用目录安装基本只会失败，而使用镜像更加鸡肋。经常挂载失败无法启动镜像，不是实体文件夹，难已备份数据，vnc启动易死，只要有些东西错误，只能前功尽弃，然后重新安装linux，又等个一个小时把之前的重新做一遍。

于是，我想到了直接在手机上刷入linux的想法，然后发现了nethunter，起初以为我的手机不支持这东西(名单里似乎只有一加和谷歌的亲儿子)，逛了几个博客发现，有个通刷包，只是不能使用外接监听网卡。很激动，花了很时间线，最后成功了。

# 在手机上刷入kali

接下来进入主题，如何在安卓手机上刷人Kali Linux NetHunter

```注: 本篇文章，只用于技术交流，旨在方便chroot运行linux，任何使用工具照成的后果与本人无关```

作者: ```PangBai```
``转载请注明来源``

准备条件
 - 一台已root的手机(带twrp就不用说了)
 - nethunter通刷包
 - 一定的刷机经验
 - 备份数据和救砖准备(其实失败几率很小)

## 下载kali nethunter
这里给出三个刷机包，请根据需要选择
arm 64位最新版
[ARM64](https://images.kali.org/nethunter/nethunter-2021.2-generic-arm64-kalifs-full.zip)
arm 32位最新版
[ARM32](https://images.kali.org/nethunter/nethunter-2021.2-generic-armhf-kalifs-full.zip)
arm 32位旧版
[ARM32](https://build.nethunter.com/installer/generic/update-nethunter-generic-armhf-20171007_215146.zip)
如果你仅仅想安装一个linux请选最后一个链接，包很小，但很多组件是旧的(upgrade直接几个G给你吃了，还耗了几个小时)

## 安装kali nethunter
手机关机，再按开机键和音量键，进入TWRP
![TWRP图](http://shp.qpic.cn/collector/1642981619/678ec442-bfd7-4ef7-bd6c-13beb8f638c7/0)
滑动允许修改系统
点击右上的清除按钮，
滑动清除(其实也许并不是特别需要)
返回主页
点击安装按钮
选择之前下载的nethunter包
刷入
等待......(包解压时会耗费很长时间)
最后成功
启动系统
开机动画很炫酷，内心激动不已
![nethunter开机图](http://shp.qpic.cn/collector/1642981619/11490d8f-43e1-4150-aa9a-a79e20095a39/0)
开机后与原来的手机没有差别，一样的锁屏，桌面，安卓软件
唯一特殊的是多了几个特别的APP
## 配置
刷入的kali系统，目录在/data/local/nhsystem/下面

新增的安卓软件有以下几个
NetHunter      -->kali系统的控制软件
NetHunter终端  -->命令行执行软件
Nethunter vnc  -->vnc链接软件
cSploit        -->移动端便捷的一键渗透工具
Hacker's keyboard -->键盘，并不好用
还有一些我不知道用途

首先进入 nethunter终端 弹出对话框
可以选择命令的执行端，果断选KALI

apt update 发现？？不行，需要换源
去安装目录下更改etc/apt/sources.list
把内容全部换掉
```java sources.list
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
```
然后运行
apt update
如果失败请去百度搜索有关数字签名教程

接下来是启动vnc图形页面
打开Nethunter
点击VNC Manager
点击SETUP LOCAL SERVER
设置密码
然后输入以下启动指令
```java VNC 
export USER=root
export HOME=/root

vncserver -kill :1
rm -rf /tmp/.X1-lock
rm -rf /tmp/.X11-unix/X1

vncserver -name remote-desktop :1
```

打开VNC链接软件(推荐去下载第三方的vnc，nethunter vnc不好用)
地址: 127.0.0.1:5901
账号: root
密码: 你设置的密码

然后看到了熟悉的桌面
![kali桌面](https://shp.qpic.cn/collector/1642981619/32a2cfd6-51e7-4fd4-ab14-e6df705e084a/0)


