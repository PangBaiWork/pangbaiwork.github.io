title: Android底部简约风格对话框
author: PangBai
date: 2021-06-19 13:39:19
tags:
- Android
categories:
- [IT, Nowork开发笔记]
---
在开发的过程中，纠结于对话框的制作，各种博客上Android对话框的制作教程很多，在此记录下自己制作对话框详细的过程。
<!--more-->
在安卓开发中，软件整体 UI 是否赏心悦目决定了用户对这个软件的评价，而对话框是用弹出的控件提醒用户操作的工具，说是 APP 的面子也不过分
我曾经在很多简约风格软件中看见过圆角的底部对话框，便感觉到了这种风格的美丽。便一直想实现它，便在网上翻找教程，但大多都是多了些复杂，而且描述并不清晰，对于我这种小白，更多带来的是头疼，经过查找资料，终于清晰了实现这种对话框的方法。
对话框可以选择```Dialog```或者 v7 包里的 ```Alertdialog```，使用方法和效果都差不太多
建议使用 Dialog，比起 v7 包的那一大堆东西，我更喜欢 ```Dialog```
这里将要用到的文件有
- a.java : 一个 Activity
- bottomdialog.xml : 对话框的布局文件
- translate_dialog_in.xml : 对话框进入动画
- translate_dialog_out.xml : 对话框关闭动画

首先要设置对话框样式，找到项目中value文件夹里的 ```style.xml``` 并在其中添加如下代码
```xml style.xml

<style name="BottomDialog" parent="@android:style/Theme.Dialog">
 <item name="android:windowContentOverlay">@null</item>
 <!-- 浮于Activity之上 -->
 <item name="android:windowIsFloating">true</item>
 <!-- 边框 -->
 <item name="android:windowFrame">@null</item>
  <!-- Dialog以外的区域模糊效果 -->
<item name="android:backgroundDimEnabled">true</item>
 <!-- 无标题 -->
<item name="android:windowNoTitle">true</item>
<!-- 半透明 -->
<item name="android:windowIsTranslucent">true</item>
<item name="android:windowBackground">@android:color/transparent</item>

</style>
<!-- 对话框动画 -->
<style name="BottomDialog.Animation" parent="@android:style/Animation.Dialog">
<item name="android:windowEnterAnimation">@anim/translate_dialog_in</item>
<item name="android:windowExitAnimation">@anim/translate_dialog_out</item>
</style>
```
在 res 中添加 anim 文件夹，用来存放动画
在文件夹中添加 translate_dialog_in.xml 和 translate_dialog_out.xml
```xml translate_dialog_in.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <translate xmlns:android="http://schemas.android.com/apk/res/android"
		android:duration="300"
		android:fromXDelta="0"
		android:fromYDelta="0"
		android:toXDelta="0"
		android:toYDelta="100%">
	</translate>
</resources>

``` 

```xml translate_dialog_out.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <translate xmlns:android="http://schemas.android.com/apk/res/android"
		android:duration="300"
		android:fromXDelta="0"
		android:fromYDelta="0"
		android:toXDelta="0"
		android:toYDelta="100%">
	</translate>
</resources>
```
接下来按照你自己的需要制作底部对话框布局
我的是这样的
```xml  bottomdialog.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:layout_gravity="center"
	android:layout_margin="0dp"
	android:padding="0dp"
	android:background="@drawable/dialog_bg">

	<LinearLayout
		android:orientation="horizontal"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:gravity="left"
		android:layout_marginLeft="20dp"
		android:layout_marginRight="20dp"
		android:layout_marginTop="20dp">

		<TextView
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:text="nowork"
			android:textSize="18sp"
			android:textStyle="bold"
			android:textColor="#ff000000"
			android:layout_gravity="center"
			android:layout_marginLeft="5dp"
			android:id="@+id/dtv1"/>

	</LinearLayout>

	<LinearLayout
		android:orientation="horizontal"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:gravity="left"
		android:layout_marginLeft="20dp"
		android:layout_marginRight="20dp"
		android:layout_marginBottom="20dp"
		android:layout_marginTop="20dp">

		<TextView
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:text="无"
			android:textSize="17sp"
			android:id="@+id/dtv2"
			android:textColor="#FF312E2E"/>

	</LinearLayout>

	<LinearLayout
		android:orientation="horizontal"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_marginLeft="25dp"
		android:layout_marginRight="25dp">

		<LinearLayout
			android:orientation="horizontal"
			android:layout_width="wrap_content"
			android:layout_height="45dp"
			android:layout_weight="5"
			android:layout_marginRight="15dp"
			android:background="@drawable/bdyj"
			android:layout_marginBottom="10dp"
			android:id="@+id/dlv3"
			android:gravity="center">

			<TextView
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"
				android:text="取消"
				android:textSize="19sp"
				android:textColor="#FF312E2E"/>

		</LinearLayout>

		<LinearLayout
			android:orientation="horizontal"
			android:layout_width="wrap_content"
			android:layout_height="45dp"
			android:layout_weight="5"
			android:layout_marginLeft="15dp"
			android:background="@drawable/bdyj"
			android:layout_marginBottom="10dp"
			android:id="@+id/dlv4"
			android:gravity="center">

			<TextView
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"
				android:text="确定"
				android:textSize="19sp"
				android:textColor="#FF312E2E"/>

		</LinearLayout>

	</LinearLayout>

</LinearLayout>

最后只有在Activity代码里调用对话框
```java a.java

      Dialog bottomDialog = new Dialog(a.this, R.style.BottomDialog);
      //指定布局
View contentView = LayoutInflater.from(a.this).inflate(R.layout.bottomdialog, null);
bottomDialog.setContentView(contentView);
ViewGroup.LayoutParams layoutParams = contentView.getLayoutParams();
layoutParams.width = getResources().getDisplayMetrics().widthPixels;
contentView.setLayoutParams(layoutParams);
//控制对话框位置
bottomDialog.getWindow().setGravity(Gravity.BOTTOM);
bottomDialog.getWindow().setWindowAnimations(R.style.BottomDialog_Animation);
//显示对话框
bottomDialog.show();

```
以下是成品
![image](http://shp.qpic.cn/collector/1642981619/d9076b1b-c52c-4d30-a834-5012ee76d3a1/0 "滑稽")



