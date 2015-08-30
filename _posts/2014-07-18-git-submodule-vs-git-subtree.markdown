---
layout: post
title: "Git submodule  VS Git Subtree"
date: 2014-07-18 14:59
comments: true
category: tech
tags: Git Submodule Subtree
---

### 下面就是正文

最近在项目中要包含其它子项目，首先想到 `git submodule`, 但是它所带来的问题很多，经同事建议，尝试 `git subtree`。

关于它的使用，有很多非常好的文章：

+ [非常棒的 SubTree 教程](http://blog.charlescy.com/blog/2013/08/17/git-subtree-tutorial/)
+ [一个真实场景的使用例子](https://gist.github.com/kvnsmth/4688345)
+ [Git Submodule 和 Git SubTree](http://blogs.atlassian.com/2013/05/alternatives-to-git-submodule-git-subtree/)


### 简单 Guide：

1. 刚开始加入子项目时：

    git subtree add --prefix=Vendor/AFNetworking --squash git@github.com:AFNetworking/AFNetworking.git master

2. 切分出相关的提交：

    git subtree split --prefix=Vendor/AFNetworking/ --branch AFNetworking

3. 提交到对应的分支：

    git push git@github.com:kvnsmth/AFNetworking.git AFNetworking:critical-bug-fix

4. 拉取下最新的代码：

    git subtree pull --prefix=Vendor/AFNetworking --squash git@github.com:AFNetworking/AFNetworking.git master

### 结论

+ Git Subtree 能满足大部分子项目需求
+ 对于贡献代码不太友好

enjoy!
