---
layout: post
title: "iOS 面试百宝箱(3):Objective-C运行时"
subtitle: "iOS Interview(3):Objective-C Runtime"
date: 2020-01-02
author: "JJP"
header-img: "img/posts/post-bg-code.jpg"
header-mask: 0.3
tags: [iOS]
---

啦啦啦, 我们继续复习iOS Objective-C的知识. 今天是Objective-C的大头----Runtime. Runtime可以说是Objective-C最难,也是最有趣的部分了. 理解好Runtime对日常开发可以说是如虎添翼. 所以面试中Runtime也是最经常被问到的知识点. 

废话不多说, 黑喂狗!

<figure class="image">
  <img src="/img/posts/post-iosinterview-mindmapoc.png" alt="mindmap"/>
  <center><figcaption><b>复习脑图Mindmap</b></figcaption></center>
</figure>

## Objective-C Runtime Q&A

#### Q: Why do we say Objective-C is dynamic language? What is Objective-C Runtime? 

**A:** Objective-c is based on C but with additional OOP features. Additionally, unlike C being a static language whose function invocation is fixed at compiling and linking time, a lot of such decision are delayed to runtime. The primary mechanism of achieving such goal is called **Messaging**.

Objective-C Runtime  is a set of API that is written by C, C++ and Assemble, whose purpose is to add OOP capability to OC and messaging mechanism.



#### Q: Explain Runtime messaging process? When will there be `unrecognized selector` Exception?

**A:** TODO

##### Follow up: Give me a usecase of it

**A:** TODO



#### Q: How Object is implemented in ObjC?

**A:** TODO



#### Q: Do you know `isa` pointer? What is it used for?

**A:** TODO

##### Follow up: Explain to me what will happen if we call `[[NSArray array] count]`?

**A:** TODO

##### Follow up: How about `[NSArray array]`?

**A:** TODO

##### Follow up: Speed up the look up?

**A:** TODO



#### Q: What is `SEL` and `IMP`? How they are used?

**A:** TODO





#### Q: What is  `class_rw_t`  and ``class_ro_t``?

**A:** TODO



#### Q: Can we add instance variable at runtime? 

**A:** TODO

##### Follow up: What can we do to add iVar at runtime?

**A:** TODO

##### Follow up: Give me a usecase of it

**A:** TODO

##### Follow up: When do we need to dealloc associated objects?

**A:** TODO



#### Q: What is the output of the following program?

{% highlight objc %}

```
@implementation Son : NSObject
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

{% endhighlight %}

**A:** TODO



#### Q: What is Method Swizzling? 

**A:** TODO

##### Follow up: Give me a usecase of it.

**A:** TODO



#### Q: NSObject `+load` and `+initialize` - What do they do?

**A:** TODO

More on this [here](https://stackoverflow.com/questions/13326435/nsobject-load-and-initialize-what-do-they-do)



#### References

- [https://juejin.cn/post/6844904079957688328](https://juejin.cn/post/6844904079957688328)

- [https://juejin.cn/post/6844904090128891912](https://juejin.cn/post/6844904090128891912)

- [https://juejin.cn/post/6844904090129039374](https://juejin.cn/post/6844904090129039374)

- [https://www.jianshu.com/p/19f280afcb24](https://www.jianshu.com/p/19f280afcb24)

- [https://www.jianshu.com/p/56e40ea56813](https://www.jianshu.com/p/56e40ea56813)

- [https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Runtime.html](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Runtime.html)

  