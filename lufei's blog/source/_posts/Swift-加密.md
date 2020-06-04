---
title: Swift 加密
date: 2020-04-29 10:32:17
tags: Swift
categories: Swift
---

# 简述

## 消息摘要算法

利用「散列函数」生成固定长度的 「散列值」，主要运用于确保信息传输的一致性。

使用比较广泛的消息摘要算法应该是「MD5」，「SHA-2」了。

[MD5](https://zh.wikipedia.org/wiki/MD5)

[SHA-2](https://zh.wikipedia.org/wiki/SHA-2)

## 对称加密算法

严格地说，消息摘要算法，不能算真正的「加密」，因为它不可逆，也就是说，只能「加密」，不能「解密」。

为了保护信息传输过程中的安全，于是对称加密算法就出现了，从名字能看出来，对称加密的「加密」和「解密」的过程使用的是相同的密钥。

对称加密算法中最著名的是「AES算法」。

[AES](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E5%8A%A0%E5%AF%86%E6%A0%87%E5%87%86)

[对称加密算法](https://zh.wikipedia.org/wiki/%E5%B0%8D%E7%A8%B1%E5%AF%86%E9%91%B0%E5%8A%A0%E5%AF%86)

## 非对称加密算法

这种算法需要两种密钥，一个公钥，一个私钥，公钥用作加密，私钥则用作解密，使用公钥把明文加密后所得的密文，只能用相对应的私钥才能解密并得到原本的明文，最初用来加密的公钥不能用作解密。

由于加密和解密需要两个不同的密钥，故被称为非对称加密。公钥可以公开，可任意向外发布，私钥不可以公开，必须由用户自行严格秘密保管，绝不透过任何途径向任何人提供，也不会透露给被信任的要通信的另一方。

「RSA算法」是非对称加密算法中运用最广的。

[RSA](https://zh.wikipedia.org/zh-cn/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)

[RSA 算法详解](http://www.guideep.com/read?guide=5676830073815040)

# Swift 中的加密

iOS13 之后，苹果推出了新的 Swift 安全加密框架 [CryptoKit](https://developer.apple.com/documentation/cryptokit)

新框架支持了很多其它现成可用的加解密算法，使用起来也比之前的 C Api 相比更加简洁。

[WWDC2019：Cryptography and Your Apps](http://awhisper.github.io/2019/06/06/wwdc2019-cryptography/)




