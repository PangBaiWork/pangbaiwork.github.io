---
title: Frida逆向实战-Live2dViewerEx破解
categories: 
- [ CTF ,逆向工程]
tags:
- 汇编
- Re
- frida
date: 2024-11-24 14:00:25
---
Live2dViewerEx，是一个安卓Live2D动态壁纸软件，这篇文章记录了对它的破解过程
<!--more-->
Live2dViewerEx是一个老牌软件了，相当于PC端的WallpaperEngine，还移植了Steam的创意工坊，安卓5.1时代我就用他当手机壁纸软件（谁能拒绝在桌面养一直萌娘呢）。我在中学生时代曾尝试过破解这个软件（无限点数，创意工坊的kawaii模型不就可以随便下了吗），但最后以失败告终（当时只会看Java层，改了几个类只能修改表层点数，真正的点数并没有办法修改）。如今逆向技术有成重新回过头来分析乐趣无穷。

![桌面](http://shp.qpic.cn/collector/1642981619/323249b8-fa7e-4f39-85d6-b6f358ddee65/0)

## 软件框架
Live2dViewerEx并不是原生应用，他核心由Unity编写，再与Java层通信渲染UI界面，软件逆向的核心是安卓Unity逆向。
## 逆向工具
Frida，Ida64，Il2CppDumper，Jadx-gui
## Java层分析
通过Java层字符串索引，和OnClick函数调用堆栈，锁定到UnityMessenger类。这个类是处理Unity和Java层的消息传递的，
`onReceive`类接收了字符串通过base64解密转化为JSON对象
```java
Java.perform(function () {
    let UnityMessenger = Java.use("com.pavostudio.exlib.unity.UnityMessenger");
    UnityMessenger["onReceive"].implementation = function (str) {

        var Base64 = Java.use("android.util.Base64");

        // 你要解码的 Base64 编码字符串
        var base64Str = str; // 这是 "Hello world!" 的 Base64 编码

        // 使用 Base64 解码
        var decodedBytes = Base64.decode(base64Str, 2);

        // 将字节数组转换为字符串
        var decodedStr = Java.use("java.lang.String").$new(decodedBytes);



        var jsonObj = JSON.parse(decodedStr);

        // // 修改 JSON 对象
        // if (jsonObj.msg == 1006 || jsonObj.msg == 1093 || jsonObj.msg == 1094) {
        //     jsonObj.i = 100;  // 例如，修改 someKey 的值
        // }

        if (jsonObj.msg == 1006) {
            let innerJsonStr = jsonObj.s;
            let innerJsonObj = JSON.parse(innerJsonStr);

            // 修改 DownloadCostPoint 字段
            innerJsonObj.DownloadCostPoint = 1;  // 修改为你想要的新值
            innerJsonObj.downloadCostPoint=1;
            innerJsonObj.minWatchReward = 1000000;
            innerJsonObj.maxWatchReward = 1000001;
            // 将修改后的内嵌 JSON 对象转换回字符串
            jsonObj.s = JSON.stringify(innerJsonObj);


        }
        if (jsonObj.msg == 1008) {
            let innerJsonStr = jsonObj.s;
            let innerJsonObj = JSON.parse(innerJsonStr);
           
            let innerJsonObj1 =innerJsonObj.setting;
            // 修改 DownloadCostPoint 字段
            innerJsonObj1.DownloadCostPoint = 1;  // 修改为你想要的新值
            innerJsonObj1.downloadCostPoint=1;
            innerJsonObj1.minWatchReward = 1000000;
            innerJsonObj1.maxWatchReward = 1000001;
            // 将修改后的内嵌 JSON 对象转换回字符串
            innerJsonObj.setting = JSON.stringify(innerJsonObj1);


            jsonObj.s = JSON.stringify(innerJsonObj);


        }
        // 将修改后的对象转回 JSON 字符串
        var modifiedJsonStr = JSON.stringify(jsonObj);


        // 输出解码后的字符串
        var bytes = Java.use("java.lang.String").$new(modifiedJsonStr).getBytes();

        // 使用 Base64 编码字节数组
        var base64Encoded = Base64.encodeToString(bytes, 2);

        if (jsonObj.msg == 1093) {
            console.log('修改后的 JSON 字符串: ' + modifiedJsonStr);
           console.log(`UnityMessenger.onReceive is called: str=${base64Encoded}`);
       }
        this["onReceive"](base64Encoded);
    };
})

```

尝试hook并修改消息内容，和以前效果一样，只是安卓端UI界面改变，下载模型点数任然按受限。
Java层的Jadx逆向数据虽然只是前端显示，但我们可以借此分析消息协议。
如：消息号1006为同步点数的消息，消息号1093为看广告增加的点数消息
## So层符号恢复
`libil2cpp.so`里是Unity代码的主逻辑，ida直接分析缺失符号信息，assets里的`global-metadata.dat`含有字符串和符号信息。
用Il2CppDumper可以生成符号导入脚本和C#的类信息。
IDA使用运行符号导入脚本，等待重新分析完成即可，时间有点长，这个时候可以看看Il2CppDumper生成的DLL文件。用dnspy打开Assembly-CSharp.dll看看，这里只有类结构和空的函数头，函数主体并不在内，但是我们可以借此看到函数偏移和类的结构信息。搜索UnityMessenger，发现一个 `onReceive `函数，还有一个`SendUnityMessage`函数，感觉很可疑。Java层的`onReceive`收到来自Unity的消息来渲染前端，那么发送这个信息的一定是一个Send。

## Frida分析Unity消息系统
等等ida分析完成进行hook`UnityMessenger_SendUnityMessage'。
```js
var mInterval = setInterval(function () {
    var sgame = Process.findModuleByName("libil2cpp.so");
    if (sgame == null) {
        console.log("无");
        return;
    }
    clearInterval(mInterval);
   
  var  addr7= sgame.base.add(0x1C2D3F4 )
    console.log("" +addr7);
    console.log(Instruction.parse(addr7).toString());
    Interceptor.attach(addr7, {
        onEnter: function (args) {
             console.log("Send2 arg1: \n"+hexdump(ptr(args[0]), { length: 100, ansi: true }));
            console.log("Send2 arg2: \n"+hexdump(ptr(args[1]), { length: 100, ansi: true }));
        }
    })
}, 1) 


```

hook到的消息正是Java层分析时`onReceive`接收到的消息，可以判断Unity消息由此函数发送。
交叉索引这个函数可以得到很多调用，那到底是那个调用发送点数消息呢，借助Java层分析可知消息号1006是同步增加的点数消息，我们在hook里打印`SendUnityMessage`函数调用堆栈，再找到消息号1006下的调用堆栈就可以知道1006同步点数消息从哪个地方被发送。


```js
var mInterval = setInterval(function () {
    var sgame = Process.findModuleByName("libil2cpp.so");
    if (sgame == null) {
        console.log("无");
        return;
    }
    clearInterval(mInterval);
   
  var  addr7= sgame.base.add(0x1C2D3F4 )
    console.log("" +addr7);
    console.log(Instruction.parse(addr7).toString());
    Interceptor.attach(addr7, {
        onEnter: function (args) {
            // console.log("Send2 arg1: \n"+hexdump(ptr(args[0]), { length: 100, ansi: true }));
            console.log("Send2 arg2: \n"+hexdump(ptr(args[1]), { length: 100, ansi: true }));
            console.log("Backtrace:\n" + Thread.backtrace(this.context, Backtracer.ACCURATE)
            .map(DebugSymbol.fromAddress).join("\n"));
       }
    })
}, 1) 


```

frida打印如下消息

```bash
Send2 arg2:
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
aae4f990  80 ae 56 f5 00 00 00 00 4c 01 00 00 7b 00 22 00  ..V.....L...{.".
aae4f9a0  6d 00 73 00 67 00 22 00 3a 00 31 00 30 00 30 00  m.s.g.".:.1.0.0.
aae4f9b0  36 00 2c 00 22 00 69 00 22 00 3a 00 31 00 30 00  6.,.".i.".:.1.4.
aae4f9c0  32 00 34 00 2c 00 22 00 69 00 32 00 22 00 3a 00  0.0.,.".i.2.".:.
aae4f9d0  30 00 2c 00 22 00 69 00 33 00 22 00 3a 00 30 00  0.,.".i.3.".:.0.
aae4f9e0  2c 00 22 00 69 00 61 00 22 00 3a 00 5b 00 5d 00  ,.".i.a.".:.[.].
aae4f9f0  2c 00 22 00                                      ,.".
Backtrace:
0xb5a01950 libil2cpp.so!0x1e01950
0xb5a012c4 libil2cpp.so!0x1e012c4
0xb583c0ac libil2cpp.so!0x1c3c0ac
0xb5bed380 libil2cpp.so!0x1fed380
0xb5bedc9c libil2cpp.so!0x1fedc9c
0xb408a128 libil2cpp.so!0x48a128
0xb3fb69c0 libil2cpp.so!0x3b69c0
0xb9da76c7 libunity.so!0x1e66c7
0xb9da9eaf libunity.so!0x1e8eaf
0xb9db823b libunity.so!0x1f723b
0xb9cd3c3d libunity.so!0x112c3d
0xb9d41cd3 libunity.so!0x180cd3
0xb9d41cf1 libunity.so!0x180cf1
0xb9d41e8d libunity.so!0x180e8d
0xb9dfc93b libunity.so!0x23b93b
0xb9e0a8a9 libunity.so!0x2498a9
```
消息是json字符串，其中`msg`是消息号1006，`i`正是我当前点数14。
可以锁定点数同步调用地址`0x1e01950`,IDA查看伪代码，核心如下。

```c
  v45 = UnityMessage__FPMMBDMKELA(1006);
  v46 = *(unsigned __int8 *)(GMAOFAGNCIO_TypeInfo + 187);
  if ( (v46 & 2) != 0 )
  {
    v46 = *(_DWORD *)(GMAOFAGNCIO_TypeInfo + 116);
    if ( !v46 )
      j_il2cpp_runtime_class_init_0(GMAOFAGNCIO_TypeInfo, 0, v44);
  }
  v47 = GMAOFAGNCIO__LJGKLPOHDKL(0, v46, v44);
  if ( !v45 )
    null(v47);
  v48 = UnityMessage__JDPPCOGEFJE(v45, v47);
  v49 = *(_DWORD *)(GMAOFAGNCIO_TypeInfo + 92);
  v50 = *(_DWORD *)(v49 + 16);
  if ( !v50 )
    null(v49);
  v51 = *(_DWORD **)(v50 + 76);
  if ( !v51 )
    null(v49);
  v53 = AdData_AdSetting__FDIJNPDBFNH(v51);
  if ( (*(_BYTE *)(EOEAIKOEFDF_TypeInfo + 187) & 2) != 0 && !*(_DWORD *)(EOEAIKOEFDF_TypeInfo + 116) )
    j_il2cpp_runtime_class_init_0(EOEAIKOEFDF_TypeInfo, 0, v52);
  v54 = toJSon((int)v53, 1, 0);
  if ( !v48 )
    null(v54);
  v55 = UnityMessage__CIJAGENPKCD(v48, v54);
  if ( !v55 )
    null(0);
  return UnityMessage__OPBMOAEIAIF(v55, 0);
```

`UnityMessage__FPMMBDMKELA(1006)`应该是消息号声明，后面一段直到发送消息为止都是构建消息内容的调用，我们只要分析这几行代码就行了。
对`GMAOFAGNCIO__LJGKLPOHDKL(0, v46, v44)`进行函数的基本礼仪（交叉引用）。查看其中一个引用`UnityMessageHandler__LAJNIAIBCOK`时代码如下
```c
 if ( GMAOFAGNCIO__LJGKLPOHDKL(0, v7, a3) < a2 * a1 )
    {
      v8 = UnityMessage__FPMMBDMKELA(1091);
      if ( !v8 )
        null(0);
      v6 = 0;
      UnityMessenger__SendUnityMessage(v8, 0);
    }
```
在消息接收处理函数里居然有对这行返回值大小的判断，可以大胆猜测这是Java传递的消耗点数的消息。
为了验证这一点，我们hook这里就好了
```js
var mInterval = setInterval(function () {
    var sgame = Process.findModuleByName("libil2cpp.so");
    if (sgame == null) {
        console.log("无");
        return;
    }
    clearInterval(mInterval);
   
  var  addr7= sgame.base.add(0x1C2D3F4 )
    console.log("" +addr7);
    console.log(Instruction.parse(addr7).toString());
    Interceptor.attach(addr7, {
        onEnter: function (args) {
            // console.log("Send2 arg1: \n"+hexdump(ptr(args[0]), { length: 100, ansi: true }));
            console.log("Send2 arg2: \n"+hexdump(ptr(args[1]), { length: 100, ansi: true }));
            console.log("Backtrace:\n" + Thread.backtrace(this.context, Backtracer.ACCURATE)
            .map(DebugSymbol.fromAddress).join("\n"));
       }
    })

        var  addr7= sgame.base.add(0x022F84F4)
    console.log("" +addr7);
    console.log(Instruction.parse(addr7).toString());
    Interceptor.attach(addr7, {
        onEnter: function (args) {
        //   console.log("gess arg1: "+hexdump(ptr(this.context.r5), { length: 100, ansi: true }))
        },
        onLeave: function (retval) {
           console.log("FunRet: "+retval)
        }
    })
}, 1) 
```
hook在frida控制台返回0xe，这正是我当前的点数，写个代码修改返回值。
```js
var mInterval = setInterval(function () {
    var sgame = Process.findModuleByName("libil2cpp.so");
    if (sgame == null) {
        console.log("无");
        return;
    }
    clearInterval(mInterval);
   
  var  addr7= sgame.base.add(0x1C2D3F4 )
    console.log("" +addr7);
    console.log(Instruction.parse(addr7).toString());
    Interceptor.attach(addr7, {
        onEnter: function (args) {
            // console.log("Send2 arg1: \n"+hexdump(ptr(args[0]), { length: 100, ansi: true }));
            console.log("Send2 arg2: \n"+hexdump(ptr(args[1]), { length: 100, ansi: true }));
            console.log("Backtrace:\n" + Thread.backtrace(this.context, Backtracer.ACCURATE)
            .map(DebugSymbol.fromAddress).join("\n"));
       }
    })

        var  addr7= sgame.base.add(0x022F84F4)
    console.log("" +addr7);
    console.log(Instruction.parse(addr7).toString());
    Interceptor.attach(addr7, {
        onEnter: function (args) {
        //   console.log("gess arg1: "+hexdump(ptr(this.context.r5), { length: 100, ansi: true }))
        },
        onLeave: function (retval) {
           console.log("FunRet: "+retval);
           retval.replace(10000)
        }
    })
}, 1) 
```
打开软件查看点数真变成10000了，而且在创新工坊，15点数的模型下载也没有问题。大喜过望，修改成功了！！！
`GMAOFAGNCIO__LJGKLPOHDKL`就是软件返回点数数据的函数。
## 修改程序
因为一开始的目的就是奔着破解软件去的，hook后还没结束，应该修改代码。
查看`GMAOFAGNCIO__LJGKLPOHDKL`的arm汇编。
ARM架构返回值通常会存储在R0寄存器。
`GMAOFAGNCIO__LJGKLPOHDKL`最后跳转到`UnityEngine.Mathf$$Max_10639724`函数，交叉引用后感觉并不是很重要的函数。直接修改函数汇编代码，让返回值改变。
```asm
il2cpp:00A2596C                 MOV             R1, #0x400
il2cpp:00A25970                 MOV             R0, #0x400
il2cpp:00A25974                 BX              LR
```
如此修改即可返回1024个点数。Apply Patch后覆盖原本的lib。
## 破解签名验证
修改了lib安装完成发现软件打开就闪退。经过Hook发现闪退位置还是在Java层的`SendUnityMessage`,仔细查看有下面的可疑代码。
```java
          case 1007:
                    UnityMessage.create(1007).setString(AppUtil.getSignature(ExApplication.get())).send();
                    return;

```
分析函数是APk签名验证的，哈哈，那我就安装原来没修改的软件，然后Hook签名数据，再修改Smali，伪装一个正确的签名。
定位Smali代码，并修改数据如下。
```smali
    invoke-static {v0}, Lcom/pavostudio/exlib/util/AppUtil;->getSignature(Landroid/content/Context;)Ljava/lang/String;

    move-result-object v0

    const-string v0, "3082019f30820108a00302010202045365a59b300d06092a864886f70d010105050030133111300f060355040313086f756b6169746f753020170d3134303530343032323733395a180f32313134303431303032323733395a30133111300f060355040313086f756b6169746f7530819f300d06092a864886f70d010101050003818d0030818902818100b2e1d67613a0fbe26fd3ea512fd7787c93d9b7053ef295b8099414cd3049916567fc1b4601ded6e18263b2a2d2b309067a912048eee368216a5dcee5ad987550de4373f7fababac7a0e3826c07b3463c10ea4a08b6986adce751b5a86f8bd062bcf866ec732fe3773d72a60144879881979d147d1c04eacdb7d7c5e89c492e2f0203010001300d06092a864886f70d0101050500038181004d5d127b7320f37271b594b59f2bb44a2fa96ca9324074268829f0ace9efc157d5844ddc5c5d0a9c71a8758a05d8544f99287ba458081abe02c91d51a6531243a31a23e1aa3cfff4b5abe6cfbdba20f34eb3c0de873b913ebdcafd0daa8c8a4a8ea17d28a76fa34f1418d201faf27305989bd2458d91b7277859fda40ee687b4"
```
到此重新签名软件，安装，软件正常打开，没有闪退。
打开软件测试，不管怎么下载模型，点数都被锁定为1024，软件破解成功，美滋滋。
## 总结
熟练掌握Frida后分析软件真是事半功倍，逆向乐趣无穷。
想要Live2dViewerEx破解版的朋友可以在评论区留下邮箱，哈哈哈哈。


