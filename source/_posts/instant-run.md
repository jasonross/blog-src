title: Instant Run 浅析
date: 2015-11-25 02:50:34
tags: [instant run,Instant Run,Android Studio,Gradle,hotfix,热补丁]
---

AS2.0 Preview 版本发布了，早上醒来就被各种刷屏，有人在担心更新，有人在拍手称快，有人在厕所哭晕。而我当然没有那么肤浅，暗想要去研究一下AS2.0最重要特性Instant Run的原理。Instant Run实际上就是一个热补丁，[@别闹腾啊](http://weibo.com/u/1851118441)在搞LayoutCast，我在搞Nuwa，对Instant Run热补丁的思路都非常感兴趣，所以就坐在一起看了两三个小时代码，对代码修复的这一部分基本上搞清楚了。

## Instant Run

google官方关于Instant Run特性的介绍在这里：[https://sites.google.com/a/android.com/tools/tech-docs/instant-run](https://sites.google.com/a/android.com/tools/tech-docs/instant-run)，按其所述，运行**FloatingActionButtonBasic**样例，实验Instant Run，豪爽豪爽。

但是，修改方法竟然不用重启Activity，这是怎么做到的？

阅读一帆关于几个热补丁思路的[介绍](http://blog.zhaiyifan.cn/2015/11/20/HotPatchCompare/)，你会发现andfix和dexposed采用jni hook方法，不用重启就能修复，而Nuwa的ClassLoader思路因为类被虚拟机加载后，不会重新加载，所以需要重启。Instant Run是怎么实现不重启加载的呢，难道也是jni hook？

## 实现原理

和nuwa类似，一个插件一个库，Instant Run用的是gradle plugin 2.0.0-alpha1和instant-run.jar。

### gradle plugin 2.0.0-alpha1

gradle plugin 2.0.0-alpha1主要有两个作用：

* 第一次运行，应用[transform API](http://tools.android.com/tech-docs/new-build-system/transform-api)修改字节码。
	
	输出目录为*Application/build/intermediates/transforms/instantRun/debug/folders/1*。
	
	* 给所有的类添加**$change**字段
	
		**$change**为**IncrementalChange**类型，**IncrementalChange**是个接口，该接口后面会讲。
	
	* 修改类的全部方法
	
		新的逻辑是：如果**$change**不为空，去调用**$change**的**access$dispatch**方法，参数为方法签名字符串和方法参数数组，否则调用原逻辑。
	
*  后续运行，dx补丁类，生成补丁dex。

	输出目录为*Application/build/intermediates/transforms/instantRun/debug/folders/4000*。

	* 被修改类对应的补丁类
	
		补丁类，并不是你修改后的类，而是由gradle plugin自动生成，实现了**IncrementalChange**接口的类。
		
		该类类名在原名后面添加**$override**，复制修改后类的大部分方法，实现**IncrementalChange** 接口的**access$dispatch**方法，该方法会根据传递过来的方法签名，调用本类的同名方法。
		
		只要把原类的**$change**字段设置为该类，那就会调用该类的**access$dispatch**方法，就会使用修改后的方法了。
	
	* 被修改类的记录类
	
		**AppPatchesLoaderImpl**记录了所有被修改的类，也会被打进补丁dex。
	
### instant-run.jar
**instant-run.jar**的路径为*Application/build/intermediates/incremental-runtime-classes/debug/instant-run.jar*。

**instant-run.jar**是gradle plugin帮我们自动打到dex中去的，省去了compile dependency这一步。它的主要作用有两个：


	
* 设置原类的**$change**字段为补丁类
	
	需要对被修改的类设置**$change**字段，那怎么知道哪些类被修改了？
	
	**AppPatchesLoaderImpl**类不但记录了全部被修改的类，还提供**load**方法支持设置被修改原类**$change**字段，当收到补丁通知时，只需新建一个DexClassLoader，去反射加载补丁dex中的**AppPatchesLoaderImpl**类，调用load方法即可，load方法中会去加载全部补丁类，并赋值给对应原类的**$change**。
	
* 重启加载补丁类

	重启后怎么办？原来的补丁文件需要加载进来。
	
	**IncrementalClassLoader**会在Application中去加载该应用cache目录中的补丁dex，把它设置为默认**PathClassLoader**的parent，由于ClassLoader采用双亲委托模型，会先去parent查找类，所以就可以加载补丁类了。
	


## 总结

google的介绍中，也提到了Instant Run的一些局限之处，比如变量、添加删除方法等，当然这一切都会持续优化，gradle plugin 2.0也还仅仅是个alpha版本而已。

Instant Run的实现思路蛮笨的，几乎所有的方法都修改逻辑，但不用重启这一点很赞，用这个思路可以搞一个热修复库了，有官方背景的库。

本文主要是探究Instant Run处理代码变化的思路，对资源变化没有分析，对于一些细节比如server实现、socket传输、application替换、updateMode处理亦未提及。

