---
layout: post
title: "在 OpenSUSE 上面搭建 SSH Tunnel"
date: 2013-01-05 13:31
comments: true
category: tech
tags: opensuse linux ssh
---

现在最新最时髦的翻墙方式是 ssh tunnel。

### 步骤：

- `ssh -D1080 hostname`

- firefox: autoproxy 插件： Preferences -> Proxy Server -> Edit Proxy Server, 找到 ssh -D， 将 Port 修改为 1080 。

- 系统: 在 Network 里面找到 Network Proxy, 设置为 Manual -> Socks Host： localhost 1080

- chrome: 暂时未成功

- OK

### 系统信息:
*opensuse 12.2 64-bit + gnome + firefox*

enjoy!
