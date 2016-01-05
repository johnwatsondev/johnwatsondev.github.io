---
title: 如何对抗众多的一键清理工具？
layout: post
guid: urn:uuid:28de6d0f-5ac3-4cd0-8630-e83713eb923a
comments: true
tags:
  - app-cleaner
---

### 需求
工程师出发后，需要每隔一分钟纪录一次工程师当前的位置，直到到达客户现场。

### 解决思路
1. 使用 `AlarmManager` 每隔一分钟通知后台 Service 启动定位，纪录数据。假如用户自行退出应用，我们不会持续定位。
2. 相比方法一，用户即使不小心退出应用后，后台服务持续驻存，类似 IM 类推送服务，直到工程师确定到达现场。

我们采用尽量减少耗电量的第一种方法。

### 碰到的挑战
国产手机是广大开发者的梦魇，许多系统自带了一键清理功能，更有甚者，默认锁屏10分钟后启动一键清理（比如小米）。  
还有 Vivo 手机，好心的系统提供了耗电优化功能，它检测到我们的应用在后台（其实我们告知工程师不许退出应用，而且 Service 都设置成了 `foreground state`）不断的唤醒 CPU 和消耗流量，直接自动把应用 “优化”，相当于绿色守护的冻结应用。  
你以为没了吗？我们还需要面对各种卫士和管家。。。如果不加入白名单，那就只有被杀死。:(

关键问题在于假如应用被杀死，连 `AlarmManager` 都会失效[泪奔]。。。

### 如何解决
抱歉，目前还在想办法解决中。。。  
QQ 和微信是如何做到后台接受消息的？各位也遇到类似的窘境吗？  
如果大家有好的思路，请不吝赐教！

### 致谢
[怎么让Android程序一直后台运行，像QQ一样不被杀死？](https://www.zhihu.com/question/29826231)  
[Android应用中，如何保证服务常驻内存？](https://github.com/android-cn/android-discuss/issues/49)  
[How to automatically restart a service even if user force close it?](http://stackoverflow.com/questions/21550204/how-to-automatically-restart-a-service-even-if-user-force-close-it)
