---
title: Node.js 中的 DES 加解密
date: 2017-01-05
image: img/nodejs.png
categories:
    - Front-end
tags:
    - Node.js
---

Node.js 自带强大的加密功能 Crypto，它是基于 OpenSSL 库实现的加密技术。
DES 是一种对称加密算法，密匙长度必须是8的整数倍，在一些简单的应用场景经常被使用。

为了网络上信息传输的安全（防止第三方窃取信息看到明文），发送发和接收方分别进行加密和解密，这样信息在网络上传输的时候就是相对安全的。

<!--more-->

DES 加密模式有： Electronic Codebook (ECB) , Cipher Block Chaining (CBC) , Cipher Feedback (CFB) ,  Output Feedback (OFB)。这里以密文分组链接模式 CBC 为例，使用了相同的 key 和 iv (Initialization Vector)

```javascript
const crypto = require('crypto')

// DES 加密
function desEncrypt (message, key) {
  key = key.length >= 8 ? key.slice(0, 8) : key.concat('0'.repeat(8 - key.length))
  const keyHex = new Buffer(key)
  const cipher = crypto.createCipheriv('des-cbc', keyHex, keyHex)
  let c = cipher.update(message, 'utf8', 'base64')
  c += cipher.final('base64')
  return c
}

// DES 解密
function desDecrypt (text, key) {
  key = key.length >= 8 ? key.slice(0, 8) : key.concat('0'.repeat(8 - key.length))
  const keyHex = new Buffer(key)
  const cipher = crypto.createDecipheriv('des-cbc', keyHex, keyHex)
  let c = cipher.update(text, 'base64', 'utf8')
  c += cipher.final('utf8')
  return c
}
```

要查看 Node.js 支持哪些 Cipher,可以通过 `crypto.getCiphers()` 查看

相关资料：  
https://nodejs.org/api/crypto.html