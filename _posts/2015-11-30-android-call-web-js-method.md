---
title: 安卓调用 Web 页 Js 方法出错
layout: post
guid: urn:uuid:22dc916e-0298-4ab1-aaf5-4903a6034e06
comments: false
tags:
  - webview
---


### 问题还原
Web 页面的 Js 方法需要本地传入 String 类型的参数，由于没看过 Js 语法，贸然写了如下代码：  

```java
String formId = "f1";
webview.loadUrl("javascript:showUserUploadPic(" + formId + ")");
```

抛出如下异常：

```java
I/chromium: [INFO:CONSOLE(1)] "Uncaught ReferenceError: formId is not defined"
```

### 问题分析
异常显示引用错误，说明 formId 被解析为变量，而非我们期待的字符串。搜索后发现，字符串参数一定要用单引号包裹，问题解决。  
等等，这就完了？在我这里是木有滴，继续测试，如果不加单引号传入 int 或 普通对象可以正常运行吗？测试后发现正常执行。

### 总结
好吧，是个很简单的问题，为什么要记录呢？很简单，该问题是如何解决的，总结解决思路，顺便加深印象。  
看某个兄台的博客，他说程序员的核心能力是分析、抽象和解决问题，总结又是帮助水平提升的不二之选！  
我觉得还要加上“坚持思考总结，养成习惯，不过分依赖记忆。”嗯，就是这样。
