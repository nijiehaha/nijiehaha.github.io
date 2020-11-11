---
title: Mac App 动态库注入
date: 2020-06-12 16:14:37
tags: 杂七杂八
categories: 杂七杂八
---

# 前言

MacOS 上的 App 的动态库注入，可以为目标 App 扩展一些新的功能，经过我的小小的探索，大概有两个方案。

# 方案一

首先利用 Xcode 生成一个动态库 `libTest.dylib`，在动态库 `Test.m` 中加入一段代码：

```
+ (void)load {
    
    NSLog(@" libTest loaded !!! ");
    
}

```
方便我们判断是否注入成功。

使用 Xcode 创建 Example App，把 `Example.app` 文件夹找到。

创建 `ExampleInject.app` 文件夹，并创建一个子文件夹，名字叫 `Contents`。

在 `ExampleInject.app/Contents` 下创建一个名为 `Frameworks` 的文件夹，将动态链接库 `libTest.dylib` 拷贝到这个文件夹中。

在 `ExampleInject.app/Contents` 下新建文件夹 `MacOS`，然后在这个文件夹下新建一个 shell 脚本文件，名字为 `ExampleInject`。

脚本内容：

```
#!/bin/sh
CurrentAppPath=$(cd $(dirname $0) && cd .. && pwd)
DYLD_INSERT_LIBRARIES=${CurrentAppPath}/Frameworks/libTest.dylib /Example.app/Contents/MacOS/Example
```

为脚本添加可执行权限

```
chmod +x ExampleInject.app/Contents/MacOS/ExampleInject
```

随后 运行 `ExampleInject` ，就可以在系统的控制台看到注入的 log :

**libTest loaded !!!**

# 方案二

利用 [insert_dylib](https://github.com/Tyilo/insert_dylib) 直接向目标 App 进行注入动态库

首先利用 Xcode 创建 动态库 `libTest.dylib` (同上)和目标 App `Example`。

把 `insert_dylib` 的可执行文件 和 `Example.app`，`libTest.dylib` 放到同一个目录下

运行下面命令：

```
./insert_dylib libTest.dylib ./Example.app/Contents/MacOS/Example

```

在 `Example.app -> Contents -> MacOS` 目录下，出现了一个 `Example_patched` 的文件，这个就是注入成功的可执行文件。

> 如果出现找不到 `libTest.dylib` 的情况，可以直接把 `libTest.dylib` 拷贝到 `Example.app -> Contents -> MacOS` 目录下

运行 `Example_patched` 文件后，同样可以在系统的控制台看到注入的 log。

# 参考

[macOS 逆向之生成动态注入 App](https://blog.nswebfrog.com/2018/02/09/make-injection-app-for-mac/)

[macOS 应用注入开发简介与实践](https://www.jianshu.com/p/44f5b98bd47f)