---
layout: post
title: "Rails 和页面缓存和清理"
date: 2014-07-23 18:04
comments: true
category: tech
tags: rails cache sweeper
---

### 背景

Page Cache 从 Rails 4 中移除了，取而代之的是 DHH's [cache digests](https://github.com/rails/cache_digests) 其优势及好处可以参见[这里](http://signalvnoise.com/posts/3113-how-key-based-cache-expiration-works)

但是对于一些分享出去的静态页面，Page Cache 还是非常有优势的。

<!--more-->

### 这里有一个怪异的地方

当你使用 [page cache](https://github.com/rails/actionpack-page_caching) , 并尝试使用 [Sweepers](https://github.com/rails/rails-observers#action-controller-sweeper) 作为清理缓存的方式，那你就要注意了：

官方文档中介绍：

> Sweepers are the terminators of the caching world and responsible for expiring caches when model objects change. They do this by being half-observers, half-filters and implementing callbacks for both roles. A Sweeper example:
>
>       class ListSweeper < ActionController::Caching::Sweeper
>         observe List, Item
>
>         def after_save(record)
>           list = record.is_a?(List) ? record : record.list
>           expire_page(:controller => "lists", :action => %w( show public feed ), :id => list.id)
>           expire_action(:controller => "lists", :action => "all")
>           list.shares.each { |share| expire_page(:controller => "lists", :action => "show", :id => share.url_key) }
>         end
>       end
>
> The sweeper is assigned in the controllers that wish to have its' job performed using the cache_sweeper class method:
>
>       class ListsController < ApplicationController
>         caches_action :index, :show, :public, :feed
>         cache_sweeper :list_sweeper, :only => [ :edit, :destroy, :share ]
>       end

在这里，只有 :edit, :destroy, :share 三个 action 定义了要调用  sweeper，直观意思是只有这三个 action 会执行 ListSweeper 的代码，但是情况并非如此。

如果你还有一个 create 方法，那就会抛出一个这样的异常：`NoMethodError (undefined method 'expire_page' for #<ListSweeper:0x0000010428dd90 @controller=nil>)`

### 手术刀来了

调来 cache_sweeper 的代码:

    def cache_sweeper(*sweepers)
      ...

      sweepers.each do |sweeper|
        ActiveRecord::Base.observers << sweeper if defined?(ActiveRecord) and defined?(ActiveRecord::Base)
        sweeper_instance = (sweeper.is_a?(Symbol) ? Object.const_get(sweeper.to_s.classify) : sweeper).instance

        if sweeper_instance.is_a?(Sweeper)
          around_filter(sweeper_instance, :only => configuration[:only])
        else
          after_filter(sweeper_instance, :only => configuration[:only])
        end
      end
    end

可以看到，首先它会将 sweeper 塞入 ActionRecord::Base.observers 里面，也就是说 sweeper 首先是一个 observer，会有 observer 一样的效果，比如说执行 after_save 等 callbacks 。

然后，会定义一个 around_filter 或者 after_filter ，参数就是 cache_sweeper 的 only 参数。

我们接着看 Sweeper 的代码：

    class Sweeper < ActiveRecord::Observer #:nodoc:
        ...

        def before(controller)
          self.controller = controller
          callback(:before) if controller.perform_caching
          true # before method from sweeper should always return true
        end

        def after(controller)
          self.controller = controller
          callback(:after) if controller.perform_caching
        end

        def around(controller)
          before(controller)
          yield
          after(controller)
        ensure
          clean_up
        end
    end

这里主要在 before 和 after 里面设置了当前 controller 。

至此，问题就清楚了，因为 `cache_sweeper :list_sweeper, :only => [ :edit, :destroy, :share ]` 并没有指定 create ，所以执行 create 方法的时候就没有执行 before ，也就是 controller 并没有设值，所以导致了在 sweeper 里面的 expire_page 没有定义了。

### 尝试下药

1. 最简单粗暴的方式就是调用 expire_page 之前先做个检查了，比如改成：

        class ListSweeper < ActionController::Caching::Sweeper
            def after_save(record)
              list = record.is_a?(List) ? record : record.list
              expire_page(:controller => "lists", :action => %w( show public feed ), :id => list.id) if controller.respond_to?(:expire_page)
              expire_action(:controller => "lists", :action => "all") if controller.respond_to?(:expire_action)
              list.shares.each { |share| expire_page(:controller => "lists", :action => "show", :id => share.url_key) } if controller.respond_to?(:expire_page)
            end
        end

2. 如果你仅仅是想清除缓存，那就不要使用 sweeper 了，试试直接在 controller 里面的 expire_page

        class ListsController < ApplicationController
          after_action :clear_list_page_cache, :only => [ :edit, :destroy, :share ]

          private
          def clear_list_page_cache
            expire_page(:controller => "lists", :action => %w( show public feed ), :id => params[:id])
          end
        end

3. 什么，你是手动党？

看看 public 下面的目录，手动清理吧，骚年！


enjoy!
