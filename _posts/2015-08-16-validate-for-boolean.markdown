---
layout: post
title: "不要校验布尔型"
date: 2015-08-16 14:22
comments: true
category: tech
tags: rails validation
---

一个小坑

<!--more-->

### 重现

有这么一个对象

    class Article
      attr_accessible :can_hidden

      validates :can_hidden, presence: true
    end

现在我创建一个不能被隐藏的文章: `Article.create!(can_hidden: false)`

Boom!!

    ActiveRecord::RecordInvalid: 验证失败: can_hidden 不能为空

### 根源

追踪 PresenceValidator 可以发现问题出在这里:

[https://github.com/rails/rails/blob/127411fdf3a3470e8830abf0c7876db67c0c344a/activemodel/lib/active_model/errors.rb#L255](https://github.com/rails/rails/blob/127411fdf3a3470e8830abf0c7876db67c0c344a/activemodel/lib/active_model/errors.rb#L255)

### 解决办法

不要对 boolean 类型做 presence 校验，设置默认值

### 其它讨论及解决方案

1. [rails/rails#2521](https://github.com/rails/rails/pull/2521)
2. [rails/rails#20343](https://github.com/rails/rails/issues/20343)
3. [rails/rails#7508](https://github.com/rails/rails/issues/7508)
4. [rails/rails#6953](https://github.com/rails/rails/issues/6953)


enjoy!
