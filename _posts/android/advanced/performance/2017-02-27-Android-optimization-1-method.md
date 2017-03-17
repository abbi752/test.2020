---
layout: post
title: 性能优化（一）：方法概述
date: 2017-2-27
excerpt: "方法概述"
categories: Android
tags: [Android 性能优化]
comments: true
lefttrees: true
---

* content
{:toc}


# 一、 布局优化

布局优化的思想：尽量减少层级和无用的控件。 

1. Layout 设计优化

    在布局设计时，就应该考虑最优化思想。下面列出一些常用的技巧：

    - 有选择地使用性能较低的ViewGroup.比如不嵌套的情况下，用LinearLayout和FrameLayout代替RelativeLayout.
        - RelativeLayout功能比较复杂，布局过程需要花费更多的CPU时间
        - 但是如果要LinearLayout嵌套来代替RelativeLayout，还是建议用RelativeLayout。因为嵌套同样会降低程序的性能  
    - 使用include实现布局重用，避免代码重复
    - 使用merge减少布局层级结构
    - 使用ViewStub实现延时加载
    - 在TextView中使用Compound drawable，取代ImageView + TextView
    - 使用LinearLayout自带的分割线: android:divider=""

2. Hierarchy Viewer工具优化布局
   

详见：[性能优化（四）Google典范之Render实践中Layout优化章节](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 二、 绘制优化

1. 移除不必要的background
2. 用clipRect优化
3. onDraw方法要避免执行大量的操作

详见：[性能优化（四）Google典范之Render实践中Overdraw章节](http://vivianking6855.github.io/2017/03/14/Android-optimization-4-Google-Publish-Render/)

# 三、 内存泄漏优化

内存泄漏是在开发过程需要重视的问题。内存泄漏优化分两个方面：

- 开发过程中避免写出有内存泄漏的代码
- 通过一些分析工具如Android Studio自动Android Monitor，[leakcanary](https://github.com/square/leakcanary)，FindBugs等

详见[Android性能优化 （二）内存 OOM](http://vivianking6855.github.io/tag/#Android%20%E8%BF%9B%E9%98%B6-ref)

# 四、 响应速度优化

响应速度优化核心思想是避免在主线程中做耗时的操作。耗时的操作应该放到异步任务中执行。

# 五、 ListView优化

构造Adapter时，没有使用缓存的 convertView，可以使用Android最新组件RecyclerView,替代ListView来避免

# 六、 Bitmap优化

主要是BitmapFactory.Options 和缓存。详见[Android Bitmap的加载和Cache](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

# 七、 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread. 控制最大并发数等。详见下面两篇blog~

[Android的线程和线程池，线程同步](http://vivianking6855.github.io/tag/#Android%20%E5%9F%BA%E7%A1%80-ref)

# 八、 耗电优化

电量其实是目前手持设备最宝贵的资源之一，大多数设备都需要不断的充电来维持继续使用。

Purdue University研究了最受欢迎的一些应用的电量消耗：

- 平均只有30%左右的电量是被程序最核心的方法例如绘制图片，摆放布局等等所使用掉的
- 剩下的 70%左右的电量是被上报数据，检查位置信息，定时检索后台广告信息所使用掉的。

如何平衡这两者的电量消耗，就显得非常重要了。有下面一些措施能够显著减少电量的消耗：

- 我们应该尽量减少唤醒屏幕的次数与持续的时间，使用WakeLock来处理唤醒的问题，能够正确执行唤醒操作并根据设定及时关闭操作进入睡眠状态。
- 某些非必须马上执行的操作，例如上传歌曲，图片处理等，可以等到设备处于充电状态或者电量充足的时候才进行。
- 触发网络请求的操作，每次都会保持无线信号持续一段时间，我们可以把零散的网络请求打包进行一次操作，避免过多的无线信号引起的电量消耗。
    - 关于网络请求引起无线信号的电量消耗，还可以参考[这里](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)
- App有电量消耗过多的问题，我们可以使用[JobScheduler](http://hukai.me/android-training-course-in-chinese/background-jobs/scheduling/index.htm) API来对一些任务进行定时处理
    - 例如我们可以把那些任务重的操作等到手机处于充电状态，或者是连接到WiFi的时候来处理。
- 使用WakeLock或者JobScheduler唤醒设备处理定时的任务之后，一定要及时让设备回到初始状态。
- 每次唤醒无线信号进行数据传递，都会消耗很多电量，它比WiFi等操作更加的耗电，[详情](http://hukai.me/android-training-course-in-chinese/connectivity/efficient-downloads/efficient-network-access.html)


我们可以通过手机设置选项找到对应App的电量消耗统计数据。我们还可以通过Android 5.0发布的Battery History Tool来查看详细的电量消耗。

请关注程序的电量消耗，用户可以通过手机的设置选项观察到那些耗电量大户，并可能决定卸载他们。所以尽量减少程序的电量消耗是非常有必要的。

# 九、 性能优化小建议

性能优化的小建议：

- 避免创建过多的对象
- 不要过多使用枚举，枚举占用内容空间比整型大
- 常量请使用static final来修饰
- 使用一些Android特有的数据结构，比如SparseArray，Pair等，它们都具有更好的性能
- 适当使用软引用和弱引用
- 采用内存缓存和磁盘缓存
- 尽量采用静态内部类，避免潜在的由于内部类而导致的内存泄漏。
- 避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外
- 自定义View中避免在onDraw方法里面执行复杂的操作，避免创建对象。
- 对于那些无法避免需要创建对象的情况，可以考虑对象池模型。
    - 通过对象池来解决频繁创建与销毁的问题。
    - 需要注意结束使用之后，需要手动释放对象池中的对象。


<br/><br/>


# Reference

<br>



> 性能优化专题

> 《Android开发艺术探索》