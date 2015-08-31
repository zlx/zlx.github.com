---
layout: post
title: "如何让 vim 与 系统共享剪切板"
comments: true
category: tech
tags: linux vim
---

## vim require

+ vim 7.3 or greater

<!--more-->

## vim compile

+ ./configure --with-features=big --enable-cscope

## step

+ add `set clipboard=unnamed` to /etc/vimrc or ~/.vimrc

## how to use

+ use `"+yy` to copy current line ===> use `p` to paste in vim and `ctrl+v` to paste outside vim

+ use `ctrl+c` to copy from somewhere outside vim  ===> use `p` to paste in vim

enjoy!
