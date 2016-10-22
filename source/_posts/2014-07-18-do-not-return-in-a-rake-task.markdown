---
layout: post
title: "不要在 Rake 中直接 Return"
date: 2014-07-18 14:59
comments: true
category: tech
tags: rake ruby
---

###  病灶

假设你有一个 Rake Task，需要记录一些用户信息，并且为了防止重复记录，你加了一段保护代码：

    Task :record_information do
      return if already_check?
      record_information
    end


看上去挺不错，但是一运行。Booom!!!
报错了 `LocalJumpError: unexpected return`

<!--more-->

### 开刀

初看很迷茫，细看悔断肠： ***Rake task is a block not a method.***

明确了原因，解决方案就显而易见了。

1. next

   当你不想继续执行后面的的代码时，经常会使用 next 来结束一个循环或者 block 。那我们这里可不可以使用呢？行不行，试试就知道：

        Task :record_information do
          next if already_check?
          record_information
        end


        bin/rake record_information  # Bingo

2. break

    在 each...do... 的循环类 block 中，我们经常使用 break 来结束，很自然地，那这里能不能用 break 来结束呢？让我们来看看，修改代码如下：

        Task :record_information do
          next if already_check?
          record_information
        end

        bin/rake record_information
        # rake aborted!
        # LocalJumpError: break from proc-closure

    看来不对。

3. 使用 abort/fail/raise

    另外，如果你认为这是异常情况，或许使用 abort/fail/raise 更符合语义。

       Task :record_information do
          abort if already_check?
          record_information
        end

        bin/rake record_information
        # abort 不返回任何信息
        # fail/raise 返回 rake aborted!

4. 将 Task 的封装成方法，直接调用方法

    在网上搜索 Rake 的最佳实践，许多人推荐，将 Task 的内容封装成方法，然后直接调用方法，这样就可以在方法体里面直接使用 return 来返回了，并且还附带了便于测试等一系列附属好处，噢耶！

       Task :record_information do
          dosomething
        end

        def dosomthing
          return if already_check?
          record_information
        end

        bin/rake record_information  # Bingo


### 下药

1. Rake 是一个 block， 并不是一个方法，所以不能直接使用 return 来返回
2. 根据实际情况来使用 next/abort/fail/raise 提早结束 Task
3. 最理想的情况下，保持每一个 Rake Task 都仅仅是调用另一个业务方法，把所有处理逻辑放在那个业务方法里面完成


### 资料

+ http://stackoverflow.com/questions/2316475/how-do-i-return-early-from-a-rake-task
+ http://readruby.io/closures


enjoy!
