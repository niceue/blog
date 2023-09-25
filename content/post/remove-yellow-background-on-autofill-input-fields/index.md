---
title: 去除 Auto-filled 输入框上的黄色背景
date: 2014-10-25 00:00:00+0000
description: 在 Chrome，当输入框自动填充值后会变成黄色背景，而且无法覆盖默认CSS样式
image: cover.webp
tags: 
    - CSS
categories:
    - Front-end
---

在 Chrome，当选择了输入框的 autocomplete 值后，输入框会变成黄色背景。同时你会发现在 user agent 样式中有如下样式：

```css
input:-webkit-autofill, textarea:-webkit-autofill, select:-webkit-autofill {
    background-color: rgb(250, 255, 189);
    background-image: none;
    color: rgb(0, 0, 0);
}
```

大多数童鞋都会采取覆盖这个默认样式的思路，甚至加 !important，结果都是无济于事。

难道真的不能破？

其实，在 [stackoverflow](http://stackoverflow.com/questions/2338102/override-browser-form-filling-and-input-highlighting-with-html-css) 上早有人问这个问题了。比较靠谱的是采用内阴影强制覆盖背景：

```css
input:-webkit-autofill,
input:-webkit-autofill:focus {
    box-shadow: 0 0 0 100px white inset;
    -webkit-text-fill-color: #333;
}
```

color 的设置同样不能覆盖，但是可以用 -webkit-text-fill-color 设定文字填充颜色的方式解决，所以上面的两行代码都是必须的，你可以根据自己的实际情况修改颜色值。还有需要注意的是，focus 的样式设置也是必须的。

最后，其实还有更简单的办法，如果你不需要 autocomplete 功能，那么关掉就不会有这个烦恼了。

```html
<form autocomplete="off">
```
  