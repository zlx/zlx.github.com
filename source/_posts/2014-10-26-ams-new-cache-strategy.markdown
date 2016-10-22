---
layout: post
title: "AWS 缓存策略设想"
date: 2014-10-26 09:54
comments: true
category: tech
tags: ams cache
---

AMS 是一个非常不错的尝试，合理使用能简化 API 的设计与实现。

目前稳定的 0.9 版本并没有自带有缓存的实现，所以出现了很多民间解法，下面根据自己的经验，提出了一种新的缓存方式。

以下所有设想及实现均基于 ActiveModel::Serializer 0.9 版本，大家可以查看[我 fake 的版本](https://github.com/zlx/active_model_serializers/tree/new_cache_strategy)

<!--more-->

### 背景分析

每一个 AMS 对象其实由两部分组成，自身的 attributes 和 关联的其他 AMS 对象。如图:

![AMS 对象组成](http://blog.zlxstar.me/images/ams_object.png)

而我们在使用 AMS 时候，其实有如下两个事实：

1. 我们经常在一个 AMS 对象里面关联多个对象
2. 一个数据的变化经常导致多个 AMS 对象都需要更新

由此我们可以设想如下：

1. 如果我们仅缓存 AMS 的 attributes 部分，可以让缓存最大限度地被重复利用
2. 如若发生数据变化，仅直接相关的 AMS 对象（attributes 里面包含发生变化的数据）需要更新，其他关联的 AMS 对象不需要更新其缓存


### 实现及性能测试

据此，我基于 0.9 版本实现了这种设想，[New Cache Strategy](https://github.com/zlx/active_model_serializers/tree/new_cache_strategy)

同时，我对其做了一个性能测试---[测试项目](https://github.com/zlx/ams-demo)

最后发现这种方式在变化率非常高的情况下依然有比较好的表现，基本保持在50倍以上的性能优化，也就是至少节约了 98% 的时间。

![性能测试结果](https://raw.githubusercontent.com/zlx/ams-demo/master/chart.png)

### 链接

+ [Active Model Serializer](https://github.com/rails-api/active_model_serializers)
+ [我的缓存设想](https://github.com/zlx/active_model_serializers/tree/new_cache_strategy)
+ [性能测试项目](https://github.com/zlx/ams-demo)

enjoy!
