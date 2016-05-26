---
layout: post
title: "Ruby 中的模块使用"
date: 2013-08-18 20:48
comments: true
category: tech
tags: ruby module include extend
---

今天我们来讲讲 Ruby 里面的模块是怎么工作的！

模块在 Ruby 里面非常重要。我们知道，在 Ruby 中，是不能直接使用多继承的，而使用了一种 mixin 的机制来实现多继承的需求。模块就是 mixin 的载体。

模块同类一样，也有 class method 和 instance method。其中 class method 在模块中称为模块方法，是可以直接调用的。

<!--more-->

    module Foo
      def self.hello
        puts 'hello world!'
      end
    end
    
    Foo.hello   #=>  'hello world!'    

而对于模块里面的 instance method， 主要有两种形式，而这取决于如何包含这个模块: include 还是 extend 。

    module Foo
      def awesome
        puts 'awesome method'
      end
    end
    
    class Car1
      include Foo
    end
    
    class Car2
      extend Foo
    end
    
    Car1.new.awesome  #=> 'awesome method'
    Car2.awesome  #=> 'awesome method'   

ok。看上去非常不错。可是，我现在有一个非常好用的模块，我希望包含这个模块后，有些方法作为实例方法，有些方法作为类方法，这时候，该如何做呢？

道高一尺，魔高一丈。你可以这样：

    module Foo
      def self.included(base)
        base.extend ClassMethods
      end
      
      module ClassMethods
        def cawesome
          puts "hello, i'm class method"
        end
      end
      
      def awesome
        puts "hello, i'm instance method"
      end  
    end
    
    class Car
      include Foo
    end
    
    Car.new.awesome  #=> "hello, i'm instance method"
    Car.cawesome     #=> "hello, i'm instance method"

So Beautiful. 这看上去非常不错。

但是，问题又来了，假如我们有另一个模块，依赖于 Foo 模块，像这样：

    module Bar
      include Foo
      def self.included(base)
        base.cawesome
      end
    end
    
    class Car
      include Bar
    end
    
    Car.cawesome   #=> NoMethodError    

出问题了，为什么 Car 中没有 cawesome 这个方法？

因为 included 是 module 的一个方法，所以 Bar 在包含 Foo 这个模块时，直接将 cawesome 变成了 Bar 的模块方法。所以在包含 Bar 时，在 Car 上面调用 cawesome 就报错了。

然后，那我们直接在 Car 里面包含 Foo 和 Bar。 像这样：

    module Bar
      def self.included(base)
        base.cawesome
      end
    end
    
    class Car
      include Foo
      include Bar
    end
    
    Car.cawesome   #=> "hello, i'm class method"    

ok， 看上去不错。

可是你是一个代码偏执狂：为什么 Bar 的依赖要 Car 来处理呢？

如果你想知道怎么做，可以参见 [ActiveSupport::Concern 使用及源码解析](/blog/2013/08/18/activesupportconcern-shi-yong-ji-yuan-ma-jie-xi)。


enjoy!
