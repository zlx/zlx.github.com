---
layout: post
title: "给 Linux 增加虚拟内存"
comments: true
published: false
category: tech
tags: linux
---

***有时候我们会感觉机器内存不够用，但是有没有可用的多余内存条或者内存插口。那么如何增加内存呢？***

首先确保你以 root 用户 或者 sudo 执行以下命令:

### 首先建立一个分区(大小为 1 G)

<!--more-->

    dd if=/dev/zero of=/home/swap bs=1024 count=1024000

### 然后将这个分区变成 swap 分区

    /sbin/mkswap /home/swap

### 接着启用这个分区

    /sbin/swapon /home/swap

现在， free 查看已经可以发现 swap 变大了. 
为了保证计算机重启后 swap 分区依然有效，我们还有第 4 步

### 在 /etc/fstab 文件中

    /home/swap swap swap default 0 0

enjoy!
