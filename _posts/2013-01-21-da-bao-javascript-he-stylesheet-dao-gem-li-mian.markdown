---
layout: post
title: "打包javascript和stylesheet到gem里面"
date: 2013-01-21 17:49
comments: true
category: tech
tags: gem assets
---

将 javascript 和 stylesheet 封装在 gem 里面，如何做呢？

假定你的 gem 名字叫做 foo
目录结构如下：

<!--more-->

    |-- assets
    |   |-- images
    |   |-- javascripts
    |   `-- stylesheets
    |-- foo.gemspec
    |-- Gemfile
    |-- lib
    |   |-- foo
    |   |   `-- version.rb
    |   `-- foo.rb
    |-- LICENSE.txt
    |-- Rakefile
    `-- README.md

首先在 foo 目录下创建一个文件 `engine.rb`

    # coding: utf-8
    module Foo
      module Rails
        class Engine < ::Rails::Engine
        end
      end
    end

和 `railtie.rb`

    # coding: utf-8
    module Foo
      module Rails
        class Railtie < ::Rails::Railtie
        end
      end
    end

然后你在 `foo.rb` 包含代码

    if ::Rails.version < "3.1"
      require "foo/railtie"
    else
      require "foo/engine"
    end

这样，他就会自动将 lib 目录下的 assets 目录导入到项目里面。

然后，在你的项目 application.css 和 application.js 里面包含你的 gem 里面的 css 和 js, 就可以直接使用了。


参考资料：

- [using railtie and rails engine in gems](http://www.tweetegy.com/2012/09/using-railtie-and-rails-engine-in-gems/)

- [adding assets to your gems](http://edgeguides.rubyonrails.org/asset_pipeline.html#adding-assets-to-your-gems)

- [social share button](https://github.com/zlx/social-share-button)

enjoy!
