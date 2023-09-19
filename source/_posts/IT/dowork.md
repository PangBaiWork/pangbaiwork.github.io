---
title: Dowork食用教程
date: 2023-09-18 17:00:21
tags:
 - Dowork
sticky: true
---
Dowork是一个在安卓运行Linux容器并且集成了终端,图形化显示,音频输出,容器管理等功能的多合一管理器

<!--more-->
## 前言
如果有人用过```Linuxdeploy```，那必定吐槽过这玩意的难用，使用其安装Linux，需要时刻关注天朝内部网络天气，并且准备好魔法上网，生成```bootstrap```的过程让人等待到厌烦，如果遇到一个包下载超时，那便只有两个选择: 从头开始安装和等待30分钟wget会自动重新下载。
如果你也遇到过以下情况: 用它启动ssh是否成功全靠运气,无法安装根文件系统,安装完成后的Linux残缺。那是时候考虑换个软件了。没错说的就是你```Dowork```
## 简单介绍
```Dowork```融合了Linuxdeploy的后端,同时支持无root权限使用,在享受到Linuxdeploy自动脚本的方便的同时,也加入了```Rootfs```在线获取和下载功能,你可以通过```rootfstool```命令行工具获取到最新构建的发行版Rootfs(终于不用看网络脸色了)。
```Dowork```使用了来自Termux的终端,提供可控性操作,按键布局继承自"Tmoe"脚本,方便实用。
```Dowork```使用Xvfb作为显示后端,提供两种显示模式(内部/外部Xvfb)，数据传输使用unix套接字(比XSDL和VNC传输性能好得多),同时支持触控会手势滚动,文本输入支持中文
```Dowork```内置pulseaduio,提供音频输出。
```Dowork```提供悬浮命令行任务输出,摸鱼和学Linux两不误
[下载Dowork(GITHUB)](https://github.com/PangBaiWork/Dowork/releases)
## 使用教程
### 创建容器
在容器页面点击加号即可创建
创建后请修改以下数据
 - 容器类型 
 根据是否具有root权限选择,若无root权限,只可选择```proot```
 - 发行版GNU/Linux
 请确定自己偏好的Linux发行版
 - 安装类型
 请选择```目录```安装(其他类型暂时未适配)
 - 安装目录
 此安装目录可自定义,一个容器对应一个目录
 若容器为proot,请默认安装在/data/data/com.pangbai.dowork/files/下目录
 若容器为chroot,任何具有执行权限的目录都可安装(最好是/data下目录)
### 安装rootfs
rootfs(根文件系统)包含容器运行所需所有文件
Dowork提供两种方式安装rootfs(Proot和Chroot都可以使用)
 - 在线构建rootfs
在线构建bootstrap与Linuxdeploy使用方法相同,考虑到网络状况，可能需要修改软件源
此方法较为反人类，此处略过。
 - 导入rootfs
返回到软件主页，点击终端符号可进入安卓shell环境(此环境在容器未安装时可用)
![进入终端](http://shp.qpic.cn/collector/1642981619/f30928dc-b327-48ac-8303-fc235be84be8/0)
进入终端后输入命令行，可查看rootfstool用法和提示
```shell
rootfstool help
```
若我们想安装最新版本的centos9,可输入以下命令获得rootfs的url，其他发行版同理
```shell
rootfstool url -a arm64 -d centos -v 9 -m bfsu
```
获取到url后我们就可以拿去导入了,在容器配置页面,点击右上角中间的按钮，把url填入，进行导入，等待导入完成，提示Succeed即导入成功。
![导入容器](http://shp.qpic.cn/collector/1642981619/334ec5fb-a476-4898-92c9-e109a25f263f/0)
此时我们返回软件主页可以看到容器的版本信息已经识别出来了
![版本信息](http://shp.qpic.cn/collector/1642981619/c0bbb380-7846-4bf9-8cb2-e40bd4aca7b1/0)
是否显示版本信息是```Dowork```判断容器是否可用的标志
此时我们再点击终端符号，就会直接进入容器
至此容器安装完成

### 图形化显示以及音频输出
 - 图形化显示
 若无特殊需求请选择内置X服务器,在显示页面可对显示屏幕进行设置,由于显示屏幕是横向放置,第一个参数width不应该大于屏幕高度，第二个参数height不应该大于屏幕宽度，否则会出现显示画面闪烁和崩溃，第三个参数为显示色深,如果没有特殊需求，不建议更改。更改后重启```Dowork```，若Xserver状态为running即正常。
 ```Dowork```会自动嗅探图形化软件的打开和关闭,并且提供浮窗显示,同时可以进行点击,滚动页面等操作，如果你是一个想在安卓上进行跨平台图形化开发的开发者，你会爱上这个功能(相比于termux-x11等应用，满足了开发者快速图形化调试的需求)。
 Dowork已在容器内预设DISPLAY环境变量,只需直接启动图形化软件即可显示
 - 音频输出到pulseaudio
 音频输出需要在容器内配置音频服务
 若是chroot容器，可以直接在容器配置页面勾选```启用允许音频输出```
 若为proot容器需要手动安装软件包(Linuxdeploy的配置页面是为chroot准备的，适配需要些时间)
  - debian系安装libasound2-plugins
  - arch系安装pulseaudio-alsa
  - centos系安装alsa-plugins-pulseaudio
  最后创建/etc/asound.conf文件,加入以下内容
  ```shell
  pcm.!default { type pulse }
  ctl.!default { type pulse }
  pcm.pulse { type pulse }
  ctl.pulse { type pulse }
  ```
 至此音频配置完成
 可以使用aplay命令测试是否成功
 ```shell
 aplay *mp3
 ```
### 容器配置和更新(仅限chroot)
在容器配置页面更改配置并退出
在容器管理页面点击更新按钮
![更新容器配置](http://shp.qpic.cn/collector/1642981619/9e4cd613-1b99-418e-9242-8464406f64cd/0)

### 进程管理
在主页会对已运行的chroot进程进行探测,可以进行手动管理
若点击umount container将会终止所有chroot进程并卸载容器
![进程管理](http://shp.qpic.cn/collector/1642981619/67a5c5ea-e46e-4934-a31e-193eee244510/0)
 
 
 