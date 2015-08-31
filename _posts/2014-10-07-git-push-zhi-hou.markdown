---
layout: post
title: "Git push 之后"
date: 2014-10-07 16:31
comments: true
category: tech
tags: git gitlab
---


Git 已经深入人心，那 `Git Push` 之后，到底发生了什么呢？今天我们以开源项目 Gitlab 为例，分析一下 `Git Push` 之后，Git 服务器是如何处理请求的。

<!--more-->

### 准备知识

+ [Git](http://git-scm.com/)： 现代代码版本管理工具
+ [GitLab](https://github.com/gitlabhq/gitlabhq): 开源 Git 项目管理工具（自建 Github）
+ [Gitolite](https://github.com/sitaramc/gitolite): 开源 Git 项目中央仓库
+ [GitLab-Shell](https://github.com/gitlabhq/gitlab-shell)： Gitlab 实现的 Gitolite 的替代品


### 开始

当我们创建好一个 Git 项目，编辑了一些文件，第一次尝试把代码提交到远程服务器时, 我们使用了这样的命令：

`git push origin master`

***注：在这之前，我们先说明一下 git 有很多种方式来传输数据，其中 ssh 是一种比较高效并且安全的方式，我们这里主要讲解 ssh 方式背后的故事。***

git 首先收集一些信息，然后向服务器发送了这样的命令：

    ssh -x git@<gitlab-server> "git-receive-pack 'schacon/simplegit-progit.git'"
    005bca82a6dff817ec66f4437202690a93763949 refs/heads/master report-status delete-refs
    003e085bb3bcb608e1e84b2432f8ecbe6306e7e7 refs/heads/topic
    0000

这里注意两点：首先，这是一个 ssh 命令，其次，发送的参数里面出现了一个 git-receive-pack 命令。

假设这个命令发送给了你的 gitlab 服务器，查看你的 gitlab 服务器上面的 .ssh/authorized_keys 文件, 可以看到类似下面的内容：

    command="/home/git/gitlab-shell/bin/gitlab-shell key-149",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AABBB3NzaC1yc2EAAAADAQABAAABAQDloHa5W8MHCZO0VOWL94gHoAtntYhQko5MHZMxPCYUQF1MhZs4TaEqUGldBK+NOhwY18or7QOylIGp7/mLN8XUza0IJqmKnb1NSTYYh2d4r/EmlT9rcsrrH/QEb8O+n4F8jt9Hk0LeaLsYF9aG+VxaybFIXiVA6sXMooUzK+RaEfjQAlsu+hTX1VDu3kZQJ5kQSUtBb1DyveFcsju6e3lSqB24GQqD13DR+GGopS3FuUoDT1UOUzvKowwzWPwQ6Ln+dUr+9LALp4ocj0BW2zCj2z08n8gIxF+4+5zMbQUS35TneW7il01/h7abTZWaAmCUY9++5QlguR+HifvPsssh zlx_star@gmail.com

经过这样的配置，原先的 ssh 命令会变成

    /home/git/gitlab-shell/bin/gitlab-shell key-149

并且携带了 `SSH_ORIGINAL_COMMAND` 这个环境变量，其中存储了客户端传过来的命令：

    "git-receive-pack 'schacon/simplegit-progit.git'"
    005bca82a6dff817ec66f4437202690a93763949 refs/heads/master report-status delete-refs
    003e085bb3bcb608e1e84b2432f8ecbe6306e7e7 refs/heads/topic

继续追踪 /home/git/gitlab-shell/bin/gitlab-shell, 发现其最后调用了 `GitlabShell.new.exec`

    class GitlabShell
      def initialize
        @key_id = /key-[0-9]+/.match(ARGV.join).to_s
        @origin_cmd = ENV['SSH_ORIGINAL_COMMAND']
        @config = GitlabConfig.new
        @repos_path = @config.repos_path
        @user_tried = false
      end

      def exec
        if @origin_cmd
          parse_cmd

          if git_cmds.include?(@git_cmd)
            ENV['GL_ID'] = @key_id

            if validate_access
              process_cmd
            else
              message = "gitlab-shell: Access denied for git command <#{@origin_cmd}> by #{log_username}."
              $logger.warn message
              $stderr.puts "Access denied."
            end
          else
            raise DisallowedCommandError
          end
        else
          puts "Welcome to GitLab, #{username}!"
        end
      rescue DisallowedCommandError => ex
        message = "gitlab-shell: Attempt to execute disallowed command <#{@origin_cmd}> by #{log_username}."
        $logger.warn message
        puts 'Not allowed command'
      end
    end

关键是两个操作， `validate_access` 和 `process_cmd`， `validate_access` 调用 Gitlab 的 API 来完成验证， `process_cmd` 则是转调 `exec_cmd`,

    def exec_cmd(*args)
      Kernel::exec({'PATH' => ENV['PATH'], 'LD_LIBRARY_PATH' => ENV['LD_LIBRARY_PATH'], 'GL_ID' => ENV['GL_ID']}, *args, unsetenv_others: true)
    end

最后可以发现还是调用原先的命令， 但是 gitlab 是如何知道有了新提交，以更新日志等一系列操作？

其实 git 还有一种 hook 的机制，在调用 `git push` 之后，会触发 `post-receive` ，而 gitlab 就是利用了这个 hook： `gitlab-shell` 目录下的 hooks/post-receive 最后发现， 它会将执行的命令保存在 redis 里面：

    def update_redis
      queue = "#{config.redis_namespace}:queue:post_receive"
      msg = JSON.dump({'class' => 'PostReceive', 'args' => [@repo_path, @actor, @changes]})
      unless system(*config.redis_command, 'rpush', queue, msg, err: '/dev/null', out: '/dev/null')
        puts "GitLab: An unexpected error occurred (redis-cli returned #{$?.exitstatus})."
        exit 1
      end
    end

巧妙的是，这里是使用 sidekiq 的 job 格式保存的，所以 gitlabhq 项目里面直接定义了一个 PostReceive 的 worker， 来处理这个请求，并完成 gitlab 更新的一系列操作。之后的故事，就请查看 Gitlabhq 的源代码吧。


### 参考文章

+ [Git Internals - Transfer Protocols](http://git-scm.com/book/en/Git-Internals-Transfer-Protocols)
+ [SSH Authorized_key](http://oreilly.com/catalog/sshtdg/chapter/ch08.html)
+ [Git Hooks](http://git-scm.com/book/en/Customizing-Git-Git-Hooks)

enjoy!
