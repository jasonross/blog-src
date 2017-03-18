title: 编写多类型列表视图，哪家强？
date: 2015-10-11 23:25:55
tags: [Android,ListView,RecyclerView,Adapter,MultiType,Epoxy]
---

在日常开发中，我们经常会遇到一些复杂的列表视图，一般都是通过多个viewtype的方式来解决，添加类型常量和ViewHolder，步骤繁琐，各种if/else判断，对扩展也不太友好。

很多人都在试图简化多类型的写法，在Github就有不少的开源项目。

### MultiType

@drakeet 的[MultiType](https://github.com/drakeet/MultiType)，添加了Item和ItemViewProvider的概念，通过全局类型池连接Item和ItemViewProvider，只需要提供Item列表，就可以自动填充View了。

感觉Item的概念有些强硬，有的Cell根本不需要Item只需要一个layout，这种情况下去创建一个空Item有些多余，当然也不是什么大问题。

### DataBindingAdapter

@markzhai 的[DataBindingAdapter](https://github.com/markzhai/DataBindingAdapter)，主要是使用DataBinding库简化了数据绑定的操作，数据和layout之间通过viewtype连接。

还是要有各种类型常量，有些麻烦。

### Epoxy

@airbnb 的 [Epoxy](https://github.com/airbnb/epoxy)添加了Model的概念，相当于是把ItemViewProvider和Item直接放在一起，Item可有可无，layoutId单独提出来作为viewType，为了避免在spanSizeLookup中各种instanceOf，spanSize放在了Model中，可以自由配置。

不强制要求Item，不需要类型常量，Epoxy的使用更加简单一些。

Epoxy也对Model操作进行了封装，支持insertModelBefore等类似操作，可以很方便的局部刷新。比如一个页面中，有上、下两部分都在进行网络请求，上面的数据返回后，需要修改ModelList，可以直接执行insertModelBefore(loadingModel)。我们现在项目中主要使用的是ListView，这个情况主要是通过Merge上、下两个Adapter，监听两个Adapter的数据变化，也是不错的思路，但是做不到局部刷新，性能上不是很好，当然这是ListView和RecyclerView的区别。

Epoxy也做了自动Diff，在Android官方的DiffUtil前，后面可能会提供两种算法支持。

------

之前在贴吧做iOS的时候，几乎所有的页面都是用UITableView实现的。Android上也可以这么做，RecyclerView+GridLayoutManager直接搞定大部分页面，再加上Epoxy这样的库，如虎添翼。