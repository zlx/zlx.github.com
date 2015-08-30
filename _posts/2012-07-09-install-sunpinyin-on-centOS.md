---
layout: post
title: "centOS 安装 sunpinyin 输入法"
comments: true
category: tech
tags: linux sunpinyin
---

近来发现之前好好的 ibus 输入法没有备选项提示了，变得非常难用。推测可能与最近的某次系统更新有关。听朋友介绍 sunpinyin 是一款不错的输入法引擎。而且它可以搭载在 ibus 或者 xcim 等输入法下面，于是决定采用它来解决我目前的问题。

<!--more-->

首先在 yum 库里面搜索了一下，发现还没有，所以开始在网上寻找安装方式。几经搜索之后，竟然发现它是一个开源的输入法。代码托管于 google code （[https://code.google.com/p/sunpinyin/](https://code.google.com/p/sunpinyin/)）。

+ 在下载页面，找到 [sunpinyin-all-in-one-2.0.3.tar.gz](https://code.google.com/p/sunpinyin/downloads/detail?name=sunpinyin-all-in-one-2.0.3.tar.gz&can=2&q=) 。

+ 下载后， 运行 `tar -xf sunpingyin-all-in-one-2.0.3.tar.gz` 解压到 sunpinyin-2.0.3 目录下。然后按照 [sunpinyin-wiki-BuildUnix](https://code.google.com/p/sunpinyin/wiki/BuildUnix) 上面的方法安装。

+ 当安装到 ibus 界面时，出现了 Checking for ibus-1.0... no 的错误，按照上述页面上的对于 ubuntu 系统的相关回复，推测 centOS 可能是 ibus-dev 的类似包，用 yum 查看了一下，果然发现有如下三个可能的包没有安装：ibus-devel ibus-devel-docs scribus-devel 。安装之，问题解决。

+ 安装好之后，我发现我的 ibus 输入法不能进入设置了，尝试重启 ibus ，执行 ibus-setup 。提示了一个 python 的语法错误。回想起之前为了安装某个软件新安装了一个 2.7 的 python 。用 whereis 查看了一下，果然发现安装了多个 python ，并且默认的版本变成了 python 2.7 。执行 `sudo ln -sf /usr/bin/python2.6 /usr/local/bin/python` 强行指定了默认 python 为 2.6 版本。

+ OK，可以正常打开设置页面了，选择 sunpinyin 为默认的中文输入法。

+ enjoy!

*吐槽一句，现在各种 linux 发行版盛行，桌面环境越来越漂亮，操作也逐渐向鼠标操作转移， CentOS 由于它社区维护的性质，保持了它的原汁原味，如果你使用它，那你一定会碰到很多安装问题。不过，这也让我们锻炼了解决问题的能力。我认为在纯文本操作的 linux 难度太高以外，试用一下 CentOS 是一次不错的经历。*
