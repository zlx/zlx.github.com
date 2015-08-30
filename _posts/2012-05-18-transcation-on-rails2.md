---
layout: post
title: rails2 的事务
comments: true
category: tech
tags: rails 事务
---

事务是一个很普遍的概念，它是要求一些相关的事件实现原子性，即 要么全部发生，要么全部不发生，这就像是一个选择题，要么是 0，要么就是 1，没有第三种选择。这在很多地方都至关重要，尤其像银行转账问题。

而在 rails 里面，同样提供了他的方式来实现事务，即 `ActiveRecord::Base#transaction`

<!--more-->

查看它的源码，很容易发现它其实只是调用底层数据库的实现。

以 mysql 为例， 它会在 block 之前加上 `begin` ， 再之后 block 之后加上 `commit` 或者 `rollback` 。 

    def transaction(start_db_transaction = true)
      transaction_open = false
        begin
          if block_given?
            if start_db_transaction
              begin_db_transaction
              transaction_open = true
            end
            yield
          end
        rescue Exception => database_transaction_rollback
          if transaction_open
            transaction_open = false
            rollback_db_transaction
          end
          raise unless database_transaction_rollback.is_a? ActiveRecord::Rollback
        end
      ensure
        if transaction_open
          begin
            commit_db_transaction
          rescue Exception => database_transaction_rollback
            rollback_db_transaction
            raise
          end
        end
      end
    end

仔细看这段源码你就会发现一个有趣的变量 `transaction_open` ,在进入 `transaction` 时，它会被设置为 `false`，并在开始执行 block 之前设为 `true`，如果发生了异常，他会判断 `transaction_open` 这个变量是否为 `true` ，再执行 rollback 的操作。否则，就执行 commit 操作。

这里如果你使用了这种

    User.transaction do
      User.create(:username => 'Kotori')
      User.transaction do
        User.create(:username => 'Nemu')
        raise ActiveRecord::Rollback
      end
    end

来试图完成事务的嵌套，那很可能就会发生一些你意想不到的错误。

原因在于在内层事务异常退出时，内层事物会 rollback ，但是被捕获了并且不再往外抛。

这种情况在 rails2.3 之后有所改善。

还有一点，如果 transaction 里面的异常被捕获了，那 transaction 就不能知道发生了异常，也就不会 rollback 了，并且 rollback 只针对于数据库操作而言。

明白了这些，你就会发现 transaction 并不是银弹，并不是什么东西套上了 transaction 就可以保住数据一致了。特别是外层的 transaction ，非但不一定能保住数据一致性，有时候反而恰恰相反，破坏了数据一致性。

比如涉及到第三方系统和外部IO的读写，一旦出错，数据库 rollback 了，外部系统或者 IO 没有恢复，导致了数据的不一致性。 而且这类错误很难排查，估计只能去翻阅庞大的 log 了。

*所以，虽然事务是一个好东西，但也不能乱用，滥用，只有合理的使用，才能发挥它巨大的价值。*
