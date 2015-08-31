---
layout: post
title: "Firefox 的坑"
comments: true
category: tech 
tags: form chrome firefox
---

今天遭遇一个 js 的兼容 bug 。

<!--more-->

    $('.delete').live('click', function(){
        if(confirm('Are you sure?')){
        var action = $(this).is('a') ? 'href' : 'act';
        var form = $('<form>').attr({
            action: $(this).attr(action),
            method: 'post'
        }).
        append('<input type="hidden" name="_method" value="delete"/>').
        append('<input type="hidden" name="authenticity_token" value="'+auth_token()+'"/>').hide();
            form.submit();
        }
        return false;
    });

以上这段代码将页面里面所有 class 为 delete 的链接增加一个 RESTful 形式的 delete 请求。

*但是这段代码在 firefox 里面并不工作*

修改成如下形式

    $('.delete').live('click', function(){
        if(confirm('Are you sure?')){
        var action = $(this).is('a') ? 'href' : 'act';
        var form = $('<form>').attr({
            action: $(this).attr(action),
            method: 'post'
        }).
        append('<input type="hidden" name="_method" value="delete"/>').
        append('<input type="hidden" name="authenticity_token" value="'+auth_token()+'"/>').hide();
        form.appendTo('body');
            form.submit();
        }
        return false;
    });

happy it works.

enjoy!
