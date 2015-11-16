---
title: 资源国际化引发的"血案"
layout: post
guid: urn:uuid:e58decc5-b50a-417d-aa8a-0926a16c345f
comments: false
tags:
  - resource
---

### 前言
以前碰到过类似的异常，我们想调用 `textview.setText(CharSequence text)`，但是传入了 **单纯的数字**，系统会调用 `textview.setText(int resid)` 方法 ，这样必然找不到资源。  
解决办法很简单，把 **数字** 转换成 `String` 即可。

### 异常
先贴出 Trace:

```java
Caused by: android.content.res.Resources$NotFoundException: String resource ID #0x7f07011e
  at android.content.res.Resources.getText(Resources.java:312)
  at android.widget.TextView.setText(TextView.java:4417)
  at com.aft.fastrepair.presentation.view.custom.PtrListView.changeHeaderViewByState(SourceFile:999)
```

找到源代码：

```java
tv.setText(R.string.p2refresh_pull_to_refresh);
```
### 原先配置
项目资源设置策略为：`values` 下默认英文，`values-zh-rCN` 下为中文。  
`p2refresh_pull_to_refresh` 只在中文资源中设置过。  
![only-set-in-values-zh-rCN](/media/files/2015/11/16/incomplete_resource.png)
上图为鼠标放在代码资源上的 doc 提示。

### 解决思路
开始以为代码混淆问题，把该类防混淆之后再测试，依然报错。:(  
之前在中文语言环境下测试已通过，今天把语言切换成英文就挂了。  
自然联想到是不是找不到英文资源呢？

猜想如下：  
当前语言既为英文，系统会在 `values-en` 和 `values` 中找，肯定不会在 `values-zh-rCN` 下找。  
不然，怎么会报错呢？抱着尝试的态度，在 `values` 中增加相应资源。  
![set-in-values-and-values-zh-rCN](/media/files/2015/11/16/complete_resource.png)
如上图所示，系统找到了 `values` 中的值，问题解决。

### 师傅，等一等！
各位老师傅，这就完了吗？  
......  
必须没有，仔细思考下，是不是漏掉了什么？  
前面只是在 `values` 中增加了资源，但是当前语言是英文，为何不报错呢？系统是如何寻找资源的呢？  
这里有个最核心的问题：系统到底是先去 `values-en` 寻找呢，还是 `values` ？  
换言之，***系统资源加载顺序是怎样的*** ？

### 遗留问题 --- 系统资源加载顺序
未完，待续。
