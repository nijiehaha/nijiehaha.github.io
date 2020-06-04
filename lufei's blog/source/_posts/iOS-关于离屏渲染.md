---
title: iOS 关于离屏渲染
date: 2020-04-16 15:57:48
tags: iOS
categories: iOS
---

先从 UIView 和 CALayer 说起，因为这是我关注到「离屏渲染」的原因。

# UIView 和 CALayer

[UIView](https://developer.apple.com/documentation/uikit/uiview) 可以理解为「显示在屏幕上的一块矩形区域」，它可以管理这块区域内的内容，可以响应事件，可以使用 Auto Layout 布局。

> UIView 的官方文档也指出了， `UIView` 必须在主线程工作

[CALayer](https://developer.apple.com/documentation/quartzcore/calayer) 理解为「图层」，没有响应事件的能力，也不能使用 Auto Layout 布局

> CALayer 的内容渲染工作可以放在一个独立线程中

在每一个 UIView 实例 view 中，都默认有一个 CALayer 实例 layer，并且 view 会自动成为 layer 的代理，之所以 view 拥有显示能力, 也是因为 layer 的存在

# 圆角

一般情况下，我们平时给一个 View 添加圆角，应该都是使用 [masksToBounds](https://developer.apple.com/documentation/quartzcore/calayer/1410896-maskstobounds) 和  [cornerRadius](https://developer.apple.com/documentation/quartzcore/calayer/1410818-cornerradius) 这两个方法，我也不例外。

然而，事实上，就算不设置 `masksToBounds`，也是可以成功设置圆角的，当然你得设置背景色，也得在 view 的 layer 上设置。

官方文档有一段这样的说明：

> Setting the radius to a value greater than 0.0 causes the layer to begin drawing rounded corners on its background. By default, the corner radius does not apply to the image in the layer’s contents property; it applies only to the background color and border of the layer. However, setting the masksToBounds property to true causes the content to be clipped to the rounded corners.

也就是说 `cornerRadius` 只适用于 layer 上的「背景色」和「边框」属性，而对于 layer 上的 contents，是不起作用的，这就是有时候光设置 `cornerRadius` 不起作用的原因。

这也是为什么我们要使用 `masksToBounds` 的原因。

而 `masksToBounds` 才是设置圆角会触发「离屏渲染」 的真正原因。

终于说到了本文的重点了～

# iOS 渲染架构

以小见大，从 UIView 和 CAlayer 的分工，以及苹果官方文档的内容来看，iOS 渲染架构大概是这样：

> 在 Application 这一层中主要是 CPU 在操作，而到了 Render Server 这一层，CoreAnimation 会将具体操作转换成发送给GPU的 draw calls（以前是 call OpenGL ES，现在慢慢转到了 Metal），CPU 和 GPU 双方同处于一个流水线中，协作完成整个渲染工作。

+ Application 是指的软件本身的一些操作（我不知道我描述的对不对）

+ Render Server 是一个单独的进程，使用 OpenGL 或 Metal 发出对 GPU 的图形调用。

CPU -> GPU -> Frame Buffer -> 显示器

CPU 计算好显示内容提交到 GPU，GPU 渲染完成后将渲染结果放入 Frame Buffer，随后视频控制器会按照 VSync 信号逐行读取 Frame Buffer 的数据，经过可能的数模转换传递给显示器显示

# 定义离屏渲染

如果要在显示屏上显示内容，我们至少需要一块与屏幕像素数据量一样大的 Frame Buffer，作为像素数据存储区域，而这也是 GPU 存储渲染结果的地方。如果有时因为面临一些限制，无法把渲染结果直接写入 Frame Buffer，而是先暂存在另外的内存区域，之后再写入 Frame Buffer，那么这个过程被称之为离屏渲染。

大概过程如下：

App -> Offscreen Buffer -> Frame Buffer

渲染结果经过了 Offscreen Buffer，再到 Frame Buffer

# CPU “离屏渲染”

大家知道，如果我们在 UIView 中实现了 drawRect 方法，就算它的函数体内部实际没有代码，系统也会为这个 view 申请一块内存区域，等待 CoreGraphics 可能的绘画操作。

对于类似这种 “新开一块 CGContext 来画图“ 的操作，有很多文章和视频也称之为“离屏渲染”（因为像素数据是暂时存入了 CGContext，而不是直接到了 Frame Buffer）。进一步来说，其实所有 CPU 进行的光栅化操作（如文字渲染、图片解码），都无法直接绘制到由 GPU 掌管的 Frame Buffer，只能暂时先放在另一块内存之中，说起来都有点像上面定义的“离屏渲染”。

但是根据 [苹果工程师](https://lobste.rs/s/ckm4uw/performance_minded_take_on_ios_design#c_itdkfh) 的说法，这种渲染过程并非真正意义上的离屏渲染。

所以这种情况更适合的名称应该是「CPU 渲染」

## 什么时候需要 CPU 渲染

渲染性能的调优，其实始终是在做一件事：平衡 CPU 和 GPU 的负载，让他们尽量做各自最擅长的工作。

绝大多数情况下，得益于 GPU 针对图形处理的优化，我们都会倾向于让 GPU 来完成渲染任务，而给 CPU 留出足够时间处理各种各样复杂的 App 逻辑。为此 Core Animation 做了大量的工作，尽量把渲染工作转换成适合 GPU 处理的形式。

但是对于一些情况，如文字（ CoreText 使用 CoreGraphics 渲染）和图片（ ImageIO ）渲染，由于 GPU 并不擅长做这些工作，不得不先由 CPU 来处理好以后，再把结果作为 texture 传给 GPU。除此以外，有时候也会遇到 GPU 实在忙不过来的情况，而 CPU 相对空闲（ GPU 瓶颈），这时可以让 CPU 分担一部分工作，提高整体效率。

比如说，我们经常会使用 CoreGraphics 给图片加上圆角（将图片中圆角以外的部分渲染成透明）。整个过程全部是由 CPU 完成的。这样一来，既然我们已经得到了想要的效果，就不需要再另外给图片容器设置 cornerRadius。另一个好处是，我们可以灵活地控制裁剪和缓存的时机，巧妙避开 CPU 和 GPU 最繁忙的时段，达到平滑性能波动的目的。

# GPU 离屏渲染

我们所说的「离屏渲染」一般都指的 GPU 离屏渲染。

对于每一层layer，我们肯定希望优先找一种通过单次遍历就能完成渲染的算法（效率最高），不然的话就只能另申请一块 Offscreen Buffer，借助这个临时中转区域来完成一些复杂的、多次的修改/剪裁操作。

# GPU 离屏渲染 性能影响

GPU 的操作是高度流水线化的。本来所有计算工作都在有条不紊地正在向 Frame Buffer 输出，此时突然收到指令，需要输出到另一块内存，那么流水线中正在进行的一切都不得不被丢弃，切换到只能服务于我们当前的 “切圆角” 操作。等到完成以后再次清空，再回到向 Frame Buffer 输出的正常流程。

在 tableView 或者 collectionView 中，滚动的每一帧变化都会触发每个 cell 的重新绘制，因此一旦存在离屏渲染，上面提到的上下文切换就会每秒发生 60 次，并且很可能每一帧有几十张的图片要求这么做，对于 GPU 的性能冲击可想而知（ GPU 非常擅长大规模并行计算，但是我想频繁的上下文切换显然不在其设计考量之中）

尽管离屏渲染开销很大，但是当我们无法避免它的时候，可以想办法把性能影响降到最低。优化思路也很简单：既然已经花了不少精力把图片裁出了圆角，如果我能把结果缓存下来，那么下一帧渲染就可以复用这个成果，不需要再重新画一遍了。

CALayer 为这个方案提供了对应的解法：[shouldRasterize](https://developer.apple.com/documentation/quartzcore/calayer/1410905-shouldrasterize) 

一旦被设置为 true，Render Server 就会强制把 layer 的渲染结果（包括其子 layer，以及圆角、阴影、group opacity 等等）保存在一块内存中，这样一来在下一帧仍然可以被复用，而不会再次触发离屏渲染。

# 离屏渲染 小结

+ CPU 渲染虽然也是“离屏”，但是通常提到的离屏渲染是发生在 GPU

+ 如果一个 layer 无法在一次遍历就完成绘制，那么就不得不触发离屏渲染

+ 离屏渲染的开销主要在于 Frame Buffer 与 Offscreen Buffer 之间的上下文切换。如果无法避免，也可以通过有效利用 shouldRasterize ，减少触发的次数

+ CPU 和 GPU 是相互扶持的关系。CPU 渲染效率不高，但是较为通用灵活；GPU 擅长并行计算，但也有捉襟见肘之时，此时 CPU 可以适当给与帮助

# 为什么 UIView 的操作必须在主线程

首先，因为 UIKit 并不是一个 线程安全 的类，UI操作涉及到渲染访问各种 View 对象的属性，如果异步操作下会存在读写问题，而为其加锁则会耗费大量资源并拖慢运行速度。

另一方面因为整个程序的起点 UIApplication 是在主线程进行初始化，所有的用户事件都是在主线程上进行传递（如点击、拖动），所以 view 只能在主线程上才能对事件进行响应。

而在渲染方面由于图像的渲染需要以 60 帧的刷新率在屏幕上「同时」更新，在非主线程异步化的情况下无法确定这个处理过程能够实现同步更新。

# 参考

[关于离屏渲染的深入研究](https://medium.com/@jasonyuh/%E5%85%B3%E4%BA%8E%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93%E7%9A%84%E6%B7%B1%E5%85%A5%E7%A0%94%E7%A9%B6-e776f56b3e60)

[AsyncDisplayKit 的一篇关于圆角的文档](https://texturegroup.org/docs/corner-rounding.html)

[CPU VS GPU](https://zsisme.gitbooks.io/ios-/content/chapter12/cpu-versus-gpu.html)

[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)




