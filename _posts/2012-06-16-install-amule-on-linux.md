---
layout: post
title: install amule on linux
publish: false
comments: true
category: tech
tags: linux amule
---

最近想玩一下使命召唤 8 ，网上找了一下，发现大多数资源都是使用迅雷下载的，资源最好的是 verycd ，但是他也只提供迅雷或者电驴下载。由于我最近开始向 linux 系统转移，所以就想能不能在 linux 下载下来。

先晒一下我的系统

<!--more-->

    uname -a
    Linux 74-e5-0b-d2-3b-7a.connectify 2.6.32-220.el6.i686 #1 SMP Tue Dec 6 16:15:40 GMT 2011 i686 i686 i386 GNU/Linux
    
    CentOS 发行版

linux 下面的下载工具比较贫乏，使用最大的就是浏览器自带下载和命令行下的 `wget` ，但这两个工具的最大问题是不支持多线程下载和断点续传。这两大功能对于我们来说是非常重要的。我检索了一下，很快发现了两个好东西： `axel` 和 `aMule` 。

`axel` 通过打开多个 HTTP/FTP 连接来将一个文件进行分段下载，从而达到加速下载的目的。这个工具直接在 yum 库有，直接安装，找了一个链接测试了一下，不错，开 20 个连接，下载速度飞快。但是它只能对于一般下载文件有用，对于网上大部分的迅雷下载链接等等就无能为力了。我仔细观察 verycd 上面的下载链接，发现它们都是以 ed2k 开头的，维基百科一下 ed2k ，了解了它是另外一种指示文件存放路径的方式，同时也知道了人工把它翻译成普通的 http 或者 ftp 形式的 url 是基本不可能的，也就是说我想把它转化为 `axel` 能处理的可能就没有了。然后我只好转向 `aMule` ：（这里吐嘈一下：鉴于国内特殊的网络环境，查找软件真是痛苦无比。）下面说明一下我成功安装的经验：

1. 我是参照这篇文章的（ [http://os.51cto.com/art/201003/191858.htm](http://os.51cto.com/art/201003/191858.htm) ）

2. 首先到 sourceforge 下载到最新版本的 aMule 源代码（这里不提供链接，因为链接很容易失效，只要到网站去搜索一下即可，一般 sourceforce 只保存最新版本），我当时的最新版本是 2.3.1 。

3. 其次确保 gcc-c++, gtk2-devel, gd-devel, libupnp, libupnp-devel, cryptopp, cryptopp-devel 都已将安装好，这些在 yum 的仓库里面都有。

4. 然后下载 wxWidgets ，（这里需要注意：这个和 aMule 可能会有依赖关系，我当时据需要 wxWidget 至少是 2.8.12 版本，而 2.8.12 在官网上不能下载，原因你懂的，所以我只好到 sourceforge 里面下了一个 2.9.3 的开发版的源代码）。然后配置安装。(这里建议直接编译安装即可，如果需要查看安装路径，可以使用`rpm -ql package_name`，如果需要卸载，请参照（[How_to_uninstall_wxWidgets](http://wiki.amule.org/index.php/How_to_uninstall_wxWidgets)）。

5. 最后配置安装 `aMule` 。

6. enjoy。
