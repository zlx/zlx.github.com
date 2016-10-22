---
layout: post
title: Rails 2 中的 flash, render 和 redirect
comments: true
category: tech
tags: rails
---

最近老是碰到flash信息问题，所以花时间总结一下，记录如下：

<!--more-->

### flash
官方的介绍如下：

> The flash is a special part of the session which is cleared with
each request. This means that values stored there will only be
available in the next request, which is useful for storing error
messages etc. It is accessed in much the same way as the session,
like a hash. Let’s use the act of logging out as an example. The
controller can send a message which will be displayed to the user
on the next request。

同时，在 rails 经常用到 render 和 redirect_to 这两个方法来实现页面跳转，现在把他们区别一下。

render 意为渲染，常用于模板的渲染（如局部模板），每个 controller 里面的 public 方法默认使用 render 方法渲染。

redirect_to 意为重定向，表示终止当前请求，重新开始一个请求。

而 flash 中的内容是在 the next request 有效，也就是说，一次重定向后，flash信息有效，再次重定向后就无效了。在 rails 中，remote 类方法，底层都是转化为 ajax 异步调用的，多用 render 来渲染模板，这样是不会发起重定向的，所以 flash 信息是不会丢失的。

同时，如果你的程序中需要让 flash 信息在多次重定向后有效，可以使用 `flash.keep` 来保持 flash 中的信息不丢失。

有关于flash，render，redirect_to的更详细信息，可以参考
[Api Dock](http://apidock.com/rails/ActionController/Redirecting/redirect_to)
