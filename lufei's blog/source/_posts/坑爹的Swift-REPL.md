---
title: 坑爹的Swift REPL
date: 2019-11-29 09:27:56
tags: LLDB
categories: LLDB
---

# 前言 
昨天稍微探索了 LLDB 如何调试 C ，随后心血来潮想要再探索一下 swift 如何利用 LLDB 来调试，不过坑爹的事情发生了。

# 问题
首先是 *swift* 和 *swiftc* (这里的 swift 和 swiftc 都是指的命令行工具)，如果你愿意的话，在 Bash 中输入`-h`展示的说明都是 Swift compiler ，这我一瞬间有点糊涂了。

赶紧利用万能的搜索引擎，去搜索了一下, 了解了一下, 结果如下，感兴趣的可以看下：

+ [swift和swiftc的区别](https://stackoverflow.com/questions/57777091/whats-the-difference-between-swift-and-swiftc)

+ [喵神的说明](https://swifter.tips/swift-cli/)

简单来说：*swift* 是一个 REPL 环境，使得使用「Swift」就像使用脚本语言一样，但实际上，还是需要编译后再运行的，所以只是表现的很像"即时的解释执行"，使用起来限制也很多。*swiftc* 就是正宗的 「Swift」的编译器前端了。

「Swift」的编译架构 是 `Swift / LLVM`

# 开始坑爹之旅
好吧，了解了这些之后，我对 *swiftc* 就失去了探索的兴趣，因为我感觉这个应该和 *clang* 使用起来差不多，事实上也确实是这样。
所以，我开始探索，如何在 swift REPL 中使用 LLDB 调试

因为在`swift -h`中，我是能看到这段说明的：
> -g <br>
 Emit debug info. This is the preferred setting for debugging with LLDB.

这让我坚信，即使在 REPL 环境，我也是能使用 LLDB 调试的。

但是万万没想到

[苹果的官方文档](https://swift.org/lldb/)的使用例子，我居然不能使用。

仔细看了说明后，发现了这句话：
> However, because of this tight integration, developers must use a matched pair of compiler and debugger built using the same sources. Debugging using any other version of LLDB can lead to unpredictable results.

看上去，可能是版本不匹配？

然后在「Swift」的官网，发现一个[swift-lldb](https://github.com/apple/swift-lldb)项目。

这里给了你如何给你本机的「Swift」编译一个匹配的 LLDB 版本。

这时候，我还不死心，我查看了一下 LLDB 的版本，以及支持的「Swift」的版本，可能因为我没更新 xcode ，我的本机「Swift」版本是5.0.1，而 lldb 的目标「Swift」版本是5.0, 我也不知道这是不是问题所在，反正，我的 swift REPL 还没办法使用 lldb 来加断点什么的，我很郁闷QAQ

但是使用 *swiftc* 的话，一切都是那么的美好！

好吧，我的天呐！

我爱 *swiftc* ！

# 加餐
[深入剖析 iOS 编译 Clang LLVM](https://github.com/ming1016/study/wiki/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90-iOS-%E7%BC%96%E8%AF%91-Clang---LLVM)





