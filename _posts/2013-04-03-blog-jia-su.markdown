---
layout: post
title: "博客加速"
date: 2013-04-03 20:50
comments: true
category: tech
tags:  blog
---

自从迁移到 [Shashank Mehta 开发的主题](http://shashankmehta.in/archive/2012/greyshade.html) 之后，访问博客一直很慢，虽然我的博客访问量不大，但是谁都不会喜欢一个访问起来很慢的网站。

<!--more-->

用 `chrome` 的开发者工具查看了一下访问的请求，按照时间排序。

![sorted request](http://blog.zlxstar.me/images/sorted_request.png)

1. 其中排在首位的是向tweet请求数据，因为国内的环境，这个访问及其缓慢甚至无法连接。砍掉！！

2. 其次，排在次席的是 disqus 这个评论系统的 js，以我目前的网络情况，这个请求大约需要 7 秒，限于目前我的博客里面的评论几乎是摆设，砍掉！

3. 然后，就是我的主页了，其中 5 秒的时间里面有 3 秒多都在等待服务器处理请求，所以这块主要是 github 处理请求以及 sinatra 的时间，暂时不优化。

4. 排在第四位的是我的一张头像的图片，需要将近 5 秒钟的时间，请求一张图片需要 5 秒钟？ 仔细看一下，我这张图片是一张 500x375 的图片，实际上头像只需要 160x120 的图片。直接缩放到 160x120 。

之后的请求都在 3 秒及以下，这次暂且不处理。

另外，我还将 jquery.min.js 下载到本地。

处理之后。新的请求访问时间为：

![new request](http://blog.zlxstar.me/images/new_request.png)


enjoy!
