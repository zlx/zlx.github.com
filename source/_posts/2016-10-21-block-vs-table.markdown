---
layout: post
title: "Display: block VS Display: table"
date: 2016-10-21 19:38
comments: true
category: tech
tags: css display
referral: true
---


`Display: block` 和 `Display: table` 是两个很相似的样式，但是在计算 table 宽度的时候，他们有一些不一样。

<!--more-->

我们来看这个例子：

```ruby
<html>
<head>
  <title>display: block VS display: table</title>
</head>
<body style='width: 400px;'>
<table class="row footer" style='display: table;width: 100%;'>
  <tbody>
    <tr>
      <td class="wrapper" style='background: yellow;'>
        Column 1
      </td>
    </tr>
    <tr>
      <td class="wrapper" style='background: yellow;'>
        Column 2
      </td>
    </tr>
    <tr>
      <td class="wrapper" style='background: yellow;'>
        Column 3
      </td>
    </tr>
    <tr>
      <td class="wrapper" style='background: yellow;'>
        Column 4
      </td>
    </tr>
  </tbody>
</table>
</body>
</html>
```

这里的 table 使用了 `display: table`

显示效果如下：

![display: table example](/images/display-table.png)

假如把 `display: table` 改成 `display: block`，显示效果就变成了

![display: block example](/images/display-block.png)

### 结论：

***使用 display: table 会根据父元素来计算宽度，display: block 则会根据子元素来计算自身所占用的宽度。***

相关代码：

- http://codepen.io/zlx/pen/bwOqLj
- http://codepen.io/zlx/pen/zKykjx


enjoy!
