---
title: Swift Package Manager tips
date: 2020-08-06 13:22:05
tags: Swift
categories: Swift
---

# 前言

Swift Package Manager 是用于管理 Swift 代码分发的工具。它与 Swift 构建系统集成在一起，可以自动执行依赖项的下载，编译和链接过程。

Xcode 11 以后集成了 Swift Package Manager ，于是除了 CocoaPods，Carthage 以外，对于 iOS 开发来说，又多了一个官方支持的包管理工具。

接下来，我分享一下使用过程遇到的小问题和解决方法

# Xcode fetch 太慢

对于这个问题，有三个方案：

+ 在终端设置 proxy，然后 `open -a Xcode.app`
+ 先下载下来，然后使用本地链接 add 
+ 先 clone 到别的仓库，然后再使用别的仓库的地址 add 

# 关于 [library(name:type:targets:)](https://developer.apple.com/documentation/swift_packages/product/2878196-library)

关于 type 这个参数，如果不传，默认是 nil，那么就是允许Xcode在静态或动态链接之间进行选择

# 参考

[Package Manager](https://swift.org/package-manager/)
