---
title: 在 Linux 上安装 Python3
slug: Installing-Python3-on-Linux
date: 2017-01-10 00:00:00+0000
image: img/python.jpg
categories:
    - 开发环境
tags:
    - Linux
    - Python
---

## 源码编译方式安装
```bash
wget https://www.python.org/ftp/python/3.5.3/Python-3.5.3.tar.xz
tar xvf Python-3.5.3.tar.xz
cd Python-3.5.3
./configure
make
make install
```

在某些系统环境下make & make install 会报错，分开执行才能安装成功
源码编译安装好后，自带 python3 和 pip3

## 文件头格式
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```
如果不是需要执行的入口文件，第一行也可以不要