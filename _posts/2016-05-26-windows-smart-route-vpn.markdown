---
published: true
title: Windows 下如何在 VPN 连断之后触发脚本
layout: post
tags: [Windows, 云梯VPN, 智能路由]
categories: [tech]
---
VPN 是开发者必备的利器，一个好用而且稳定的 VPN 是绝对是给予你很大的帮助，良心推荐 [云梯VPN](http://igetvpn.com/?r=c8892698ee152ac1)。

VPN 是一种全局的代理，我们不需要做任何额外设置就可以实现设备上的任何应用程序的代理。好处显而易见，坏处则是无形中让你正常的国内网页浏览也被记入了流量，VPN 大多数是按照流量和设备数定价，保证正常使用的前提下，流量当然是越省越好。如果我们能够区分国内网站和国外网站，那就可以实现国内外网站分流，从而实现节省流量的目的。

如果你是使用 Windows ，那我今天介绍的办法或许适合你。

<!-- more -->

### 废话结束

今天的主角是 －－ Windows 里面的计划任务（Scheduler Tasks）。

打开计划任务，左上角选择操作（Action），选择创建任务（Create Task）:

![Trigger 01](http://blog.zlxstar.me/images/trigger-01.png)

在接下来的窗口里面输入任务名字（Name），并勾选以管理员角色执行（Run with highest privileges）:

![Trigger 02](http://blog.zlxstar.me/images/trigger-02.png)

然后选择上面的触发器（Triggers）, 选择新建（New），会跳出下面的窗口。

按照下图设置好参数：***特别注意 Source 和 Event ID***， 这个事件是在连上 VPN 以后触发的，这是关键。确定并退出

![Trigger 03](http://blog.zlxstar.me/images/trigger-03.png)

然后选择上面的操作（Actions），在如下图窗口里面选择你要执行的脚本，确认并退出。

![Trigger 04](http://blog.zlxstar.me/images/trigger-04.png)

最后确认退出任务创建窗口。

创建完成后会在任务列表里面出现你刚创建的任务，可以手动执行确定脚本位置正确。你现在可以连接上 VPN 试试了。

对于 VPN 断开之后执行的任务如上类似，只是需要修改之前的 ***Event ID***，如下图所示：

![Trigger 05](http://blog.zlxstar.me/images/trigger-05.png)

完成了以上两个任务的创建之后，你就可以在 VPN 连接和断开之后通过脚本来修改路由表了。

### 说明

以上内容如果对你有帮助，请评论或者转发以支持原作者，或者使用我的小尾巴 [云梯VPN](http://igetvpn.com/?r=c8892698ee152ac1) 购买。