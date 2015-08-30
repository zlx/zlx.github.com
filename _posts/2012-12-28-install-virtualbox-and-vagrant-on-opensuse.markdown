---
layout: post
title: "安装 VirtualBox 和 vagrant"
date: 2012-12-28 15:53
comments: true
category: tech
tags: VirtualBox vagrant linux
---

系统: opensuse 12.2， kernel 3.4.11-2.10-desktop x86_64

# 整个安装过程： #

<!--more-->

    rpm -i VirtualBox-xxx.rpm

    gem install vagrant

    vagrant box add lucid32 http://files.vagrantup.com/lucid32.box

    vagrant init lucid32

    vagrant up

# 错误处理 #

#### 提示依赖包 libpng12.so.0 找不到

    安装依赖包 libpng12-devel

#### 提示 kernel 模块不存在和文件头找不到

查看系统 kernel 版本号， 安装对应 kernel-devel

如果 当前版本低于最新 kernel-devel 版本

可以使用 `sudo zypper update` 更新内核

enjoy!


