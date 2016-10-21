---
layout: post
title: "新仔杂谈第010期"
date: 2014-05-07 15:13
comments: true
category: 新仔杂谈
---

最近一段时间一直忙于工作，没有时间能静下心来好好整理，今天算是补上一期。

下面是我发现的有意思的东西。

<!--more-->

## 新大陆

1. Facets

   这个工具能够根据你的 Gemfile.lock 检测出你可能有哪些安全隐患，对于注重系统安全的朋友是一个不可多得的工具。

   [https://hakiri.io/facets](https://hakiri.io/facets)

2. Make your code support console

   对于 Rails 的 console，是不是感觉非常方便。如果你的代码也可以支持 console，对于新用户的来探索使用你的接口会不会方便很多。

   在这里，仅 7 行代码让你的代码具备这项功能。

   [7-lines-every-gems-rakefile-should-have/](http://erniemiller.org/2014/02/05/7-lines-every-gems-rakefile-should-have/)

3. anonymous controller

   你有没有碰到需要测试 controller 里面的某些公共方法或者功能，但是又不想随便使用一个 controller 实例来增加测试的依赖，今天就介绍一个东西可以帮助你：anonymous controller

   [how-do-i-test-an-application_controller-on-a-rails-app/](http://erniemiller.org/2014/02/05/7-lines-every-gems-rakefile-should-have/)
   [anonymous-controller](https://www.relishapp.com/rspec/rspec-rails/docs/controller-specs/anonymous-controller)

4. Recommundle

   看到你长长的 Gemfile 列表，是不是想看看有没有更好的 Gem 可以使用。看看这个：

   [recommundle](http://recommundle.com/)

5. rubygeocoder

   地理位置服务是一个现在非常流行的服务，而 rubygeocoder 提供了一个完全的 Ruby 解决方案。

   [rubygeocoder](http://www.rubygeocoder.com/)


enjoy!
