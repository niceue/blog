---
title: 在 Linux 上安装 Node.js
slug: Installing-Nodejs-on-Linux
date: 2017-01-05
image: img/nodejs.png
categories:
    - 开发环境
tags:
    - Linux
    - Node.js
---

## 方式一：yum 安装
```bash
# 其中 setup_7.x 中的 7.x 为要安装的版本
wget -qO- https://rpm.nodesource.com/setup_7.x | bash -
yum install -y nodejs
```

## 方式二：编译安装
```bash
wget https://npm.taobao.org/mirrors/node/latest/node-v7.4.0-linux-x64.tar.xz
tar xvf node-v7.4.0-linux-x64.tar.xz
cd node-v7.4.0-linux-x64.tar.xz
./configure
make && make install
```