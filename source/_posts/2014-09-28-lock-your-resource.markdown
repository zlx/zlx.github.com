---
layout: post
title: "资源加锁"
date: 2014-09-28 22:53
comments: true
category: tech
tags: ruby lock
---

在 Rails 当中，经常需要将某些任务作为定时任务执行，而对于系统的定时任务而言，到点就启动一个进程来处理，相互之间是独立的，这就有可能导致某一些进程同时操作某个资源，有可能导致发生出现竟态，而导致一些问题。

通常的一个思路是通过一些外部的标志来达到加锁的作用，比如说文件。

<!--more-->

来看一段代码：

    lock do
      # handle the limited resource
    end

这里的 `lock` 就是一个加锁和解锁的过程。

lock 的一个简单实现是：

    def lock
      lock_file = "resource.lock"
      return if File.exists? lock_file
      FileUtils.touch lock_file

      yield
    ensure
      FileUtils.rm_rf lock_file
    end

还有一种更强大更优雅的做法:

    def lock(timeout = 10)
      File.open(lock_file, "w+") do |f|
        begin
          f.flock File::LOCK_EX
          Timeout::timeout(timeout) { yield }
        ensure
          f.flock File::LOCK_UN
        end
      end
    end

**注意**

- 其中 `File::LOCK_EX` 是一个排他锁，在持有这个锁期间，只能有一个进程操作该文件。`File::UN` 代表解锁。
- 另外，这里还做了一个超时判断，来防止进程死锁。

enjoy!
