---
layout: post
title: SSH 验证
comments: true
category: tech
tags: github ssh
---

多次碰到通过 ssh 验证用户身份的问题，记录下来方便记忆。

* 系统环境： linux 

* 具体步骤：

* 在 linux 下，首先进入家目录下的 .ssh 目录下（如果没有就创建一个），执行 ssh-keygen ，安装提示输入文件名（如 id_rsa ）。

* 然后将 .pub 后缀的文件内容加到你的 ssh 服务器信任列表里。例如 github 上面个人设置里面的 ssh keys 。

* enjoy!

