---
title: 保存 Activity 状态
layout: post
guid: urn:uuid:07872e31-fca1-4965-b87f-2e7267a24ac9
comments: true
tags:
  - activity state
---

作者：[JohnWatsonDev](http://www.johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 为什么要单独讨论这个话题

熟悉 Activity 生命周期的朋友也许会呲之以鼻，这不太简单了，直接 `onPause()` 中保存呗。

嗯。。。够狠～

仔细一想，在 `onPause()` 中保存的到底是什么？一般来说是还未来得及持久化的数据。这些数据有可能下一次进入 Activity 继续展示或者根据这些数据展示另一些数据。

有没有别的意外情况呢？请看下面的场景：

假设用户在注册 Activity 表单中输入了一些数据，这时微信突然来消息，用户点击通知栏切换到微信，注册 Activity 进入后台。请注意，此时微信界面和注册 Activity 在同一个 Task 中。通常我们不会在 `onPause()` 方法中保存表单数据，而是在用户点击提交按钮时直接使用即可。用户在微信中稍作停留后，点击 Back 键返回注册 Activity 。假设在使用微信过程中，系统回收内存，恰好把注册 Activity 销毁了。那么，用户返回后还能看到原先填入表单的数据吗？

先让思路飞一会儿，请接着往下看。

### 什么情况下需要保存状态

当 Activity 暂停 （paused）或停止（stoped）时，它的状态是被保留的。这是正确的，因为它还在内存中被持有。因此，用户在此 Activity 中作出的所有改变都被保留下来，直到该 Activity 重新返回前台（resume），那些改变仍然在那里。

但是，当系统需要回收内存而销毁 Activity 后，系统不能简单的恢复（resume）它，并且回到原来的状态。系统必须重新创建该 Activity 。然而，用户并不知道系统销毁并重建了该 Activity 。因此，用户可能期望 Activity 还是原先离开时的样子。

这种情况下，我们必须通过实现 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法来保存 Activity 重要的状态信息。

系统会在 Activity 变得容易被销毁前调用 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 。系统会给该方法一个 [Bundle](http://developer.android.com/reference/android/os/Bundle.html) 参数，用来通过键值对方式保存状态信息。这样，如果系统杀死应用进程，用户返回 Activity 后，系统会重新创建 Activity 并且通过 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle\)) 和 [onRestoreInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onRestoreInstanceState(android.os.Bundle\)) 方法传入 [Bundle](http://developer.android.com/reference/android/os/Bundle.html)。使用其中任何一个方法，我们可以通过 [Bundle](http://developer.android.com/reference/android/os/Bundle.html) 取出保存的状态信息，进而还原状态。如果没有状态信息需要还原，传入的 [Bundle](http://developer.android.com/reference/android/os/Bundle.html) 为 null （比如Activity 第一次创建时）。


![restore_instance](/media/files/2016/01/18/restore_instance.png)

> 注意：[onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法并不保证在 Activity 销毁前调用，因为在一些情况下不需要保存状态，比如用户点击 Back 键退出。如果系统调用 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法，一定会在 [onStop()](http://developer.android.com/reference/android/app/Activity.html#onStop(\)) 之前，可能在 [onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause(\)) 之前。

然而，即使我们什么也没做，也没有实现 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法，[Activity](http://developer.android.com/reference/android/app/Activity.html) 类默认实现了 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法，因此有些状态会被自动保存和恢复。特别是默认实现调用了布局中每个 [View](http://developer.android.com/reference/android/view/View.html) 相应的 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法，该方法允许 View 提供自己应该保存的信息。Android Framework 中几乎每个 widget 都恰当地实现了该方法，比如任何可见性的改变都会自动保存，并且在 Activity 重建时恢复。举个例子，[EditText](http://developer.android.com/reference/android/widget/EditText.html) 控件会保存用户输入，[CheckBox](http://developer.android.com/reference/android/widget/CheckBox.html) 会自动保存是否选中。我们唯一需要做的就是：通过 [android:id](http://developer.android.com/guide/topics/resources/layout-resource.html#idvalue) 属性为想自动保存状态的控件提供唯一的 ID 。若没有设置 ID ，系统将不会自动保存其状态。

尽管 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 默认实现保存了 Activity UI 的有用信息，我们仍需要复写该方法保存其他信息。比如：我们可能需要保存成员变量的值，它们可能会在 Activity 生命周期中改变。

> 我们可以通过设置 [android:saveEnabled](http://developer.android.com/reference/android/R.attr.html#saveEnabled) 属性值为 false ，或者 [setSaveEnabled()](http://developer.android.com/reference/android/view/View.html#setSaveEnabled(boolean\)) 方法来阻止布局中的某个 View 保存其状态。通常，我们不应该这么做，除非我们想使 Activity 恢复不同的状态。

由于 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 默认实现保存 UI 状态，因此我们在复写此方法时，务必首先调用父类实现。对于 [onRestoreInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onRestoreInstanceState(android.os.Bundle\)) 方法也是如此。

> 注意：由于 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 方法是不保证被调用的，所以它只适合保存临时的状态信息，而绝不能保存持久化的数据。当用户离开 Activity 时，我们应该使用 [onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause(\)) 方法保存持久化数据（比如应该保存到数据库的数据）。

一个简单的测试应用具有恢复状态能力的方法是：旋转设备从而使设备方向改变。设备方向改变后，系统会销毁和重新创建 Activity 以便为新的屏幕配置应用替代资源。仅仅出于这个原因，我们的 Activity 在重建后可以恢复到之前的状态是很重要的，因为用户在使用应用时会经常旋转屏幕。

### 小结

若保存临时状态，主要是 UI 状态和成员变量的值，我们该使用 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 保存，在 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle\)) 或 [onRestoreInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onRestoreInstanceState(android.os.Bundle\)) 中恢复。

若保存持久化数据，比如写入 Sharedpreferences 、本地文件或数据库的数据。应当在 [onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause(\)) 中保存。

### 相关资源

[Saving activity state](http://developer.android.com/guide/components/activities.html#SavingActivityState)
