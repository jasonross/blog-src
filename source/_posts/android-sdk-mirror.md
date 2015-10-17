title: Android SDK镜像服务器搭建
date: 2015-10-11 23:51:19
tags: [Android,Android SDk,镜像,nginx]
---

前段时间XcodeGhost闹的沸沸扬扬，意外发现迅雷下载真的是可以被污染的，详情见[对迅雷下载进行投毒的简单尝试](http://gold.xitu.io/entry/56023eabddb263b560c8d254)，也提醒了开发者不要随意从非官方渠道下载开发工具。

我们都知道Android SDK由于国内网络环境的原因，下载速度很慢，有很多[国内搭建的镜像](http://www.androiddevtools.cn/)，但镜像服务器下载带宽肯定有一定限制，并且内网一般也会限制下载速度。所以在内网搭建一个Android SDK镜像服务器很有必要，一方面可以保证速度，几兆每秒肯定不成问题，另一方面大家从内网下载，也可以保证安全性。貌似有的团队用FTP存储SDK，还是太Low了，用起来肯定不如镜像方便。

搭建Android SDK镜像服务器，有两个现成的开源解决方案，[苏州GDG](https://github.com/renfeng/android-repository)和[中科院开源镜像小组](https://github.com/opencas/mirrors)，苏州GDG牛皮吹的很响，貌似还在某个会议上发表过演讲，但估计是个人作品，东西不够完善，更新缓慢。中科院开源镜像小组开源的方案非常不错，并且也对外开放了镜像服务，一直运行良好，Github上还有定时同步和Nginx的配置信息。

花了一个下午的时间，搞定了内网Android SDK镜像服务器，下载速度平均5M/s，效果杠杠的。

## 下载SDK

直接运行[android.py脚本](https://github.com/opencas/mirrors/blob/master/script/bin/android.py)，由于SDK下载需要翻墙，所以你可能需要代理。命令行程序默认不走你的全局代理，我在家用sock5代理，在mac下用的是[proxychains4](http://www.dreamxu.com/proxychains-ng/)，公司内网有Http代理服务器，想在命令行中使用，需要设置`http_proxy`和`https_proxy`环境变量，临时使用直接在终端中输入：

```
export http_proxy=http://yourproxyaddress:proxyport
export https_proxy=http://yourproxyaddress:proxyport
```

细看脚本，你会发现脚本会先去下载`repository-*.xml`和`addons_list-*.xml`，xml里面是一些文件的具体下载地址，包含sdk、build_tools、support包等等，然后去下载具体的文件。

## 服务器配置

因为最终搭建的镜像要作为代理服务器，在SDK Manager中配置，所以需要你配置ngnix的虚拟主机。在`/etc/nginx/conf.d/default.conf`文件中，修改`server_name`字段：

```
server_name dl.google.com
            dl-ssl.google.com;
```
因为当SDK Manger配置你的镜像作为代理后，首先会拉取xml配置文件，比如`https://dl.google.com/android/repository/repository-11.xml`，最终会找到镜像服务器的`android/repository/repository-11.xml`文件，下载其他SDK文件也是同样，所以需要你把之前运行脚本下载的文件放到正确的目录中。

## 使用
别人要使用你的镜像，需要配置SDK Manager，直接去看[androiddevtools](http://www.androiddevtools.cn/)的介绍吧。