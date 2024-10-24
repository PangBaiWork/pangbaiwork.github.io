title: Github CI自动构建使用教程
author: PangBai
date: 2021-09-24 13:39:19
tags:
- Github
categories:
- 随时随地进行软件构建
---
可爱的Github为开发者们提供了一项名为Github CI的功能，帮助开发者在线构建各种项目，为构建耗时项目和没有构建设备的开发者提供帮助
<!--more-->
当一提到开源，每个人应该都会想到~~白嫖~~和全球最大的开源社区-Github。自动构建作为免费开放的功能可谓非常好用。
* * *
### 前言
我找到一个开源的安卓NDK项目，但因为没有电脑，作为小白的我起初使用AIDE进行构建，安装了ndk 支持后发现，构建出的东西没有so文件？重试很多次后我才知道，这东西不支持Cmake?Cmake是个啥？去问度娘后才得到答案。然后我开始考虑用termux容器运行arch Linux进行构建，安装sdkman，安装gradle，ndk没有arm架构的？然后到处查找，在github上找到一个，搞过来用，好的，用不了。纯小白便开始迷茫。逛了逛Github想起来，不是还有github ci吗？以前也没怎么接触过这个，就开始试试吧。
### 教程
条件
- 一个github仓库
- 对git操作流程有一定了解
- 安装git(手机请用termux)

#### 配置Github
仓库的建立和远程控制，网络上已经有了许多教程，在这里就不一一赘述了。
打开Github项目仓库页面，点击Actions
在这个页面寻找你需要构建的项目，展开更多后，可以找到```Android Ci```的选项，选择它。
可以看到如下页面，直接点击下面的 ```Start commit```


打开终端，在你的项目文件夹初始化git仓库
使用ssh将当前仓库和github的仓库建立远程连接

执行
```sh
git pull
```
把远程仓库文件同步过来
然后执行
```sh
cd .github/workflows
```
进入自动构建的配置文件夹
然后对配置文件```android.yml```进行编辑
 - 把java版本更改为8，以提高兼容
 - 确定你需要设置构建的项目文件地址，并且存在gradlew(用于gradle构建)
 - 可以在run后面执行命令行，但不要用cd之类的指令，实测不能使用
 _mulu 
 构建完成后所有生成的文件都会被删除
 - 构建好后的文件可以通过artifact打包成zip，在构建控制页面下载
以下为实例
```yaml
name: Android CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
# 在linux上运行构建(必要的话可以改为windows)
    runs-on: ubuntu-latest

# 初始化构建工具
    steps:
    - uses: actions/checkout@v2
    - name: set up JDK 8
      uses: actions/setup-java@v2
      with:
        java-version: '8'
        distribution: 'adopt'
        cache: gradle
# 修改权限并进行构建
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      run: ./gradlew build
# 查找构建好的文件      
# 这里为寻找.apk和.aar后缀文件的路径
    - name: find apk
      run:  find ./ -regex ".*\.apk\|.*\.aar"
           
# 将构建好的含.apk的文件上传
    - name: Upload APK
      uses: actions/upload-artifact@v2
      with:
          name:  ok
          path: app/build/outputs/apk/debug/*.apk
	  #这里的path路径是来自于find apk那一步的结果
```

修改完成后
将本地仓库推送github，项目的自动构建就会按照配置文件运行
第一次构建成功可能因为路径问题不会上传apk文件 

这时我们需要去github ci的log页面，寻找find apk那一行查看apk所在路径
如果其输出路径为 app/build/outputs/apk/，就打开终端，把配置文件upload APK那一步的path改为
app/build/outputs/apk/*.apk。
保存文件重新将本地仓库推送到github
这次就可以得到构建好的apk了
![githubci](http://shp.qpic.cn/collector/1642981619/8c8a9739-dbf2-412f-9d5e-7b0f6556ac42/0)
