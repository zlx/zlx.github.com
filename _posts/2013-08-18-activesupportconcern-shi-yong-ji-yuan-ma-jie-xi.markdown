---
layout: post
title: "ActiveSupport::Concern 使用及源码解析"
date: 2013-08-18 19:56
comments: true
category: tech
tags: rails concern module
---

今天我们讲讲 Rails 里面的 ActiveSupport::Concern 。

ActiveSupport::Concern 是 Rails 提供的一个处理模块的小工具。

<!--more-->

首先我们需要了解 Ruby 里面的模块是怎么工作的！如果你还不知道，可以参见我的 [Ruby 中的模块使用](/blog/2013/08/18/ruby-zhong-de-mo-kuai-shi-yong) 这篇博客。


Ruby 的模块在大部分时候工作良好，但在有些情况下还需要一些 hack ！

### 问题

#### 我们想要包含模块后，部分方法表现为实例方法，另外一些方法表现为类方法?

Ruby 世界中的最佳实践是：

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
    Car.cawesome     #=> "hello, i'm class method"

看上去很不错。既然大家都这么做，为什么每次都要自己去写 `base.extend ClassMethods` 呢。好吧， 这就是 Rails 世界的规则：约定大于配置。

所以，你可以这样写：

    require 'active_support/concern'
    module Foo
      extend ActiveSupport::Concern
      
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
    Car.cawesome     #=> "hello, i'm class method"

Good. 

#### Ruby 里面模块依赖要宿主类自己去处理？

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
    
    module Bar  
      def self.included(base)
        base.cawesome   #=> "hello, i'm class method"
      end
    end
    
    class Car
      include Foo
      include Bar
    end
    
    Car.cawesome   #=> "hello, i'm class method"

So Terrible！

[Rails 的厨师们](http://david.heinemeierhansson.com/2012/rails-is-omakase.html) 认为我们可能会喜欢这样:

    require 'active_support/concern'
    module Foo
      extend ActiveSupport::Concern
      
      module ClassMethods
        def cawesome
          puts "hello, i'm class method"
        end
      end
      
      def awesome
        puts "hello, i'm instance method"
      end  
    end
    
    module Bar
      extend ActiveSupport::Concern
      include Foo
    
      included do |base|
        base.cawesome   #=> "hello, i'm class method"
      end
    end
    
    class Car
      include Bar
    end
    
    Car.cawesome   #=> "hello, i'm class method"

看上去的确不错，不是吗？

### 看看实现

查看 [concern 的源代码](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/concern.rb)，可以看到主要就三个方法： #extended, append_features, 和 included 。

    def self.extended(base) #:nodoc:
      base.instance_variable_set("@_dependencies", [])
    end

extended 是 Ruby 提供的一个钩子,每次执行 `extend <module name>` 的时候就会被调用。它在宿主上设置了一个实例变量：@_dependencies ，便于之后使用。

    def included(base = nil, &block)
      if base.nil?
        @_included_block = block
      else
        super
      end
    end

included 不同于 self.included 方法，每次 extend Concern 这个模块时，这会变成宿主类的一个类方法(对于模块而言就是模块方法)。它接受两个参数， base 和 block。 如果 base 为空， 就将 block 保存在 @_included_block 这个变量里面，方便之后使用。如果base 存在，就调用 super 调用原来的 included 方法。

    def append_features(base)
    	if base.instance_variable_defined?("@_dependencies")
    	  base.instance_variable_get("@_dependencies") << self
    	  return false
    	else
    	  return false if base < self
    	  @_dependencies.each { |dep| base.send(:include, dep) }
    	  super
    	  base.extend const_get("ClassMethods") if const_defined?("ClassMethods")
    	  base.class_eval(&@_included_block) if instance_variable_defined?("@_included_block")
    end

append_features 是 Ruby 提供的另一个钩子，每次执行 `include <module name>` 时，都会被调用。我们来看看这里都做了什么。

第一条线，如果 base 定义了 @_dependencies ，则把当前模块加入 base 类的@_dependencies 中, 然后直接返回。之前那个例子中 Foo 就被放在了 Bar 的 @_dependencies 中。

第二条线，如果 base 没有定义 @_dependencies， 也就是 Car 那个类的情况。

1. 如果 `base < self` 就返回 false ，这是为了避免重复包含；
2. 然后检查 @_dependencies ,将之前记录下来的依赖模块都包含进来；
3. 调用 super 执行真正的 include 操作（Ruby 实现的 append_features 方法）；
4. 如果当前模块中有 ClassMethods 模块，就 `extend ClassMethods`；
5. 如果有 @_include_block 就在当前作用域中执行

ok, 至此，就可以保证模块里面的依赖关系得以解决，而不需要由宿主类来处理。


### 参考资料

+ [ActiveSupport::Concern 代码解读](http://www.zhlwish.com/2012/07/23/rails-activesupport-concern/)
  
enjoy!
