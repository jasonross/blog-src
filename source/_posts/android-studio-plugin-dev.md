title: Android Studio插件开发
date: 2015-10-11 23:25:55
tags: [Android Studio,插件,Android,IntelliJ IDEA,Java]
---

很早就想开发个Android Studio的插件了，本来想撸个清理无用资源的，但Android项目使用Gradle构建后，Lint检测出来的内容有所变化，不是很好判断要清理的资源，并且开启shrinkResources选项可以轻松的完成，所以放弃了。正好最近公司有个需求，可以做成个插件一键搞定，花了一天半的时间就搞定了。

## 插件开发
Android Studio基于IntelliJ IDEA开发，而IntelliJ IDEA是捷克软件公司JetBrains的作品，JetBrains成立十五年，做了很多语言的IDE，硕果累累，甚至还设计了Kotlin语言。

Android Studio自身不具备开发插件的功能，由于Android Studio基于IntelliJ IDEA，所以可以为IntelliJ IDEA开发插件，同样可以用在Android Studio上。

IntelliJ IDEA插件开发的官方文档可以看[这里](http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html)，需要下载IntelliJ IDEA，在IntelliJ IDEA上开发，方便调试的话需要下载源代码。大多数插件可能添加个action，再加点图形界面就差不多了，界面就是Swing那一套，当年写了那么多Swing程序，没想到现在还用的上。IntelliJ IDEA的API很多，可能很难都去读，用到什么现查吧，大多数直接用JDK就可以了。如果有些地方不太清楚的话，可以去Github上看看一些开源的插件是怎么搞的，比着葫芦画瓢估计也够用了。

## Tips

插件开发中遇到一些琐碎的东西搞不清楚，文档也不是很全，最终基本上也都找到解决方案，列几个Tips吧：

* Toolbar图标

	plugin.xml中的action可以设置icon属性，但若把图片放在普通路径下会提示找不到，看到文档说放在固定目录下，但貌似也不起作用，最终放在`out>production>project`下才可以，要知道这个目录下都是一些编译好的class文件，直接放在这也是醉了。

* Dialog居中显示

	`setLocationRelativeTo（null）`可以使其屏幕居中，但如果IDE不全屏，显示的效果就不好看了，想使Dialog在IDE窗口居中显示，可以这样设置：

	```
	setLocationRelativeTo(WindowManager.getInstance().getFrame(actionEvent.getProject())
	```
* Android SDK
你要用Lint，那要找到Android SDK的目录，用下面这段代码：

	```
		public static Sdk findAndroidSDK() {
	        Sdk[] allJDKs = ProjectJdkTable.getInstance().getAllJdks();
	        for (Sdk sdk : allJDKs) {
	            if (sdk.getSdkType().getName().toLowerCase().contains("android")) {
	                return sdk;
	            }
	        }
	
	        return null; // no Android SDK found
	    }
	```
* 状态栏显示信息

	```
	StatusBar statusBar = WindowManager.getInstance().getStatusBar((Project) DataKeys.PROJECT.getData(actionEvent.getDataContext()));
	statusBar.setInfo("要显示的内容");
	```
* 文件即时刷新

	修改文件后，不会在IDE中实时刷新，需要你调用VirtualFile的refresh方法。
	
	```
	VirtualFile vf = LocalFileSystem.getInstance().findFileByIoFile(new File(projectbuildFilePath));
vf.refresh(true, false);
	```