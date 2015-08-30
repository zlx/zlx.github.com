---
layout: post
title: "debug your rails app"
date: 2013-01-08 10:10
comments: true
category: tech
tags: rails debug
---

Debug 是一个程序员必备的技能。那么，在 rails 里面如何快速方便地 debug 呢？

*友情提醒：*在 rails 里面，你可以尝试先写测试，再写代码，如此就可以避免漫长的 debug 过程了

<!--more-->

    # debugger
    gem 'debugger'

将上述代码加入你的 Gemfile 文件。

### Rails server

在 controller 里面加入 ***debugger*** 关键字:

    class BlogsController < ApplicationController
    def index
      debugger
      @blog = Blog.new
      @blogs = Blog.all
    end

当你访问到 `http://localhost:3000/blogs` 时候，就会在 rails server 控制台出现如下提示:

    [2013-01-08 10:17:06] INFO  WEBrick::HTTPServer#start: pid=4084 port=3000
    /mnt/soffolk/home/soffolk/work/task/app/controllers/blogs_controller.rb:6
    @blog = Blog.new
    (rdb:1) 

并且，你可以使用很多调试命令，如： `l: 列出当前执行位置， s: 步入， n: 下一步， q: 退出， c: 继续执行， h: 帮助`

### Rails console

在 model 某个方法里面加上 ***debugger*** 关键字:

    class MyTask < ActiveRecord::Base
      def overdue?
        debugger
        !complete && dead_line < Time.now
      end
    end

如此，当你在 console 里面调用 `@task.overdue?` 时候， 就会出现提示： `(rdb:1) ` 如此，你就可以如同在 server 控制台一样使用调试命令了。

### 其它

其它时候，如同测试等都可以加上 ***debugger*** 关键字，那样就可以在执行到相关代码时跳出 `(rdb:1) ` 提示。


# pry

    gem 'pry-rails'

将上述代码加入你的 Gemfile 文件中

当你执行 rails console 之后，就会出现提示：

    Loading development environment (Rails 3.2.8)
    [1] pry(main)> 

然后你就可以使用 pry 那令人兴奋的功能了。help 可以列出你可以使用的命令。

### pry 相关参考
- [pry-rails](https://github.com/rweng/pry-rails)
- [http://pryrepl.org/](http://pryrepl.org/)
- [pry-everywhere](http://lucapette.com/pry/pry-everywhere/)

enjoy!
