---
layout: post
title: "CentOS 安装 phpmyadmin"
published: false
comments: true
published: false
category: tech
tags: linux mysql
---

对于一般开发者而言，数据库客户端还是很有必要的。Linux 下面的数据库客户端不多，我选择安装的是 phpmyadmin 。

# 其实安装很简单。 #

<!--more-->

## 安装 phpmyadmin 

如果你只是想用一下，那直接运行 `yum install phpmyadmin` 就可以了。

如果你希望那个体验最新版本，可以到 [phpmyadmin官网](http://www.phpmyadmin.net/home_page/index.php)下载源码包。并编译安装。

## 安装 apache

另外，你还需要安装 apache 服务器，直接执行 `yum install httpd` 就可以了。

## 配置 apache

打开 /etc/httpd/conf/http.conf 和 /etc/httpd/conf.d/phpmyadmin.conf 文件。

下面有两种做法：

1. 将 phpmyadmin 安装目录移到 /var/www/html 目录下（这是 apache 的默认网站根目录，一般在 /etc/httpd/conf/http.conf 里面配置）。

2. 在 /etc/httpd/conf.d 目录下创建一个 phpmyadmin.conf 文件，然后加入 

``` sh
<Directory "/path/to/your/phpmyadmin/dir">
  Order Deny,Allow
  Deny from all
  Allow from localhost
  Allow from 127.0.0.1
</Directory>
```

其中 /path/to/your/phpmyadmin/dir 是你的 phpmyadmin 安装路径，*但是，一般这些在安装的时候会帮你做好, 不过查看一下会比较放心 ^-^*

NOTE: 较早的版本配置一般都在 /etc/httpd/conf/http.conf 里面，现在一般都是在 /etc/httpd/conf.d 目录下的不同文件。

***但是，千万要注意，你的系统上一定要安装 php***

这个问题导致我痛苦了很久。没安装的表现是: php 命令找不到，可以查看 phpmyadmin 的目录结构（php文件不会被解析）。

说了很多废话，其实，最简单的安装方法是

- 安装 apache : `sudo yum install httpd`

- 安装 php : `sudo yum install php php-common php-mysql` #其它你认为会用到的东西

- 安装 phpmyadmin : `sudo yum install phpmyadmin`

- OK！ 打开 `http://127.0.0.1/phpmyadmin` 输入用户名: `root` 和 `` 登陆。

注意，这只是我的安装过程，所以与大家可能会有些不同，可以参考，不可照搬。另外需要考虑 

- 数据库用户名密码

- phpmyadmin 目录访问权限

- phpmyqdmin 验证方式为 http

enjoy!
