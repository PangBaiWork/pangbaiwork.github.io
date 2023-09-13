---
title: 利用JavaScript实现循环打字效果
date: 2021-03-13 19:27:55
tags: html前端
categories:
- [html,HTML特效]
---
++一个自己制作的JS打字效果++&#123;.primary&#125;
# 效果
- [x] 循环打字
- [x] 控制打字速度
- [x] 纯JS实现
- [x] 适用于网页美化
效果如下


# `JavaScript`教程
首先这个效果的核心是`setTimeout()`方法和`slice()`方法
这两个JS方法的用途分别是每隔一段时间执行一次代码和截取固定长度的字符串
```js setTimeout的用法
setTimeout(执行代码,执行时间)
```
```js slice()的用法
对象.slice(截取起始,截取末尾)
```
接下来只需要把效果拼接一下就可以了
使用length()取得文本长度
如果没到达文本总长度就逐增截取字符并显示
达到了文本长度就逐减截取字符并显示
下面给出JavaScript实例 
 !!这些只是旁白自己随手写的，可能代码有些粗糙，有实力的大佬可以自行修改!!
```js 打字机实例
var i = 0,
timer = 0,
s = 1,
str; //申明将用到的变量，并初始化
var divTyping = document.getElementById('text')
//获得文本元素，你需要在html里添加一个带id的文本元素                                            //如 <h1 id="text"></h1>

function typing () {
  if (s == 1){
    str = '时间虽短 情谊长存';
      }
  if (s == 2){
    str = '苟富贵无相忘';
      }
  if (s == 3){
str = 'No matter where \nWe called home';

}
//如果你要循环打出不同的几段话，就使用上面这个函数
//如果你只想要一段话重复打字，直接设置str变量的值即可。

if (i <= str.length) {
 divTyping.innerHTML = str.slice(0, i++) + '_';
 timer = setTimeout(typing, 300);
// 如果长度未到达，就逐步增字
// '_'为光标符号，可以换成'|'
}
  else {
//如果长度到达，结束打字,移除打字机，并启动扣字机
clearTimeout(timer);
typin();
}
}


//这里是扣字机
function typin () {

 if (i > 0) {
 divTyping.innerHTML = str.slice(0,i--) + '_';
 timez = setTimeout(typin, 60);
//如果文本长度不为0就逐步扣字
  }
 
 else {	
// 如果文本长度为0就结束扣字机，开启打字机
 clearTimeout(timez);
 typing();
 if (s == 3){
 s = 0;
 }
//控制循环打字
 s++;
}
}

```

复制粘贴即可食用
`本文来自旁白`
转载请注明原地址
