---
title: Swift 闭包的循环引用
date: 2020-08-14 14:22:42
tags: Swift
categories: Swift
---
# 闭包

闭包是可以在代码中被传递和引用的功能性独立代码块。Swift 中的闭包和 C 以及 OC 中的 blocks 很像，还有其他语言中的匿名函数也是类似的。

***闭包能够捕获和存储定义在其上下文中的任何常量和变量的引用***

Swift 闭包大概有以下三种：

+ 全局函数是一个有名字但不会捕获任何值的闭包;

+ 内嵌函数是一个有名字且能从其上层函数捕获值的闭包;

+ 闭包表达式是一个轻量级语法所写的可以捕获其上下文中常量或变量值的没有名字的闭包;

# Swift 内存管理

Swift 是自动管理内存的

释放的原则遵循了自动引用计数 (ARC) 的规则：当一个对象没有引用的时候，其内存将会被自动回收。这套机制从很大程度上简化了我的编码，我们只需要保证在合适的时候将引用置空 (比如超过作用域，或者手动设为 nil 等)，就可以确保内存使用不出现问题。

但是，所有的自动引用计数机制都有一个从理论上无法绕过的限制，那就是循环引用 (retain cycle) 的情况。

# 闭包循环引用

简单来说就是 self (指代当前的实例对象) 间接或直接持有闭包，闭包也持有 self 的时候，就会导致闭包的循环引用

比如说，下面这段代码：

```
class People  {
    
    lazy var closure = {
        print(self.name)
    }
    
    var name = "路飞"
    
    deinit {
        print("\(name) 销毁了")
    }
    
}

let lufei = People()
lufei.closure()
```

上面这段代码就会造成循环引用了，People 实例持有 closure，closure 也持有了 People 实例，于是造成了循环引用，导致内存泄漏。

# 解决循环引用

## Weak 

weak 引用只是一个指向那个对象的指针，但是它不会保护这个对象，所以当没有对象持有它的时候，它将会被 ARC 释放。

在 swift 中，所有的弱引用都会作为 Optional

于是上面的代码可以这么改：

```
lazy var closure = { [weak self] in
    print(self?.name)
}
```

## Unowned

unowned 引用和 weak 很像，不过还是有区别的。

unowned 引用意味着对应的类型 不可能为 Optional 类型。

所以如果你确定 unowned 引用的对象在访问的时候不会被释放的话，可以这样改：

```
lazy var closure = { [unowned self] in
    print(self.name)
}
```

# 参考

[Automatic Reference Counting](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)

[Swift之你真的知道为什么使用weak吗？](Swift之你真的知道为什么使用weak吗？)

[[译]Swift中的Weak，Strong和Unowned](https://medium.com/@zkh90644/%E8%AF%91-swift%E4%B8%AD%E7%9A%84weak-strong%E5%92%8Cunowned-1cb64b01f933)