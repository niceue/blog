---
title: 使用国内 npm 镜像
slug: using-domestic-npm-mirrors
date: 2015-05-31 00:00:00+0000
description: 利用国内 npm 镜像源来加速模块安装
image: npm-cover.png
categories:
    - Front-end
tags: 
    - Node.js
---

## 国内 npm 镜像源：
- npmmirror 镜像源： [https://registry.npmmirror.com](https://registry.npmmirror.com)
- 华为云镜像源：[https://mirrors.huaweicloud.com/repository/npm/](https://mirrors.huaweicloud.com/repository/npm/)


## 修改 npm 镜像源

### 方式一、命令行修改配置
```bash
# 配置 registry
npm config set registry https://registry.npmmirror.com
# 验证配置是否修改成功
npm config get registry
```

### 方式二、添加 .npmrc 配置文件
```bash
registry = https://registry.npmmirror.com
```
`.npmrc` 按以下 4 级逐级查找，可以按需求选择配置的地方
1. 项目目录：/path/to/my/project/.npmrc
2. 用户目录：~/.npmrc
3. 全局配置：$PREFIX/etc/.npmrc
4. 内置配置：/path/to/npm/.npmrc
