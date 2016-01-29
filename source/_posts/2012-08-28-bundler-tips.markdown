---
layout: post
title: "Bundler 知识点"
date: 2012-08-28 15:40
comments: true
category: tech 
tags: Bundler
---

Bundler 现在俨然是 Rails 的一个标准装备了。想将我使用 Bundler 的心得记录如下：

## Rationale

最简单使用 Bundler 的方式的是在当前目录下创建一个 Gemfile 文件，然后运行 Bundle 。(当然需要先安装 Bundler)

<!--more-->

    touch Gemfile
    Bundle

但是现在 Gemfile 里面没有任何内容，所以会报一个 `WARNING： The Gemfile specifies no dependencies`
将下列内容写入 Gemfile 文件。

    source "http://rubygems.org"
    
    gem "rails", "3.0.0.rc"
    gem "rack-cache"
    gem "nokogiri", "~> 1.4.2"

再次运行 `Bundle` , 你会发现当前目录下多了一个 Gemfile.lock 文件。

*Bundler 会在当前目录下寻找 Gemfile 文件，然后按照他的规则解析，并安装 gems 最后将安装好的 gems 版本和依赖等信息记入 Gemfile.lock 文件*

## source

注意之前的 Gemfile 文件内容中的：

    source "http://rubygems.org"

这是指定 gem 安装的搜索源。默认是 http://rubygems.org , 这是官方源。国内可以使用 http://ruby.taobao.org/ (由衷感谢淘宝)
你可以指定多个 source ，bundler 会按照顺序一个个查找。

## Group

在 Gemfile 中， 你可以指定 group

    source "http://rubygems.org"
    
    gem "rails", "3.2.2"
    gem "rack-cache", :require => "rack/cache"
    gem "nokogiri", "~> 1.4.2"
    
    group :development do
      gem "sqlite3"
    end
    
    group :production do
      gem "pg"
    end

然后我们就可以使用 `bundle install --without production` 跳过 :production 组的 gem 的安装。

另外，我们使用 `Bundler.require(:default, :development)`, 来 require `rails rack/cache nokogiri sqlite3` 而不 require `pg`

## require

默认情况下： Bundle.require 会直接 require 和 gem 同名的文件。

但是你可以使用如下三种形式改变它：

    gem "redis", :require => ["redis/connection/hiredis", "redis"]
    gem "webmock", :require => false
    gem "rack-cache", :require => "rack/cache"

其中第一种会在 require `redis` 时候去加载 `redis/connection/hiredis` 和 `redis` 两个文件。
第二种不会加载任何文件。
第三种会在 require `rack-cache` 时候去加载 `rack/cache` 文件。

## version

关于 Gemfile 中的 gem 的版本问题，主要是涉及到之后的 gem 升级。
随着时间的推移，我们项目中使用的 gem 如果不及时更新，就有可能给我们的项目造成潜在的危险（比如说某个 gem 的 bug 导致我们的网站有可能遭受攻击）

对于这个问题，一般有三种态度

1. Optimistic Version Constraint（乐观）

2. Pessimistic Version Constraint（悲观）

3. Absolute Version Constraint（超级悲观）

### 第一种

    gem 'devise', '>= 1.3.4'

假定 1.3.4 之后的 devise 版本都是可用的，这时候运行 bundle update 后，会获取 devise 的最新版本安装。

### 第二种

    gem 'library', '~> 2.2'

这种情况下，bundle update 会获取 library 2.2 到 3.0 之间的最新版本。

### 第三种

    gem 'library', '2.2'

这种情况下，bundle update 不会更新 library

三种情况下，多数人会推荐使用第二种。这样既能保证 gem 及时更新，又能保证尽量不会破坏我们的功能。因为按照版本命名规范第一个版本号不改变代表不改变接口。
详细参考[gem 的 versioning policies 说明](http://docs.rubygems.org/read/chapter/7)
