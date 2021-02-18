---
layout: post
title: "iOS 面试百宝箱(4):多线程"
subtitle: "iOS Interview(4):Multithread"
date: 2021-02-14
author: "JJP"
header-img: "img/posts/post-bg-code.jpg"
header-mask: 0.3
tags: [iOS]
---

啦啦啦, 我们来复习iOS 多线程编程. 这部分知识点也是面试的高频问题, 特别是关于NSRunLoop和GCD的知识点.  这两块知识点在iOS程序优化和底层编程中经常用到, 所以面试官比较看重. 大家面试前可以重点复习.

废话不多说, 黑喂狗!

<figure class="image">
  <img src="/img/posts/post-iosinter-multithread-mindmap.png" alt="mindmap"/>
  <center><figcaption><b>复习脑图Mindmap</b></figcaption></center>
</figure>

## iOS Multhread Programming Q&A



#### Q: Compare GCD and NSOperationQueue?

**A:** Both GCD and NSOperationQueue help us manages tasks running on multiple thread. But NSOperationQueue is a heavier framework compared to GCD. Internally, NSOperationQueue provides a few additional functionanlities that GCD can not provide or hard to implement with. For example, dependencies between tasks,  management of life cycle of tasks. NSOperationQueue is implemented by GCD internally.



#### Q: How do you prevent racing condition in iOS?

**A:**  There are many ways to prevent race condition. 

1. Dispatching tasks on a serial queue
2. Locks/mutex
3. Semaphore



#### Q: Why don't we use `atomic` to ensure thread safety?

**A:** From [stackoverflow](https://stackoverflow.com/questions/588866/whats-the-difference-between-the-atomic-and-nonatomic-attributes#:~:text=Atomic%20means%20only%20one%20thread,unsafe%2C%20but%20it%20is%20fast.): With "atomic", the synthesized setter/getter will ensure that a *whole* value is always returned from the getter or set by the setter, regardless of setter activity on any other thread. That is, if thread A is in the middle of the getter while thread B calls the setter, an actual viable value -- an autoreleased object, most likely -- will be returned to the caller in A.

What "atomic" does **not** do is make any guarantees about thread safety. If thread A is calling the getter simultaneously with thread B and C calling the setter with different values, thread A may get any one of the three values returned -- the one prior to any setters being called or either of the values passed into the setters in B and C. Likewise, the object may end up with the value from B or C, no way to tell. `atomicity` of a single property also cannot guarantee thread safety when multiple dependent properties are in play.

#### Q: Tell me at least 4 different kinds of locks in iOS. What are their differences and usecase?

**A:**  Two major kinds of locks:

- Busy-waiting locks 
  - OSSpinLock
  - os_unfair_loc
- Sleep lock
  - pthread_mutex
  - dispatch_semaphore
  - NSLock 
  - NSRecursiveLock
  - NSCondition
  - NSConditionLock
  - @synchronized

Busy-waiting locks keeps running and trying to acquire the lock. Obviously it consumes more CPU, however, it is faster compared to sleep locks which become inactive when the lock is unavailable, because of there's no context switching for busy waiting locks. Below are the summary of the performance for different locks.

<figure class="image">
  <img src="/img/posts/post-iosinter-multithread-lockperform.png" alt="performance"/>
  <center><figcaption><b>Performance of locks</b></figcaption></center>
</figure>

more on this [here](https://juejin.cn/post/6844903914190405640)

##### Follow up: Why do we stop using `OSSpinLock`

**A:**  Because of the issue of **Priority Reversing**. Priority reversing describes the problems that when two threads with different priority try to accquire the lock, the higher priority thread always get to run, even if the thread with lower priority currently holds the lock, which leads to a deadlock problem.

more on this [here](https://xuyunan.github.io/posts/ios-lock-osspinlock/)



#### Q: How `@synchronized` is implemented? 

**A:** In essense, `@synchronized` is implemented with the recursive lock. The advantage of recursive locks compares to normal lock is that there will not be deadlock if you accquire the lock multiple times within the same thread. 

Runtime has a so called `SyncList` data structure , in which each node called `SyncData`. And each `SyncData` structure has a dedicated recursive lock. When we call `@synchronized(obj)`, the runtime will find the `SyncData` using the index value of the synchronized object, which is calculated by its memory address hash value and accquire the lock of that `SyncData`.

**Follow up: Why happends when you do `@synchronized(nil)`**

**A:** no locks will be used. The code within the block will run normally.



#### Q: Imagine you have 3 tasks, A, B and C. You want C to execute only after A and B have both completed execution. What do you do?

**A:** There are multiple ways, I give two solution here:

1. Using simple a  simple global variable `count`. 

{% highlight objc %}

```objective-c
dispatch_queue_t queue = dispatch_queue_create("testqueue",DISPATCH_QUEUE_CONCURRENT);

__block int finishCount = 0;
dispatch_block_t taskC = ^(){
  //do task C
}
dispatch_block_t taskA = ^(){
  //do task A
  finishCount += 1
  if (finishCount == 2) {
    taskC()
  }
}
dispatch_block_t taskB = ^(){
  //do task B
  finishCount += 1
  if (finishCount == 2) {
    taskC()
  }
}
dispatch_async(queue, taskA);
dispatch_async(queue, taskB);
```

{% endhighlight %}

2. Using `dispatch_group`. Cleaner than the method using external count.

{% highlight objc %}

```objective-c
dispatch_block_t taskC = ^(){
  //do task C
}
dispatch_queue_t disqueue = dispatch_queue_create("testqueue",DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t disgroup = dispatch_group_create();
dispatch_group_async(disgroup, disqueue, ^{
  //do task A
});

dispatch_group_async(disgroup, disqueue, ^{
  //do task B
});

dispatch_group_notify(disgroup, disqueue, ^{
  taskC();
});
```

{% endhighlight %}

#### Q: Imagine you want to support read/write concurrent ops to a shared resources. How do you make sure writing ops exclusively access the resources?

**A:** We could use `dispatch_barrier`.  It creats a sync point within a concurrent queue, such that the block in the `dispatch_barrier ` will always run without other concurrent operations. 

We can utilize this feature to support synchronization between concurrent writes and reads.

{% highlight objc %}

```objective-c
dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(queue, ^{
  //read task1
});
dispatch_async(queue, ^{
   //read task2
});

dispatch_barrier_async(queue, ^{
  // write task, which will execute exclusively
  // in this case, after task1 and task2 are completed
});

dispatch_async(queue, ^{
	// read task3
});
```

{% endhighlight %}



#### Q: What is NSRunLoop?

**A:** From [Apple documentation](https://developer.apple.com/documentation/foundation/nsrunloop?language=objc): A `NSRunLoop` object processes input for sources such as mouse and keyboard events from the window system, [`NSPort`](https://developer.apple.com/documentation/foundation/nsport?language=objc) objects, and [`NSConnection`](https://developer.apple.com/documentation/foundation/nsconnection?language=objc) objects. A `NSRunLoop` object also processes [`NSTimer`](https://developer.apple.com/documentation/foundation/nstimer?language=objc) events. 

But I prefer how one [Stackoverflow](https://stackoverflow.com/questions/12091212/understanding-nsrunloop) post explains it: 

> A run loop is an abstraction that (among other things) provides a mechanism to handle system input sources (sockets, ports, files, keyboard, mouse, timers, etc).
>
> Each NSThread has its own run loop, which can be accessed via the currentRunLoop method.
>
> In general, you do not need to access the run loop directly, though there are some (networking) components that may allow you to specify which run loop they will use for I/O processing.
>
> A run loop for a given thread will wait until one or more of its input sources has some data or event, then fire the appropriate input handler(s) to process each input source that is "ready.".
>
> After doing so, it will then return to its loop, processing input from various sources, and "sleeping" if there is no work to do.

So in summary, Runloop

- Keep the program running continuously
- Monitor and process various events in App (touch events, timer events, selector events)
- Save CPU resources and improve program performance: do things while doing, rest when doing things
- A RunLoop loop is responsible for drawing all points on the screen.



#### Q: What are the different modes in NSRunLoop? What are their differences?

**A:**  Refering to the blow image, thread and runloop are one-to-one  relation. However,  we need to create runloop and assign it to a thread (except main runloop/main thread which is created automatically by the system).

Each runloop has multiple modes. Modes are a set of sources, timers and observers, which are events that current thread handles. Runloop can switch modes and by switching modes, thread handles different set of events with different priorities.  Here are some system defined modes:

1. `kCFRunLoopDefaultMode`：the default mode of the main runloop. 
2. `UITrackingRunLoopMode`：a mode for ScrollView tracking，which ensures scrolling  and UI updates have higher priority.
3. `UIInitializationRunLoopMode`: the first mode when app starts, unused afterwards.
4. `GSEventReceiveRunLoopMode`: a mode for receiving internal events, not used often
5. `kCFRunLoopCommonModes`: a placeholder mode for labelling both `kCFRunLoopDefaultMode and UITrackingRunLoopMode`. 

<figure class="image">
  <img src="/img/posts/post-iosinter-runloop.png" alt="runloop"/>
  <center><figcaption><b>NSRunloop and modes</b></figcaption></center>
</figure>



#### Q: Give me some usecase of `NSRunloop`?

**A:**  More on this [here](https://www.jianshu.com/p/adf9eb244e81)





#### References

- https://www.cnblogs.com/lxlx1798/p/10051974.html
- https://vincents.cn/2017/03/14/ios-lock/
- https://juejin.cn/post/6844903914190405640
- https://blog.csdn.net/totogo2010/article/details/8016129
- https://www.jianshu.com/p/adf9eb244e81
- https://juejin.cn/post/6844903604965523464

