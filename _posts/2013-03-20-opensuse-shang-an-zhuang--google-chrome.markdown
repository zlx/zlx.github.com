---
layout: post
title: "opensuse 上安装 google chrome"
date: 2013-03-20 08:06
comments: true
category: tech
tags: linux chrome 
---

今天又装了一个 opensuse 的 64 版系统，再次遇到 google chrome 无法安装的问题，特此记录一下。

<!--more-->

## step 1

首先去[官网]()下载一个 google chrome 的 opensuse 64 版 

## step 2

命令行执行 `sudo rpm -i google-chrome-xxx.rpm`

## 错误

安装时提示  `lsb > 0 is needed`

解决方案： 直接使用 `sudo zypper -y install lsb` 安装这个依赖包然后再次安装即可。

enjoy!
