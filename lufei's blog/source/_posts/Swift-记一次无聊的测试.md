---
title: Swift 记一次无聊的测试
date: 2020-07-10 15:52:33
tags: Swift
categories: Swift
---

# 奇怪的一段代码

```
class Test {

    lazy var label:TestLabel = {
        let label = TestLabel()
        return label
    }()

    deinit {
        print("页面销毁")
    }
    
    var testAttribute:() -> () { return {} }
    
    func testFunc() {}

}

var retainClosure:(() -> ())?
class TestLabel {
    func test(closure: @escaping () -> ()) {
        retainClosure = closure
    }
}
```

# 分别测试这段代码

测试 A

```
let test:Test? = Test()
test?.label.test(closure: test!.testAttribute)
```

控制台输出：页面销毁

测试 B

```
let test:Test? = Test()
test.label.test(closure: test.testFunc)
```
控制台无输出

# 分析

从结果来看，当把 testFunc 作为 Closure 传递给函数 `TestLabel.test` 的时候，testFunc 自动捕获了 test, 而把 testAttribute 作为 Closure 传递的时候，没有发生自动捕获 test 的行为，这应该是 Swift Class 中对待 Func 和 Closure Property 的区别？暂时还木有找到官方文档中的理论依据。

# 打印自动引用计数证实猜想

```
print(CFGetRetainCount(test as CFTypeRef))
```

通过两次引用计数对比，发现确实如上面所说，传递 testFunc 的时候，导致了 retainClosure 持有了 test，而 testAttribute 却没有影响

# 总结

得好好再读一下官方指南～～～QAQ，还有研究一下 Swift 编译前端～～～待续



