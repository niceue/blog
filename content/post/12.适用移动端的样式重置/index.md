---
title: 适用移动端的样式重置
slug: style-reset-for-mobile
date: 2017-11-24 22:12:00+0000
image: img/code.jpg
description: 简洁可靠的移动端样式重制方案
categories:
    - Front-end
tags:
    - CSS
---

```css
/**********
 * reset
 *********/
* {box-sizing: border-box; -webkit-tap-highlight-color: rgba(0,0,0,0);}
body,
dl, dt, dd, ul, ol, li,
h1, h2, h3, h4, h5, h6,
pre, code, form, fieldset, legend, input, textarea,
p, blockquote, th, td, hr, button,
article, aside, details, figcaption, figure, footer, header, menu, nav, section {
  margin: 0;
  padding: 0;
  border: 0;
}
/* 默认不要下划线 */
a {text-decoration: none;}
/* 按钮文本不可选 */
button {user-select: none;}
img {vertical-align: middle;}
/* 加载不出来的图片不要显示灰色边框 */
img:not([src]),img[src=""] {opacity: 0;}
ul, ol {list-style: none;}
table {border-collapse: collapse; border-spacing: 0;}
input, select, button, textarea {font-size: 100%; font: inherit;}
html, body {height: 100%; overflow-x: hidden;}
```