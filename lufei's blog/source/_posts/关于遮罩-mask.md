---
title: 关于遮罩(mask)
date: 2019-11-26 13:20:32
tags: iOS
categories: iOS
---

遮罩(Mask)，就是一幅只有单通道，肉眼看上去只有黑白，和黑白之间颜色的图片，通过可透过光和不可透过光，来遮挡下面的图片，达到不同的效果，可以理解为四通道图片中的alpha通道，所以也可以叫alpha图。

遮罩图中，直观肉眼看上去，0代表黑色，255代表白色。

遮罩图和被遮罩图融合一般情况都是作为前景图，在这种情况下：
>遮罩图在和被遮罩图融合的时候，0代表透明度为0，也就是完全透明，1代表不透明，肉眼看上去，就是白色部分透明，黑色的部分不透明。融合的话，可以使用[CGBitmapContextCreate](https://developer.apple.com/documentation/coregraphics/1455939-cgbitmapcontextcreate?language=objc)拿到遮罩图和被遮罩图的像素数组，自己组合计算，也可以利用[CGImageCreateWithMask](https://developer.apple.com/documentation/coregraphics/1456337-cgimagecreatewithmask)，直接利用系统api帮助你合成。

甚至可以用`CGContext`来为自己合成，类似这样
```
UIGraphicsBeginImageContextWithOptions(self.size, NO, 0.0f);

[tintColor setFill];

CGRect bounds = CGRectMake(0, 0, self.size.width, self.size.height);

UIRectFill(bounds);

[self drawInRect:bounds blendMode:blendMode alpha:1.0f];

if (blendMode != kCGBlendModeDestinationIn) {
    [self drawInRect:bounds blendMode:kCGBlendModeDestinationIn alpha:1.0f];
}

UIImage *tintedImage = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();
```

遮罩图作为背景图来融合另一张图的时候，和作为前景图融合是相反的，也就是黑是透明，白是不透明，这种情况比较少见。

在iOS中，CALayer，处理[mask](https://developer.apple.com/documentation/quartzcore/calayer/1410861-mask)和[contents](https://developer.apple.com/documentation/quartzcore/calayer/1410773-contents)属性的时候，肉眼直观显示，黑色代表可透过光，白色不可透光。

所以，**在一张肉眼可见的黑白的alpha单通道图上，透明和不透明，是通过白色和黑色来处理的**，这就为我们的画板增加了想像的空间，也就是说:

**在遮罩图上，没有透明色(clearColor), 只有白，黑，及黑白之间的颜色**

事实上单纯的单通道alpha图，看上去是黑白的，但是我们使用他是为了遮挡和显示被遮罩图的，所以，为了在显示的时候，能够直观的看到效果，[CGImageMaskCreate](https://developer.apple.com/documentation/coregraphics/1455089-cgimagemaskcreate)是一个重要的api, 可以把"看上去黑白"的遮罩图，转为可以在iOS中使用的遮罩图。

这一块的东西比较绕，所以我写个博客记录一下，便于以后弄晕了，再来看看QAQ

[测试demo](https://github.com/nijiehaha/testMaskDrawBoard)




