---
layout: post
title: "Flutter从零单排(1):环境搭建和第一个Flutter App"
subtitle: "Learn Flutter(1): Environment Setup and first Flutter App"
date: 2020-11-25
author: "JJP"
header-img: "img/post-bg-default.jpg"
tags: [Flutter, Study]
---

大家好, 欢迎来到**Flutter从零单排**系列的第一期. 

熟悉Dota圈的朋友应该发现了, 这个名字是受了大酒神的视频启发. 当时他的系列视频爆红网络, 模拟路人小白单排上分,代入感极强. 我就取了类似的名字. 一是自己也是从头开始学习, 二是希望大家能够一起踩坑,一起进步. 

这一系列文章记录性比较强, 多是记录我的学习过程, 还有学习过程中的一些小想法. 所以不会特别Technical, 高分大神可以跳着看.

## Why Flutter? 为啥要学Flutter

学习都是有成本的, 在开始学习任何东西之前, 都要想想"为什么"这个哲学问题. 

对于我来说, 这个问题不难回答. 

1. 我是iOS开发, Flutter是新兴的客服端开发技术框架, 有很多新的程序设计理念.
2. 想学习开发Android. 虽然重新学习Android开发问题不大, 但学习Flutter貌似一举两得.
3. Flutter在大厂受到重视, 大规模应用已有范例 (闲鱼App). 对找工作有帮助.

综上, 学习Flutter对我来说是个受益大的事情. 所以这段时间提上日程. 屏幕前的你也可以自己斟酌下. 

当然还有同学会问:问啥不学ReactNative呢? 哈哈, 这是另一个技术选型的问题了. 网上有很多分析的文章, 对比两个框架. 比如[这里](https://nevercode.io/blog/flutter-vs-react-native-a-developers-perspective/)和[这里](https://www.zhihu.com/question/384934444).  引用下别人的回答

> 在自己可以选择的条件下：react-native 更适合前端领域的开发者，flutter 更适合原生领域的开发者，同时还和你想面试的岗位和产品有关系



## Env Set-up 环境搭建

好啦, 分析完学习的动机, 让我们开始撸起袖子, 做我们今天要干的第一件事: 开发环境搭建.  那就参考[官网](https://flutter.dev/docs/get-started/install)一步步来吧

- 安装下载Flutter
- 然后安装IDE及插件 
- 跑第一个HelloWorld程序

结果, 以为很简单半小时能搞定的流程, 果然踩了很多坑.

- 坑1: [Unable to access Android SDK add-on list](: https://stackoverflow.com/questions/28918069/unable-to-access-android-sdk-add-on-list/53615144 )
  - 问题: 第一次启动Android Studio跳出警告框:Unable to access Android SDK add-on list.
  - 原因: 被墙
  - 解决: For MAC OSX, right click androidstudio->Show Package->Contents->bin->idea.properties, then add "disable.android.first.run=false" the line in this answer.
- 坑2: [flutter pub get failed](https://stackoverflow.com/questions/49056332/flutter-pub-get-failed)
  - 问题: 更新flutter package结束不了
  - 原因: 被墙
  - 解决: 按照这个[指导](https://flutter.dev/community/china#configuring-flutter-to-use-a-mirror-site)设置国内镜像地址

- 坑3: `flutter run` 失败
  - 问题: 无法成功运行Demo, `flutter run -v` 可以看到信息
  - 原因: 被墙
  - 解决: 手动下载[gradle-5.6.2](https://services.gradle.org/distributions/gradle-5.6.2-all.zip)放到对应的folder里

总结起来, 都是因为网络环境(被墙)的问题, 尴尬🤦. 

忙忙碌碌一个小时后, 看到simulator跑起来的HelloWorld, 终于露出了欣慰的笑容

## First Flutter APP 第一个Flutter App

接着, 跟着官方的 **Write your first Flutter app, part 1,2**, 完成一个Infinitely Scrolling List App. 应该没有什么可以卡壳地方.

这是我们第一次接触Dart语法和Flutter的Widget. 个人感觉, Widget来做layout的语法和跟新的SwiftUI差不多, 各种嵌套的UI构造函数, 都是descriptive UI programming. Widget的作用个我的感觉有点像iOS开发里的UIViewController, Widget的State有点像MVVM结构里的ViewModel. 还挺趣的, 期待后面的进一步了解.

<img src="/img/posts/post-flutter-helloworld.jpg" alt="drawing" width="250"/>

好了, 这就是Flutter从零单排的第一期的内容了. 这一期我们简单的安装了环境, 跑通了一个HelloWorld. 下一期我们要一起学习Flutter的语言Dart, 同时把它和Objective-C和Swift做做对比.

