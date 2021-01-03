---
layout: post
title: "iOS 面试百宝箱(3):Objective-C运行时"
subtitle: "iOS Interview(3):Objective-C Runtime"
date: 2021-01-03
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

**A:** Objective-c is based on C but with additional OOP features. Objective-C Runtime  is a set of API that is written by C, C++ and Assemble, whose purpose is to add OOP capability to OC and messaging mechanism. Additionally, unlike C being a static language whose function invocation is fixed at compiling and linking time, a lot of such decision are delayed to runtime. The primary mechanism of achieving such goal is called **Messaging**.



#### Q: Explain Runtime messaging process? 

**A:** Function call to object will be translate to 

{% highlight objc %}

objc_msgSend(id self, SEL op, ...>)

{% endhighlight %}

whereby `id` is the receiving object and `SEL` is the encoded method name.  Then Runtime will follow the object's `isa` pointer and look up the `IMP` (function implmentation) in the class object's `methodLists`. If not found, it will follow its super pointer and look up the `SEL` in the super class object until reach root class object `NSObject` or `NSProxy`. Moreover, this look up process can be speed up by caching.

More on this [here](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Runtime.html#%E6%B6%88%E6%81%AF%E4%BC%A0%E9%80%92%E6%9C%BA%E5%88%B6).

#### Q: When will there be an `unrecognized selector` Exception?

**A:** If the `IMP` lookup fails, the runtime would proceed with the **Dynamic Method Resolution** process:

1. **Deciding whether to add method dynamically**: Runtime first invoke `+resolveInstanceMethod` to give us an opportunity to add method dynamically . If we return YES, we need to `class_addMethod` to add method and handle the message. Otherwise it would go to next step.

2. **Deciding which object to respond:** In this step, runtime will invoke `+forwardingTargetForSelector` to decide which object handle the message. If return `nil`, then go to step 3.  If we return a non-nil object, runtime will pack the invocation and send it to the returned object.

3. **Deciding whether to change the message:** In this step, runtime allows us to change the message signature by calling  `+methodSignatureForSelector`.  If we return `nil`,  then runtime would raise `unrecognized selector sent to instance` exception. Otherwise, it will go to `+forwardInvocation` . If succeeds, then the process ends with success. Otherwise, runtime raises the exception.

<figure class="image">
  <img src="/img/posts/post-iosinter-methodresolve.png" alt="method resolve"/>
</figure>

##### Follow up: Give me a usecase of Dynamic Method Resolution

**A:** If a class has too many method, it would take a lot of time loading it. We can add it via runtime API `class_addMethod` lazily in `+ (BOOL)resolveInstanceMethod:(SEL)sel`. 



#### Q: How Object is implemented in ObjC?

**A:** Object is `objc_class`, which means every object in Object has a pointer to its class object

{% highlight objc %}

```
/// Represents an instance of a class.
struct objc_object {
    Class isa;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

{% endhighlight %}

Then the `Class` object is defined as follow:

{% highlight objc %}

```
struct objc_class : objc_object {
    // Class ISA;
    Class superclass; //父类指针
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags 

    class_rw_t *data() { 
        return bits.data(); // &FAST_DATA_MASK
    }
}
```

{% endhighlight %}



#### Q: What is  `class_rw_t`  and ``class_ro_t``?

**A:** `class_rw_t` and `class_ro_to` are defined as follow:

{% highlight objc %}

```
struct class_rw_t {
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;

    Class firstSubclass;
    Class nextSiblingClass;
};

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
    uint32_t reserved;

    const uint8_t * ivarLayout;

    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;
};
```

{% endhighlight %} 

Every object has pointer to `class_ro_t` and ``class_rw_t``.  At compile time, `class_ro_t` struct is fixed. The `bits` field in `objc_class` stores the address of it，At runtime (specially after runtime's `realizeClass` method), `class_rw_t` is created and it contains the old `class_ro_t` . Then class object's `bits` pointer is set to newly created ``class_rw_t` address.

<figure class="image">
  <img src="/img/posts/post-iosinter-clsro.png" alt="clsro"/>
  <center><figcaption><b>Before realizeClass</b></figcaption></center>
  <img src="/img/posts/post-iosinter-clsrw.png" alt="clsrw"/>
  <center><figcaption><b>After realizeClass</b></figcaption></center>
</figure>

#### Q: Do you know `isa` pointer? What is it used for?

**A:** `isa` pointer of an object instance points to its **class object**. `isa` pointer of a class object points to its **meta class** object.  The class object decides the behaviour of an instance, and the meta class object decides that for class object.  The relationship is best illustrated by this diagram:

<figure class="image">
  <img src="/img/posts/post-iosinter-isa.jpg" alt="isa"/>
</figure>

##### Follow up: Give me a scenario whereby you can use this.

**A:** **Automatic encoding and decoding of object**.  

If you ever implements of serialisation, you would know that it is troublesome to implements `<NSCoding>` protocol.  Let's say you have following class and need to make it confirm to `<NSCoding>`:

{% highlight objc %}

```
@interface Movie : NSObject<NSCoding>

@property (nonatomic, copy) NSString *movieId;
@property (nonatomic, copy) NSString *movieName;
@property (nonatomic, copy) NSString *pic_url;
@end

@implementation Movie

- (void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:_movieId forKey:@"id"];
    [aCoder encodeObject:_movieName forKey:@"name"];
    [aCoder encodeObject:_pic_url forKey:@"url"];

}

- (id)initWithCoder:(NSCoder *)aDecoder
{
    if (self = [super init]) {
        self.movieId = [aDecoder decodeObjectForKey:@"id"];
        self.movieName = [aDecoder decodeObjectForKey:@"name"];
        self.pic_url = [aDecoder decodeObjectForKey:@"url"];
    }
    return self;
}
@end
```

{% endhighlight %}

In the above case, we only have 3 properties, what if we have 100 ones? In this case we can apply our runtime knowledge to make it simple:

{% highlight objc %}

```
#import "Movie.h"
#import <objc/runtime.h>
@implementation Movie

- (void)encodeWithCoder:(NSCoder *)encoder

{
    unsigned int count = 0;
    Ivar *ivars = class_copyIvarList([Movie class], &count);

    for (int i = 0; i<count; i++) {
        Ivar ivar = ivars[i];
        const char *name = ivar_getName(ivar);
        NSString *key = [NSString stringWithUTF8String:name];
        id value = [self valueForKey:key];
        [encoder encodeObject:value forKey:key];
    }
    free(ivars);
}

- (id)initWithCoder:(NSCoder *)decoder
{
    if (self = [super init]) {
        unsigned int count = 0;
        Ivar *ivars = class_copyIvarList([Movie class], &count);
        for (int i = 0; i<count; i++) {
        	Ivar ivar = ivars[i];
        	const char *name = ivar_getName(ivar);
        	NSString *key = [NSString stringWithUTF8String:name];
        	id value = [decoder decodeObjectForKey:key];
       	 [self setValue:value forKey:key];
        }
        free(ivars);
    }
    return self;
}
@end
```

{% endhighlight %}

In the above code, we use runtime API to get all ivar and their name and use their names to encode the properties automatically.



#### Q: Can we add instance variable at runtime? 

**A:**  1. Can not add iVar to compiled class  2. Can add iVar to created object at runtime

The reason that we can not add iVar at runtime is that `objc_ivar_list` and `instance_size` are fixed at compile time.  you can refer to the `class_rw_o` for this.



##### Follow up: What can we do to add iVar at runtime for a compiled object?

**A:** We can use runtime API `objc_setAssociatedObject` and `objc_getAssociatedObject` to add associated object in a class's category.

##### Follow up: When do we need to dealloc associated objects?

**A:** Associated objects are deallocated later than the class itself. They are deallocated in the `NSObject`'s dealloc method. Here is the order of deallocation:

1. Current class calls `-dalloc` , reference count set to 0.
2. Super class calls `-dealloc` 
3. `NSObject` calls `-dealloc`:  `NSObject`'s `object_dispose()` is called
4. Within `object_dispose()`,  C++ iVar's destructors are called.  Instances' `-release` are called. All runtime Associated objects are disassociated.



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

**A:**  Both lines are `Son`. 

`super` is not a pointer to super class, but will be translated to `objc_msgSendSuper`. The receiver is still the current object.  `objc_msgSendSuper` is defined as follow:

{% highlight objc %}

```
id objc_msgSendSuper(struct objc_super *super, SEL op, ...)

 struct objc_super {
 __unsafe_unretained id receiver;
 __unsafe_unretained Class super_class;
 };
```

{% endhighlight %}

So when we call `[super class] ` , what actually happen is that the runtime packs the current class and super_class in the `objc_super` struct, and look up the function in super class. After found the implementation, it will call `objc_msgSend(objc_super->receiver, @selector(class))`, which is the same as calling `[self class]`. Hence the output are the same.



#### Q: What is Method Swizzling? 

**A:** Exchanging a method implementation with another method.  Specically, because method's implementation `IMP` is looked up via its selector `SEL` , we can exchange two methods Implementation by switching mapping of their `SEL`.  This is usually done by `method_exchangeImplementations` in class `+load`. 

{% highlight objc %}

```
+ (void)load {
   
    Method oriMethodImp = class_getClassMethod(self, @selector(oriMethod:));
    Method swiMethodImp = class_getClassMethod(self, @selector(swiMethod:));   
    method_exchangeImplementations(oriMethodImp, swiMethodImp);
}

```

{% endhighlight %}

##### Follow up: Give me a usecase of Method Swizzling.

**A:** We can use method swizzling to add hook to some method. For example, if we want to monitor how many times a method is called, we can swizzle a new method, in which we notes down invocation and call the original method implementation.



#### Q: Explain `+load` and `+initialize` ?

**A:**  I belive [this stackoverflow post]((https://stackoverflow.com/questions/13326435/nsobject-load-and-initialize-what-do-they-do)) explain this perfectly.

> The runtime sends the `load` message to each class object, very soon after the class object is loaded in the process's address space. For classes that are part of the program's executable file, the runtime sends the `load` message very early in the process's lifetime. For classes that are in a shared (dynamically-loaded) library, the runtime sends the load message just after the shared library is loaded into the process's address space.

> The runtime calls the `initialize` method on a class object just before sending the first message (other than `load` or `initialize`) to the class object or any instances of the class. This message is sent using the normal mechanism, so if your class doesn't implement `initialize`, but inherits from a class that does, then your class will use its superclass's `initialize`. The runtime will send the `initialize` to all of a class's superclasses first (if the superclasses haven't already been sent `initialize`).



#### References

- [https://juejin.cn/post/6844904079957688328](https://juejin.cn/post/6844904079957688328)

- [https://juejin.cn/post/6844904090128891912](https://juejin.cn/post/6844904090128891912)

- [https://juejin.cn/post/6844904090129039374](https://juejin.cn/post/6844904090129039374)

- [https://www.jianshu.com/p/19f280afcb24](https://www.jianshu.com/p/19f280afcb24)

- [https://www.jianshu.com/p/56e40ea56813](https://www.jianshu.com/p/56e40ea56813)

- [https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Runtime.html](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Runtime.html)

  