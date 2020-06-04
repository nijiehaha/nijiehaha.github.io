---
title: iOS13移除UIWebView之后
date: 2020-03-20 13:42:02
tags: iOS
categories: iOS
---

# 前言

最近提交应用到 App Store 收到了苹果的提醒，大概是关于 iOS13 废弃的api`UIWebView`的，[官方的通知](https://developer.apple.com/news/?id=12232019b)

好吧，那就开始检查项目中可能使用`UIWebView`的地方吧，首先本项目本身的代码，其实没有的，因为项目做的晚，都是直接用的`WKWebView`，不过依赖的很多第三方，为了兼容，都还有对`UIWebView`的引用。

检查第三方库SDK是否含有`UIWebView`的命令如下：

```
find . -type f | grep -e ".a" -e ".framework" | xargs grep -s UIWebView
```

[iOS13适配参考](https://juejin.im/post/5d00af64e51d455d88219ee2)

找到包含的第三方之后，就开始按顺序更新，升级，适配吧～

# 开始

第一个就是`WechatOpenSDK`，版本号`1.8.6`以上的就是我们需要更新的版本

[iOS微信SDK接入教程](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html)

有个很大的坑，就是接入新版`微信SDK`，必须接入[Universal Links](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)

啊，麻烦啊！！！注意事项：

+ 接入`Universal Links`之后，会生成一个`AppNameRelease.entitkements`文件，从名字上也能发现，在测试的时候，需要切换到`Release`模式来检验测试，描述文件最好也重新编辑生成一下，并且只有第一次安装的时候，苹果会在适当的时候，根据 xcode 中注册的域名去拿到`apple-app-site-association`文件（所以需要反复测试的时候，记得移除，重新安装）

+ `apple-app-site-association`文件虽然内容是`json`，但千万不要有后缀，注册的`Universal Links`必须支持`https`，`content-type`最好是`Application/json`，还有需要在域名对应的服务器的根目录下，新建一个`.well-known`文件夹，在这个目录下，放置`apple-app-site-association`文件

+ 接入微信的时候，还有一个容易遗忘的步骤，需要重写`AppDelegate`的`continueUserActivity`方法，xcode11以后，`SceneDelegate`也有对应的方法。

+ 新版微信分享，第一次触发的时候，会有一个授权的过程，如果发现了，不要太惊讶，之后就不会有了

如果你对你的`apple-app-site-association`内容不太自信，可以使用下面的办法：

+ [苹果提供的检测apple-app-site-association是否有效的地址](https://search.developer.apple.com/appsearch-validation-tool/)，这种办法应该是需要你把`Universal Links`集成到一个页面中，用爬虫去爬到`apple-app-site-association`文件内容，从而去验证，测试链接形如：`https://www.myWeb.com/.well-known/apple-app-site-association`

+ 我们输入对应的通用链接和文件名，能在浏览器查看到`json`内容，就基本没什么问题了

以上就基本完成了

# 最后

当然还有其他的第三方需要更新适配，不过最麻烦的一关已经过了

我给苹果和微信两位大佬跪了～～～哭～～～




