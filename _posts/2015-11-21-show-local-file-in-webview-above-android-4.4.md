---
title: Android 4.4 以上 Webview 中显示本地图片
layout: post
guid: urn:uuid:a4715c2e-774d-4643-8656-41175234f752
comments: true
tags:
  - webview
---

### 问题背景
Android 版本碎片化导致原生开发比较艰难，你懂的，各种蛋＊的版本适配。所以，H5 加本地混合开发在中小型项目上有一定的优势。比如：功能迭代更快，兼容性更佳（有些方面）等等。  

在这种策略下，本地尽量配合 H5 实现完整交互，而不是大包大揽。  

我们的产品有上传图片的功能，并且在 web 页面展示。按理说不是太难的东西，可是这里面坑叫一个多。还是谷歌挖的。。。

### 出现问题
```java
INFO/chromium(6759): [INFO:CONSOLE(9)] "Not allowed to load local resource: content://com.testing.image/kitkat.png"
```

### 解决方案
把图片压缩后，转换成 base64 格式，直接通过 Js 方法传给网页，直接通过 img 标签设置。  
缺点：不适合显示大量图片。

```java
  ArrayList<Base64Photo> targetList = new ArrayList<>(event.getPicList().size());
  String base64Str;//编码后的图片内容
  for (String path : event.getPicList()) {
    base64Str= PictureUtil.bitmapToString(path);
    if (!TextUtils.isEmpty(base64Str)) {
      targetList.add(new Base64Photo(path, base64Str));
    }
  }

  webview.loadUrl("javascript:showUserUploadPic(" + new Gson().toJson(targetList,
    new TypeToken<ArrayList<Base64Photo>>() {}.getType()) + ")");
```

```java
/**
 * Js 端接收对象
 */
public class Base64Photo {
  /**
   * 图片路径
   */
  public String path;
  /**
   * 编码后的图片内容
   */
  public String base64;

  public Base64Photo(String path, String base64) {
    this.path = path;
    this.base64 = base64;
  }
}
```

### 致谢
[Kitkat-WebView](https://github.com/henrychuangtw/Kitkat-WebView)  
[Android webview加载本地图片](http://blog.csdn.net/candyguy242/article/details/16947445)  
[Android 4.4 WebView cannot load "content://" urls in html page](https://code.google.com/p/android/issues/detail?id=63033)
