---
layout: post
title: "Flutter从零单排(2):Dart语法"
subtitle: "Learn Flutter(2): Dart basics"
date: 2020-11-26
author: "JJP"
header-img: "img/posts/post-bg-mobile.jpg"
header-mask: 0.3
tags: [Flutter, Study]
---

往期回顾:

- [Flutter从零单排(1):环境搭建和第一个Flutter App]({% post_url 2020-11-24-flutter-1-env %})

大家好, 欢迎来到**Flutter从零单排**系列的第二期. 这期里我们来一起学习Dart. 

我把Dart的学习分成了两部分, 在今天的第一部分我们看看基本语法, 第二篇学习Dart的高阶内容, 包含异步编程模型, Isolate和EventLoop. 

## Dart Overview 

Dart是Google开发的一门语言, 最开始想拿来代替JavaSc, 结果却因为Flutter的原因才获得了很多关注. 不管怎样, 作为一门新生的面向对象编程语言, 它有很多特点:

- **Type Safe**: 在Dart里, 虽然类型不是强制需要标识出来, 但是Dart是statically typed, 所以很多问题在编译时候暴露出来, Dart Runtime也会帮助我们进行type checking

- **Highly Portable**: Dart可以编译成MachineCode, 也可以转译成JavaScript, 也可以在Dart VM的帮助下, 编程类似python的interpreted language. 这也是Dart支持hot-reload的原因. 

- **Single Thread**: Dart是单线程语言. 通过EventLoop处理多任务. 虽然是单线程语言, Dart支持async/futures. 这个我们下期会单说.

- **Memeory Managed by Gabage Collection**: 不用手动管理内存, 或者像Reference Counting那样需要注意循环引用

- **Visibility by convention**: Dart没有Java那种private, public的标识符. By convention, private的method的名字前加"_"

  

## Dart 基础语法 Basics

因为篇幅有限, 这里我会把Dart基本语法里比较有趣的地方提出来. 其它的大家参考官网的[Language Tour](https://dart.dev/guides/language/language-tour). 

{% highlight dart %}

```dart
/*
====================
Variable and modifier
====================
*/

// every variable is an object, even 'null'
var a = null;

// Uninitialized variables have an initial value of null
int lineCount;
assert(lineCount == null)
  
//const and final.
/* 
"final" means single-assignment to modify variable. 给你一次赋值的机会, 别改了
"const" means a compile-time computed value. 
*/
const hour = 3600
const min = 60
final name = 'Bob'
const names = ['Bob', 'Alice']
another_names = const ['Bob']
x name = 'Alice' //wrong
x names = ['Bob'] //wrong
another_names = ['Alice'] //OK
  
  
// bool checking
// MUST explicit check bool, can not do if (aObject)
/*
 Dart不允许null在所有地方里被silently ignored. (包括if statement)
 objc,python和一些其他的语言允许bool exp里面的非bool类型经常会有意想不到的bug.
 比如python里, 经常可以 if list: 来判断list是否为空
*/
var fullName = null;
if (fullName) { //wrong
  
}

// list spread operator ... and null-aware ...?
/*
 这个地方也是, Dart不允许null在所有地方里被silently ignored. 
*/
var list = null;
var list2 = [0, ...?list];
assert(list2.length == 1);

var list3 = [0, ...list]; //wrong. 

// 但list包含null是可以的
var things = [2, null, 3];
var more = [1, ...things, 4]; // [1, 2, null, 3, 4].
var also = [1, ...?things, 4]; // [1, 2, null, 3, 4].
// 除掉包含的null, 需要explicitly处理
var more = [1, ...things.where((thing) => thing != null), 4]; // [1, 2, 3, 4].


// collection for & collection if
/*
很像python的list comprehension: [i for i in range(10)], 当更灵活
*/
var listOfInts = [1, 2, 3];
var listOfStrings = [
  '#0',
  for (var i in listOfInts) '#$i'
];
assert(listOfStrings[1] == '#1');


// Rune and grapheme clusters
import 'package:characters/characters.dart';
...
var hi = 'Hi 🇩🇰';
print(hi);
print('The end of the string: ${hi.substring(hi.length - 1)}');
print('The last character: ${hi.characters.last}\n');

// high order function
/* 跟swift很像 */
var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
list.forEach((item) {
  print('${list.indexOf(item)}: $item');
});

// if null
String playerName(String name) => name ?? 'Guest';

//Cascade notation
/* 挺方便的, 多次build */
final addressBook = (AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();


/*
====================
Classes面向对象
====================
*/

//constant constructor
var p = const ImmutablePoint(2, 2);

/*
Dart的constructor挺奇怪的, 子类不继承父类的constructor. 默认调用父类no-arg constructor
调用方式是先父类,再子类
*/
class Person {
  String firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  //调用父类constructor
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
  
  //你还可以指定如何实现constructor
  Employee() : super.fromJson(defaultData);
  // ···
  
  //Named Constructor如同其他语言自己的class factor method
  Employee.withDefaultName() {
    firstName = 'default';
    lastName = 'default';
  }
 }

//Initializer list
/*
就是给我们机会在init前,跑一系列的expression
虽然我还不是特别明白有什么用处
*/
Point.fromJson(Map<String, double> json)
    : x = json['x'],
      y = json['y'] {
  print('In Point.fromJson(): ($x, $y)');
}


//Factory constructory
/*
提示用API的人, 这个constructor出来的instance不一定是新的, 可能有cache
*/
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to
  // the _ in front of its name.
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(
        name, () => Logger._internal(name));
  }

  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}

//Implicit interfaces
/*
Dart里, 每个class隐型定义了interface, 其它class可以直接implement, 不用继承.
*/
class Impostor implements Person {
  get _name => '';
  String greet(String who) => 'Hi $who. Do you know who I am?';
}

//noSuchMethod():
/*
Dart里, 如果定义了noSuchMethod, 会自动调用这个, 很有趣. 相当于包装起来ObjeC自己需要处理的问题
*/
class A {
  // Unless you override noSuchMethod, using a
  // non-existent member results in a NoSuchMethodError.
  @override
  void noSuchMethod(Invocation invocation) {
    print('You tried to use a non-existent member: ' +
        '${invocation.memberName}');
  }
}


/*
====================
Generic泛型
====================
*/
class Foo<T extends SomeBaseClass> {
  String toString() => "Instance of 'Foo<$T>'";
}

// generics in method
T first<T>(List<T> ts) {
  // Do some initial work or error checking, then...
  T tmp = ts[0];
  // Do some additional checking or processing...
  return tmp;
}
```

{% endhighlight %}



好了, 这就是今天的学习笔记了. 下期我们来研究Dart的异步编程和内存模型. 



*References*:

- [dart.dev](https://dart.dev/)
- [Introduction to Dart VM](https://mrale.ph/dartvm/)
- [Dart (DartLang) Introduction: Getting started with Dart/Flutter](https://medium.com/run-dart/dart-dartlang-introduction-getting-started-with-dart-flutter-c5f59b8c456b)
- [Understanding Dart](https://awaiscs.medium.com/understanding-dart-ffa4f975fddd)

