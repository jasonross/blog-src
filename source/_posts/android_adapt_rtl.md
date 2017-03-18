title: Android App 适配RTL
date: 2016-10-08 23:51:19
tags: [Android,RTL]
---

我们的APP打算进入阿拉伯市场，但阿拉伯语不同于其他语言，布局和文字方向都是从右向左，也就是本文要讲的RTL语言，right-to-left。

下面对RTL的适配进行一下总结。

### AS一键适配

AS支持一键适配RTL，主要是更新Manifest文件和在Layout中添加Start和End属性（你是不是从来没有用过paddingStart、marginStart?)

Start属性在LTR中对应Left，在RTL中对应Right，在API 17开始支持，为了兼容低版本，需要同时有Left和Start. 

![](http://mmbiz.qpic.cn/mmbiz_png/Nb5MFRT1ibkcJkVwkIibfWXEJK8RTMGzpu2N4c5oric3jibmX0cuumQaayl6CDq2IE2ibXx5JTSSaCYLw3cESuakoicw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 图标翻转

有些图标（比如箭头）本身是有方向的，在RTL下也需要进行翻转。可以使用autoMirrored特性，API 19开始支持。

### 动画翻转

可以新建anim-ldrtl文件夹，将对应的动画进行反向处理。

### include/style不支持

AS的一键化工具不支持include和style标签，需要自己手动添加Start/End。

### TextView wrap_content

大多数情况下，TextView的宽度使用wrap_content而不是match_parent, 在RTL显示英文会更正常。

### paddingleft和paddingStart

如果同时设置了paddingLeft和paddingStart，而没有设置paddingRight，在RTL下，会同时具备paddingLeft和paddingRight属性，显示会出现问题。可以考虑使用margin替换padding，同时设置marginLeft和marginStart, 在RTL下只会有marginLeft属性。

### ViewPager适配

android官方控件大多支持RTL，ViewPager除外，可以考虑将list翻转处理。

### 自定义view适配

对于自定义View，也要单独针对RTL进行适配。判断RTL代码如下： 
![](http://mmbiz.qpic.cn/mmbiz_png/Nb5MFRT1ibkcJkVwkIibfWXEJK8RTMGzpu09MWLyYibxCX8tfs2HoibBZwHIat7zOJHGN3xP2iatKRkWiavd32rE3qTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### RTL预览

AS在xml文件的Design标签下支持RTL预览，在日常开发中可以使用。