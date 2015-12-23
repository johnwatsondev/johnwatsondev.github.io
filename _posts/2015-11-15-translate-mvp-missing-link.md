---
title: (译) MVP 缺失的纽带
layout: post
guid: urn:uuid:00974858-998d-4dbe-95a0-d770662a421d
comments: true
tags:
  - mvp
---

原文：[MVP: The Missing Link](http://blog.sqisland.com/2015/11/mvp-missing-link.html)  
译者：[JohnWatsonDev](http://www.johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

<div class="video-container"> <iframe width="560" height="315" src="https://www.youtube.com/embed/AoqL1PN8hCk" frameborder="0" allowfullscreen> </iframe> </div>

我已经想学 [Model View Presenter](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter) 有一阵了，非常高兴昨晚在 [GDG Boulder](http://www.meetup.com/Google-Developer-Group-Boulder/events/226164202/) 上听 [Michael Cameron](https://twitter.com/darxval) 的讲座。  
![mvp with View Interface](/media/files/2015/11/15/missing_link_MVP.png)

### 界面接口
我之前模糊地认为 MVP 有利于单元测试，现在终于知道原因啦。秘密就是缺失的界面接口，它不是 MVP 首字母缩写的一部分。界面接口在 view 层和 presenter 层之间定义了一份约定，它允许你在测试其中之一时模拟另一个。
你能够把安卓 UI 代码划分到 view 层（通常是 `Fragment` 或 `Activity`），view 层和 presenter 层可以通过界面接口单独地通信。你可以不写一行安卓代码模拟界面接口，这意味能在 JVM 上使用 JUnit 测试 presenter 层。

### 通过 build flavors 来模拟
模拟对于封闭测试是必要的，不过大家常常使用依赖注入框架，比如 [Dagger](http://google.github.io/dagger)。如果你喜欢的话，可以使用 build flavors 提供针对测试的模拟。这是个很好的技巧。  
![mock via build flavors](/media/files/2015/11/15/missing_link_mock.png)

### Mockito
[Mockito](http://mockito.org/) 是一个很好的模拟框架而且能通过注解模拟。使用 `@Mock` 标注你想模拟的成员变量，在标注了 `@Before` 注解的方法中调用 `MockitoAnnotations.initMocks()` 完成初始化。

```java
@Mock
private MyThing myThing;

@Mock
private OtherThing otherThing;

@Before
public void setUp() throws Exception {
  MockitoAnnotations.initMocks(this);
}
```
等价于：

```java
private MyThing myThing;
private OtherThing otherThing;

@Before public void setUp() throws Exception {
  myThing = Mockito.mock(MyThing.class);
  otherThing = Mockito.mock(OtherThing.class);
}
```
这是我做的 Sketchnotes:  
![sketch_notes](/media/files/2015/11/15/sketch_notes.jpg)  
幻灯片: [原文](http://www.slideshare.net/DarxVal/model-view-presenter-presentation) [译文]()

### 代码实验室
Google 已经发布了一个很好的使用 MVP 架构的代码示例 [Android Testing Codelab](http://www.code-labs.io/codelabs/android-testing/) 。  
在示例中你可以看到 JUnit 和 Espresso 测试的作用，通过 build flavors 模拟，还有其他的奇技淫巧。快去看看吧！[译文在此]()

### 译者注
为何会发现这篇博文？  
源于一个 issue 讨论 --- [Is the MVP architecture better than MVC?](https://github.com/futurice/android-best-practices/issues/83)  
顺便吐槽一下 Google 的效率，在市面上 MVP 流行之后才发布一篇简单的指导文章，当然更多的是在讲 [ATSL](http://google.github.io/android-testing-support-library/) 。  
搞不懂 Google 为何不早点推出官方的指导框架呢？
