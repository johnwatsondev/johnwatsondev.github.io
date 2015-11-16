---
title: MVP 介绍(译文)
layout: post
guid: urn:uuid:5fd9acec-8cba-4c27-9b85-ae780e057ed5
comments: true
tags:
  - mvp
---

原文：[model-view-presenter-presentation](http://www.slideshare.net/DarxVal/model-view-presenter-presentation)  
译者：[JohnWatsonDev](http://www.johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 阅读须知
括号内均为译者自己标注

### MVC 
优点和缺点？（挖坑，待填，网上资料很多，我没有认真思考比对过，所以不会随便引用。）

### MVP （继续挖坑... 没有深入比对市面上各种 MVP 架构，不敢妄言！）

#### Views
- 不要和 widget.view 混淆 （界面的抽象）
- The view is a tattle tale  （这句真的不怎么会翻译，大家可以留言 lol）
- 要对 View 严加看管，你拥有绝对控制权 :)
  - 告诉它做什么  （你需要有个秘书来发号施令）
  - 让 View 告诉你它发生的一切  （你需要有个秘书来接收消息）

（所以你需要一个秘书，秘书是谁呢？后面就知道了）

#### View Interfaces
- 不依赖任何类，总是有个约定（接口）
- 约定细节
  - presenter 必须告诉 View 做什么
  - presenter 必须知道 View 接收到的任何事件 （比如用户点击了一个按钮）

#### Presenter
- 告诉 View 和 Model 对象去做什么 （presenter 就是辣个秘书）
- 处理事件

### 假如做一个名叫 TaskIt 的 APP

- 增加任务
- 获取一个任务列表
- 详细查看一个任务 （包含修改和删除）

### 如何测试呢？

控制反转 （[中文](https://zh.wikipedia.org/wiki/控制反转) [英文](https://en.wikipedia.org/wiki/Inversion_of_control)）  
你需要做的就是注入模拟对象代替具体对象来测试应用的一部分 （单元测试）

Mock 和 Espresso

### 鸣谢
[nucleus](https://github.com/konmik/nucleus)  
[android-testing](http://code-labs.io/codelabs/android-testing)  
[幻灯片背景图](https://plus.google.com/u/0/photos/102898026333733818285/albums/6081914916274644193/6081914917317951522?pid=6081914917317951522&oid=102898026333733818285)

### 译者注
本人水平有限，如有错误，请及时指正。  
文中还有很多坑没填，后续会继续跟进，若有好的建议，欢迎留言！
