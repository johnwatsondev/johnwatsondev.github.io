---
layout: post
title: 在 ListView 的 onScroll() 中移除 footerView 失败
guid: true
comments: false
tags: [ListView]
---

作者：[JohnWatsonDev](http://johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 问题
最近优化之前写的 ListView 下拉刷新/加载更多 Demo，在 `onScroll()` 中移除 listView 的 footerView 时，抛出：`java.lang.IndexOutOfBoundsException: Invalid index 0, size is 0`.

### 问题来源
为何要在 `onScroll()` 中移除？为了提升 [UX](https://en.wikipedia.org/wiki/User_experience) 。  
之前在滚动到底部时设置 footerView 状态，footerView 中内容变化会被用户看到，开始滚动时，判断当前是否启用加载更多，若启用则添加 footerView，否则移除。  
这样，当滚动到底部时 footerView 的状态已经提前设置完成，[UX](https://en.wikipedia.org/wiki/User_experience) 较佳。

### 异常
异常信息：

{% highlight java %}
FATAL EXCEPTION: main
Process: com.jwd.ptrlv.v2, PID: 4255
java.lang.IndexOutOfBoundsException: Invalid index 0, size is 0
at java.util.ArrayList.get(ArrayList.java:308)
at android.widget.HeaderViewListAdapter.isEnabled(HeaderViewListAdapter.java:164)
at android.widget.ListView.dispatchDraw(ListView.java:3307)
at android.view.View.draw(View.java:15234)
at android.widget.AbsListView.draw(AbsListView.java:4110)
at android.view.View.updateDisplayListIfDirty(View.java:14167)
at android.view.View.getDisplayList(View.java:14189)
at android.view.View.draw(View.java:14959)
at android.view.ViewGroup.drawChild(ViewGroup.java:3405)
at android.view.ViewGroup.dispatchDraw(ViewGroup.java:3198)
at android.view.View.updateDisplayListIfDirty(View.java:14162)
{% endhighlight %}

### 分析
我们看到在 `ListView.dispatchDraw()` 中抛出了异常，此方法主要作用为：被 `View.draw(Canvas canvas)` 方法调用绘制孩子视图，子类可以重写该方法，在孩子绘制之前并且在自身绘制完毕之后调用。  
接着往下走，来到 `HeaderViewListAdapter.isEnabled(int position)` 方法，主要判断位于 postion 位置的 item 是否为可选择和可点击的。  
最终报错的一行代码为：

{% highlight java %}
return mFooterViewInfos.get(adjPosition - adapterCount).isSelectable;
{% endhighlight %}

错误为下标越界，那么再梳理下逻辑：  
我们在 `onScroll()` 方法里移除 footerView，调用 `ListView.removeFooterView(View v)` 方法，该方法继而调用 `HeaderViewListAdapter.removeFooter(View v)` 方法，然后 HeaderViewListAdapter 中的 mFooterViewInfos 移除掉 footerView，那么 mFooterViewInfos 将立即发生变化，而 `HeaderViewListAdapter.isEnabled(int position)` 传入的 position 来自 `ListView.dispatchDraw()` 中循环所有孩子视图（headerView ＋ item ＋ footerView）的个数，所以在循环开始后，position 无法及时变化，必然会发生越界行为。

用一句话总结，就是在视图绘制的过程中，强行把视图数据改变，势必出现错误，那最后的解决办法是什么呢？

### 解决办法

{% highlight java %}
this.post(new Runnable() {

      @Override
      public void run() {
        PtrListView.this.removeFooterView(mFooterView);
        PtrListView.this.requestLayout();
      }
});
{% endhighlight %}

原先的错误代码为：
{% highlight java %}
PtrListView.this.removeFooterView(mFooterView);
this.post(new Runnable() {

      @Override
      public void run() {
        PtrListView.this.requestLayout();
      }
});
{% endhighlight %}

微小改动的奥秘在哪里呢？`View.post(Runnable action)` 方法的代码在主线程执行，当 ListView 移除 footerView 时，视图已经绘制完毕，再次绘制视图便没有问题。

附上代码链接 [PtrListView.java](https://gist.github.com/johnwatsondev/5922a3be8bc2555bff93)
