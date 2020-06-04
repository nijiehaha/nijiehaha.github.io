---
title: 你可能不知道的UICollectionView
date: 2020-04-03 13:36:39
tags: iOS
categories: iOS
---

# 前言

最近遇到一个跟 [UICollectionView](https://developer.apple.com/documentation/uikit/uicollectionview) 相关的控件问题，就是如何给 `UICollectionView` 的不同 *Section* 设置不同的背景色，进而想到如何给它们设置不同的属性。

为什么想到这个问题，是因为以前遇到这种情况，我往往都是不假思索分成多个 *Collection View* 来处理的，完全忘记了其实 `UICollectionView` 是可以分组的，哎，我以前到底绕了多少弯路啊，苦笑～

# 了解 UICollectionView

`UICollectionView` 是 iOS6 以后引入 *UIKit* 的新的视图控件，Api 设计和 `UITableView` 类似，基础用法也类似。

不过 *UICollectionView* 在 *UITableView* 的基础上做了一些扩展，其中最强大的部分就是完全灵活的布局结构。

*UITableView* 和 *UICollectionView* 都是 [data-source 和 delegate](http://developer.apple.com/library/ios/#documentation/general/conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html) 驱动的。它们在显示其子视图集的过程中仅扮演容器角色，且对子视图集真正的内容毫不知情。

*UICollectionView* 在此之上进行了进一步抽象。它将其子视图的位置，大小和外观的控制权委托给一个单独的布局对象。通过提供一个自定义布局对象，你几乎可以实现任何你能想象到的布局。布局对象继承自 [UICollectionViewLayout](https://developer.apple.com/documentation/uikit/uicollectionviewlayout) [抽象基类](https://zh.wikipedia.org/wiki/%E8%99%9A%E5%87%BD%E6%95%B0#%E6%8A%BD%E8%B1%A1%E7%B1%BB%E5%92%8C%E7%BA%AF%E8%99%9A%E5%87%BD%E6%95%B0)。

> iOS6 中以 [UICollectionViewFlowLayout](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout) 类的形式展现了一个具体的布局实现。<br>
一般情况下，在实际开发中，我们使用 `UICollectionViewFlowLayout` 就够了

为了适应任意布局，*collection view* 建立了一个类似、但比 *table view* 更灵活的视图层级。

和 *table view* 一样，你的主要内容显示在 *Cell* 中，cell 可以被任意分组到 *Section* 中。*Collection view* 的 cell 必须是 `UICollectionViewCell` 的子类。

除了 *Cell*，*collection view* 额外管理着两种视图：

+ **Supplementary Views** (追加视图)
+ **Decoration Views** （装饰视图）

*collection view* 中的 *Supplementary views* 相当于 *table view* 的 *section header* 和 *footer views*。像 *cells* 一样，他们的内容都由数据源对象驱动。然而和 *table view* 中用法不一样，*supplementary view* 并不一定会作为 *header* 或 *footer view*，它们的数量和放置的位置完全由 `UICollectionViewLayout` 控制。

*Decoration views* 纯粹为一个装饰品。他们完全属于 `UICollectionViewLayout`，并被布局对象管理，他们并不从 *data source* 获取内容。当 `UICollectionViewLayout` 指定需要一个 *decoration view* 的时候，*collection view* 会自动创建，并将 `UICollectionViewLayout` 提供的布局参数应用到上面去。并不需要为 *decoration view* 准备任何内容。

> *Supplementary views* 和 *decoration views* 必须是 `UICollectionReusableView` 的子类。

*Supplementary views* 和 `UICollectionViewCell` 都需要在 `collection view` 中注册，这样当 *data source* 让它们从 *reuse pool* 中出列时，它们才能够创建新的实例。如果你是使用的 *Interface Builder* ，则可以通过在可视编辑器中拖拽一个 cell 到 *collection view* 上完成 cell 在 *collection view* 中的注册。同样的方法也可以用在 *supplementary view* 上，前提是你使用了 `UICollectionViewFlowLayout`。如果没有，你只能通过调用 `registerClass:` 或者 `registerNib:` 方法手动注册视图类了。你需要在 `viewDidLoad` 中做这些操作。

> *Decoration views* 需要使用 `UICollectionViewLayout` 注册

# 自定义 UICollectionViewFlowLayout

前面也说了，日常开发中，大部分情况下，`UICollectionViewFlowLayout`足够使用了，需要自定义的时候，也可以继承 `UICollectionViewFlowLayout`，进行进一步的定制。

当然，如果实在出现了某种需要自定义 `UICollectionViewLayout` 的情况，第一步是去仔细阅读 [UICollectionViewLayout 文档](https://developer.apple.com/library/ios/documentation/uikit/reference/UICollectionViewLayout_class/Reference/Reference.html)

我的需求，只需要进一步定制 `UICollectionViewFlowLayout` 就够了。

大概流程如下：

首先是我们需要自定义一个 [UICollectionViewLayoutAttributes](https://developer.apple.com/documentation/uikit/uicollectionviewlayoutattributes)的子类，并添加一个 `backgroundColor` 属性。

然后自定义一个 *Decoration view* ，显然它继承自 [UICollectionReusableView](https://developer.apple.com/documentation/uikit/uicollectionreusableview)，因为我们需要 `UICollectionViewLayoutAttributes` 的子类新添加的 `backgroundColor` 对 `UICollectionReusableView` 的子类视图生效，所以还需要重写一下 [apply](https://developer.apple.com/documentation/uikit/uicollectionreusableview/1620139-apply) 方法。

最后在`UICollectionViewFlowLayout` 的子类中注册我们自定义的 `Decoration view`。

> 想要计算 *Decoration view* 的 `frame`，其中一个方法是，获取每一组的第一个 *item* 和 最后一个 *item* ，利用 [union(_:)](https://developer.apple.com/documentation/coregraphics/cgrect/1455837-union) 方法来实现。

当然还可以为 `UICollectionViewDelegateFlowLayout` 添加一个新的关于背景色的代理方法。

# 实践

[参考代码](https://github.com/nijiehaha/CustomLayoutTool/blob/master/SectionBgFlowLayout.swift)

# 参考

[自定义 Collection View 布局](https://objccn.io/issue-3-3/)

[Collection View Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40012334-CH1-SW1)

# UIcollectionView 其他可能有用的小知识

## 也许有时候你会觉得 `UICollectionView` 的刷新动画有点多余。

我有时候就会这么想，于是想关闭 `UICollectionView` 刷新时候的隐式动画。

此时，`UICollectionView` 的 [performBatchUpdates(_:completion:)](https://developer.apple.com/documentation/uikit/uicollectionview/1618045-performbatchupdates) 方法，就派上了用场：

```
UIView.setAnimationsEnabled(false)

collectionView.performBatchUpdates({
    collectionView.reloadItems(at: indexPaths)
}, completion: { (_) in
    UIView.setAnimationsEnabled(true)
})
```

当然还有更酷的方法，那就是 `UIView`的 [performWithoutAnimation(_:)](https://developer.apple.com/documentation/uikit/uiview/1622484-performwithoutanimation) 方法了：

```
UIView.performWithoutAnimation {
    collectionView.reloadItems(at: indexPaths)
}
```

> 当然，设置 UIView 动画时间为0，也可以算第三种办法，此处就不多说了。

但是不知道为什么 `reloadData` 在这两个个方案中的表现，总是不那么和我美好的想象相符合，尤其是在`performBatchUpdates`里，直接导致 UI 刷新无效了，原因目前还不太清楚，不过 [reloadData](https://developer.apple.com/documentation/uikit/uicollectionview/1618078-reloaddata) 的文档里倒是做了这样的说明：

> You should not call this method in the middle of animation blocks where items are being inserted or deleted. Insertions and deletions automatically cause the collection’s data to be updated appropriately.

好吧，乖乖听话～～

