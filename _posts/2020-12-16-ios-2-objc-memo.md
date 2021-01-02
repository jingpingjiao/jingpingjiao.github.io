---
layout: post
title: "iOS 面试百宝箱(2):Objective-C内存管理"
subtitle: "iOS Interview(2):Objective-C Memory Management"
date: 2020-12-16
author: "JJP"
header-img: "img/posts/post-bg-code.jpg"
header-mask: 0.3
tags: [iOS]
---

啦啦啦, 今天开始我们来复习iOS面试中重要的一块知识点:Objective-C(OC).  OC是iOS开发的大头, 虽然很多新的代码建议大家用Swift, 还是有很多很多legacy code还是用的OC, 特别是大厂的大App. 

<figure class="image">
  <img src="/img/posts/post-iosinterview-mindmapoc.png" alt="mindmap"/>
  <center><figcaption><b>复习脑图Mindmap</b></figcaption></center>
</figure>
因为ObjC东西太多了, 我只能把内容分成几个部分. 这篇就着重复习ObjC的内存管理Memory Management.  有些没能展开细讲的知识点, 我会附上一些相关博客链接供大家查阅.  文章的形式我就以英文问答的形式吧, 模拟英文面试, 习惯中文的小伙伴也可以趁机练练英文阅读. 

废话不多说, 黑喂狗!

## Objective-C Memory Management Q&A

#### Q: What is Reference Counting in Objective-C?

**A:** Reference counting is a way of the system to manage memory, which is by keeping a number of times an object is referred to by other pointers. If the reference count of an object is zero, the system will release its memory at some point of time. 

##### Follow up: Tell me the difference between MRC and ARC?

**A:** MRC is short for Manual Reference Counting and ARC is automatic reference counting. Because in Objective-C, almost all object are heap allocated and its developer's responsibility to correctly manage the memory. Before OC5.0, it's manual, hence Manual Reference Counting.  For example, We use `[obj retain]` to tell the runtime we increase the reference count, and `[obj release]` to reduce reference count. After OC5.0, ARC comes and these things are managed by compiler, we no longer need to do that by ourselves.

#### Q: Explain retain cycle and how to avoid it?

**A:** As I mentioned before, objects will only be released if their reference count become 0. Retain cycle happens if two or more objects hold strong reference to each other, which form a cycle, thus their reference count can never be 0. For example, a common retain cycle is a ViewController holds a strong reference to block which is a network API callback. If the block capture self strongly, then you have a retain cycle. A way to resolve it is to declare a `weak self` and let block capture the `weak self` instead.

##### Follow up 1: How do you detect retain cycle?

**A:** There are multiple techniques can be used. For example:

1. FBRetainCycleDetector
2. Symbolic breakpoint
3. Instruments Memory Graph
4. Instruments Allocation and Leaks

More on this in [here](https://sarunw.com/posts/easy-way-to-detect-retain-cycle-in-view-controller/), [here](https://medium.com/zendesk-engineering/ios-identifying-memory-leaks-using-the-xcode-memory-graph-debugger-e84f097b9d15) and [here](https://doordash.engineering/2019/05/22/ios-memory-leaks-and-retain-cycle-detection-using-xcodes-memory-graph-debugger/).

##### Follow up 2: Explain how improper use of `NSTimer` can lead to retain cycle? How to solve it?

**A:** If we use the API `[NSTimer scheduleWithTarget: selector:]`,  `NSTimer` will hold strong reference to its target. If you fire a `NSTimer` in a ViewController and the ViewController hold strong reference to it, it will be a reference cycle. You have to invalidate the timer and set the reference to nil. Or we can use another API `[NSTimer scheduledTimerWithTimeInterval: repeats:block:]`, as long as we use `weak self` , there won't be retain cycle.

##### Follow up 3: What if your `UIViewController` don't hold strong reference to the `NSTimer`?

**A:** There will still be a memory leak. `UIViewController` will not be dealloc. Because `NSRunLoop` will hold strong ref to `NSTimer`, `NSTimer` holds ref to `UIViewController`. 

<figure class="image">
  <img src="/img/posts/post-iosinter-nstimer.png" alt="retaincycle"/>
</figure>

More on this in [here](https://juejin.cn/post/6907514351180218381)

#### Q: Explain difference and usage of `weak, strong , assign`?

**A:** They are property modifiers, which describe different reference types of the pointer . `strong` and `weak` are pointer types for objects and `assign` is for basic types. **Basic types are  stack-allocated** which are automatically managed by the system; **ObjC objects are heap-allocated** which should be managed by developer. mostly   `strong` means that the current object will hold strong reference to the ivar, which leads the reference count for the ivar to increase by 1. `weak` on the other hand would not affect the ivar's reference count. Moreover, if the ivar is deallocated, the `weak` pointer will be set to `nil`.

##### Follow up 1: Can you use assign to an object? Why so or why not so? 

**A:** Technically you can use `assign` for object. But the problem is the system won't set the pointer to `nil` after deallocating the referred object. So if you try to use the same pointer, it will cause a `EXC_BAD_ACCESS` runtime error, which essentially says that you are accessing a wild pointer. 

For example, say you have a VC which has a `UIButton` property:

{% highlight objc %}

@interface ViewController
@property(nonatomic, assign) UIButton *btn;
@end {% endhighlight %}

​		Then you create a `UIButton` and assign to the VC:

{% highlight objc %}

UIButton *btn = [[UIButton alloc] initWithFrame:....];
vc.btn = btn;
btn = nil;
//later, somewhere else
NSLog(@"%@", vc.btn); //Error. EXC_BAD_ACCESS (code=....){% endhighlight %}

##### Follow up 2: Do you know how the system implements `weak` pointer?  Namely how a `weak` pointer be set to `nil` after the object being deallocated?

**A:** In principle, the runtime will hold a weak pointer mapping table, whose keys are object memory address and values are lists of weak pointer addresses who point to the object. Everytime, you assign let a weak pointer points to an object, the system will add the pointer list for the corresponding object. When the object is deallocated, the runtime will lookup its memory address, get the weak pointer list, and set them all to `nil`.

<figure class="image">
  <img src="/img/posts/post-iosinter-weakptr.png" alt="weak ptr"/>
</figure>

More on this in [here](https://juejin.cn/post/6844903735651647502),

#### Q: What is AutoReleasePool and why we need it?

**A:** AutoReleasePool is memory managing mechanism. It's a way of keeping an object around you are not sure when to release it. 

Whenever an object instance is marked as "autoreleased" (e.g `NSArray* arry = [[NSArray array] autorelease];`), it will have a retain count of +1 at that moment in time. And the object is add to AutoReleasePool. At the end of the run loop, the pool is drained, and any object marked autorelease then has its retain count decremented.  But with ARC, developer not longer needs to manually call `[obj autorelease]`, but only needs to enclose the code using `@autoreleasepool{}`

##### Follow up 1: Do you know how it's implemented?

`@autoreleasepool{}` can be loosely translate into:

{% highlight objc %}

```
@autoreleasepool {
    void *ctx = objc_autoreleasePoolPush(){
        void *objc_autoreleasePoolPush(void)
                                |
        void *AutoreleasePoolPage::push(void)
     };
 
    objc_autoreleasePoolPop(ctx){
        void objc_autoreleasePoolPop(void *ctxt)
                                  |
        AutoreleasePoolPage::pop(void *ctxt)
    };

```

{% endhighlight %}

`AutoReleasedPool` is implemented by a doublly-link list of a struct called `AutoReleasePoolPage`. Each `AutoReleasePool` holds a stack, every autorelease object will be push to the current `AutoReleasePage`'s stack. Each AutoReleasePoolPage has fixed memory size, so if there are too many autorelease objects in one's stack, system will create one new AutoReleasePoolPage, push the object to its stack and add the new AutoReleasePoolPage to the doubly-link list.

<figure class="image">
  <img src="/img/posts/post-iosinter-autoreleasepool.png" alt="autoreleasepool"/>
</figure>

At end of a AutoReleasePool, AutoReleasePoolPage will pop objects in its stack and send `[obj release]` to them all.  

Moreover, at the beginning of `autoreleasepool`, `AutoReleasePoolPage` will push a Sentinel object, which actually a nil object, to mark the beginning of current auto release pool. So we will only release objects up to the Sentinel object when we reach end of current auto release pool. 

<figure class="image">
  <img src="/img/posts/post-iosinter-nestautoreleasepool.png" alt="autoreleasepool"/>
</figure>

##### Follow up 2: How many AutoReleasePoolPage can a RunLoop have?

We can see from the `AutoReleasePoolPage` simplified definition below, each corresponds to one thread. 

<figure class="image">
  <img src="/img/posts/post-iosinter-autoreleasepage.png" alt="autoreleasepool"/>
</figure>

But in case one `AutoReleasePoolPage` can not fit all the autorelease objects, there would be more `AutoReleasePoolPage.

##### Follow up 3: When will object be released in AutoReleasePool?

1. RunLoop will crate a AutoReleasePool at beginning of each loop, and destroy runloop at end of each loop.  
2. For user created AutoReleasePool, at end of the scope.

##### Follow up 4: Give me an example of using it?

{% highlight objc %}

```
for(int i=0;i<100;i++) {
    UIView *view = [UIView new];
    // do something
}
// do more thing
```

{% endhighlight %}

For example, in the above code, many objects are created without deallocation, which will cause intense usage of memory. What we can do is to wrap the creation part with `@autoreleasepool`, so that objects will be released in time.

{% highlight objc %}

```
for(int i=0;i<100;i++) {
  	@autoreleasepool{
			UIView *view = [UIView new];
    	// do something
   	}
}
// do more thing
```

{% endhighlight %}

More on this in [here](https://juejin.cn/post/6844904013465387022) and [here](https://blog.csdn.net/ochenmengo/article/details/105069297).

