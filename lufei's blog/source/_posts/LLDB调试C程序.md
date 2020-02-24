---
title: LLDB调试C程序
date: 2019-11-28 15:26:54
tags: LLDB
categories: LLDB
---

# 前言

[Clang](https://zh.wikipedia.org/wiki/Clang)是一个C、C++、Objective-C和Objective-C++编程语言的编译器前端。它采用了LLVM作为其后端。

[LLDB](https://lldb.llvm.org/)是一个支持C, Objective-C and C++的调试器，内置于xcode。

# 开始
c测试代码, 文件名mylldb.c
```
#include <stdio.h>

int main() {
    
    int i = 0;
    
    printf("hello lldb\n");

}
```
# clang生成输出文件
使用`-g`和`-o`生成调试信息和输出文件，我们这里分别是`mylldb.DSYM`和`mylldb`。

# 进入LLDB
使用`-f`载入`mylldb`

使用`breakpoint set --file`设置断点

使用`run`启动程序

接下来程序会停在断点处，`thread`, `print`, `expression`等等一系列调试命令就都可以使用了

[LLDB命令](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html#//apple_ref/doc/uid/TP40012917-CH3-SW5)

# 参考
[LLDB调试器使用简介](http://southpeak.github.io/2015/01/25/tool-lldb/)
