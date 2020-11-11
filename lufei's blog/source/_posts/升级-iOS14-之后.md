---
title: 升级 iOS 14 之后
date: 2020-10-10 15:42:01
tags: iOS
categories: iOS
---

# 升级 xcode 12 之后，模拟器编译报错

报错内容：`building for iOS Simulator, but linking in object file built for iOS, xxxx for architecture arm64`

因为 xcode 12 之后，Build Settings 不再默认包含 Valid Architectures 编译选项，作为替代品，以 VALID_ARCHS 的形式放到了 User-Defined 的编辑分组下。

并且，因为 arm 的 mac 的推出，xcode 12 也不再默认支持 x86_64 了，需要手动添加才能跑模拟器。

简单直接的解决方法：

删掉 User-Defined 下的 VALID_ARCHS

> 在 project 下，选中 User-Defined 下的 VALID_ARCHS，按 delete 删除

[Xcode 12 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-12-release-notes)

# UISlider 的奇怪显示

在 iOS 14 之前，我使用 slider 的 subViews 的 lastView，作为滑块的位置来更新特殊的 UI 和 value，之前工作得都很不错，直到我更新了 iOS 14 ，一切都变得糟糕起来了。

iOS 14 的 UISlider 的 subViews 的 lastView 不再是滑块的位置，好吧，只能放弃以前的硬核方案了。

解决方案：

```
let trackRect = slider.trackRect(forBounds: slider.bounds)
let thumbRect = slider.thumbRect(forBounds: slider.bounds, trackRect: trackRect, value: slider.value)
```

# Xcode 12 放弃了对 iOS 8 的支持导致的 CocoaPods 警告

警告内容: `The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 8.0, but the range of supported deployment target versions is 9.0 to 14.0.99.`

在 CocoaPods 解决此问题之前，可以将以下内容添加到 Podfile 中作为临时解决方法:

```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings.delete 'IPHONEOS_DEPLOYMENT_TARGET'
    end
  end
end
```

[Xcode 12 drops support for iOS 8 and how to fix deployment target warnings in CocoaPods](https://www.jessesquires.com/blog/2020/07/20/xcode-12-drops-support-for-ios-8-fix-for-cocoapods/)

# iOS 14 UIDatePicker 适配

准确地说，iOS 13.4 之后，UIDatePicker 的默认样式改变了，如果还是需要展示原来的样式，可以这样修改：

```
if #available(iOS 13.4, *) {
  datePicker.preferredDatePickerStyle = .wheels
} else {
  // Fallback on earlier versions
}
```

# UITableView 的组尾高度问题

我遇到的问题是（UITableView 的样式使用的是默认的 plain）：我把 UITableView 所有分组中不需要组尾的高度设置为 0.01（当然不建议这么做，历史遗留问题），并且在代理中把组尾的 view 返回 nil，但是这样的设置在 iOS 14 之后，似乎失效了，而且出现了奇怪的显示问题（具体就是出现了一条大概一个像素的线），实验了一下，大概有两种解决办法：

+ 把组尾的高度改成更小的浮点数，比如 0.0000001（并且最好不要把组尾的 view 直接返回 nil，这一点存疑）。

+ 把 UITableView 的样式改成 group。
