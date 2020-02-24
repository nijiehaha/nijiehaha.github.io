---
title: 坑爹的Swift REPL
date: 2019-11-29 09:27:56
tags: LLDB
categories: LLDB
---

# 前言 
昨天稍微探索了LLDB如何调试C，随后心血来潮想要再探索一下swift如何利用LLDB来调试，不过坑爹的事情发生了。

# 问题
首先是swift和swiftc，如果你愿意的话，在Bash中输入`-h`展示的说明都是Swift compiler，这我一瞬间有点糊涂了。

赶紧利用万能的搜索引擎，去搜索了一下, 了解了一下, 结果如下，感兴趣的可以看下：

+ [swift和swiftc的区别](https://stackoverflow.com/questions/57777091/whats-the-difference-between-swift-and-swiftc)

+ [喵神的说明](https://swifter.tips/swift-cli/)

简单来说：swift是一个REPL环境，使得使用swift就像使用脚本语言一样，但实际上，还是需要编译后再运行的，所以只是表现的很像"即时的解释执行"，使用起来限制也很多。swiftc就是正宗的swift的编译器前端了。
swift的编译架构 是 `Swift / LLVM`

# 开始坑爹之旅
好吧，了解了这些之后，我对swiftc就失去了探索的兴趣，因为我感觉这个应该和clang使用起来差不多，事实上也确实是这样。
所以，我开始探索，如何再swift REPL中使用LLDB调试

因为在`swift -h`中，我是能看到这段说明的：
> -g 

> Emit debug info. This is the preferred setting for debugging with LLDB.

这让我坚信，即使在REPL环境，我也是能使用LLDB调试的。

但是万万没想到

[苹果的官方文档](https://swift.org/lldb/)的使用例子，我居然不能使用。

仔细看了说明后，发现了这句话：
> However, because of this tight integration, developers must use a matched pair of compiler and debugger built using the same sources. Debugging using any other version of LLDB can lead to unpredictable results.

看上去，可能是版本不匹配？

然后在swift的官网，发现一个[swift-lldb](https://github.com/apple/swift-lldb)项目。

这里给了你如何给你本机的swift编译一个匹配的LLDB版本。

这时候，我还不死心，我查看了一下LLDB的版本，以及支持的swift的版本，可能因为我没更新xcode，我的本机swift版本是5.0.1，而lldb的目标swift版本是5.0, 我也不知道这是不是问题所在，反正，我的swift REPL还没办法使用lldb来加断点什么的，我很郁闷QAQ

但是使用swiftc的话，一切都是那么的美好！

好吧，我的天呐！

我爱swiftc！

# 加餐
[深入剖析 iOS 编译 Clang LLVM](https://github.com/ming1016/study/wiki/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90-iOS-%E7%BC%96%E8%AF%91-Clang---LLVM)





