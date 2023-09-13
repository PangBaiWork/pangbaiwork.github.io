---
title: Linux桌面使用随机二刺螈壁纸
date: 2023-09-13 16:48:30
tags:
 - Linux
 - shell
---

在 Linux 桌面使用随机二刺螈壁纸
一个很简单的 shell
<!--more-->
## 前言

总所周知，涩图是第一生产力\
如果桌面壁纸单调那么，那一定会影响工作和学习效率 (bushi\
壁纸千千万，均以自然风景和抽象图片居多，在 linux 下更是如此，\
如果我们想要看不完的二次元壁纸，那指定会要点功夫\
以下我利用简单的 shell 脚本实现随机桌面壁纸。

## 材料

- [x]  一个随机壁纸 api:https://api.suyanw.cn/api/comic(可到百度寻找)
- [x] 一个 Linux 壁纸软件:variety (桌面环境自带的也许也行，但是效果不确定)
- [x] 一双手，一个脑子，一只像 PangBai 一样热心网友 (doge)

## 原理

没啥原理，就是通过启动终端的配置文件执行指定脚本，脚本会下载图片，下载完成后替换图片，壁纸自动刷新显示\
因为不希望耗费太多资源，壁纸更换就是当你每次启动终端的时候\
如果你想要定时更换功能，可以自己设计
## 步骤
编辑一个脚本文件，这里我们叫它 start_run, 位置随意
```shell
wget  --no-proxy -q -O  临时壁纸存放位置 https://api.suyanw.cn/api/comic 
if [ $? -eq 0 ]; then
    # 下载成功，替换旧图片
    mv 零食壁纸存放位置 当前壁纸所在位置
else
    # 下载失败，不替换图片
    echo "下载失败"
fi
```
解释一下脚本执行
绕过代理，静默下载壁纸文件
这里存在两个路径，一个是临时壁纸存放位置如 /tmp/download.jpg，用于下载缓冲一个当前壁纸所在位置 (如 /home/pangbai/ 图片 /random.jpg), 这个就是我们在 variety 要选择的壁纸，
有人可能会问，为什么要两个文件呢，那是因为下载壁纸覆盖的过程会导致壁纸空缺，很难看。



修改 shell 启动配置
这里涉及到你使用的 shell 的类型
执行
```shell
chmod +x start_run
echo SHELL
```

查看你当前 shell, 一般是 bash 或者 zsh
bash 就编辑\～/.bashrc,zsh 就编辑\～/.zshrc
在文件最后一行添加脚本执行命令如
```shell
/home/pangbai/start_run &
```
#
修改壁纸设置
把 variety 默认壁纸来源全部取消了
添加壁纸图片 /home/pangbai/图片/random.jpg


## 完结
每次你启动命令行执行命令都会为你更新一次壁纸，很方便呢