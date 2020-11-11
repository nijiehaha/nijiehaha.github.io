---
title: iOS Auto Layout
date: 2020-07-08 09:41:27
tags: iOS
categories: iOS
---

# 前言

看了 [钟颖大神的微博](https://m.weibo.cn/1765732340/4435328616116891) 之后，惭愧于自己关于 Auto Layout 知识的缺失，于是有了这篇文章。

## 钟大微博里的不太明白的词

+ RTL 一种书写方向，如阿拉伯文和希伯来文

+ A11Y 全称 Accessibility 指「可访问性」，在苹果的定义下指「无障碍使用」，[Accessibility on iOS](https://developer.apple.com/accessibility/ios/)

+ Baseline 指基线，在 Auto Layout 的定义下，一般指「在视图底部上方放置文字的地方」

# Cassowary 算法

[Cassowary](http://constraints.cs.washington.edu/cassowary/) 能够有效解析线性等式系统和线性不等式系统，用户的界面中总是会出现不等关系和相等关系，Cassowary 开发了一种规则系统可以通过约束来描述视图间关系。约束就是规则，能够表示出一个视图相对于另一个视图的位置。

2011年苹果将这个算法运用到了自家的布局引擎中，也就是 Auto Layout。

# Auto Layout 的生命周期

在得到自己的 layout 之前 Layout Engine 会将 Views ，约束，Priorities（优先级）， instrinsicContentSize（主要是 UILabel, UIImageView 等）通过计算转换成最终的效果。

Layout Engine 的循环机制：

约束变化 -> Deferred Layout Pass -> 应用 Run Loop -> 约束变化

## 生命周期中需要注意的事项

+ 不要期望 frame 会立刻变化

+ 在重写 layoutSubviews() 时需要非常小心。

# 关于约束

Auto Layout 的视图层级里，所有视图通过放置在它们里面的约束来动态计算的它们的大小和位置。一般控件需要四个约束决定位置大小，如果定义了 [intrinsicContentSize](https://developer.apple.com/documentation/uikit/uiview/1622600-intrinsiccontentsize) 的比如 UILabel 只需要两个约束即可。

## IntrinsicContentSize / Compression Resistance Priority / Hugging Priority

具有 instrinsic content size 的控件，比如 UILabel ，UIButton ，选择控件，进度条和分段等等，可以自己计算自己的大小，比如 label 设置 text 和 font 后大小是可以计算得到的。

但是当同时有多个 Intrinsic Content Size 控件需要布局的时候，就会出现 「Intrinsic 冲突」，此时就需要 Compression Resistance Priority 和 Hugging Priority。

+ [Hugging priority](https://developer.apple.com/documentation/uikit/uiview/1622485-setcontenthuggingpriority) 抗拉伸优先级，值越大，越难拉伸
 
+ [Content Compression Resistance](https://developer.apple.com/documentation/uikit/uiview/1622526-setcontentcompressionresistancep) 抗压缩优先级，值越大，越难压缩

## 约束方程式

view1.attribute1 = mutiplier * view2.attribute2 + constant

## Api 添加约束

[NSLayoutConstraint 官方参考](https://developer.apple.com/documentation/uikit/nslayoutconstraint)

## VFL 语言添加约束

[VFL 语言指南](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html)

## UIStackView

由于前面两种方案过于麻烦，苹果还提供了 UIStackView

[UIKit Framework Reference UIStackView Class Reference](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/AutoLayoutWithoutConstraints.html)

# 布局过程

updateConstraints -> layoutSubViews -> drawRect

## viewDidLayoutSubviews，-layoutSubviews

> 使用 Auto Layout 的 view 会在 viewDidLayoutSubviews 或 －layoutSubview 调用 super 转换成具有正确显示的 frame 值。

## View 的改变会调用哪些方法

+ 改变 frame.origin 不会掉用 layoutSubviews

+ 改变 frame.size 会使 superVIew 的 layoutSubviews 调用

+ 改变 bounds.origin 和 bounds.size 都会调用 superView 和自己 view 的 layoutSubviews 方法

# 参考

[Auto Layout Guide](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/index.html)

[深入剖析Auto Layout，分析iOS各版本新增特性](https://ming1016.github.io/2015/11/03/deeply-analyse-autolayout/)

[AutoLayout的一些基本概念](https://github.com/ming1016/study/wiki/Masonry)


