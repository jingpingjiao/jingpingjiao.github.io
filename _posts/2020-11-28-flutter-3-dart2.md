---
layout: post
title: "Flutter从零单排(3):Dart异步编程"
subtitle: "Learn Flutter(3): Dart Async Programming"
date: 2020-11-28
author: "JJP"
header-img: "img/posts/post-bg-mobile.jpg"
header-mask: 0.3
tags: [Flutter, Study]
---

往期回顾:

- [Flutter从零单排(1):环境搭建和第一个Flutter App]({% post_url 2020-11-24-flutter-1-env %})
- [Flutter从零单排(2):Dart语法]({% post_url 2020-11-26-flutter-2-dart1 %})

欢迎来到**Flutter从零单排**系列的第三期. 上一期里我们学习了Dart的基本语法, 这期稍微进阶一下, 学习Dart的异步编程. 

## Dart Async Programming

相信大家对同步异步(Sync/Aynsc)操作都有所了解, 特别是熟悉iOS GCD的朋友应该比较熟悉这个.

- 同步操作synchronous operation: 同步操作阻塞其它操作直到结束
- 同步方法synchronous function:同步函数只调用同步操作
- 异步操作asynchronous operation: 异步操作的同时可以执行其它操作
- 异步方法asynchronous function: 异步函数至少调用一个异步操作

### Await, Async, Future

那么在Dart里, 异步编程通过await, async, Future几个关键的东西实现

- **`async`** , 加在异步方法的声明里, 表示当前方法是异步的. 注意: 如果一个方法只是返回Future, 并不用标识成async, 比如下面的例子. 
- **`await`**,  加在调用异步方法前, 表示调用的是异步方法. 然后第一个await前所有异步方法都会同时进行.

- **`Future<T>`**.   Future表示是对一个异步操作结果的抽象, 可以想成异步函数给你的一个"保证", 保证之后某一时间你可以通过Future object拿到异步执行的结果. Future有两个状态: uncomplete和complete. 

  - Uncomplete: 调用异步方法后状态
  - Complete: 异步方法执行完后, 要不完成成功返回结果, 要不完成失败返回错误

  它的抽象类型`<T>`就是成功完成后返回值的类型.

例子:

{% highlight dart %}

```dart
Future<String> createOrderMessage() async {
  var order = await fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() {
  return Future.delayed(Duration(seconds: 4), () => 'Large Latte');
}

Future<void> main() async {
  print('Fetching user order...');
  print(await createOrderMessage());
}

//output
/*
Fetching user order...
Your order is: Large Latte
*/
```

{% endhighlight %}

大概就是这样. 不熟的同学可以试一试[官网上的小练习](https://dart.dev/codelabs/async-await#exercise-practice-using-async-and-await)

### Async Stream

Dart除了Future这种一次性完成的异步编程, 还有Stream来应对一连串的异步时间---流数据.

- 用`await for`来接收流数据. 挺直观的, await对应异步, for对应流
- 用`yield`生成流(当然有其他生成流的方式)

{% highlight dart %}

```dart
import 'dart:async';

Future<int> sumStream(Stream<int> stream) async {
  var sum = 0;
  await for (var value in stream) {
    sum += value;
  }
  return sum;
}

Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    yield i;
  }
}

main() async {
  var stream = countStream(10);
  var sum = await sumStream(stream);
  print(sum); // 55
}
```

{% endhighlight %}

当然Stream的应用远不止于此. Dart也支持Functional Programming的信息流处理, 比如then, map, reduce, bind. 我准备用到的时候再细看.

### Async API in Flutter

我们来看看Async API在下面这个Flutter应用的例子. 假如说调用network service的`fetchNetworkData`API返回用户数据, 然后展示给用户. 你可以用FutureBuilder调用`fetchNetworkData`, 然后给他一个builder function. 这个builder function会在fetchNetworkData后调用, 并且重绘UI.

{% highlight dart %}

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Use a FutureBuilder.
    return FutureBuilder<String>(
      future: _fetchNetworkData(5),
      builder: (context, snapshot) {
        if (snapshot.hasError) {
          // Future completed with an error.
          return Text(
            'There was an error',
          );
        } else if (snapshot.hasData) {
          // Future completed with a value.
          return Text(
            json.decode(snapshot.data)['field'],
          );
        }
        throw UnimplementedError("Case not handled yet");
      },
    );	
  }
}
```

{% endhighlight %}

## Isolates

我们对Dart的语法和编程模式已经有了基本的了解, 接下来我们来了解一些Dart原理的东西. **Isolate**. 建议大家读一下[这篇博客](https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a). 图文并茂, 讲的很好, 能给我们一个大概的了解. 在这里我大概提炼一下博客的内容.

我们可以把Isolate想象成每个thread的独有空间, 有自己独有的内存空间和eventloop(见下段).  这样的好处很明显, 代码在不同的Isolate里不用考虑locking, 因为一个Isolate只有一个线程, 独享了内存空间. 比如在iOS开发里, main thread主要拿来做UI展示, 我们一般会把需要大量计算的任务通过GCD或者其他方法新建一个dispatch_queue, 这样不会影响UI thread的性能. 

​										下图就是两个Isolate, 两个thread互不干扰 ([图片出处](https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a)).

![Image for post](https://miro.medium.com/max/1260/1*MnJQG2bdbcqmEwu1lSL52Q.png)

那EventLoop是什么呢? 顾名思义, Eventloop就是一个Event Processing的loop. 你可以想象一个永远在运行的whileloop, 平时沉睡, 有事件(timer, touch, I/O, etc.)的时候就被唤醒, 处理事件. 基本等同于iOS开发里的RunLoop.

​																			  ([图片出处](https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a))

![Image for post](https://miro.medium.com/max/1248/1*t-mHg6JrdkKCs65UvD70-Q.png)



今天我们简单的学习了下Dart的异步编程以及异步编程背后的Isolate和Eventloop. 对Dart的粗浅学习到这儿, 之后在编写Flutter应用时继续边用边学. 



*References*:

- [Asynchronous programming: futures, async, await](https://dart.dev/codelabs/async-await)
- [Dart asynchronous programming: Isolates and event loops](https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a)
- [Dart asynchronous programming: Futures](https://medium.com/dartlang/dart-asynchronous-programming-futures-96937f831137)