---
title: 检查运行时权限工具类
layout: post
guid: urn:uuid:f9b93732-9806-4fd1-ba4b-4705a0743cd3
comments: true
tags:
  - permission
---

作者：[JohnWatsonDev](http://www.johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 前言
在 6.0 上使用 [PhotoPicker](https://github.com/donglua/PhotoPicker) ，由于默认情况下应用权限是未通过的，所以出现问题。  

### 异常
```java
Caused by: java.lang.SecurityException: Permission Denial:  
reading com.android.providers.media.MediaProvider uri content://media/external/images/media  
requires android.permission.READ_EXTERNAL_STORAGE, or grantUriPermission()
```

显然是木有权限呀，一番查证后是 [Runtime Permissions](http://developer.android.com/training/permissions/requesting.html) 问题。参考文档后抽取工具类如下：

{% gist johnwatsondev/2d9a068931f88a3b2470%}

### 解释

重点解释下 `checkPermissionGranted` 方法中的最后两个参数。

+ 当权限未被授予且需要向用户说明原因时，`ShowRationaleCallback` 回调函数会发出通知，在实现类的 `void needShow(final int requestCode)` 方法中自定义提示，弹出 Toast 或 对话框 等等。  
+ 布尔型的 `needRequestPermission` 参数是当权限未被授予且需要向用户说明原因时，是否直接帮你请求该权限，省去手动发起。

### 示例
请参考 [Demo](https://github.com/johnwatsondev/PhotoPicker.git) 中的 [`MainActivity`](https://github.com/johnwatsondev/PhotoPicker/blob/master/photopickerdemo/src/main/java/me/iwf/PhotoPickerDemo/MainActivity.java) 。
