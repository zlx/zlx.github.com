---
layout: post
title: "在rails项目中集成nginx和unicorn"
date: 2013-02-01 15:17
comments: true
category: tech
tags: nginx unicorn rails
---
 
nginx 是一个非常高效的 http server, 而 unicorn 是一个非常高效的 app server。高效开发的 web 框架 rails 搭载上 这两个利器，可谓是如虎添翼。

下面我主要来介绍一下如何在一个 rails 项目里面集成 nginx  和 unicorn， 仅作为配置记录。

<!--more-->

## 首先安装 nginx

    在 opensuse 里面
    sudo zypper in nginx
    
    在 ubuntu 里面
    sudo apt-get install nginx

其它系统使用各自自己的软件管理工具安装即可。 详见 [nginx](http://nginx.com/)

## 配置 nginx

在项目 config 目录下创建文件 `nginx.conf`

    server {
      listen 80 default;
      root /path/to/your/app/directory/public;
      
      try_files $uri/index.html $uri @unicorn;
      location @unicorn {
        proxy_pass http://localhost:3000;
      }
    
      error_page 500 502 503 504 /500.html;
    }

然后执行

    sudo mkdir /etc/nginx/conf.d
    sudo ln -s /path/to/your/app/directory/config/nginx.conf /etc/nginx/conf.d/nginx.conf

并在 `/etc/nginx/nginx.conf` 里面 http 模块下面添加 `include conf.d/*.conf;`


运行

    rails s

    sudo service nginx start


现在访问 http://localhost 就可以访问项目了。
*如果设置合适的 host 还可以实现访问 app.local 来访问项目了。happy?*

## 配置 unicorn

unicorn 是一个非常高效的 app server， 它按照 unix 的哲学来设计，充分利用了 unix 系统的特点，以达到高速响应的目的。

### 首先安装 unicorn
修改项目目录下的 Gemfile 文件, 添加这一行

    gem 'unicorn'

然后运行 `bundle install`

安装之后在 config 目录下创建 unicorn.rb 文件

    working_directory "/path/to/your/app/directory"
    pid "/path/to/your/app/directory" + "/tmp/pids/unicorn.pid"
    stderr_path "/path/to/your/app/directory" + "/log/unicorn.log"
    stdout_path "/path/to/your/app/directory" + "/log/unicorn.log"
    
    listen "/tmp/unicorn.#{appname}.sock"
    worker_processes 2
    timeout 30

然后编辑之前的 buychina.conf 文件, 加上

    upstream unicorn {
      server unix:/tmp/unicorn.#{appname}.sock fail_timeout=0;
    }

并将 `proxy_pass http://localhost:3000;` 修改为 `proxy_pass http://unicorn;`

然后执行

    unicorn -D -c config/unicorn.rb
    sudo service nginx restart

访问 http://localhost 查看是否工作良好。


## 配置 unicorn 开机启动

如果在服务器上，有时候我们会希望我们的服务是开机启动的。

切换到 `/etc/init.d/` 目录下

运行

    curl -o unicorn_init.sh https://raw.github.com/defunkt/unicorn/master/examples/init.sh

运行

    bundle install --binstubs

生成 gem 的可执行命令

编辑 unicorn_init.sh 文件
    APP_ROOT 修改为你的项目目录
    CMD 修改为 `$APP_ROOT/bin/unicorn -D -c $APP_ROOT/config/unicorn.rb`

然后可以将 nginx 和 unicorn_init 设置为你的开机启动 

    sudo chkconfig nginx on
    sudo chkconfig unicorn_init on


## 参考

- [unicorn source code](https://github.com/defunkt/unicorn/tree/master/examples)
- [railscasts nginx-unicorn](http://railscasts.com/episodes/293-nginx-unicorn)
- [nginx + unicorn + performance tweaks](http://vasil-y.com/2012/10/21/nginx-unicorn-performance-tweaks/)
- [Configuring Nginx and Unicorn](http://sleekd.com/general/configuring-nginx-and-unicorn/)


enjoy!
