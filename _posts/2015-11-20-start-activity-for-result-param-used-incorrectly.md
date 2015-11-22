---
title: startActivityForResult 参数值引发的异常
layout: post
guid: urn:uuid:1a74f0fd-5214-40a7-8c4c-6eecfd9ce409
comments: false
tags:
  - bugfix
---

### 想当然要不得
编程的世界就是这么奇妙，一个微小的“疏忽大意”便会造成意想不到的后果。  
这么讲太给自己面子，其实是“想当然”引发的不必要 bug 。  
注释文档里会给出详细的解释，可惜。。。

### 踩坑代码
```java
public void startActivityForResult(android.content.Intent intent, int requestCode)
```
这个方法最熟悉不过了，容易出问题的地方在于哪里呢？请先看注释：  

```java
Parameters:
intent - The intent to start.
requestCode - If >= 0, this code will be returned in onActivityResult() when the activity exits.
```
没错，你猜对了，就是那个 `requestCode` ，它的值必须是非负数，才能在 `onActivityResult()` 中收到回调。。。  
当时看着是个 `int` 值，想也没想直接上 －2 ，结果调试了几分钟才发现问题。

### 脸好疼
与其说从未遇到这个 bug ，不如说是不知道自己这么“随便”。。。  
“啪啪啪”，好疼。
