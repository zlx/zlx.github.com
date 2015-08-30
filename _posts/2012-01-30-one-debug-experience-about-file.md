---
layout: post
title: 记录一次文件操作 debug 经历
comments: true
category: tech
tags: rails debug
---

今天要自己实现一个文件上传的功能，碰到一个很奇怪的错误。

`ERROR upload file error:Timeout::Error: execution expired`

<!--more-->

最后定位到这里

    client 端：
        upload_file = ActionController::UploadedStringIO.new
        upload_file.write File.new(split_attchment_name).read
        upload_file.original_path = split_attchment_name.split("/")[-1]
        upload_file.content_type = "application/octet-stream"

    server 端：
        params[:upload].read

*** server 端 `params[:upload].read` 读到的结果为 `nil` 。***

找到原因，修改就容易了。
修改后的代码

    client 端：
        upload_file = ActionController::UploadedStringIO.new
        upload_file.write File.new(split_attchment_name).read
        upload_file.original_path = split_attchment_name.split("/")[-1]
        upload_file.content_type = "application/octet-stream"
        upload_file.pos = 0

    server 端：
        params[:upload].read

ok， 任务完成。


Tip: 

`IO.read` 默认是以 `r` 模式来读的
