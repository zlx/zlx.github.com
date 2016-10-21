---
layout: post
title: "魔法之 Octopress"
date: 2013-04-02 22:43
comments: true
category: tech
tags: git octopress
---

从几个月前开始用 [jekyll](http://jekyllrb.com/)
写博客开始，就没有再登录我的网易博客和blogcn博客了。 `jekyll`
果然是程序员的最爱博客系统。

<!--more-->

## 问题背景

使用 [octopress](http://octopress.org/)
来搭建博客非常方便、快捷。但同时，如果出问题了也非常痛苦。

最近由于要帮女友学程序，所以建了一个博客用来记录她的学习过程：
[金丹的程序之路](http://jindan-programming.github.com/)

这个博客刚开始是我来帮她记录，之后可能会由她自己记录。所以这个需要能够支持多人共同维护，所以我使用的
github 上面的组织功能，把我们都加到这个组里面。

## 问题描述

我先在我的电脑上用 octopress 搭建了一个博客，然后推到 github 上面。

然后我用她的电脑拉下代码，_发现我无法使用 `rake deploy` 部署博客。_

## 原理阐述

这里先插一下 octopress 的基本流程：

1. 先 `rake new_post` 创建一篇新博客

2. 然后，编辑博客

3. 运行 `rake generate` 转化为 html

4. (可选) 运行 `rake preview` 在本地预览一下

5. 运行 `rake deploy` 部署到 github 或者 你的服务器上

6. 然后运行以下命令把你的代码推到 github 上面

    git add .
    git commit -m 'add post xxx'
    git push origin source

其它的我就不说了，如果你查看 `Rakefile` 文件可以发现： 

当你运行 `rake deploy` 时候，它会把 `public` 目录下的文件拷贝到 `_deploy` 目录下，然后在 `_deploy` 目录下执行

    git add .
    git add -u
    git commit -m '.....'
    git push origin master --force

然后就部署成功了。

这里有一个区别是你在项目根目录下是 `push` 到 `source` ，然后在 `_deploy`
目录下是 `push` 到 `master`
。 以前一直没有深究，直到这次出现问题我才去仔细看了一下：原来除了根目录下有一个
`.git` 文件夹以外，`_deploy` 目录下也有一个 `.git`
文件夹。也就是实际上是一个项目的两个分支分别在不同的目录下。这和我们一般的使用习惯不同，所以会让人感觉奇怪。

## 问题根源

而我之所以出现这个错误，是由于我迁下来的项目的 `_deploy` 目录下没有了 `.git`
文件夹，也就是说 `_deploy` 目录不是一个项目的分支。

## 解决方案

明白了原因，解决起来就容易了：

1. 我先在 `_deploy` 文件夹下运行 `git init`

2. 然后将我的电脑上面的 `_deploy` 目录下的 `.git/config` 配置文件拷贝到这边

好，问题解决


enjoy!
