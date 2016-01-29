---
layout: post
title: "TL-WR703N 路由器上安装并更新 OpenWRT"
date: 2012-11-30 21:20
comments: true
category: tech
tags: linux openwrt
---

wr-tp703n 搭载 openwrt 果然是神机。

首先从 [openwrt官网](http://downloads.openwrt.org/snapshots/trunk/ar71xx/) 上面下载对应的 factory.bin 文件

登录路由器，在系统升级页面升级安装.

<!--more-->

...数分钟后

+ `telnet 192.168.1.1` 登录上你的新系统

+ `passwd`  修改你的 root 密码 

+ `ssh root@192.168.1.1` ssh 方式登录系统

+ `vi /etc/config/network`  修改你的网络配置

+ `/etc/init.d/network restart` 重启网络

+ `opkg update` 更新软件包

+ `opkg install luci` 安装 luci

+ `/etc/init.d/uhttp enable`  启用 luci

+ `/etc/init.d/uhttp start`  开启 luci

如果发现系统版本过低，可能需要升级，这时候可以去之前的下载页面找 sysupgrate.bin 文件升级。升级方法详见[官方文档](http://wiki.openwrt.org/doc/howto/generic.sysupgrade)

Tip:

### 如果发生配置错误无法连接 openwrt 的情况。

1. 可将路由器重启，并迅速连击 reset 键，直至指示灯联闪。
2. 然后用网线连接你的电脑和路由器(注意:电脑要开启 eth0 连接)。这样可以 telnet 上你的路由器。然后 firstboot 还原设置。
3. 退出重启路由器后就可以正常登录了。

### 相关参考

- [openwrt 官网](https://openwrt.org/)
- [openwrt 中文技术网](http://www.openwrt.org.cn/bbs/forum.php)
- [WR703N 如何安装 openwrt](http://riverslee.com/2012/wr703n-openwrt/)
- [恩山论坛-openwrt专版](http://www.right.com.cn/forum/forum-72-1.html)
