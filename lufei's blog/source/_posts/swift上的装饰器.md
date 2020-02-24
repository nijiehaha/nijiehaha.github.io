---
title: swift上的装饰器(Decorator)
date: 2019-11-15 08:47:16
tags: Swift
categories: Swift
---

作为一名swift萌新，已经深深爱上了这门语言，也认识到这门语言可没有看上去这么简单，于是总是想折腾一点不一样的东西，目的都是为了更好的理解swift这门语言

# python上的装饰器(Decorator)
python上的装饰器很酷，提供了@这个语法糖，当你想为你的函数提供一个新的功能，却不想对原来函数做改变的时候，这个时候装饰器的作用就来了，我们可以用下面的方式：

```
from functools import wraps
 
def log(func):
    @wraps(func)
    def with_logging(*args, **kwargs):
        print(func.__name__ + " was called")
        return func(*args, **kwargs)
    return with_logging
 
@log
def addition_func(x):
   return x + x

result = addition_func(4)
```

这样就为`addition_func`方法功能扩展了一个日志打印的功能

当然，其他语言也有自己的装饰器实现，但是对我来说，比较好理解的就是python的装饰器实现了。

从上面的代码也能看出来，把原来函数传进去，先执行装饰函数，再返回原来的函数，执行原来的函数。

# swift中的函数和闭包
首先`swift`中的函数和闭包，对我来说是等价的，无论是`func`还是`{}`，在我的理解下，都是等价的，他们有个函数名称，有一个参数列表，有一个返回值列表，并且他们都是`tuple`，也就是元组，就算是`void`，在`swift`中也是一个空元组`()`，这是一个很关键的部分。

# 泛型编程
泛型是通过参数化类型来实现在同一份代码上操作多种数据类型。利用“参数化类型”将类型抽象化，从而实现灵活的复用.
第一次听说泛型是在C++中，类似像这样的：

```
class Task<T> {
    public T obj;
    public Task(T t){
        this.obj = t;
    }     
 }
```
直到你实现这个具体类型之前，没人知道它到底是什么，这是我粗浅的理解，swift也提供了这样的能力，大概是这样：

```
func goToOnePiece<T>(product: T) {
    return T
}
```
swift的内置类型，函数，大范围运用了泛型编程，比如数组中的`element`

# 开始实现swift中的装饰器
有了函数和泛型这两个武器，我们开始实现了，首先第一步，给这个装饰器取个名字，叫`decorate`吧，初始的时候，这个函数应该是这样的：
```
func decorate<T, U>(_ function: @escaping(T) -> U, decoration: @escaping(T, U) -> U) -> (T) -> U {
    return { args in
        decoration(args, function(args))
    }
}
```
`function`就是原来的函数，`decoration`就是装饰函数，最终需要返回原来的`function`, `T`和`U`，分别是参数列表和返回值列表，也就是`tuple`

使用方法：
```
let add = decorate(+) { (args, rv) in
    print("In: \(args)")
    print("Out: \(rv)")
    return rv
}

let tuple = (2,4)
add(tuple)
```

不过这里，我还有点不满足，我想把`decorate`自定义成一个操作符`@`，可惜swift中已经有这个高级操作符了，所以，就作罢了，不过我的目的，也基本达到了，嘻嘻嘻，很快乐

当然，其实这个实现是没什么意义的，只是我个人的突发奇想，除了练习swift以外，确实没啥大用，哈哈，因为本身swift有更好的方式实现装饰器模式，比如用swift的`protocol`，使用起来更加的优雅


