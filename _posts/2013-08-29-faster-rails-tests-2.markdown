---
layout: post
title: "Faster Rails Tests 2"
date: 2013-08-29 15:04
comments: true
category: tech
tags: rails test
---

接上文 [Faster Rails Test](http://blog.zlxstar.me/blog/2013/08/29/fast-rails-tests/)

这里我们就来重点讲讲如何在代码层面加速测试。

这里我将从以下几点来介绍：

1. FactoryGirl
2. Mock
3. Without Rails

<!--more-->

### FactoryGirl

[FactoryGirl](https://github.com/thoughtbot/factory_girl) 是一个很好的插件，大部分时候它都给我们提供了很多方便。但是不可否认，它很慢。

原因也显而易见。但我们项目变得很复杂时， Model 之间的关联也会很复杂，而 FactoryGirl 会帮我们创建好这些关联记录。

比如说我们是一个电子商务系统，那 Order 可能就是我们的一个非常关键的对象, 也就是 God Object。

现在我们有一个这样的测试：

    describe "#full_name" do
      it "returns the order placer's concatenated name" do
        order = FactoryGirl.create(:order, first_name: 'Ben', last_name: 'Orenstein')
        order.full_name.should == 'Ben Orenstein'
      end
    end

这个测试很简单，就是测试订单的 full_name 这个方法。
但是这个测试要花多少时间？ 7.3 seconds 没错，可能你的实际情况中不会那个久。

现在我们来换一种方式：

    describe "#full_name" do
      it "returns the order placer's concatenated name" do
        order = Order.new(first_name: 'Ben', last_name: 'Orenstein')
        order.full_name.should == 'Ben Orenstein'
      end
    end

这里我们仅仅用 `Order.new` 代替了 `FactoryGirl.create` 但是这个测试跑了多长时间？1/20000 seconds

整整快了 35000 倍！！！

*注明：例子参考 [Ben's Mail List](http://www.fastrailstests.com/)* 

尽管你的项目中可能不会有这么大的提升，但是，相信我：当你的测试跑得太慢时，使用原来的对象创建方式来替换掉 FactoryGirl ，你的测试会快很多。

关于这个主题，可以参考这些文章

+ [Keep performance in mind when using FactoryGirl in your test suite](http://blog.12spokes.com/web-design-development/how-factorygirl-can-slow-down-your-test-suite-aka-factory-build-vs-blank-activerecord-objects/)
+ [Faster test suite boot times with Ruby on Rails](http://blog.codeship.io/2013/08/21/faster-test-suite-boot-times-with-ruby-on-rails.html?utm_source=rubyweekly&utm_medium=email)


### Mock

Mock 是 Ruby 社区近年来很火的一个概念，适当的使用 Mock 可以让你的测试跑的又快又健壮。

拿上面的那个例子举例，假如 Order 是这样的，

    class Order < AR::Base
      def full_name
        "#{self.first_name} #{self.last_name}"
      end
    end  
    
如果使用 Mock ，可以这样：

    describe "#full_name" do
      it "returns the order placer's concatenated name" do
        order = mock("Order")
        order.stub(:first_name => 'Ben')
        order.stub(:last_name => 'Orenstein')
        order.full_name.should == 'Ben Orenstein'
      end
    end

关于 Mock 对象，可以单独形成一篇博客，也有很多人谈论了这些概念，这里列出几篇文章

+ [Mocks Are not Stubs](http://martinfowler.com/articles/mocksArentStubs.html)
+ [stubbing is not enough](http://gmoeck.github.io/2011/10/26/stubbing-is-not-enough.html)
+ [An introduction to mock objects](http://jamesmead.org/talks/2007-07-09-introduction-to-mock-objects-in-ruby-at-lrug/)
+ [spy vs spy](http://robots.thoughtbot.com/post/159805295/spy-vs-spy)

同样，你如果要在测试中使用 Mock 这种思想，这里推荐几个 Gem：

+ [FlexMock](https://github.com/jimweirich/flexmock)
+ [Mocha](https://github.com/freerange/mocha)
+ [RSpec-mocks](https://github.com/rspec/rspec-mocks)
+ [RR](https://github.com/btakita/rr)
+ [bourne](https://github.com/thoughtbot/bourne)

其实还有一类是模拟真实环境的，这种 Mock 又叫 Fake，比如说模拟数据库，网络，文件，甚至时间等。

* [Mock Time](https://github.com/travisjeffery/timecop)
* [Fake File System](https://github.com/defunkt/fakefs)
* [Fake Web Request](https://github.com/chrisk/fakeweb)
* [Fake Redis](https://github.com/guilleiguaran/fakeredis)
* [Fake S3](https://github.com/jubos/fake-s3)
* [Fake Request like VCR](https://github.com/vcr/vcr)

Mock 是一个非常实用的功能，合理利用它，你不但可以让你的测试跑得更快，也可以做到测试隔离，降低你的依赖，让你的测试更健壮。

### Without Rails

或许很多人会感到奇怪，我们讲 Rails 测试，为什么会说 without Rails 呢。

正如 Gary Bernhardt 所说：

>Rails is not your application

我们要测试的是我们的项目，不是 Rails，所以我们为什么要加载 Rails 环境呢？

假如我们有一个 ShoppingCart ：

    class ShoppingCart < ActiveRecord::Base  
      has_many :products  
      def total_price    
        products.map(&:price).inject(:+)  
      end
    end

然后我们要测试 total_price 这个方法。自然我们需要加载 Rails 环境，然后构造一个 ShoppingCart 对象，然后往里面放几个 Product 。

现在我们来稍微改造一下：

    # lib/calculate_total_price.rb
    class CalculateTotalPrice  
      def self.of(elements)    
        elements.map(&:price).inject(:+)  
      end
    end
    
    class ShoppingCart < ActiveRecord::Base  
      has_many :products  
      def total_price    
        CalculateTotalPrice.of(products)  
      end
    end

我们把计算逻辑移到 **CalculateTotalPrice** 这个类, 然后直接调用这个类来完成计算。那我们就可以这样测试：

    # spec/lib/calculate_total_price_spec.rb
    require 'calculate_total_price'
    
    describe CalculateTotalPrice do  
      it "returns 0 when there are no products" do    
        expect(CalculateTotalPrice.of([])).to eq(0)  
      end  
      
      it "returns the sum of prices of the products" do    
        products = [stub(price: 5), stub(price: 10)]
        expect(CalculateTotalPrice.of(products)).to eq(10)  
      end
    end

这里，我们并没有 `require 'spec_helper'` 也就是说我们并没有加载 Rails 环境，并且直接使用 Mock 对象来完成数据的构造。所以这个测试就是一个 Ruby 的测试，可以想象，这会有多快！！


关于这块内容，[Corey Haines - Fast Rails Tests](http://vimeo.com/30893836) 非常值得一看。

它给了我们一些指导，如何加速我们的测试：

1. Extract business logic into modules
2. Extract domain objects into classes

并且提出了的 TDD 的一种实践方式（[Test Driven Design](http://stackoverflow.com/questions/7538744/is-test-driven-development-the-same-as-test-driven-design)）

他们提出要倾听你的测试，让测试来驱动你的设计。

关于这块内容有很多尝试，这里列出一些：

+ [Talk Review: Fast Rails Test](http://fespinoza.github.io/blog/2013/03/29/talk-review-fast-rails-tests/)
+ [4 Steps to Faster Rails Tests](http://tom-clements.com/blog/2011/10/23/4-steps-to-faster-rails-tests/)
+ [Play by Play: Gary Bernhardt](https://peepcode.com/products/play-by-play-bernhardt)

## Conclusions

测试是一种有效保证代码质量的手段，但是如果测试非常慢，我估计大多数人都会丧失测试的热情。所以加快你的测试是非常重要也是必要的一件事情，我觉得一个比较合理的目标是：

> 每个 unit test & functional test 都必须小于 0.5s

我们如果合理利用各种手段，可以让我们的测试变得得又快又健壮。或许能真正做到 Kent Beck 说的：

> I get paid for code that works, not for tests, so my philosophy is to test as little as possible to reach a given level of confidence. 


## 扩展阅读

+ [Writing Sensible Tests for Happiness](http://fredwu.me/post/59395419899/writing-sensible-tests-for-happiness)
+ [这个关于测试的讨论也有很多料](http://ruby-china.org/topics/13574)

enjoy!
