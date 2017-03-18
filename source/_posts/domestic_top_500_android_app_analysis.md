
title: 国内Top500Android应用分析报告
date: 2016-07-11 0:16:16
tags: [Android,APP,RxJava,RN]
---

## 背景

笔者目前在开发的应用在线上有不少OOM的问题，经过一番优化后，OOM降了一个量级，但由于产品中Gif是一特色，内存占用优化空间比较有限，想要减少OOM最简单的办法可能就是开启largeHeap。Google官方是不推荐使用largeheap的，更大的内存意味着更长的垃圾回收时间，所以我就想看看国内的这些应用是否有打开largeheap。用AS的APK Analyzer看了下微信和QQ空间，发现大家都打开了，然后我就安心了。

后来，我们打算把Jackson库换掉，太占用方法数了，可选择的有Gson和Fastjson。同样去看看国内的App都在用什么吧，本以为Fastjson号称速度最快，国内又是主场，应该用的人更多一些，但在手动分析了几个APP后，发现大家都在用Gson，Fastjson反而没见到几个。

再后来，有一天晚上，好像看了篇TK教主的文章就来了灵感，觉得可以批量分析一下国内主流的APP，看看大家都在用什么库，教父整天说据我所知，国内用哪些团队在用rxjava，我可以来个更全面准确的分析，顺便可以看看我的Nuwa是否有人在用？

## 分析
豌豆荚正好那几天被收购了，那就拿他开刀吧，脚本下载了排行榜（周下载量）Top500的应用，使用Android SDK自带的appt和dexdump，分析Manifest文件和dex文件，拿到一些APK基本信息和dex中的Class Descriptor。


有些有插件化能力的或者没有用MultiDex的应用，可能dex不在常规位置，比如淘宝把dex伪装成.so文件，所以需要查看文件的magic number来找到dex文件。

涉及到数据的排序和整理，所以使用了mysql数据库，Class Descriptor通过package分段聚合，package最多保留四段，比如Lcom/jiajixin/nuwa/utils/dex/,会有`com/*`、`com/jiajixin/`、`com/jiajixin/nuwa/`、`com/jiajixin/nuwa/utils/`入库,然后group by后根据count倒序输出，人工匹配对应库名。

## 数据

废话不多说，直接展示数据。

### APK基本信息

![minSdkVersion](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-0%402x.png)


![largeHeap](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-1%402x.png)

![process count](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-2%402x.png
)
![method count](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-4%402x.png
)

### 开源项目

Network:

Okttp|Volley|android-async-http
---|---|---
135|105|58

ImagecLoader:

Universal Image Loader|Fresco|Glide|Picasso
---|---|---|---
130|88|54|38

Dependency Injection:

ButterKnife|Dagger|Android Annotations|Roboguice
---|---|---|---
63|16|11|5

Json:

Gson|Fastjson|Jackson
---|---|---
227|87|26

Protocol Buffers:

Wire| Protobuf
---|---
72|67

DataBase:

GreenDao|OrmLite
---|---
41|27

Event:

EventBus|Otto
---|---
132|12

HotFix:

Andfix|Dexposed|Nuwa
---|---|---
34|24|7

Plugin:

Droidplugin|Dynamic-Load-Apk|DynamicAPK|Pluginmgr
---|---|---|---|---
8|5|7|1|1

PullToRefresh:

Android-PullToRefresh|android-Ultra-Pull-To-Refresh
---|---
92|27

Name| Users 
---|---
NineOldAndroids|150
zxing|121
okio|119
thrift|97
bolts|76
android-gif-drawable|72
spdy|63
photoview|62
iflytek|61
rxjava|50
pinyin4j|46
viewpagerindicator|46
jsr305|42
ijkplayer|41
DanmakuFlameMaster|36
retrofit|26
DragSortListView|23
exoplayer|23
MPAndroidChart|23
rebound|22
jsoup|21
leakcanary|21
android-gpuimage|20
daimajia|20
xutils|19
Android-wheel|18
roundedimageview|18
PagerSlidingTabStrip|16
SmoothProgressBar|16
stetho|16
aspectj|15
bouncycastle|14
soloader|14
tagsoup|14
vitamio|14
dom4j|13
afinal|12
cropper|12
Android-Easing|11
react-native|11
slidingmenu|11
zip4j|11
disklrucache|10
swipebacklayout|10
java_websocket|9
realm|4

这些产品在使用rxjava：
![](http://7fviov.com1.z0.glb.clouddn.com/ECharts.png)

这些产品在使用react-native：
![](http://7fviov.com1.z0.glb.clouddn.com/rn.png)

上面这两张图献给教父。

### 第三方服务

![](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-5%402x.png)
![](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-6%402x.png)
![](http://7fviov.com1.z0.glb.clouddn.com/QQ20160711-7%402x.png)


