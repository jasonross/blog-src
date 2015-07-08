---
title: 关于Android的Data Binding技术
date: 2015-05-31 20:13:13
tags: [Android,Data Binding,Google IO,MVVM]
---
此次Google IO大会，给Andorid开发者带来了很多福利。我对其中的Data Binding技术最感兴趣，所以花时间研究了一下。

## Data Binding与MVVM
Data Binding即数据绑定，在很多前端框架中都有应用，比如Google维护的AngularJS就支持View和Model的双向绑定。当数据更新，框架收到通知，视图可以自动更新，View和Model可以做到松耦合。

有了Data Binding库的支持，在Android中可以很方便的实现MVVM的开发模式。MVVM，是Windows社区的一种叫法，即Model-View-ViewModel。在Android中，Activity，Fragment，Dialog之类都属于View层；Model层主要负责数据相关部分，可以从数据库或者网络获取；ViewModel主要是把Model转换为View可以呈现的东西，例如把一个Date类型转化为格式化后的String。

ViewModel在哪里呢？在Data Binding插件的支持下，会实时生成以Layout名称命名的Binding类，比如Layout名称为activity_main，则会生成ActivityMainBinding类，数据绑定在Layout文件中XML标注，最终在ActivityMainBinding中转化为Java代码，所以ActivityMainBinding类扮演了ViewModel的角色。

在我看来，MVVM相对于传统的MVC，把一些业务逻辑抽离，放到ViewModel中，可以减轻Activity和Fragment的压力，同时数据绑定也大大减少了逻辑代码，另一方面对于测试来说，数据变化引起视图自动更新，可以更加方便测试。但在XML中进行数据绑定的操作，可能不利于调试。

在Google官方发布Data Binding库之前，已经有类似的开源项目，如[RoboBinding](http://robobinding.github.io/RoboBinding/getting_started.zh.html)，支持双向绑定等，比较成熟，Google的库目前只支持单向绑定，为Beta版本，不知道后面是不是还有很多特性改进。你会发现，由于Android的开源特性，开发者在很多方面走在了Google的前面，比如这次发布的权限控制，国内很多ROM早就有了这个功能。我在想，当Google发布新版本的时候，会不会去看看MIUI有什么特性优化呢。

##参考资料
[Data Binding Guide](https://developer.android.com/tools/data-binding/guide.html) （官方资料）

[Web开发的MVVM模式](http://www.cnblogs.com/dxy1982/p/3793895.html)（MVC VS. MVP VS. MVVM）

[MVVM介绍](http://objccn.io/issue-13-1/)（iOS中MVVM的一种实现，对概念的理解有帮助）

[MVVM on Android: What You Need to Know](http://www.willowtreeapps.com/blog/mvvm-on-android-what-you-need-to-know/) （这个博客很不错）

[Animating Android Binding Transitions](http://www.willowtreeapps.com/blog/animating-android-binding-transitions/)（Data Binding技巧）
##Sample
官方貌似没有Data Binding的Sample，我写了个简单的Demo，Github地址：
https://developer.android.com/tools/data-binding/guide.html