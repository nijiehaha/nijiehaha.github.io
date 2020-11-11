---
title: iOS 仿射变换
date: 2020-09-18 13:51:07
tags: iOS
categories: iOS
---

# 仿射变换简述

[仿射变换](https://zh.wikipedia.org/wiki/%E4%BB%BF%E5%B0%84%E5%8F%98%E6%8D%A2)，是指在几何中，对一个向量空间进行一次线性变换并接上一个平移，变换为另一个向量空间。

具体到 iOS，苹果在 Core Graphics 中使用 [CGAffineTransform](https://developer.apple.com/documentation/coregraphics/cgaffinetransform#declaration) 来表示这个变化，或者说映射关系。

简单来说，CGAffineTransform 是一个可以和二维空间向量（例如 CGPoint ）做乘法的矩阵。

当对图层应用变换矩阵，图层矩形内的每一个点都被相应地做变换，从而形成一个新的四边形的形状。UIView 有一个 [transform](https://developer.apple.com/documentation/uikit/uiview/1622459-transform) 属性就是用来接收 CGAffineTransform，并对当前 view 做变换的。

上面也说了，CGAffineTransform 的背后实际上是矩阵的相乘，所以当你实际在做仿射变换的时候，你旋转，缩放，平移的顺序不同，可能导致结果也不相同。

为了解决这个问题，我选择分开记录旋转，平移，缩放的参数，自己计算生成一个 CGAffineTransform 来进行变换

[CGAffineTransform 接口测试 demo](https://github.com/nijiehaha/AffineTransformTest/blob/master/AffineTransformTestDemo.swift)

说到 CGAffineTransform ，很难不说说 iOS 的坐标系，在这次测试中，我也对 iOS 的坐标系统有了新的认识

# iOS 坐标系简述

每一个图形上下文都有三个坐标系：

+ 绘图（用户）坐标系，用来绘制的坐标系

+ 视图坐标系，相对于视图的坐标系

+ 物理设备坐标系，展示屏幕像素点的坐标系

这些坐标系通过一一映射的关系的构成了 iOS 上的坐标系统。

大多数情况下，我们都在和 UIKit 打交道，所以我们大多人认识到的 iOS 坐标系原点都是在左上角的。

不过当你需要使用 Core Graphics 绘图的时候，你就会发现，映射到 UIView 上的结果和你想象的不太一样，这是因为 Core Graphics 默认的坐标原点是在左下角的。

当然 UIKit 给了我们很多方式，可以不需要直接使用 Core Graphics，就能享受到 Core Graphics 的方便，但是当你需要直接使用 Core Graphics 来绘制图形的时候，你可能需要[翻转你的坐标](https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/GraphicsDrawingOverview/GraphicsDrawingOverview.html#//apple_ref/doc/uid/TP40010156-CH14-SW26)

# 参考

[The Math Behind the Matrices](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/dq_affine/dq_affine.html#//apple_ref/doc/uid/TP30001066-CH204-CJBECIAD)

[Coordinate Systems and Drawing in iOS](https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/GraphicsDrawingOverview/GraphicsDrawingOverview.html)




