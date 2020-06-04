---
title: 初学Method Swizzling
date: 2017-01-09 15:39:34
tags: iOS
categories: iOS
---

最近开始学习 **Method Swizzling**，分享一些学习收获吧。

提到 **Method Swizzling**，就得提到OC的方法替换了，总结一下，大概应该有这样几种吧：

1.**重写类的方法(Overriding Methods)**：这个就不多说了

2.**伪装(Posing)**：Posing是个很有趣的技术，不过已经过时了，因为64位和iPhone环境下的Objective-C Runtime中不再支持它了. 通过这个伪装(posing),你可子类化，然后将这个子类伪装成它的父类。像变魔术一般，Runtime会让这个子类应用于各处，这时方法复写又有了用处。既然被抛弃了，也就不必多费口舌了

3.**归类(Categories)**：这种方法其实仅仅适用于复写目标类的父类中实现的函数。如果直接复写目标类中的方法，使用归类会带来问题：第一，它无法调用方法的之前的实现。替换掉后，之前的实现就被完全改写了。但大部分情况下，只是想增加些功能，并不期望完全替代。第二，如果被多个**category**复写，运行时(**runtime**)并不保证哪个真正会被使用到。

下面就正式进入正题吧，**Method Swizzling** 解决了这个问题

> **原理**：**Method Swizzing**是发生在运行时的，主要用于在运行时将两个Method进行交换，我们可以将**Method Swizzling**代码写到任何地方，但是只有在这段**Method Swilzzling**代码执行完毕之后互换才起作用。
而且**Method Swizzling**也是iOS中AOP(面相切面编程)的一种实现方式，我们可以利用苹果这一特性来实现AOP编程。

我的理解**Method Swizzling**的工作原理：原本**SEL1**对应**IMP1**，**SEL2**对应**IMP2**，但是经过**Method Swizzing**变换后，**SEL1**对应**IMP2**，**SEL2**对应**IMP1**，两个**IMP**交换一下，从而达到了修改方法的目的。

**Method Swizzling使用**

在实现Method Swizzling时，核心代码主要就是一个**runtime**的C语言API：
```
 OBJC_EXPORT void method_exchangeImplementations(Method m1, Method m2)
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
```

**举个例子**：

```

#import "UIViewController+MyMethod Swizzling.h"
#import <objc/runtime.h>

@implementation UIViewController (MyMethod Swizzling)

+ (void)load {

// 通过class_getInstanceMethod()函数从当前对象中的method list获取method结构体，如果是类方法就使用class_getClassMethod()函数获取。

Method fromMethod = class_getInstanceMethod([self class], @selector(viewDidLoad));

Method toMethod = class_getInstanceMethod([self class], @selector(swizzlingViewDidLoad));

/**

*  我们在这里使用class_addMethod()函数对Method Swizzling做了一层验证，如果self没有实现被交换的方法，会导致失败。

*  而且self没有交换的方法实现，但是父类有这个方法，这样就会调用父类的方法，结果就不是我们想要的结果了。

*  所以我们在这里通过class_addMethod()的验证，如果self实现了这个方法，class_addMethod()函数将会返回NO，我们就可以对其进行交换了。

*/

if (!class_addMethod([self class], @selector(swizzlingViewDidLoad), method_getImplementation(toMethod), method_getTypeEncoding(toMethod))) {

method_exchangeImplementations(fromMethod, toMethod);

}

}

// 我们自己实现的方法，也就是和self的viewDidLoad方法进行交换的方法。

- (void)swizzlingViewDidLoad {

NSString *str = [NSString stringWithFormat:@"%@", self.class];

// 我们在这里加一个判断，将系统的UIViewController的对象剔除掉

if(![str containsString:@"UI"]){

NSLog(@"看看类型 : %@", self.class);

}

[self swizzlingViewDidLoad];

}

@end

```

**注意**：刚开始学习，说法什么的用的不准确，也是看的别人的文章，跟着边敲边理解，如果有新的收获，会继续分享的QAQ

**参考文章**：

[iOS黑魔法 - Method Swizzling](http://www.jianshu.com/p/ff19c04b34d0)

[Runtime Method Swizzling 实践小结](https://www.zybuluo.com/xifenglang-33250/note/618461)
