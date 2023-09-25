---
title: 禁止密码表单被自动填充
slug: disable-form-autofill
date: 2015-05-14 00:00:00+0000
description: Chrome 浏览器的自动填充密码功能会带来安全隐患，如何禁用自动填充呢？
image: img/stop.jpg
categories:
    - Front-end
tags: 
    - Chrome
    - Security
---

## 记住密码带来安全隐患

当别人打开你的浏览器，就能直接登录进去。

更进一步，若网站被 xss 攻击者利用，可以轻松获取到你的明文密码信息。攻击者只要构造一个包含用户名和密码的表单插入到 DOM，延时等待浏览器的自动填充，然后就可以获取到密码值。

即使不被 xss 攻击，也可能被他人打开你的浏览器直接获取到明文密码。

## 如何禁止表单自动填充

### 更改浏览器设置
作为个人用户可以考虑禁用这个选项，但作为网站还是得寻求其他方式，我们不能保证用户都更改设置。

### 给表单设置 autocomplete="off" 
实践证明 autocomplete 这个属性可以关掉输入框获得焦点时的下拉选项，但不能禁用表单被自动填充。在 chrome 开发工具中可以看到，当 input 被自动填充后，会有一个 input:-webkit-autofill 的样式，但这个样式不可更改。而且也没有提供一个 autofill 的 DOM 接口来改变这个行为。

### 有效的方式
用 Javascript 动态设置 password 的 type，来绕过浏览器的自动填充。经过实践，下面代码被证明有效果：
```js
const input = document.querySelector('input[type="password"]');
input.setAttribute('type', 'text');
setTimeout(function() {
    input.setAttribute('type', 'password');
}, 500);
```