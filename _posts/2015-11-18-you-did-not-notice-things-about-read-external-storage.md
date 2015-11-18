---
title: 你可能忽略的读取外部存储权限问题
layout: post
guid: urn:uuid:749f748e-fd43-4352-a89f-12a407d0b971
comments: true
tags:
  - permission
---

作者：[JohnWatsonDev](http://www.johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 来源
这个问题源自最近要做获取本机图片和拍照的功能，本着不重复造轮子滴原则找到了 [PhotoPicker](https://github.com/donglua/PhotoPicker) 开源库，效果还不错，非常感谢作者！  
我没有使用 aar ，因为要自己最大程度定制嘛。:) 读源码的过程中，发现该库只使用了一个权限 `android.permission.WRITE_EXTERNAL_STORAGE` ，怀着好奇心，抛出一个问题：为什么只声明写权限即可，读取权限不需要呢？  
带着这个问题搜索，有了本篇总结。

### 官网解释
官网永远是我们最好的伙伴，没有之一。各位先来看[原文](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)解释：  

> Allows an application to read from external storage.

> Any app that declares the WRITE_EXTERNAL_STORAGE permission is implicitly granted this permission.

> This permission is enforced starting in API level 19. Before API level 19, this permission is not enforced and all apps still have access to read from external storage. You can test your app with the permission enforced by enabling Protect USB storage under Developer options in the Settings app on a device running Android 4.1 or higher.

> Also starting in API level 19, this permission is not required to read/write files in your application-specific directories returned by getExternalFilesDir(String) and getExternalCacheDir().

> Note: If both your minSdkVersion and targetSdkVersion values are set to 3 or lower, the system implicitly grants your app this permission. If you don't need this permission, be sure your targetSdkVersion is 4 or higher.

> Protection level: dangerous

> Constant Value: "android.permission.READ_EXTERNAL_STORAGE"

### 译文（略去废话版）
任何声明了 `WRITE_EXTERNAL_STORAGE` 权限的应用都会隐式地获取到此权限。  
此权限在 API 19 之后被加强了，在 API 19 之前，所有的应用能从外部存储读取数据(注意：此处的存储是指整个外部存储，不光你自己应用的，还有系统的和别人的)。  
如果系统版本是 4.1 或者更高，那么恭喜你，你可以自己决定是否开放此权限，请打开 “保护 USB 存储”，它在 “设置” －－ “开发者选项”下面。

> 注意：开发者文档有时也很坑。。。我尝试在 6.0 下开启此选项，结果没找到。
> 谷歌关键词 `Protect USB storage android`，找到一篇[报道](http://www.androidcentral.com/jelly-bean-brings-new-permission-along-read-external-storage)，通过文末的 [Android 4.1 APIs](http://developer.android.com/about/versions/android-4.1.html)，才总算找到理解这段话的钥匙。  
> 事实上，这个 “保护 USB 存储” 并不是在 4.1 之后一直存在，起码 Nexus 5 的 6.0 官方镜像没有。。。这不是重点，我们先接着翻译，后面详细讲解这个权限的前世今生。:)

从 API 19 开始，应用在 `getExternalFilesDir(String)` 和 `getExternalCacheDir()` 方法返回的存储目录下读写文件不需要此权限。  
注意：如果 `minSdkVersion` 和 `targetSdkVersion` 的值是 3 或以下，系统将会自动授予应用此权限。如果不需要此权限，请保证 `targetSdkVersion` 的值为 4 或以上。(为啥不需要的话要保证 4 或以上呢？说的不明不白。:(  理解的小伙伴请解释下，谢谢！)

保护级别：危险  
常量值：`android.permission.READ_EXTERNAL_STORAGE`

### 该权限何时加入的
答案是：API 16 ，也就是 [Jelly Bean 4.1](http://developer.android.com/about/versions/android-4.1.html#Permissions) 。  
在 4.1 中，默认情况下所有的应用仍然有读取权限。如果应用已经获取了写权限，那么自动获取读取权限。  
好奇的你肯定会问：**特殊情况**是啥呢？  
问得好，这就是我在上面译文中疑惑的地方，**保护 USB 存储** 是个神马玩意儿？其实它就是在 4.1 引入的，请看图片：  

![Protect USB storage](/media/files/2015/11/18/protect_usb.jpg)

当勾选此选项后，会有什么行为呢？暂时没有测试，暂定使用虚拟机测试。  
当然后来在 6.0 上为何看不到，目前还不清楚，如果你查到原因了，请告诉我哈！

### KitKat 中的变化
在 [Important Behavior Changes](http://developer.android.com/about/versions/android-4.4.html#Behaviors) 和 [App Permissions](http://developer.android.com/about/versions/android-4.4.html#Permissions) 中阐述了变化。  
大意是：在 4.4 上，应用不能直接读取外部存储的共享文件，除非获取了读权限。由 `getExternalStoragePublicDirectory()` 方法返回目录(外部存储的共享区域)下的文件，没有读取权限时无法访问。但是可以在没有任何权限时(`WRITE_EXTERNAL_STORAGE` 和 `READ_EXTERNAL_STORAGE` 均不需要)，访问(读写)由 `getExternalFilesDir()` 方法返回的目录（应用特定的外部存储，你自己的地盘）下的文件。  

### Marshmallow 中运行时权限
[Runtime Permissions](http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-runtime-permissions) 有说明。  
大意为：用户可以在运行时决定是否授权给某个应用。类似与 iOS 上弹出对话框询问是否打开第三方的应用，手机主人真正当家做主啦。:)  
在 6.0 及以上系统要在运行时判断是否获取了相应权限，根据用户的授权状态给出恰当的交互。更多信息请看 [Working with System Permissions](http://developer.android.com/training/permissions/index.html) 和 [Requesting Permissions at Run Time](http://developer.android.com/training/permissions/requesting.html) 。

### 总结
其实就是译文的浓缩版：  
如果已经声明了 `WRITE_EXTERNAL_STORAGE` 权限，那么不需要显示地声明 `READ_EXTERNAL_STORAGE` 权限。  
API 19 之前所有的应用能从外部存储读取数据(注意：此处的存储是指整个外部存储，不光你自己应用的，还有系统的和别人的)。  
从 API 19 开始，应用在 getExternalFilesDir(String) 和 getExternalCacheDir() 方法返回的存储目录（应用特定的外部存储，你自己的地盘）下读写文件不需要此权限，而在 `getExternalStoragePublicDirectory()` 方法返回目录(外部存储的共享区域)下的文件，没有读取权限时无法访问。  
关于运行时权限，请看本篇：[检查运行时权限工具类](http://www.johnwatsondev.com/2015/11/17/runtime-permission-util.html) 。
