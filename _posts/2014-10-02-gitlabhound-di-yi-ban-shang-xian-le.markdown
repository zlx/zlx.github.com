---
layout: post
title: "gitlab-hound 上线了"
date: 2014-10-02 23:52
comments: true
category: tech gitlab hound
tags: 
---

Gitlab-Hound 上线了。

正如你说看到的，Gitlab-Hound 抄袭自 [Hound](https://github.com/thoughtbot/hound)，由于原项目只支持 Github ，考虑到很多国内团队使用自己搭建的 Gitlab 作为代码仓库，所以有了这个项目。

<!--more-->

### 目前 Gitlab-Hound 已有的功能：

1. 支持同时检测多个项目
2. 支持创建新 MR，触发代码规范的检查
3. 支持已有 MR 提交新代码，触发代码规范检查
4. 支持 Ruby 代码规范的检查
5. 支持自定义 Ruby 代码规范


### 安装方式

1. 从 [Gitlab-Hound](https://github.com/zlx/Gitlab-Hound) 下载最新代码
2. 将 config/database.yml.example 拷贝到 config/database.yml 并修改为你的数据库账号和密码
3. 将 config/secrets.yml.example 拷贝到 config/secrets.yml, 并修改为你的配置，其中
    + 本地使用 bundle exec rake secrets 生成一个 secret, 替换掉现有的 secret_key_base
    + 在你的 Gitlab 上面创建一个功能强大的账号，可以管理你需要检测的项目（master 或者 owner）,将用户名和私钥填在 main_username 和 main_private_token
    + 在你的 Gitlab 上面创建一个专门用来检查代码规范的账户，将用户名和私钥填在 comment_username 和 comment_private_token
    + 该项目需要使用 Sidekiq 来执行后台任务，所以需要你配置一个 redis_url
    + 另外，我们会使用 Gitlab 的 Web Hook 机制来获取通知，所以需要配置一个 hook_base_url, 确保该地址可以通过你的 Gitlab 访问到。一般情况下，如果你的 Gitlab-Hound 可以从外网访问，直接使用你的外网域名地址即可
    + 将你需要检测的项目全称填写到 active_repos 下面，切记是 git@github.com:gitlabhq/gitlabhq.git 里面的 ***gitlabhq/gitlabhq*** 部分

4. 安装依赖 `bundle install`
5. 构建数据库 `bundle exec rake db:setup`
6. 启动服务器 `bundle exec unicorn_rails -c config/unicorn.rb -D`
7. 启动 Sidekiq 服务 `bundle exec sidekiq -e production -d`


*** 注意事项：***

另外该项目使用了添加评论的 API，现有 Gitlab API 不能给对应行添加 Comment， 所以需要打上我这个 [patch](https://github.com/gitlabhq/gitlabhq/pull/7839) ，这是我给 gitlabhq 提交 PR， 如果大家觉得有用，可以在该 PR 下面支持一下，早日合并到主版本。


### 最后，

Gitlab-Hound 还处于非常初期的阶段，功能非常简陋，代码也非常粗糙，欢迎大家多多贡献力量。


enjoy!
