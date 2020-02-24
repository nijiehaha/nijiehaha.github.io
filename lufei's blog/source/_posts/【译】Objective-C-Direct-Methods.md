---
title: 【译】Objective-C Direct Methods
date: 2020-01-20 13:29:37
tags:
categories:
---

# 原文
[Objective-C Direct Methods](https://nshipster.com/direct/#objc_direct-propertydirect-and-objc_direct_members)

# 正文
当听说OC引入新功能的时候，我已经很难感到兴奋了。最近的一些关于OC的改进，都是为了服务于和Swift的互通，而不是关于OC这门语言本身了（看看[nullabiltiy](https://developer.apple.com/swift/blog/?id=25)和[lightwight generics](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/using_imported_lightweight_generics_in_swift)）

因此，当我了解到[最近的Clang的合并补丁](https://reviews.llvm.org/D69991)为OC的方法添加了直接分配的机制的时候，感到十分的兴奋。

这种新功能的起源还不是很清楚；能得到的线索止于苹果内部的 Radar 号（2684889），除了能借此推出这项功能的相对年龄（估计是本世纪初的什么时候），就得不到更多的信息了。幸好，这个功能拥有充足的文档和测试范围，能让我们很好的了解它的原理。(对实施者Pierre Habouzit，审核经理John McCall和其他LLVM贡献者表示由衷的感谢)

本周在NSHipster上，我们借此机会回顾了OC方法的分派，并试图了解这一新语言功能对未来代码库的潜在影响。

(**Direct**方法最早可能会出现在Xcode 11.x上，但很可能会在WWDC 2020上宣布。)

要了解Direct方法的重要性，您需要了解一些有关OC运行时的知识。但是，在此之前，让我们从oop本身的起源开始我们的讨论：

# 面向对象编程

Alan Kay在1960年初期创造了这个词，之后，在Adele Goldberg，Dan Ingalls和他在Xerox parc的其他同事的帮助下，Kay通过创建Smalltalk编程语言，在70年代将该想法付诸实践。

(在此期间，Xerox PARC 的研究人员还开发了 Xerox Alto，这将成为苹果Macintosh和所有其他GUI计算机的灵感来源)

在1980年代，Brad Cox和Tom Love开始研究OC的第一个版本，这是一种语言，旨在采用Smalltalk的面向对象范式，并在C的坚实基础上加以实施。 90年代，该语言成为NeXT（后来成为Apple）的官方语言。

对于我们这些在iPhone时代开始学习OC的人来说，该语言通常被视为另一项苹果专有技术，是该公司“Not invented here”文化的众多，晦涩的副产品之一。然而，我们并不能说 Objective-C 只是「面向对象的 C」而已，它是最早的几个面向对象语言之一，像其他语言一样，都有着强烈的面向对象特质。

现在，oop是什么意思？这是个好问题。上世纪90年代的炒作周期使该词几乎毫无意义。但是，就我们今天的目的而言，让我们集中讨论Alan Kay在1998年写的一句话：

> I’m sorry that I long ago coined the term “objects” for this topic because it gets many people to focus on the lesser idea. The big idea is “messaging”

# 动态分派和OC运行时

在OC中，程序由一组对象组成，这些对象通过传递依次调用方法或函数的消息而彼此交互。消息传递的这种行为由方括号语法表示：

```
[someObject aMethod:withAnArgument];
```

编译OC代码后，消息发送将转换为对名为`objc_msgSend`的函数的调用（字面意思是“将消息发送至带有参数的某个对象”）。

```
objc_msgSend(object, @selector(message), withAnArgument);
```

+ 第一个参数是接收方（实例方法本身）

+ 第二个参数是_cmd：选择器或方法的名称

+ 任何方法参数都作为附加函数参数传递

Objc_msgSend负责确定响应此消息而调用哪个基础实现，该过程称为方法分派。

在Objective-C中，每个类维护一个调度表来解析在运行时发送的消息。调度表中的每个条目都是一个方法，该方法将选择器（SEL）键入相应的实现（IMP），该实现是指向C函数的指针。当对象收到消息时，它查询其类的调度表。如果可以找到选择器的实现，则调用关联的函数。否则，对象将查询其超类的调度表。这将继续沿继承链向上进行，直到找到匹配项或根类（NSObject）认为选择器无法识别。

(更不用说OC还可以让您做诸如替换方法实现和在运行时动态创建新类之类的事情。您可以做的事相当宽广。)

如果您认为所有这些间接听起来都需要大量工作，从某种意义上来说，那是对的！

如果您的代码中有一条常使用的代码调用路径，比如一个频繁被调用的消耗巨大的方法，那么您可以想象避免所有这些间接操作会有所益处。为此，一些开发人员已使用C函数作为替换动态分派的一种方式。

# 用C函数直接分派

正如我们在`objc_msgSend`中所看到的，任何方法调用都可以通过将隐式`self`作为第一个参数传递而由等效函数表示。

例如，考虑使用常规的动态分派方法对OC类进行以下声明。

```
@interface MyClass: NSObject
- (void)dynamicMethod;
@end
```

如果开发人员希望在`MyClass`上实现某些功能, 但是不要遍历所有消息来发送，那么，他们可以声明一个静态的C函数，该函数将`MyClass`的实例作为参数。

```
static void directFunction(MyClass *__unsafe_unretained object);
```

以下是每种方法的调用方式的展示：

```
MyClass *object = [[[MyClass] alloc] init];

// Dynamic Dispatch
[object dynamicMethod];

// Direct Dispatch
directFunction(object);
```

# Direct 方法

Direct方法具有常规OC方法的外观，但是具有C函数的行为。调用Direct方法时，它直接调用其基础实现，而不是通过objc_msgSend。

有了这个新的LLVM补丁，您现在可以有选择性地注释掉Objective-C方法，从而避免使用动态分派。

# objc_direct, @property(direct), 和 objc_direct_members

要使实例或类方法使用Direct分派，可以使用objc_direct[Clang属性](https://nshipster.com/__attribute__/)对其进行标记。同样，可以通过使用`direct`声明Object-C属性的方法来使其实现Direct分派。

```
@interface MyClass: NSObject
@property(nonatomic) BOOL dynamicProperty;
@property(nonatomic, direct) BOOL directProperty;

- (void)dynamicMethod;
- (void)directMethod __attribute__((objc_direct));
@end
```

根据我们的计算，`direct`的添加使`@property`属性的总数达到16：

+ `getter` 和 `setter`
+ `readwrite` 和 `readonly`
+ `atomic` 和 `nonatomic`
+ `weak`, `strong`, `copy`, `retain`，和 `unsafe_unretained`
+ `nullable`, `nonnullable`, 和 `null_resttable`
+ `class`

当使用`objc_direct_members`属性注释类别或类扩展的`@interface`时，其中包含的所有方法和属性声明都将视为direct的，除非该类事先声明过。

> 您无法使用`objc_direct_members`属性注释原始类接口。

```
__attribute__((objc_direct_members))
@interface MyClass ()
@property (nonatomic) BOOL directExtensionProperty;
- (void)directExtensionMethod;
@end
```

用`objc_direct_members`注释`@implementation`具有相似的效果，会使未先前声明的成员被视为direct成员，包括任何由属性综合产生的隐式方法。

```
__attribute__((objc_direct_members))
@implementation MyClass
- (BOOL)directProperty {…}
- (void)dynamicMethod {…}
- (void)directMethod {…}
- (void)directExtensionMethod {…}
- (void)directImplementationMethod {…}
@end
```

> 动态方法不能在子类中被直接方法覆盖，direct方法也不能被覆盖。              
  协议不能声明direct方法要求，而类也不能使用direct方法来实现协议要求。

之前的示例已经示范过如何使用了，我们可以看到在调用方式上direct方法和动态方法是如何区分的：

```
MyClass *object = [[[MyClass] alloc] init];

// Dynamic Dispatch
[object dynamicMethod];

// Direct Dispatch
[object directMethod];
```

对于我们当中注重性能的开发人员而言，direct方法似乎是精彩的方案。但是有一个小转折：

***在大多数情况下，direct方法可能不会带来明显的性能优势。***

事实证明，`objc_msgSend`非常快。由于积极的缓存，广泛的低级优化以及现代处理器的固有性能，`objc_msgSend`的开销非常低。

我们早已可以将iPhone硬件合理地描述为资源受限的环境了。因此，除非苹果正在为新的嵌入式平台做准备（AR眼镜？），否则我们对苹果在2019年实施OC的**Direct**方法的最合理的解释是性能以外的原因。

# 隐藏的动机

当将OC方法标记为direct方法时，其实现可以隐藏可见性。也就是说，direct方法只能在同一模块内调用。它甚至不会出现在OC运行时中。

隐藏可见性有两个直接的优点：

+ 较小的二进制文件大小
+ 没有外部调用

没有外部可见性，也没有从OC运行时动态调用它们的方法，直接方法实际上是私有方法。

> 如果您想使用direct分派，但仍然希望使您的API可以从外部访问，则可以将其包装在C函数中。                          

``` 
static inline void performDirectMethod(MyClass *__unsafe_unretained object) {
    [object directMethod];
}
```

尽管Apple可以使用隐藏可见性来防止混乱和私有API的使用，但这似乎并不是主要动机。

根据实施此功能的Pierre所说，此优化的主要好处是减少了代码大小。据报道，未使用的O元数据的权重可占已编译二进制文件中`__text`部分的5 – 10％。

您可以想象，从现在起到明年的开发者大会，可以有几个工程师遍历每个SDK框架，用`objc_direct`注释私有方法，并用`objc_direct_members`注释私有类，这是逐步加强其SDK的轻量级方法。

如果的确如此，那么我们对Objective-C的新功能添加的理由也持怀疑态度。我认为既然不是服务于swift语言， 那应该就是服务于苹果公司的吧。尽管这个功能在编程历史和Apple本身中占有重要地位，但已经很难不把OC视为历史了。