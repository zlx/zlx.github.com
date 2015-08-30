---
layout: post
title: "搭建 openvpn"
comments: true
category: tech
tags: linux openvpn
---

今天刚到手 linode 的一个 VPS 。立马着手搭建 VPN 。

从 [rubychina](http://ruby-china.org) 社区得到一些建议，决定搭建 openvpn 。

<!--more-->

按照 [linode官网帮助](http://library.linode.com/networking/openvpn/centos-5) 建议，搭建好了 openvpn 。

WARNING: *dnsmasq 部分不起作用，配置了以后反而导致不能联网，去掉之后能正常联网，但是想一些国外网站（例如： [twitter.com](https://twitter.com) [facebook.com](https://facebook.com) ）依然不能使用。初步估计可能是官网提供的 DNS 服务器有问题。正在进一步研究。*
