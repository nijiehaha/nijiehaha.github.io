---
title: iOS 图片处理
date: 2020-07-22 13:25:43
tags: iOS
categories: iOS
---

# 位图

位图就是一个像素数组，数组中的每个像素就代表着图片中的一个点，无论是 JPEG，还是 PNG 都是一种压缩的位图图形格式，但它们都不是图片的原始数据，在 iOS 中，可以简单的通过下面的方法获取图片的原始数据：

```
let rawData = image?.cgImage?.dataProvider?.data
```

在将图片渲染到屏幕之前，必须要先获得原始数据，才能执行后续的绘制操作，这就是为什么需要对图片解压缩的原因。

## 从原始位图数据恢复 CGImage 代码示例
```
let image = UIImage(contentsOfFile: path)

let originCgImage = image?.cgImage

let rawData = image?.cgImage?.dataProvider?.data

let imgDataProvider = CGDataProvider.init(data: rawData)
            
let cgImage = CGImage.init(width: originCgImage.width, 
                height: originCgImage.height, 
                bitsPerComponent: originCgImage.bitsPerComponent, 
                bitsPerPixel: originCgImage.bitsPerPixel, 
                bytesPerRow: originCgImage.bytesPerRow, 
                space: originCgImage.colorSpace!, 
                bitmapInfo: originCgImage.bitmapInfo, 
                provider: imgDataProvider!, 
                decode: nil, 
                shouldInterpolate: false, 
                intent: originCgImage.renderingIntent)
```

# 图片解压缩

在 iOS 的图片工作流中，默认情况下，系统将图片的解压缩工作是在主线程处理的，这是一个相当耗时的 CPU 操作，尤其是有大量图片且快速滑动的列表上，性能影响很大。

于是很多大神研究出了一套「强制解压缩」的方案：

> 当未解压缩的图片将要渲染到屏幕时，系统会在主线程对图片进行解压缩，而如果图片已经解压缩了，系统就不会再对图片进行解压缩。因此，方案就是在子线程提前对图片进行强制解压缩。

「强制解压缩」的原理：对图片进行重新绘制，得到一张新的解压缩后的位图

核心函数：CGContext 的 [init(data:width:height:bitsPerComponent:bytesPerRow:space:bitmapInfo:)](https://developer.apple.com/documentation/coregraphics/cgcontext/1455939-init)

这个函数创建一个位图上下文，用来绘制一张宽 width 像素，高 height 像素的位图。

## 参考 YYModel 中的解压缩实现

```
CGImageRef YYCGImageCreateDecodedCopy(CGImageRef imageRef, BOOL decodeForDisplay) {
    ...

    if (decodeForDisplay) { // decode with redraw (may lose some precision)
        CGImageAlphaInfo alphaInfo = CGImageGetAlphaInfo(imageRef) & kCGBitmapAlphaInfoMask;

        BOOL hasAlpha = NO;
        if (alphaInfo == kCGImageAlphaPremultipliedLast ||
            alphaInfo == kCGImageAlphaPremultipliedFirst ||
            alphaInfo == kCGImageAlphaLast ||
            alphaInfo == kCGImageAlphaFirst) {
            hasAlpha = YES;
        }

        // BGRA8888 (premultiplied) or BGRX8888
        // same as UIGraphicsBeginImageContext() and -[UIView drawRect:]
        CGBitmapInfo bitmapInfo = kCGBitmapByteOrder32Host;
        bitmapInfo |= hasAlpha ? kCGImageAlphaPremultipliedFirst : kCGImageAlphaNoneSkipFirst;

        CGContextRef context = CGBitmapContextCreate(NULL, width, height, 8, 0, YYCGColorSpaceGetDeviceRGB(), bitmapInfo);
        if (!context) return NULL;

        CGContextDrawImage(context, CGRectMake(0, 0, width, height), imageRef); // decode
        CGImageRef newImage = CGBitmapContextCreateImage(context);
        CFRelease(context);

        return newImage;
    } else {
        ...
    }
}
```
大致步骤如下：
+ 使用 CGBitmapContextCreate 函数创建一个位图上下文；
+ 使用 CGContextDrawImage 函数将原始位图绘制到上下文中；
+ 使用 CGBitmapContextCreateImage 函数创建一张新的解压缩后的位图。

## 注意

在我实际使用 `init(data:width:height:bitsPerComponent:bytesPerRow:space:bitmapInfo:)` 过程中，对 bitmapInfo 的选择有一些疑惑，尤其是 premultiplied-alpha，经过搜索查资料，iOS 只支持 premultiplied-alpha，macOS 可以支持非 premultiplied-alpha

[Which CGImageAlphaInfo should we use?](https://stackoverflow.com/questions/23723564/which-cgimagealphainfo-should-we-use)

[官方的说法是](https://developer.apple.com/documentation/uikit/1623912-uigraphicsbeginimagecontextwitho)：

> You use this function to configure the drawing environment for rendering into a bitmap. The format for the bitmap is a ARGB 32-bit integer pixel format using host-byte order. If the opaque parameter is true, the alpha channel is ignored and the bitmap is treated as fully opaque (CGImageAlphaInfo.noneSkipFirst | kCGBitmapByteOrder32Host). Otherwise, each pixel uses a premultipled ARGB format (CGImageAlphaInfo.premultipliedFirst | kCGBitmapByteOrder32Host).

# 图片的编解码

主要是 Image/IO 的用法

## 解码

解码，指的是讲已经编码过的图像封装格式的数据，转换为可以进行渲染的图像数据。具体来说，iOS 平台上就指的是将一个输入的二进制 Data ，转换为上层 UI 组件渲染所用的 UIImage 对象。

以静态图为例：

+ 创建 CGImageSource
+ 读取图像格式元数据（可选）
+ 解码得到 CGImage
+ 生成上层的 UIImage，清理

示例：
```
let data = try! Data.init(contentsOf: URL(fileURLWithPath: path!)) as CFData
let source = CGImageSourceCreateWithData(data, nil)
let cgImage = CGImageSourceCreateImageAtIndex(source!, 0, nil)
```

## 渐进式解码

渐进式解码（Progressive Decoding），即不需要完整的图像流数据，允许解码部分帧（大部分情况下，会是图像的部分区域），对部分使用了渐进式编码的格式，则更可以解码出相对模糊但完整的图像。

比如说，JPEG 支持三种方式的渐进式编码，包括 Baseline，interlaced，以及progressive (参考：[iOS 处理图片的一些小 Tip](https://blog.ibireme.com/2015/11/02/ios_image_tips/))

对于 Image/IO 的渐进式解码，其实和静态图解码的过程类似。但是第一步创建 `CGImageSource` 时，需要使用专门的 `CGImageSourceCreateIncremental` 方法，之后每次有新的数据（下载或者其他流输入）输入后，需要使用 `CGImageSourceUpdateData`（或者 `CGImageSourceUpdateDataProvider`）来更新数据。注意这个方法需要每次传入所有至今为止解码的数据，不仅仅是当前更新的数据。

之后的过程，就和普通的解码一致。

## 编码

编码过程，这里指的就是将一个 UIImage 表示的图像，编码为对应图像格式的数据，输出一个 Data 的过程。Image/IO 提供的对应概念，叫做 `CGImageDestination` ，表示一个输出。之后的编码相关的操作，和这个 Destination 一一对应。

以静态图为例：

+ 创建 CGImageDestination
+ 添加图像格式元数据（可选）和CGImage
+ 编码得到输出，清理

示例：

```
let outData = NSMutableData()
let destination = CGImageDestinationCreateWithData(outData, kUTTypeJPEG, 1, nil)
CGImageDestinationAddImage(destination!, image!.cgImage!, nil)
CGImageDestinationFinalize(destination!)
```

# 图片的读写

## 读取图片

想要读取原图大小，最好的办法，就是拿到图片 url，根据 url 来读取 Data。

示例：
```
let data = Data(contentsOf: imageUrl)
```

> 展示适合的字节展示格式，一种方式是使用 ByteCountFormatter

## 写入图片

iOS 中写入图片，一般是指保存到相册（当然还有其他的，这里主要谈论保存到相册）

iOS 中，所有 接收 UIImage 的 保存到相册的 Api 中，都会对图片进行跟当前设备相关的压缩，但是如果保存的是 Data ，误差在 1 K左右

示例：
```
import Photos

/// 保存 二进制 数据 到相册
PHPhotoLibrary.shared().performChanges ({

    let options = PHAssetResourceCreationOptions()
    let createRequest = PHAssetCreationRequest.forAsset()
    createRequest.addResource(with: .photo, data: image.pngData()!, options: options)

}) { (_, _) in
}

let allPhotosOptions = PHFetchOptions()

allPhotosOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: true)]

// 获取所有照片
let asset = PHAsset.fetchAssets(with: allPhotosOptions)[1]
        
let options: PHContentEditingInputRequestOptions = PHContentEditingInputRequestOptions()
        
options.canHandleAdjustmentData = {(adjustmeta: PHAdjustmentData) -> Bool in
   return true
}
// 检查效果
asset.requestContentEditingInput(with: options, completionHandler: { (contentEditingInput, info) in

    let url = contentEditingInput?.fullSizeImageURL
    let jpgData = try! Data(contentsOf: url!)
    let byteFormatter = ByteCountFormatter()    
    byteFormatter.countStyle = .file
    byteFormatter.includesActualByteCount = true
    print(byteFormatter.string(fromByteCount: Int64(jpgData.count)))

})
```

# 参考

[Quartz 2D Programming Guide](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007533-SW1)

[Image I/O Programming Guide](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/ImageIOGuide/imageio_intro/ikpg_intro.html#//apple_ref/doc/uid/TP40005462-CH201-TPXREF101)

[图片Premultiplied Alpha到底是干嘛用的](https://segmentfault.com/a/1190000002990030)

[iOS 图片的解压缩](https://www.cnblogs.com/dins/p/ios-tu-pian.html)

[iOS平台图片编解码入门教程（Image/IO篇）](https://dreampiggy.com/2017/10/30/iOS%E5%B9%B3%E5%8F%B0%E5%9B%BE%E7%89%87%E7%BC%96%E8%A7%A3%E7%A0%81%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B%EF%BC%88Image:IO%E7%AF%87%EF%BC%89/)
