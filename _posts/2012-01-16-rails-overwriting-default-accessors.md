---
layout: post
title: overwrite default accessors in rails
comments: true
category: tech
tags: rails activerecord
---

在 rails 中，所有的字段都会自动生成读写方法。但是，有时候，你想自己构造出这些方法时，你可以重写这些方法：（与字段名相同的是读方法，多一个=的是写方法），然后，你可以通过 `read_attribute(:attribute)` 来获得字段值，通过 `write_attrbute(:attribute, value)` 来写字段值。

<!--more-->

Example:

    class Song < ActiveRecord::Base
        # Uses an integer of seconds to hold the length of the song
        def length=(minutes)
          write_attribute(:length, minutes.to_i * 60)
        end
        def length
          read_attribute(:length) / 60
        end
    end

*通过这种方式，你可以简化一些表单处理，将 controller 里面的逻辑移到了 model ，清除了部分 bad smell 。*

另外，你还可以用

    self[:attribute]
    self[:attribute]=(value)

来代替

    read_attribute
    write_attribute

最新资料请参考 [rails api](http://rails.rubyonrails.org/classes/ActiveRecord/Base.html) 
