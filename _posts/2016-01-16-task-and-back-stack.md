---
title: (译) 任务和返回栈
layout: post
guid: urn:uuid:5bb78698-6bd9-42ff-8b6a-43be0ea8a86e
comments: true
tags:
  - task
---

原文：[Tasks and Back Stack](http://developer.android.com/guide/components/tasks-and-back-stack.html)  
译者：[JohnWatsonDev](http://johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 概念及行为解释

---

简单来讲，任务就是用户当前在执行一组操作时打开的 N 个 Activity 的集合。这些 Activity 被排列在一个栈中，这个栈就叫返回栈。

#### 返回栈

返回栈和任务具有一一对应关系。

一个任务中既可包含本应用的 Activity 也可包含其他应用中的 Activity 。大多数任务是在桌面或者启动器（Launcher）启动的。如果任务不存在（最近没有打开过），则创建一个新的任务，栈底是主 Activity 。

当前 Activity 打开新 Activity 时，新 Activity 会压入栈顶并且获得焦点。前一个 Activity 会保留在栈中，但是会 stop 。当 stop 时，系统保留界面的状态。点击 Back 键时，栈顶的 Activity 弹出，并且 destroy 掉。相应的前一个 Activity 显示，之前保留的界面被恢复。请看下图：

![diagram_backstack](/media/files/2016/01/16/diagram_backstack.png)

栈中的 Activity 绝不会被重新排列，只能压入或弹出。返回栈遵循“后进先出”原则。所有的 Activity 被移出栈后，任务将会消失。

#### 多任务

当用户按下 Home 键或新启动一个任务时，原先的任务会进入后台状态。当进入后台状态时，任务中所有 Activity 是被 stop 掉的，任务的返回栈保持原封不动，这个任务只是简单地失去焦点，另一个任务取代了它原先的位置。请看下图：

![diagram_multitasking](/media/files/2016/01/16/diagram_multitasking.png)

> 注意：同一时间点，后台可以持有多个任务，然而，如果用户同一时间点运行了很多后台任务，后台有可能会销毁 Activity 来释放内存，从而造成 Activity 状态丢失。

在应用中，某个 Activity 可能被实例化多次，甚至出现在不同的任务中。当然我们可以改变它的行为，下文会提到。

#### Activity 和任务行为总结

- 当 Activity A 启动 B ，A 会 stop ，但是系统会保留其状态（比如滚动位置和表单中输入的文字）。如果在 B 中按下返回键，A 会恢复其之前存储的状态。

- 当用户按下 Home 键离开某个任务，当前 Activity 会 stop ，并且任务进入后台状态。系统保持任务中每个 Activity 的状态。如果用户稍后按下启动器图标恢复任务，任务会重新回到前台状态，并且栈顶的 Activity 也会恢复。

- 如果用户按下返回键，当前 Activity 会从栈中弹出，并且销毁。栈中的前一个 Activity 恢复显示。当 Activity 被销毁掉，系统不再保存其状态。

- Activity 能够被实例化多次，甚至来自不同的任务。

### 保存 Activity 状态

---

上面提到，当 Activity stop 时，系统默认会保存其状态。这种情况下，当用户返回前一个 Activity 时，用户界面完好如初。然而，我们能够并且应该主动通过回调方法保存 Activity 状态，以防 Activity 被销毁，而且必须重新创建。

当系统 stop 了任务中的一个 Activity 时（比如新启动一个 Activity 或任务进入后台状态），如果系统需要回收内存的话，系统可能会完全销毁该 Activity 。当发生这种情况时，Activity 的状态信息会丢失。如果发生这种情况，系统仍然知道该 Activity 在返回栈中的位置，如果其到达栈顶，系统必须 recreate 它（而不是 resume ）。为了不丢失用户的工作，我们应该在 Activity 中实现 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle\)) 回调方法主动保存状态。

更多关于如何保存 Activity 状态信息，请查看 [Activities](http://developer.android.com/guide/components/activities.html#SavingActivityState) 文档。

### 管理任务

---

有时候，我们想在新的任务中启动一个 Activity ，或者，当启动一个 Activity 时，可以直接把已经存在的对象拉到前台，而不是在栈顶启动新的对象。或者，当用户离开任务时，我们想使返回栈清除所有的 Activity 除了根 Activity 。

这个时候，我们可以通过以下几种方式来实现：

#### 定义启动模式

启动模式允许我们定义一个 Activity 的新对象如何与当前任务相关联。我们可以通过 manifest 文件或 intent flag 定义。

假设 Activity A 启动 B ，B 可以在 manifest 文件中定义如何与当前任务关联，A 可以请求定义 B 如何与当前任务关联。如果二者都定义了 B 的启动模式，以 A 在 intent flag 中定义的为准，而不是 B 在 manifest 文件中所定义的。

> 注意：一些启动模式只在 manifest 文件中可用，而不能在 intent flag 中添加，同样的，一些启动模式只在 intent flag 中可用，而非 manifest 文件。

##### manifest 文件

[launchMode](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode) 标签可以设置四种指令，以便控制 Activity 如何进入任务。

- `standard` 默认模式

系统在启动该 Activity 的任务中创建一个新对象，并且发送 intent 。它能够被多次创建新对象，每个对象能属于不同的任务，并且一个任务中可以有多个对象。

- `singleTop`

如果当前任务的顶部已经存在一个 Activity 的对象，系统将会通过调用 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法给该对象发送 intent ，而非创建新对象。它能够被多次创建新对象，每个对象可以属于不同的任务，并且一个任务可以拥有多个对象（只有当返回栈顶的 Activity 对象不是该 Activity 对象）。

例如当前返回栈为 A-B-C-D ，D 在栈顶，现在从 D 要启动 D ，如果 D 是 `standard` 模式，那么会创建新对象，栈变为 A-B-C-D-D 。然而，如果 D 的启动模式为 `singleTop` ，已经存在的 D 对象通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 接收 intent ，因为其在栈顶，栈仍然是 A-B-C-D。但如果启动 B ，则创建新的 B 对象加入栈顶，即使它的启动模式为 `singleTop` 。

- `singleTask`

系统创建新的任务，并且实例化新的对象作为 root 。然而，如果该 Activity 对象在单独的任务中已经存在，系统会通过调用 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 方法发送 intent 给此对象，而非创建新对象。同一时间点只能存在一个对象。

> 注意：尽管 Activity 在新任务中启动，但是按下 Back 键仍然返回上一个 Activity 。

- `singleInstance`

和 `singleTask` 相同，除了该对象独占一个任务，该任务中有且仅有一个该 Activity 对象。任何被它启动的 Activity 对象会新启动一个单独的任务。

无论新启动的 Activity 是否在新任务中，按下 Back 键总是为用户呈现前一个 Activity 。然而，如果你启动的 Activity 指定了 `singleTask` 启动模式，并且如果后台任务中已经存在该 Activity 的对象，那么整个任务都要被拉到前台。此时，栈顶包含了被拉到前台任务的所有 Activity 。下图展示了这种场景：

![diagram_backstack_singletask_multiactivity](/media/files/2016/01/16/diagram_backstack_singletask_multiactivity.png)

> 注意：通过 [launchMode](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode) 标签指定的启动模式能够被 intent 包含的 flag 所覆盖。

##### 使用 intent flags

* [FLAG_ACTIVITY_NEW_TASK](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)

在新任务中启动 Activity 。如果一个任务中已经运行了要启动的 Activity ，这个任务会被拉到前台，该 Activity 会通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 接收 intent 。

该 flag 和 `singleTask` 具有相同的行为。

* [FLAG_ACTIVITY_SINGLE_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_SINGLE_TOP)

如果将要启动的 Activity 是当前的 Activity （在返回栈顶），那么当前 Activity 会通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 接收 intent 。而非创建新对象。

该 flag 和 `singleTop` 具有相同的行为。

* [FLAG_ACTIVITY_CLEAR_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TOP)

如果将要启动的 Activity 已经在当前任务中运行，那么不会创建新对象，在它顶部的所有 Activity 被销毁，intent 通过 [onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent\)) 发送到这个 Activity 中（现在已经到了栈顶）。

launchMode 标签中没有和它行为一致的值。

`FLAG_ACTIVITY_CLEAR_TOP` 通常和 `FLAG_ACTIVITY_NEW_TASK` 一起使用。当它们一起使用时，将会在另一个任务中寻找已存在的 Activity 对象，并且把它放在可以响应 intent 的位置上。

> 注意：如果指定启动的 Activity 是 `standard` 模式，它也会被从栈中移出，并且在相同位置上重新创建新对象处理 intent ，这是由于当启动模式为 `standard` 时，总要为新 intent 创建新对象。


### 进一步了解

---

[android:launchMode](http://developer.android.com/guide/topics/manifest/activity-element.html#lmode)  
[Understand Android Activity's launchMode: standard, singleTop, singleTask and singleInstance](http://inthecheesefactory.com/blog/understand-android-activity-launchmode/en) [(译文)](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)
