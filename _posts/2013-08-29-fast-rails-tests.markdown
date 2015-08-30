---
layout: post
title: "Faster Rails tests"
date: 2013-08-29 10:27
comments: true
category: tech
tags: rails test
---

近段时间研究了一下 Rails 里面 Faster Test 的主题，现将研究结果记录下来，方便自己查阅和有类似需求的人参考。如有错误，欢迎指正！


针对 Rails 项目，Faster Test 主要从以下四个方面寻求突破：

1. Boot Time
2. Multi-Core
3. GC
4. Code

<!--more-->

## Boot Time

我们知道，当我们的 Rails 项目达到一定规模之后，启动时间就会变的越来越慢。对于测试而言，它需要加载 Rails 环境，包含各种 Gem 依赖，所以这是一大块时间支出。

关于这点，我相信大家都有深刻的体会，我就不举实际的例子了，直接拿出我们的解决方案。

### ZEUS

ZEUS 是一个 go 语言实现的 Gem， 通过它，我们可以加载一次 Rails 环境，然后使用它的命令来跑测试，这样就可以省掉 Rails 启动时间了。

详细使用方法请参考 [ZEUS](https://github.com/burke/zeus)


### Spring

Spring 是类似功能的 Gem, 但它是纯 Ruby 的实现，使用方式也不一样。

详细使用说明请参考 [Spring](https://github.com/jonleighton/spring)

### Do not load entire Rails

既然加载 Rails 环境很慢，那我们就不要加载它，正如 Gary Bernhardt 所说

> Rails is not your application

关于这点，我们将在 [Code](http://blog.zlxstar.me/blog/2013/08/29/faster-rails-tests-2/) 一节详细介绍。


实现类似需求的 Gem 还有 [Spork](https://github.com/sporkrb/spork) [Commands](https://github.com/rails/commands)

## Multi-Core

我们现在的机器都很先进了，动不动双核，四核。但是我们跑测试的时候只能用一个核，是不是太浪费了。如果我们利用了多核的优势，那我们的测试将 2x 或者 4x ，这是多么理想呀。

很幸运，我们生活在一个工具丰富的时代：

### Parallel Tests

Parallel_tests 是做的一个很完善的工具，它支持 Test::Unit, Rspec, Cucumber

使用之前，你需要配置一下 database.yml:

    test:
      database: yourproject_test<%= ENV['TEST_ENV_NUMBER'] %>

然后你就可以

1. 使用 `rake parallel:test` 来跑 Test::Unit 的测试
2. 使用 `rake parallel:spec` 来跑 Rspec 的测试
3. 使用 `rake parallel:features` 来跑 Cucumber 的测试

其它还有更丰富的使用方法，请参考 [parallel_tests](https://github.com/grosser/parallel_tests)

另外还有人将 parallel_tests 和 zeus 结合，写了 [zeus-parallel_tests](https://github.com/sevos/zeus-parallel_tests)

既节省了启动时间，有利用了多核技术，测试时间立马飞涨！！！

### Specjour

Specjour 是利用多核技术加速测试的另一种实现，它目前支持 Rspec 和 cumcuber 

具体可以参考 [Specjour]((https://github.com/sandro/specjour)


## GC

我们知道 Rails 是基于 Ruby 的一个 web 框架，而 Ruby 是一门虚拟机语言，同 Java 一样，在 GC 上面优化依然可以加快测试。

Ruby 2.0 之前的 GC 性能很不好，Ruby 2.0 对 GC 进行了很大的优化。关于 Ruby 2.0 的 GC 实现，我们这里并不涉及。我们仅仅介绍一下我们可以利用 GC 做那些优化。

### GC report

知道了 GC 的具体状态，我们才能更好地优化。

如果是 Ruby 2.0 , 你可以使用 **GC::Profiler** 来查看 GC 的时间，次数等等信息。

    GC.enable          开始统计
    GC.disable         结束统计
    GC.raw_data        GC 信息， Hash 形式返回
    GC.result          GC 信息， 表格形式返回
              
更多信息请参考 [GC::Profiler](http://ruby-doc.org/core-2.0/GC/Profiler.html)


如果是 Ruby 1.9 或者 Ruby 1.8 ，你需要打一个 GC 的 Patch 才能查看。[这里](https://github.com/skaes/rvm-patchsets) 有各版本 Patch 的集合 

	GC.enable_stats    开始统计
	GC.disable_stats   结束统计
	GC.collections     GC 次数
	GC.time            GC 时间
	GC.clear_stats     清空统计结果

更详细的参数可以参考 [Garbage collector statistics
](http://www.rubyenterpriseedition.com/documentation.html#_garbage_collector_statistics)

### GC configration

+ RUBY_HEAP_MIN_SLOTS
+ RUBY_HEAP_SLOTS_INCREMENT
+ RUBY_HEAP_SLOTS_GROWTH_FACTOR
+ RUBY_GC_MALLOC_LIMIT
+ RUBY_HEAP_FREE_MIN

这是 GC 里面的一些参数，通过合理的配置他们，你可以明显减少 GC 的次数和时间，有效提高测试的速度。关于这点我并没有太多经验，这里仅仅展示一下 37Signals 和 twitter 的配置：

37Signals's Settings

> RUBY_HEAP_MIN_SLOTS=600000
RUBY_GC_MALLOC_LIMIT=59000000
RUBY_HEAP_FREE_MIN=100000

twitter's settings
 
>RUBY_HEAP_MIN_SLOTS=500000
RUBY_HEAP_SLOTS_INCREMENT=250000
RUBY_HEAP_SLOTS_GROWTH_FACTOR=1
RUBY_GC_MALLOC_LIMIT=50000000

关于这一块，可以参考一下其他人的优化经验

+ [37signals -- the road to faster tests](http://37signals.com/svn/posts/2742-the-road-to-faster-tests)
+ [Rails Tests Run in 2/3 Time w/ GC Tuning](http://blog.winfieldpeterson.com/2010/12/10/rails-tests-run-in-23-time-w-gc-tuning/)

## Code

这块是我想重点介绍的，请移步 [Faster Rails Tests 2](http://blog.zlxstar.me/blog/2013/08/29/faster-rails-tests-2/)

enjoy!
