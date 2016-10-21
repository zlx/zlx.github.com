---
layout: post
title: 关注内存－小心大数组
category: tech
tags: Ruby Memory-leak Array
comments: true
---

Ruby 有垃圾回收机制，以致于大多数 Ruby 开发人员（包括我😫）平时并不关心内存使用情况，所以很有可能不经意间就产出了非常消耗内存或者说是内存不友好的代码。

### 我们都喜欢栗子

现在我们先来看一段测试代码：

<!--more-->

    class Foo
      def large_return
        return mini_return.map { |i| gen_str(i) }
      end

      def large_return_with_freeze
        return mini_return.map { |i| gen_str(i).freeze }
      end

      def mini_return
        return (1...10_000)
      end

      def gen_str(index)
        's' * (100 + index)
      end
    end

    class FooTest
      def test_method_momery(method)
        before = get_memory_usage
        foo = Foo.new
        100.times do |i|
          foo.send(method).each do |i|
            j = yield(i, foo)
            j.gsub(/s{50}/, '').length == 50 ? '.' : 'x'
          end
        end

        after = get_memory_usage
        puts "Memory: #{(after-before) / 1000}M"
      end

      def test_large_return
        test_method_momery(:large_return) {|i| i }
      end

      def test_large_return_with_freeze
        test_method_momery(:large_return_with_freeze) {|i| i }
      end

      def test_mini_return
        test_method_momery(:mini_return) {|i, foo| foo.gen_str(i) }
      end

      def test_another_large_array
        before = get_memory_usage
        foo = Foo.new
        100.times do |i|
          large_array = foo.mini_return.map{ |i| foo.gen_str(i) }
          large_array.each do |j|
            j.gsub(/s{50}/, '').length == 50 ? '.' : 'x'
          end
        end

        after = get_memory_usage
        puts "Memory: #{(after-before) / 1000}M"
      end

      def get_memory_usage
        `ps -o rss= -p #{Process.pid}`.to_i
      end
    end


    t = FooTest.new
    #t.test_large_return  #=> Memory: 241M
    #t.test_mini_return  #=> Memory: 113M
    #t.test_another_large_array #=> Memory: 227M
    #t.test_large_return_with_freeze #=> Memory: 243M


最后面四行注释是代表运行对应方法所占用的内存大小。

对比 `test_large_return` 和 `test_mini_return` 两个方法可以看出，两个相似的实现，内存占用相差了2倍多。两个实现的区别在于 `large_return` 返回了一个大数组，而 `mini_return` 返回了一个 Range 对象。

我们知道 Ruby 的 GC 会自动回收垃圾对象，而垃圾对象则必须没有其它引用。一个方法的返回值会被调用者引用，所以就不能被垃圾回收，从而占用了内存。

### 一句话结论

如果可以选择，尽量避免使用大数组，如果非用不可，请及时消除引用。


### 参考文章

+ [Ruby Uses Memory](http://www.sitepoint.com/ruby-uses-memory/)
+ [How to debug Ruby Memory issue](http://eng.rightscale.com/2015/09/16/how-to-debug-ruby-memory-issues.html)
