---
title: 'Kingfisher源码阅读:命名空间'
date: 2019-12-16 09:17:42
tags: iOS
categories: iOS
---

# 前言

拜读喵神的[Kingfisher](https://github.com/onevcat/Kingfisher)，学习Swift，第一个就是这个`kf`命名空间的实现，好酷。

以前写OC的时候，给系统方法扩展，官方都是推荐使用增加`前缀_`的方式，swift的推出，改变了这种情况，因为语言特性和苹果力荐的面向协议编程，有了更酷的实现。

+ [喵神的实现](https://github.com/onevcat/Kingfisher/blob/master/Sources/General/Kingfisher.swift)

+ [swift中的lazySequence](https://github.com/apple/swift/blob/master/stdlib/public/core/LazySequence.swift)

# 思路

实现一个包裹泛型Base类的Struct或者Class，实现`Protocol` 和你想要的命名空间，将 Protocol 加载到所需的 Base 类并通过`Extension + where Base`实现 Base 类的特定代码

# 实现

参考上面的源码和思路，我自己试着写了一遍，代码如下：

```
/// 包裹NJBase的struct
struct NijieWrapper<NJBase> {
    
    let base:NJBase
    
    init(_ base: NJBase) {
        
        self.base = base
        
    }
    
}

/// 实现协议
protocol NijieCompatible: AnyObject {}
extension NijieCompatible {
    
    var nj:NijieWrapper<Self> {
        
        get { return NijieWrapper(self) }
        set {}
        
    }
    
}

/// 为UIImageView添加扩展
extension UIImageView: NijieCompatible {}
extension NijieWrapper where NJBase: UIImageView {
    
    func setNijieImage(image:UIImage? = nil) {
        
        base.image = image
        
    }
    
}

```

实现协议的时候，之所以使用扩展，是因为需要给所有遵守协议的类型提供默认实现。

使用：

```
let imageView = UIImageView()
imageView.frame = CGRect(x: 100, y: 100, width: 100, height: 100)
imageView.nj.setNijieImage(image: UIImage(named: "test123"))
```

# 结尾

这样子，命名空间就实现了，感恩swift




