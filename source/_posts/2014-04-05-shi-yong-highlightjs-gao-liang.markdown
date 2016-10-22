---
layout: post
title: "使用 Highlight.js 高亮你的代码"
date: 2014-04-05 18:03
comments: true
category: tech
tags: octopress highlightjs
---

今天给博客换了一个代码高亮插件 highlight.js 可以支持大多数语言，自动识别语言，而且可以根据心情随时切换代码块样式。

<!--more-->

## 第一步

在 head.html 文件中加入：

    <link rel="stylesheet" href="http://yandex.st/highlightjs/8.0/styles/solarized_dark.min.css">
    <script src="http://yandex.st/highlightjs/8.0/highlight.min.js"></script>
    <script>hljs.initHighlightingOnLoad();</script>

## 第二步

完成第一步之后发现效果已经有了，但是多了一块白边，所以手动做了一个 hack，在 sass/parts/_syntax.scss 注释掉一行：

    article{
      ...
      pre{
        ...
        /*padding: 5px 15px;*/
        ...
      }
    }

采用的主题不同，可能处理方式不一样，请参考。

## 第三步

直接使用在代码块之前空 4 个空格来高亮

    initializer "active_record.initialize_database" do |app|
     ActiveSupport.on_load(:active_record) do
       self.configurations = app.config.database_configuration || {}
       establish_connection
     end
    end

## 最后

配合 Mou 写 markdown，非常方便。而且根据喜好随时更换代码块颜色：[highlight.js Style](https://github.com/isagalaev/highlight.js/tree/master/src/styles)


enjoy!
