---
title: 阻止请求 favicon.ico 图标
slug: blocking-requests-for-the-favicon-icon
date: 2016-11-23 00:00:00+0000
description: favicon.ico 图标用于收藏夹图标和浏览器标签上的显示，如果不设置，浏览器会请求网站根目录的这个图标，如果网站根目录也没有这图标会产生 404。
image: img/stop.jpg
categories:
    - Front-end
tags: 
    - HTML
---

> 出于优化的考虑，要么就有这个图标，要么就禁止产生这个请求。

在页面的 head 区域，加上如下代码实现屏蔽：
```html
<link rel="icon" href="data:;base64,=">
```

或者详细一点
```html
<link rel="icon" href="data:image/ico;base64,aWNv">
```
