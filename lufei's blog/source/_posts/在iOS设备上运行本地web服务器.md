---
title: 在iOS设备上运行本地web服务器
date: 2020-03-18 16:41:27
tags: iOS
categories: iOS
---

# 前言

有这个奇怪想法的起因是因为，突然有个「H5离线展示」的需求，最简单的方法就是将前端代码下发到本地，直接加载，不过还有个办法就是通过起本地服务的方式，这样的方式，似乎更加优雅。

有了这个想法之后，我就开始寻找比较好的 [socket](https://en.wikipedia.org/wiki/Berkeley_sockets) 框架，想试一试，不过令我比较失望的是，大多数都是 OC 的，或者不支持最新的 swift 版本，理想情况下，我想找个纯 swift 的 socket 框架，这可难住我了，最终只能求助于苹果开源的一个偏底层的网络框架[SwiftNio](https://github.com/apple/swift-nio)

# 开始

`SwiftNio`虽说跨平台，但是移植到 iOS 上还需要一些特殊的扩展[NIOTransportServices](https://github.com/apple/swift-nio-transport-services)

依赖管理利用的苹果自带的`Swift Package Manager`，官方自带，xcode11 以后就自动集成了这个包管理工具，取代`CocoaPods`只是时间问题，并且`CocoaPods`的使用也越来越不如以前便捷了。

> `Swift Package Manager`也是可以加载本地依赖包的，在文件路径前加上 `file://` 就可以了。[了解 Swift Package Manager](https://swift.org/package-manager/)

因为我只需要加载一些本地资源文件，所以起一个本地 http 服务就够用了，不需要很复杂，通过参考文档，以及苹果提供的示例，踉踉跄跄的完成了这个小目标，长舒一口气。

> 有一点，我还挺奇怪，这个框架2018年就开源了，网上关于它的文章却不是很多，基本上很难搜索到我需要的东西，可能是我姿势不太对吧，郁闷。

我的`web server`主要依赖`NIO`, `NIOHTTP1`, `NIOTransportServices`三个模块。

思路就是在 app 启动时，开启 http 服务，然后使用`wkWebView`加载 `localhost:port`，就可以了。

代码就不贴了，项目地址如下：

[demo地址](https://github.com/nijiehaha/MyLocalServer/blob/master/MyLocalServer/MyLocalServer/ViewController.swift)

# 总结

`SwiftNio`虽然最终实现了我的想法，但是我依然有一些问题留在心里，假如我想实现一个像`Nginx`那样的`web server`，据我目前从`SwiftNio`文档中获取的知识远远不够，我期待以后能多一些 api 的示例，demo，光靠文字，我还是理解的不是很好，期待有一天，我能实现我自己的完整的纯 swift 的 web 服务器。

好像有点跑偏了原本的主题，哈哈哈～

# 参考
[Running a SwiftNIO Server in an iOS app](https://diamantidis.github.io/2019/10/27/swift-nio-server-in-an-ios-app)




