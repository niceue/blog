---
title: 在 Electron 使用 Node.js 原生模块
slug: Using-Node.js-Native-Modules-in-Electron
date: 2018-03-28 16:43:00+0000
image: img/electron.webp
description: 使用 Node.js 原生模块，扩展 Electron 能力
categories:
    - Front-end
tags:
    - Electron
    - Node.js
---


Node.js 原生模块是用 C++ 编写的 Node.js 扩展。C++ 源码通过 [node-gyp](https://github.com/nodejs/node-gyp) 编译为 .node 后缀的二进制文件（类似于 .dll 和 .so）。在 Node.js 环境中可以直接用 require() 函数将 .node 文件初始化为动态链接库。一些 npm 包会包含 C++ 扩展，例如： [node-ffi](https://github.com/node-ffi/node-ffi)、[node-iconv](https://github.com/bnoordhuis/node-iconv)、[node-usb](https://github.com/tessel/node-usb)，但都是源码版本，在安装后需要编译后才能被 Node.js 调用。

Electron 同样也支持 Node 原生模块，但由于和官方的 Node 相比使用了不同的 V8 引擎，如果你想编译原生模块，则需要手动设置 Electron 的 headers 的位置。

## 环境准备
不管是 Node.js 环境或是 Electron 中使用原生模块，都需要准备一个编译工具 [node-gyp](https://github.com/nodejs/node-gyp)。我们这里使用的是 windows 环境开发，参考 node-gyp 的安装说明还需要安装 [windows-build-tools](https://github.com/felixrieseberg/windows-build-tools)。

用管理员权限打开 CMD 或 PowerShell 窗口，运行以下命令：
```shell
npm i -g node-gyp
npm i -g --production windows-build-tools
```
windows-build-tools 安装时间可能会长一点，要耐心等待。

## 项目配置
在安装 npm 模块之前还要设置一些环境变量，建议在项目目录下放一个 `.npmrc` 文件，内容如下：
```yaml
registry=https://registry.npm.taobao.org
NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron
arch=ia32
target_arch=ia32
msvs_version=2015
disturl=https://atom.io/download/electron
runtime=electron
target=1.8.4
build_from_source=true
```

参数说明：
- registry - 配置 npm 包镜像
- NODEJS_ORG_MIRROR - 配置 Node.js 头文件下载镜像
- ELECTRON_MIRROR - Electron 下载镜像
- disturl - Electron 头文件镜像（用了 electron-rebuild 模块才需要）
- arch、target_arch - 根据目标环境定义为 ia32 或 x64

## 编译
通过以上配置不需要 [electron-rebuild](https://github.com/electron/electron-rebuild) ，直接用 npm 或 yarn 安装新原生包的时候，会自动编译为适用当前 electron 版本的原生模块到 `{module_name}/build/Release/xxx.node`。由于这个构建后的路径是动态的，`node-ffi` 等第三方模块会使用 [bindings](https://github.com/TooTallNate/node-bindings) 去动态找到这个 .node 文件。bindings 的原理是，首先定位到当前包的目录，然后通过预设一些搜寻路径，一个个尝试读取，直到找到为止。

## 项目构建

### 问题：找不到 .node 文件
bindings 的搜寻方式在 js 源码未压缩的情况下当然没问题，但我们的项目中通常还使用了 webpack。
在开发模式下能够找到 node_modules 下的文件也没有问题，构建到生产环境后就没有 node_modules 了，而且 webpack 也不支持打包动态路径的文件。我想到两种解决方案：

方案一、将 node-ffi 拷贝一份修改 bindings 为写死路径，当然每个用到 bindings 的包都要修改。
```javascript
// ffi 下的 bindings.js
module.exports = require('bindings')('ffi_bindings.node')
// 改为
module.exports = require('../build/Release/ffi_bindings.node')
```

方案二、自己实现一个 bindings 映射，并利用  webpack 的 alias 功能替换 bindings 模块。

1. 增加 bindings.js
```javascript
function bindings (name) {
  if (name === 'ffi_bindings.node') {
    return require('ffi/build/Release/ffi_bindings.node')
  }
  if (name === 'binding') {
    return require('ref/build/Release/binding.node')
  }
}

module.exports = bindings
```
2. 配置 webpack 别名
```javascript
// webpack.main.config.js
module.exports = {
  ...,
  resolve: {
    extensions: ['.js', '.json', '.node'],
    alias: {
      // 如果用 bindings 包，就会找不到 .node 模块，这里替换成自己的实现
      'bindings': path.resolve(__dirname, '../addons/bindings.js')
    }
  },
  ...
```

显然方案二更好一点，只需要自己实现一个 bindings.js，而不用去动第三方包的源码，所以我们直接用了方案二。

### 问题：打包 asar 后，提取出 .node 文件
Electron 官方文档[应用程序打包](https://electronjs.org/docs/tutorial/application-packaging)有说明，二进制文件不要在 asar 中执行，需要 unpack 出来。我们用了 [electron-packager](https://github.com/electron-userland/electron-packager) 可以通过 [asar](https://github.com/electron-userland/electron-packager/blob/master/docs/api.md#asar) 参数配置：
```javascript
  asar: {
    unpack: '*.node'
  }
```
这样，会把 .node 文件都提取到 `app.asar.unpacked` 目录。但是它只负责提取并不会自动更新 .node 文件的访问地址到新的路径。所以我想到了用 webpack 的 `file-loader`：
```javascript
// webpack.main.config.js
module: {
  rules: [
    {
      test: /\.node$/,
      loader: 'file-loader',
      options: {
        name: '[name].[ext]',
        outputPath: 'addons',
        publicPath: '../../app.asar.unpacked/addons'
      }
    }
  ]
}
```
我 app 打包后的项目结构是这样的
```
app.asar.unpacked
app
---main
---renderer
---package.json
```
JS 是在 main 目录的主线程里面访问原生模块的，打包出来的路径也是这样。所以相对路径是要向上两级找到 app.asar.unpacked 目录。

运行一下 `npm run build` 打包。

![smile.png](img/emoji/smile.png)

文件按我预期那样生成好了!

好的，运行一下程序：
> TypeError: Cannot read property 'int64' of undefined

程序跑不起来了！

![kill.png](img/emoji/kill.png)

bindings 也是直接 require('addon.node') 呀，Node.js 官网[也是这样说的](http://nodejs.cn/api/addons.html#addons_loading_addons_using_require)。怎么构建后就不行了呢？

后面，我在webpack 的 loader 中找到 [node-loader](https://doc.webpack-china.org/loaders/node-loader/)，里面说明
> 在 enhanced-require 中执行 node add-ons
所以，node-loader 是针对魔改过的 `require`（非 node 环境 require）的。这有可能是在 Electron 或是 webpack 发生的。

于是我对`.node`文件增加一个`node-loader`
```javascript
module: {
  rules: [
    {
      test: /\.node$/,
      use: [
        'node-loader',
        {
          loader: 'file-loader',
          options: {
            name: '[name].[ext]',
            outputPath: 'addons',
            publicPath: '../../app.asar.unpacked/addons'
          }
        }
      ]
    }
  ]
}
```
然后重新构建。**一切问题都解决了！**

![cool.png](img/emoji/cool.png)


**相关资料**：  
https://electronjs.org/docs/tutorial/using-native-node-modules  
https://nodejs.org/dist/latest/docs/api/addons.html  
https://github.com/nodejs/node-gyp/  
https://doc.webpack-china.org/loaders/node-loader/  
