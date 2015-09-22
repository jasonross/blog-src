title: 构建神器Gradle
date: 2015-08-07 16:58:59
tags: [Android Studio,Gradle,Android,Groovy]
---

**本文为CSDN约稿文章，首发地址为：[Android项目中如何用好构建神器Gradle?](http://www.csdn.net/article/2015-08-10/2825420)。如需转载，请与CSDN联系。原文错误，会在本站更新。**

最近在忙团队并行开发的事情，主要是将各个团队的代码分库，一方面可以降低耦合，为后面模块插件化做铺垫，另一方面采用二进制编译，可以加快编译速度。分库遇到了一些问题，很多都要通过Gradle脚本解决，所以稍微花时间研究了一下。

Gradle虽为构建神器，但感觉学习曲线比较陡峭。[Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)内容很多，但有点太多了，多的你看不完，[Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)一篇文章主要讲了Android相关的配置，看完可能感觉马马虎虎会用，但到了修改一些构建流程的时候还是不知所措。经过一段时间的摸索，我觉得在Android项目中用好Gradle，你要做到以下三点：

1. 了解 `Groovy` 基本语法。
2. 粗读 `Gradle User Guide` 和 `Gradle Plugin User Guide`。
3. 实战，实战，再实战。(三遍，你懂的)

涉及到的知识点和内容比较多，我不会一一讲解，本文主要会解答自己学习过程中的一些疑问，讲解一些相关概念和实战经验，过程中也会推荐一些有质量的博客文章。

## Groovy语言

Gradle基于Groovy语言，虽然接触Gradle比较久，甚至写过一点Groovy语句，但对语言本身并不了解。为什么用Groovy呢？Groovy运行在JVM上，在Java语言的基础上，借鉴了脚本语言的诸多特性，相比Java代码量更少，Groovy兼容Java，可以使用Groovy和Java混合编程，可以直接使用各种Java类库。

Groovy语法的学习，推荐官方文章[Differences with Java](http://www.groovy-lang.org/differences.html)和IBM developerWorks的[精通Groovy](http://www.ibm.com/developerworks/cn/education/java/j-groovy/j-groovy.html)。了解了基本语法，对读写gradle脚本都会有帮助，比如随便举下面几个例子：

1. 比如为何在gradle脚本中使用InputStream不用import包，而使用ZipFile需要import包？因为groovy默认import了下面的包和类，无需再import.

	```
	java.io.*
	java.lang.*
	java.math.BigDecimal
	java.math.BigInteger
	java.net.*
	java.util.*
	groovy.lang.*
	groovy.util.*
	```

2. 经常看到${var1}的用法是怎么回事？
这是Groovy中的[GString](http://blog.csdn.net/hivon/article/details/2271000)，可以在双引号中直接使用，用于字符串叠加非常方便。

	```
	def dx = tasks.findByName("dex${variant.name.capitalize()}")
	```

3. 下面的代码你真的能看懂吗？

	```
	//apply是一个方法，plugin是参数，值为'com.android.application'
	apply plugin: 'com.android.application'

	/**
	*buildscript,repositories和dependencies本身是方法名。
	*后面跟的大括号部分，都是一个闭包，作为方法的参数。
	*闭包可以简单的理解为一个代码块或方法指针。
	*/
	buildscript {
	    repositories {
	        jcenter()
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:1.2.3'
	    }
	}

	//groovy遍历的一种写法 each后面是闭包
	android.applicationVariants.each { variant ->
	}
	```

## Gradle概念
下面讲几个Gradle相关的概念，几个比较重要的吧，更多的东西还是要自己去看`Gradle User Guide`。

### 生命周期
Gradle构建系统有自己的[生命周期](https://docs.gradle.org/current/userguide/build_lifecycle.html)，初始化、配置和运行三个阶段。

1. 初始化阶段，会去读取根工程中setting.gradle中的include信息，决定有哪几个工程加入构建，创建project实例，比如下面有三个工程：
```
include ':app', ':lib1', ':lib2'
```
2. 配置阶段，会去执行所有工程的build.gradle脚本，配置project对象，一个对象由多个任务组成，此阶段也会去创建、配置task及相关信息。
3. 运行阶段，根据gradle命令传递过来的task名称，执行相关依赖任务。

### 任务创建
很多文章都会告诉你，任务创建要这样：

```
task hello {
    doLast {
        println "hello"
    }
}
```
或者用`<<`替换doLast，那我就很纳闷，定义个任务怎么这么麻烦，还要加什么doLast，我直接这样不行吗？

```
task hello {
    println "hello"
}
```
上面的这种写法，“hello” 是在gradle的配置阶段打印出来的，而前面的写法是在gradle的运行阶段打印出来的，所以怎么写要看你的需求了。

另外task中有一个action list，task运行时会顺序执行action list中的action，doLast或者doFirst后面跟的闭包就是一个action，doLast是把action插入到list的最后面，而doFirst是把action插入到list的最前面。

### 任务依赖
当我们在Android工程中执行./gradlew build的时候，会有很多任务运行，因为build任务依赖了很多任务，要先执行依赖任务才能运行当前任务。任务依赖主要使用dependsOn方法，如下所示：

```
task A << {println 'Hello from A'}
task B << {println 'Hello from B'}
task C << {println 'Hello from C'}
B.dependsOn A
C.dependsOn B
```
了解更多，可以看一下侦跃翻译的[Gradle tip #3-Task顺序](http://blog.csdn.net/lzyzsd/article/details/46935405)。

### 增量构建
你在执行gradle命令的时候，是不是经常看到有些任务后面跟着[UP-TO-DATE]，这是怎么回事？

在Gradle中，每一个task都有inputs和outputs，如果在执行一个Task时，如果它的输入和输出与前一次执行时没有发生变化，那么Gradle便会认为该Task是最新的，因此Gradle将不予执行，这就是增量构建的概念。

一个task的inputs和outputs可以是一个或多个文件，可以是文件夹，还可以是project的某个property，甚至可以是某个闭包所定义的条件。自定义task默认每次执行，但通过指定inputs和outputs，可以达到增量构建的效果。

### 依赖传递
Gradle默认支持传递性依赖，比如当前工程依赖包A，包A依赖包B，那么当前工程会自动依赖包B。同时，Gradle支持排除和关闭依赖性传递。

之前引入远程AAR，一般会这样写：

```
compile 'com.somepackage:LIBRARY_NAME:1.0.0@aar'
```

上面的写法会关闭依赖性传递，所以有时候可能就会出问题，为什么呢？本来以为@aar是指定下载的格式，但其实不然，远程仓库文件下载格式应该是由pom文件中[packaging属性](http://www.infoq.com/cn/news/2011/06/xxb-maven-9-package)决定的，@符号的真正作用是[Artifact only notation](https://docs.gradle.org/current/userguide/dependency_management.html),也就是只下载文件本身，不下载依赖，相当于变相的关闭了依赖传递，可以看一下sf的[这个问题](http://stackoverflow.com/questions/22795455/transitive-dependencies-not-resolved-for-aar-library-using-gradle)，通过添加transitive=true可以解决。但其实如果远程仓库有pom文件存在，compile后面根本不需要加"@aar"，也就不会遇到这个问题了。

## Android Gradle实战
下面讲讲在Android Gradle实战中遇到的一些问题和经验，感觉还是蛮多干货的。

### productFlavors
这个东西基本上已经烂大街了，gradle的项目一般都会使用Product Flavor，看完美团的文章，你应该就懂了。

>[美团Android自动化之旅—适配渠道包](http://tech.meituan.com/mt-apk-adaptation.html)

### buildTypes
很多App有内测版和正式版，怎么让他们同时安装在一个手机上？同时安装在一个手机上，要求packageName不同的，用productFlavors可以解决，但可能不够优雅，alpha版本还要来个debug和release版本岂不是很蛋疼？可以用buildTypes来解决，淘宝朱鸿的[文章](http://hugozhu.myalert.info/2014/08/03/50-use-gradle-to-customize-apk-build.html)有比较详细的讲解，但有些内容可能有些过时了，需要更改脚本。

### 依赖更新
项目依赖的远程包如果有更新，会有提醒或者自动更新吗？

SNAPSHOT(changing)和+号(dynamic)版本默认24小时自动更新，通过更改resolutionStrategy可以修改检查周期。

```
configurations.all {
    // check for updates every build
    resolutionStrategy.cacheDynamicVersionsFor 0, 'seconds'
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}
dependencies {
	compile 'com.dianping.nova.business:search:1.0.+'
	compile 'com.dianping.nova.business:selectdish:1.0.6-SNAPSHOT'
}
```

### 上传aar到Maven仓库
在工程的build.gradle中添加如下脚本：

```
apply plugin: 'maven'
uploadArchives {
    repositories {
        mavenDeployer {
            pom.groupId = GROUP_ID
            pom.artifactId = ARTIFACT_ID
            pom.version = VERSION
            repository(url: RELEASE_REPOSITORY_URL) {
                authentication(userName: USERNAME, password: PASSWORD)
            }
        }
    }
}
```

在build.gradle同目录下添加gradle.properties文件，配置如下：

```
GROUP_ID=dianping.android.nova.thirdparty
ARTIFACT_ID=zxing
VERSION=1.0
RELEASE_REPOSITORY_URL=http://mvn.dp.com/nova
USERNAME=hello
PASSWORD=hello
```

gradle.properties的属性会被build.gradle读取用来上传aar，最后执行`./gradlew :Zxing:uploadArchives`即可。

更多配置，可参考[建立企业内部maven服务器并使用Android Studio发布公共项目](http://blog.csdn.net/qinxiandiqi/article/details/44458707)。

### 取消任务
项目构建过程中那么多任务，有些test相关的任务可能根本不需要，可以直接关掉，在build.gradle中加入如下脚本：

```
tasks.whenTaskAdded { task ->
    if (task.name.contains('AndroidTest')) {
        task.enabled = false
    }
}
```

tasks会获取当前project中所有的task，enabled属性控制任务开关，whenTaskAdded后面的闭包会在gradle配置阶段完成。

### 加入任务
任务可以取消了，但还不尽兴啊，想加入任务怎么搞？前面讲了dependsOn的方法，那就拿过来用啊，但是原有任务的依赖关系你又不是很清楚，甚至任务名称都不知道，怎么搞？

比如我想在执行dex打包之前，加入一个hello任务，可以这么写：

```
afterEvaluate {
    android.applicationVariants.each { variant ->
        def dx = tasks.findByName("dex${variant.name.capitalize()}")
        def hello = "hello${variant.name.capitalize()}"
        task(hello) << {
			println "hello"
        }
        tasks.findByName(hello).dependsOn dx.taskDependencies.getDependencies(dx)
        dx.dependsOn tasks.findByName(hello)
    }
}
```

afterEvaluate是什么鸟？你可以理解为在配置阶段要结束，项目评估完会走到这一步。

variant呢？variant = productFlavors+ buildTypes，所以dex打包的任务可能就是dexCommonDebug。

你怎么知道dex任务的具体名称？Android Studio中的Gradle Console在执行gradle任务的时候会有输出，可以仔细观察一下。

hello任务定义的这么复杂干啥？我直接就叫hello不行吗?不行，each就是遍历variants，如果每个都叫hello，多个variant都一样，岂不是傻傻分不清楚，加上variant的name做后缀，才有任务的区分。

关键来了，dx.taskDependencies.getDependencies(dx)会获取dx任务的所有依赖，让hello任务依赖dx任务的所有依赖，再让dx任务依赖hello任务，这样就可以加入某个任务到构建流程了，是不是感觉非常灵活。

我突然想到，用doFirst的方式加入一个action到dx任务中，应该也可以达到上面效果。

### gradle加速
gradle加速可以看看这位朋友写的[加速Android Studio/Gradle构建](http://blog.isming.me/2015/03/18/android-build-speed-up/)，我就不多嘴了。并行编译，常驻内存，还有离线模式这些思路对gradle的加速感觉还是比较有限。

想要更快，可以尝试下Facebook出品的[Buck](https://buckbuild.com/)，可以看一下Vine团队适配Buck的[技术文章](http://engineering.vine.co/post/117873038742/reducing-build-times-by-adopting-buck)，我们的架构师也有适配Buck，加速效果在10倍左右，但有两个缺点，不支持Windows系统，不支持远程依赖。


### 任务监听
你想知道每个执行任务的运行时间吗？你想知道每个执行任务都是干嘛的吗？把下面这段脚本加入build.gradle中即可：

```
class TimingsListener implements TaskExecutionListener, BuildListener {
    private Clock clock
    private timings = []

    @Override
    void beforeExecute(Task task) {
        clock = new org.gradle.util.Clock()
    }

    @Override
    void afterExecute(Task task, TaskState taskState) {
        def ms = clock.timeInMs
        timings.add([ms, task.path])
        task.project.logger.warn "${task.path} took ${ms}ms"
    }

    @Override
    void buildFinished(BuildResult result) {
        println "Task timings:"
        for (timing in timings) {
            if (timing[0] >= 50) {
                printf "%7sms  %s\n", timing
            }
        }
    }

    @Override
    void buildStarted(Gradle gradle) {}

    @Override
    void projectsEvaluated(Gradle gradle) {}

    @Override
    void projectsLoaded(Gradle gradle) {}

    @Override
    void settingsEvaluated(Settings settings) {}
}

gradle.addListener new TimingsListener()
```

上面是对每个任务计时的一个例子，想要了解每个任务的作用，你可以修改上面的脚本，打印出每个任务的inputs和outputs。比如assembleDebug那么多依赖任务，每个都是干什么的，一会compile，一会generate，有什么区别？看到每个task的输入输出，就可以大体看出它的作用。如果对assemble的每个任务监听，你会发现改一行代码build的时间主要花费在了dex上，buck牛逼的地方就是对这个地方进行了优化，大大减少了增量编译运行的时间。

### buildscript方法
Android项目中，根工程默认的build.gradle应该是这样的：

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

一会一个jcenter()这是在干什么？buildscript方法的作用是配置脚本的依赖，而我们平常用的compile是配置project的依赖。repositories的意思就是需要包的时候到哥这里来找，然后你以为`com.android.tools.build:gradle:1.2.3`会从jcenter那里下载了是吧，图样图森破，不信加入下面这段脚本看看输出：

```
buildscript {
    repositories {
        jcenter()
    }
    repositories.each {
        println it.getUrl()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}
```

结果是这样的：
>file:/Applications/Android%20Studio.app/Contents/gradle/m2repository/
>https://jcenter.bintray.com/

我靠，仓库竟然直接在Android Studio应用内部，下面还有更爽的，我们知道有依赖传递，上面classpath 中的`gradle`依赖`gradle-core`，`gradle-core`依赖`lint`，`lint`依赖`lint-checks`，`lint-checks`最后依赖到了`asm`，并且这个根目录中的依赖配置会传到所有工程的配置文件，所以如果你要引用asm相关的类，不用设置classpath，直接import就可以了。你怎么知道前面的依赖关系的？看上面m2repository目录中对应的pom文件就可以了。

为什么讲到ASM呢？[ASM](http://asm.ow2.org/)又是个比较刁的东西，可以直接用来操纵Java字节码，达到动态更改class文件的效果。可以用ASM[面向切面编程](http://developer.51cto.com/art/201309/410861_all.htm)，达到解耦效果。[Android DEX自动拆包及动态加载简介](http://tech.meituan.com/mt-android-auto-split-dex.html)中提到的class依赖分析和R常量替换的脚本都可以用ASM来搞。

### 引入脚本
脚本写多了，都挤在一个build.gradle里也不好，人长大了总要自己出去住，那可以把部分脚本抽出去吗？当然可以，新建一个other.gradle把脚本抽离，然后在build.gradle中添加`apply from 'other.gradle'`即可，抽出去以后你会发现本来可以直接import的asm包找不到了，怎么回事？根工程中配置的buildscript会传递到所有工程，但只会传到build.gradle脚本中，其他脚本可不管，所以你要在other.gradle中重新配置buildscript，可以在other.gradle中加入：

```
buildscript {
    repositories {
       jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}
```

### 获取AndroidManifest文件
[ApplicationId versus PackageName](http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename)提到，gradle中的applicationid用来区分应用，manifest中packageName用来指定R文件包名，并且各个productFlavor 的manifest中的packageName应该一致。applicationid只是gradle脚本中的定义，其实最后生成的apk中的manifest文件的packageName还是会被applicationid替换掉。

那获取R文件的包名怎么搞？要获取AndroidManifest中package属性，并且这个manifest要是起始的文件，因为最终文件中的package属性会被applicationid冲掉，由于各个manifest中的package属性一样，并且非主manifest可以没有package属性，所以只有获取主manifest的package属性才是最准确的。

```
def manifestFile = android.sourceSets.main.manifest.srcFile
def packageName = new XmlParser().parse(manifestFile).attribute('package')
```

### 无用资源

无用的资源就不要打包进APK了。
>[Resource Shrinking](http://tools.android.com/tech-docs/new-build-system/resource-shrinking)

### 一个Bug
之前在创业公司，用[Travis](https://travis-ci.org/)做持续继承，遇到一个让我很纠结的问题。在Travis上执行构建脚本如下：

```
./gradlew clean
./gradlew assembleXR
```

最后生成的APK在运行的时候报错，提示找不到某个.so文件，解压发现APK中果然缺少某个库工程的.so文件，但在本地运行的时候却是没有问题，纠结了好久，后来研究发现Android Studio中执行Clean Project的时候，会执行generateSources的任务，把它加入构建脚本后才打包正确。最近发现，这原来是个[Bug](https://code.google.com/p/android/issues/detail?id=106579&thanks=106579&ts=1421971822)，并且已经在android gradle1.3被修复了。

匆匆忙忙间，写了很多东西。读完此文，希望你能感受到构建神器的魅力，感受到它的灵活强大，当然也希望能让你使用Gradle更加得心应手。