---
layout: post
title: Rails 启动模式小谈
comments: true
category: tech
tags: rails
---

玩过rails的人都知道，rails里面默认提供了三种开发模式：development、production 和 test 模式。

development 模式一般在开发阶段使用，这个阶段的一个显著特点是代码改动比较频繁，所以相应的配置文件 development.rb 里面包含 `config.cache_classes = false` , 它表示每次请求时都会重新加载 class 。这虽然会导致程序速度变慢一点，但是一个最大的好处是不需要每次修改代码重启服务器（实际上并不是所有类都会重新加载的, 详见 [auto_load_paths](http://guides.rubyonrails.org/configuring.html#rails-general-configuration) ）。

<!--more-->

production 和 test 模式都一般会设置 `config.cache_classes` 的值为 `true` 。这样可以使得程序运行速度更快！

同时对应三种启动模式还有三套数据库，在 `config/database.yml` 里面。

这几种不同的模式带来的最大好处是区分对待：使得在开发阶段便于开发，生产阶段突出性能，测试阶段模拟生产却又不会破坏实际数据。

## 但是也有需要注意的问题 #

而他带来的主要问题是我们需要特别注意类变量的使用。在rails，有时候我们需要类变量来保存一些数据来达到优化性能的目的。

首先我们不考虑类变量的使用是否合理，我们需要注意的是在生产环境下，类变量是一直存在的，直到服务器停止才会销毁。而在开发环境下，相关的类变量在每次新的请求时都会重新设置。

另外，由于类变量是一直可用的，并且对所有该类的实例都可见，所以可能还需要考虑并发的问题。

如果你已经明确这些类变量的特点，那它可以帮你完成一些很难的事情，但是，如果你并不知道可能会发生什么时，请谨慎使用类变量，否则很可能得到的是一个惊喜！！

- Tip：

使用不同的模式启动server和console命令：

    ruby script/server -e XXX
    ruby script/console XXX

在 rails 3 里面是

    rails s -e XXX
    rails c XXX

其中 XXX 是对应的模式名称。默认是 development，也可以是 production 或者 test ，甚至是你自己定义的一种模式（只需要在 `config/enviroments` 目录下加上相应的配置文件 `config/database.yml` 里面加上对应的配置即可 ）。
