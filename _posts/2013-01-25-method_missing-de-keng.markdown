---
layout: post
title: "method_missing的坑"
date: 2013-01-25 15:04
comments: true
category: tech
tags: 
---

Ruby 里面的 method_missing 绝对是一个创举，有了她，你可以实现一些很动态的内容。

比如说 rails 里面的 find_xxx 方法，就是使用 method_missing 实现的，使用懒定义的方式来定义这些方法（也就是第一次调用时在 method_missing 里面定义，之后直接调用）

如果你想自己实现这些方法，可以这样做：

<!--more-->

    def method_missing(name, *argv, &block)
      # some code
      if # some condition 
        # some code you expect call
      else
        super
      end
    end

并且，通常，你要实现 respond_to? 方法，来保证它的行为一致。

    def respond_to?(name, include_private = false)
      # some code
      if # some condition
        true
      else
        super
      end
    end

OK, 到此为止，世界都还是很美好的，下面我要揭露一下其中的黑暗面了。^-^

## 坑来了

> 如上所说，rails 里面使用 method_missing 来实现一些魔法。

> 于是，问题出现了：

> *如果你在 method_missing 里面调用了某些方法，而在这些方法里面也调用了 method_missing ，那么......邪恶的堆栽溢出出现了..hehe*

> Anyway, 使用 method_missing 千万要小心又小心。。切记，切记！！

### 附上已发现的 rails 里面使用了 method_missing 的方法：

- ActiveRecord::Base#find_xxx

- ActiveRecord::Base#exists?


enjoy!
