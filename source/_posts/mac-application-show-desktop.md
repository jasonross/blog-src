---
title: Mac下显示桌面应用开发
date: 2015-02-21 20:13:13
tags: [mac开发,显示桌面,开机启动,状态栏,快捷键]
---

由于开始接触iOS开发，所以最近一直在Mac下工作。在Windows上，有个显示桌面的图标，对于有点洁癖的人来说，点击一下还我清爽，非常实用。在Mac下，⌘+F3快捷键虽然可以达到类似的效果，但触发后桌面边缘有阴影，且点击桌面文件后会重新显示之前隐藏的应用，不是很爽。知乎上有人问过Mac显示桌面的问题，应该有不少Mac用户需要这个功能，AppStore搜索了一下，真有相关应用，但需要付费，所以我就打算自己开发一款。

##技术要点
没有太多精力，第一个版本我只想实现两个功能：

1. 状态栏点击图标，即显示桌面，所有显示窗口最小化。
2. 开机默认自启动，无需用户配置。

###状态栏
首先要在状态栏显示图标,代码如下：
<pre>
statusItem = [[NSStatusBar systemStatusBar] statusItemWithLength:NSVariableStatusItemLength];
[statusItem setImage:[NSImage imageNamed:@"StatusIcon"]];
[statusItem setHighlightMode:YES];
[statusItem setAction:@selector(onStatusItemClicked:)];
[statusItem setTarget:self];
</pre>
**注意**：由于只显示状态栏图标，不显示窗口和菜单栏，所以需要在plist文件中设置`Application is background only`为`YES`。

###显示桌面
如何实现显示桌面呢？
其实显示桌面是有快捷键的，`CMD+ALT+H+M`，按下CMD+ALT+H就会隐藏当前窗口除外的窗口，CMD+M会隐藏当前窗口，所以同时按下CMD+ALT+H+M就会达到显示桌面的效果。

所以现在需要做的就是模拟按键，之前做Java桌面应用开发的时候，有个Robot类非常方便，查阅了一下资料，Mac开发也有类似方法：
<pre>
GEventSourceRef src = CGEventSourceCreate(kCGEventSourceStateHIDSystemState);
CGEventRef cmdDown = CGEventCreateKeyboardEvent(src, kVK_Command, true);
CGEventPost(kCGHIDEventTap, cmdDown);
</pre> 
其中`kVK_Command`参数是指虚拟按键，对应的是`0x37`,代指Command按键。`kVK_Command`在`<Carbon/Carbon.h>`中枚举定义，所以需要`#import <Carbon/Carbon.h>`。
###开机启动
Mac开发实现开机启动有多种方式，比较方便的是使用`LSSharedFileListInsertItemURL`方法，腾讯的老谭有篇[文章](http://www.tanhao.me/pieces/380.html)，参考了一下，不过其中的`LSSharedFileListItemResolve`已被废弃，需要用`LSSharedFileListItemCopyResolvedURL`代替。
##成果
###效果图
代码比较简单，也已在[Github](https://github.com/jasonross/ShowDeskTop)上开源，最终效果图如下:
![演示动画](http://jiajixin.qiniudn.com/show_desktop.gif)

###应用下载
[http://jiajixin.cn/ShowDeskTop.app.zip](http://jiajixin.cn/ShowDeskTop.app.zip)

###不足之处
时间仓促，应用有些不足之处：

1. 不会作图，所以图标不是很优雅，只能说看的过去，如果你有更好的图标，可以给我提交[Pull Request](https://github.com/jasonross/ShowDeskTop)。
2. 不支持自定义快捷键和退出功能，由于状态栏图标要保持操作简单，所以不会添加菜单选项，后面可以考虑在Dock中添加。
3. 不支持复制自身到`应用程序`文件夹中，后续会添加此功能。