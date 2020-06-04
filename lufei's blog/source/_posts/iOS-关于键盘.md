---
title: iOS 关于键盘
date: 2020-06-04 09:11:55
tags: iOS
categories: iOS
---

# 前言

iOS 上多个输入框，可能出现的键盘遮挡，始终是一个有一丢丢麻烦的问题，当然，有很好的第三方 [IQKeyboardManager](https://github.com/hackiftekhar/IQKeyboardManager)，帮助我们一行代码解决了这个问题，不过，我还是想自不量力的做一些小小的总结。

# iOS 键盘的基本知识

在 iOS 上，键盘的变化由 [NSNotificationCenter](https://developer.apple.com/documentation/foundation/nsnotificationcenter) 控制的，所以要想拿到这些关于键盘的变化信息，你得提前在 NSNotificationCenter 上添加观察者，来监听这些关于键盘的通知。

那么具体有哪些通知呢？比如下面这些吧：

`keyboardWillChangeFrameNotification`： 在键盘 frame 更改之前立即发出

`keyboardWillShowNotification`: 在键盘出现之前立即发出

当然还有其他的，就不一一列举了～

除了键盘的通知，通知的内容，也是我们需要关注的，这些信息都保存在 [NSNotification](https://developer.apple.com/documentation/foundation/nsnotification) 的 [userInfo](https://developer.apple.com/documentation/foundation/nsnotification/1409222-userinfo) 字典中，我们可以通过一些特殊的 key 来获取，比如：

`keyboardFrameBeginUserInfoKey`：返回开始时键盘在当前屏幕中的 frame

那么我们可以这样获取 rect

```
let noti = NSNotification()
let rect = noti.userInfo?[UIResponder.keyboardIsLocalUserInfoKey] as? CGRect
```

除了键盘本身的这些通知，还有一些关于和输入框相关的通知 `UITextFieldTextDidBeginEditingNotification` 等等，这些使用方法和上面类似

# 键盘遮挡解决方案

可以看出来，无论是什么关于键盘遮挡的方案，一定是基于通知的。

[大名鼎鼎的 IQKeyboardManager 也是这么做的](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L2186) 

通过参考 IQKeyboardManager 的源码，键盘遮挡解决方案大概是这样：

+ 首先，设计一个类，遵循单例模式，注册并监听键盘相关的通知

+ 初始化一个 UITapGestureRecognizer，在点击 UITextField 对应的 UIWindow 的时候，收起键盘

+ 初始化一些默认属性，例如键盘距离、覆写键盘的样式等

+ 设置不需要解决键盘遮挡问题的类

+ 其他

基本上，整个解决方案其实都是基于 iOS 中的通知系统的；在事件发生时，调用对应的方法做出响应。

# 举个例子（ IQKeyboardManager 源码理解）

比如一个 UITextField 被点击的时候，[textFieldViewDidBeginEditing（这个方法只是和代理方法同名） 被调用](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L1757)，然后在这个方法中，做了这样几件事：

+ [为 UITextField 添加 IQToolBar](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L1780)

+ [添加手势，用来关闭键盘](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L1803)

+ [在调整 frame 前，保存当前 frame，以备之后键盘隐藏后的恢复](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L1810)

+ [将视图移动到合适的位置](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L1836)

其中由两个比较重要的方法：

+ [adjustPosition](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L949) 根据键盘弹出的位置调整视图

+ [restorePosition](https://github.com/hackiftekhar/IQKeyboardManager/blob/master/IQKeyboardManagerSwift/IQKeyboardManager.swift#L1444) 恢复视图调整前的位置

以上大概就是 IQKeyboardManager 的关键内容了

# 小结

当然 IQKeyboardManager 的内容肯定不止这些，比如 UITextField 和 UITextView 的通知机制，IQToolBar 的实现，IQTextView 的实现啊等等，欢迎大家还是去读源码（也是在跟自己说哒～）

# 参考

[IQKeyboardManager](https://github.com/hackiftekhar/IQKeyboardManager)

[『零行代码』解决键盘遮挡问题（iOS）](https://draveness.me/keyboard/)












