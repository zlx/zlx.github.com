---
layout: post
title: 你需要知道的 linux
category: tech 
comments: true
tags: linux
---

## linux下的软件安装方式：

常见的有三种：

- 二进制安装
在linux下，只需解压就可以运行。这种安装方式优点是安装方便，缺点是一般软件的二进制安装包都是较最新版本落后的。

- 源代码安装
这种安装方式需要自己编译再安装，通常包含：解压（bunzip2,tar），配置安装方式（configure），编译(make)，编译安装(make
install)几个步骤。这种安装通常能获得最新的版本，而且能直接修改源代码来修复错误，但是需要使用人员具备一定的知识基础。

- rpm安装方式
这种安装包多是以rpm为后缀的软件包。这是很多的linux发行版中支持的一种安装方式，rpm工具可以方便地管理软件在安装，卸载，更新，查看状态等。
另外有一个叫做yum的软件安装工具，支持软件的在线下载安装，且在解决软件包依赖方便较rpm工具方便很多，但是，通常你需要指定一下软件库。

<hr />

## linux网络配置

1. 查看ip地址情况：ifconfig
2. 配置:setup
3. 配置完后你需要重启网络，修改才会生效，可以使用service network [restart|stop|start]等命令


_网络配置中涉及到的文件有/etc/network/目录下的一些网卡配置文件和DNS服务配置情况。_

<hr />

## linux下的压缩工具

在linux下，通常使用tar工具来收集并归档文件，gzip，bzip2,lzop等工具来压缩归档文件。

### tar有二十几个选项参数，并且可以支持多参数同时使用，这样就使得它的功能非常强大。

常用的参数组合如下：
- tar czf myfiles.tar.gz *.txt  ##--> 将目录下的所有txt后缀的文件归档到myfiles文件中。
- tar czvf myfiles.tar.gz *.txt ##--> 其它和上面一样，v参数代表显示过程中的详细信息。
- tar xzvf myfiles.tar.gz ##--> 同上一个例子的不同在于这个命令是解压文件到当前目录下，而上面一个是压缩。

<hr />

### gzip工具

其实，tar加上参数可以实现gzip压缩包，如上面的三个例子。
gzip工具的直接使用例子如下：

- gzip myfile  ##--> 压缩myfile文件，命名为myfile.gz
- gzip -v myfile ##--> 压缩myfile文件，并输出详细信息
- gzip -1 myfile ##--> 选择压缩级别1压缩文件(压缩级别越小，压缩速度越快，压缩比越小)

<hr />

### bzip2工具
bzip2能提供较高的分辨率，相对的速度也最慢

- bzip2 myfile  ##--> 压缩文件myfile，命名为myfile.bz2
- bunzip2 myfile.bz2  ##--> 解压缩文件myfile.bz2

基本参数同gzip相同。

<hr />

## VMWare相关

1. 在使用锐捷上网的朋友可以借鉴下：在VMWare下要安装VMWare Tool然后选择NAT上网方式可以与主机共享网络。

2. 在VMWare下切换X-windows图形界面和命令界面可能会遇到一些问题：原因是VMWare的hot-key与切换键冲突：解决方式的修改VMWare的Hot-key。
3. 在fedora 13下测试发现图形界面是ctrl+alt+F1而不是传统的ctrl+alt+F7
