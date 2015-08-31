---
layout: post
title: "让你的浏览器编程便捷的代码编辑器"
date: 2013-01-31 15:04
comments: true
category: tech
tags: 
---

让你的浏览器编程一个便捷漂亮的代码编辑器

<!--more-->

    data:text/html,
    <style type="text/css">
    #e {
    	position:absolute;
    	top:0;
    	right:0;
    	bottom:0;
    	left:0;
    }
    </style>
    <div id="e"></div>
    <script src="http://d1n0x3qji82z53.cloudfront.net/src-min-noconflict/ace.js" type="text/javascript" charset="utf-8"></script>
    <script>
    	var e=ace.edit("e");
    	e.setTheme("ace/theme/monokai");
    	e.getSession().setMode("ace/mode/ruby");
    </script>

enjoy!
