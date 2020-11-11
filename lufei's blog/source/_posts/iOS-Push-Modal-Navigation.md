---
title: iOS Push & Modal Navigation
date: 2020-07-07 09:08:21
tags: iOS
categories: iOS
---

最近项目中开始集成一些模块化的页面，所以如何「恰当」地实现页面跳转成为了一个需要特别关心的问题，在了解一些后，在这里做个小小的总结。

# Push Navigation

这是 iOS 中最常见的 Navigation 机制。

将一个 VC Push 到 Navigation 栈中，由导航栏统一管理。内存管理也由编译器来负责。

简单易用，满足了大部分情况下的页面跳转需求。

主要 api：

+ [pushViewController(_:animated:)](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621887-pushviewcontroller)

+ [popViewController(animated:)](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621886-popviewcontroller)

# Modal Navigation

模态 Navigation 是指「一个 VC，以模态形式呈现另一个 VC 」。 VC 不必是导航控制器的一部分，以模态呈现的 VC 通常被视为呈现（父）VC 的“子代”。模态呈现的 VC 通常没有任何导航栏或标签栏。（父） VC 还负责消除其创建和呈现的模态 VC（内存管理由父 VC 负责）。

主要 api：

+ [present(_:animated:completion:)](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621380-present)

+ [dismiss(animated:completion:)](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621505-dismiss)

# 拦截导航栏返回事件

两种方案：

+ [UIViewController-BackButtonHandler](https://github.com/onegray/UIViewController-BackButtonHandler/blob/master/UIViewController%2BBackButtonHandler.m)

+ 自定义一个 UINavigationController 子类，并遵守 UINavigationBarDelegate

# 'whose view is not in the window hierarchy!' error 

Modal Navigation 偶尔会引发上面这个错误。

大部分情况下都是因为当前 controller 的 view 还没有加入当前的 window hierarchy，可以在 `ViewDidAppear` 方法中再做 Modal Navigation

# 参考

[Pushing, Popping, Presenting, & Dismissing ViewControllers](https://medium.com/@felicity.johnson.mail/pushing-popping-dismissing-viewcontrollers-a30e98731df5)

[Understanding Navigation in iOS](https://guides.codepath.com/ios/Understanding-Navigation-in-iOS#modal-navigation)

[你真的了解iOS中控制器的present和dismiss吗？](https://www.jianshu.com/p/dd6180bc340a)


