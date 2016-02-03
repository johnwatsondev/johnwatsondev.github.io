---
title: Android 4.4 以上 Webview 中显示本地图片
layout: post
guid: urn:uuid:a4715c2e-774d-4643-8656-41175234f752
comments: true
tags:
  - webview
---

作者：[JohnWatsonDev](http://johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 问题背景
公司 App 分用户端和工程师端，用户端用户体验要求较高，大部分功能用 Native 方式开发，而工程师端在分析利弊之后选择混合开发模式，即 Native 做壳，大部分功能放到 Web 页，Web 实现效果不理想的采用回调 Native 代替。  
这是为何？难道工程师端体验不重要？  
当然不是，工程师端和用户交互较少，大部分界面只展示内容，少量的轻量级交互为输入文本或上传图片，加上 Native 开发的各种坑（屏幕适配和版本碎片化导致的 bug，关键是神奇的 WebView 从中作梗，众所周知，版本碎片化问题是有多严重，由于安全问题屏蔽了许多方法，而且不推出相应替代方法。
），还有 Web 页全终端适配优势和版本迭代更快等原因。综合考量之后使用混合开发方式。

我们的产品有上传图片的功能，并且在 web 页面展示。按理说不是太难的东西，可是这里面坑叫一个多。还是谷歌挖的。。。

### 出现问题
```java
INFO/chromium(6759): [INFO:CONSOLE(9)] "Not allowed to load local resource: content://com.testing.image/kitkat.png"
```

### 解决方案
最后，我们采用本地选择图片，把图片压缩后，转换成 base64 格式，直接通过 WebView 的 Js 接口回调 Web 页面，最终 Web 页完成上传工作。
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
