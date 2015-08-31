---
layout: post
title: "解决 Chrome 的 SSL ERROR"
date: 2012-12-08 21:20
comments: true
category:  tech
tags: linux goagent 
---

作为一个技术宅男，翻墙是一项必备技能。而 goagent 提供了一种非常灵活方便并且免费的方式来达到我们的这个目的（感谢 google 和 goagent 这个项目的相关开发人员)。

好，进入正题。。。

<!--more-->

***首先注意我的系统是 opensuse 12.2 ,不同的 linux 发行版可能有所不同***

### 安装这个工具

    sudo zypper install mozilla-nss-tools

### 然后找到 local 目录下的 CA.crt 文件, 并导入

    certutil -d sql:$HOME/.pki/nssdb -A -t TC -n "goagent" -i /path/to/goagent/local/CA.crt 

OK!


## 参考资料

- [Managing_certificates_with_Google_Chrome](https://www.assembla.com/spaces/confusa/wiki/Managing_certificates_with_Google_Chrome)
- [LinuxCertManagement](http://code.google.com/p/chromium/wiki/LinuxCertManagement)
- [goagent security certificate is not trusted](http://cc.bingj.com/cache.aspx?q=goagent+The+site%27s+security+certificate+is+not+trusted!&d=4598622319608652&mkt=zh-CN&setlang=zh-CN&w=L4On790V2UcrNmU7krg1L1Y2kz4fMb9X)

enjoy!
