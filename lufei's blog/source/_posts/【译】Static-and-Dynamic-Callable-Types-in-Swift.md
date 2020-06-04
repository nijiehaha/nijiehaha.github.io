---
title: 【译】Static and Dynamic Callable Types in Swift
date: 2020-02-25 09:31:20
tags: 翻译计划
categories: 翻译计划
copyright: false
---
# 原文
[Static and Dynamic Callable Types in Swift](https://nshipster.com/callable/)

# 正文

上周，苹果发布了[Xcode 11.4的第一个测试版本](https://developer.apple.com/documentation/xcode_release_notes/xcode_11_4_beta_release_notes)，这次更新也被证明为是近期更新中最重要的一次。`XCTest`(Xcode提供的用于测试的框架)得到了[巨大的推动](https://developer.apple.com/documentation/xcode_release_notes/xcode_11_4_beta_release_notes#3530390)，比如大量便于日常测试的改善，还有[模拟器](https://developer.apple.com/documentation/xcode_release_notes/xcode_11_4_beta_release_notes#3530393)，也得到了一些体贴的优化。但是Swift的变化引发了极大的关注。

在Xcode11.4中，swift的编译时间全面缩短（从许多开发者的回复来估计，大概在他们的项目中改善了10%到20%）。并且由于[新的语言检查平台](https://swift.org/blog/new-diagnostic-arch-overview/)的引入，编译器总能返回有用的错误信息。这也是第一个附带着新的[sourcekit-lsp服务器](https://nshipster.com/language-server-protocol/)的Xcode版本，该服务器使VSCode等编辑器可以更有意义地与Swift一起使用。

然而，尽管进行了所有这些改进（这确实是Apple开发人员工具团队的一项令人难以置信的成就），但许多早期反馈集中增加最明显的还是关于Swift 5.2。 并且来自Twitter，Hacker News和Reddit的的peanut galleries的回应-坦白地说-是“混乱的”。

如果您像我们大多数人一样，不适应[Swift演变](https://apple.github.io/swift-evolution/)的来龙去脉，那么Xcode 11.4就是您第一次接触到该语言的两个新功能：[key path expressions as functions](https://github.com/apple/swift-evolution/blob/master/proposals/0249-key-path-literal-function-expressions.md)和[callable values of user-defined nominal types](https://github.com/apple/swift-evolution/blob/master/proposals/0253-callable.md)。

第一个，就是用那些允许的键路径来替代通过函数展开只有一个表达式的闭包，比如所`map`

```
// Swift >= 5.2
"🧁🍭🍦".unicodeScalars.map(\.properties.name)
// ["CUPCAKE", "LOLLIPOP", "SOFT ICE CREAM"]

// Swift <5.2 equivalent
"🧁🍭🍦".unicodeScalars.map { $0.properties.name }
```

第二个，就是允许拥有`callAsFunction`方法的类型的实例被调用的时候，就好像它们是函数一样

```
struct Sweetener {
    let additives: Set<Character>

    init<S>(_ sequence: S) where S: Sequence, S.Element == Character {
        self.additives = Set(sequence)
    }

    func callAsFunction(_ message: String) -> String {
        message.split(separator: " ")
               .flatMap { [$0, "\(additives.randomElement()!)"] }
               .joined(separator: " ") + "😋"
    }
}

let dessertify = Sweetener("🧁🍭🍦")
dessertify("Hello, world!")
// "Hello, 🍭 world! 🍦😋"
```

当然，这两个例子都是可怕的。 那就是问题所在。

通常，大多数包含“swift新功能”的描述仅仅是对swift推进提案的反驳，充满动机不佳（且经常带有表情符号）的示例。这种处理方式不能很好地描述Swift语言的功能，并且在Swift 5.2的情况下，可以使流行的批评认为这些都是轻浮的补充-仅仅是语法糖。

(在某种程度上，我们对此感到内………我们的错)

本周，我们希望通过提供一些历史和理论背景来理解这些新功能，以达到该问题的中心。

# Swift中的语法糖

如果您对“key path as function”过于关注而感到厌烦，请回想一下，现状并非没有甜头。考虑一下以前的糖精示例：

```
"🧁🍭🍦".unicodeScalars.map { $0.properties.name }
```

该表达方式至少依赖于四个不同的语法的让步：
+ 尾随闭包语法，该函数允许省略函数的最终闭包参数标签
+ 匿名闭包参数，允许闭包中的参数按位置使用（$ 0，$ 1，…），而无需绑定到命名变量。
+ 推断参数和返回值类型
+ 单表达式闭包的隐式返回

如果您想完全减少饮食中的糖分，最好选择Mavis Beacon，因为您将进行更多打字。

```
"🧁🍭🍦".unicodeScalars.map(transform: { (unicodeScalar: Unicode.Scalar) -> String in
    return unicodeScalar.properties.name
})
```
(另外，谁知道map中的参数标签是“ transform”？)

实际上，正如我们在后面的示例中所看到的，从句法上讲，Swift是冬天的棉花糖世界。从初始化程序，方法调用到可选方法和方法链，几乎所有有关Swift的内容都可以描述为棉花糖的旋律-它实际上仅取决于您在“语言功能”和“语法糖”之间的界限。

要了解原因，您必须首先了解我们如何到达这里，这需要一些历史，数学和计算机科学。准备吃蔬菜

# λ演算和基础计算机科学发展遐想

所有编程语言都可以看作是表示λ演算的各种尝试。编写代码所需的一切-变量，绑定，应用程序-全部都在那里，被大量希腊字母和数学符号掩埋。

除了句法上的差异外，每种编程语言都可以通过其能力的组合来理解，以使程序更易于编写和阅读。对象，类，模块，可选，文字和泛型之类的语言功能都只是基于λ微积分的抽象。

与纯数学形式主义的任何其他偏离都可以归因于现实世界的限制，例如1870年代的打字机，1920年代的打孔卡，1940年代的计算机体系结构或1960年代的字符编码。

最早的编程语言是 Lisp ，ALGOL 和 COBOL ，几乎所有其他语言都源于这种语言。

(由于缺少易于访问的 ALGOL 环境，我们在这里使用 FORTRAN 作为替代。)

*Lisp*
```
(defun square (x)
    (* x x))

(print (square 4)) 
;; 16
```

*FORTRAN*
```
pure function square(x)
  integer, intent(in) :: x
  integer :: square
  square = x * x
end function

program main
  integer :: square
  print *, square(4)
end program main
! 16
```

*COBOL*
```
IDENTIFICATION DIVISION.
       PROGRAM-ID. example.
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  x   PIC 9(3) VALUE 4.
       01  y   PIC 9(9).
       PROCEDURE DIVISION.
           CALL "square" USING
               BY CONTENT x
               BY REFERENCE y.
           DISPLAY y.
           STOP RUN.
       END PROGRAM example.

IDENTIFICATION DIVISION.
       PROGRAM-ID. square.
       DATA DIVISION.
       LINKAGE SECTION.
       01  x   PIC 9(3).
       01  y   PIC 9(3).
       PROCEDURE DIVISION USING x, y.
           MULTIPLY x BY x GIVING y.
           EXIT PROGRAM.
       END PROGRAM square.
* 016000000
```

在这里，您可以了解三个截然不同的时间表；我们的现实是，ALGOL的语法（第二个）“胜过”了其他选择。在ALGOL 60中，您可以画一条直线，从1963年的CPL到1967年的BCPL和1972年的C，然后是1984年的Objective-C和2014年的Swift。这是一个沿袭，可以告诉您可调用的类型以及如何调用它们。

现在，我们回到swift

# Swift中的函数类型

在Swift中，函数是一等对象，这意味着它们可以分配给变量，存储在属性中，并作为参数传递或作为其他函数的值返回。

函数类型与其他值的区别在于它们是可调用的，这意味着您可以调用它们以产生新的值。

# 闭包

Swift的基本功能类型是闭包，即功能齐全的独立单元。

```
let square: (Int) -> Int = { x in x * x }
```

作为函数类型，您可以通过在开括号和闭括号之间传递必要数量的参数来调用闭包（）

> 函数类型采用的参数数量取决于它自身的元数

闭包之所以称为闭包，是因为它们关闭并从定义它们的上下文中捕获对任何变量的引用。但是，捕获语义并不总是令人满意的，这就是为什么Swift为特殊的闭包（称为函数）提供专用语法的原因。

# 函数

在顶级/全局范围内定义的函数称为不捕获任何值的闭包。在Swift中，您可以使用func关键字声明它们：

```
func square(_ x: Int) -> Int { x * x }
square(4) // 16
```

与闭包相比，函数在传递参数方面具有更大的灵活性。

函数参数可以使用命名标签，而不是闭包中未标记的位置参数-这对说明代码在其调用位置的作用大有帮助：

```
func deposit(amount: Decimal,
             from source: Account,
             to destination: Account) throws { … }
try deposit(amount: 1000.00, from: checking, to: savings)
```

函数可以是通用的，允许将它们用于多种类型的参数：

```
func square<T: Numeric>(_ x: T) -> T { x * x }
func increment<T: Numeric>(_ x: T) -> T { x + 1 }
func compose<T>(_ f: @escaping (T) -> T, _ g: @escaping (T) -> T) -> (T) -> T {
    { x in g(f(x)) }
}

compose(increment, square)(4 as Int) // 25 ((4 + 1)²)
compose(increment, square)(4.2 as Double) // 27.04 ((4.2 + 1)²)
```

函数还可以采用可变参数，隐式闭包和默认参数值（允许使用`#file`和`#line`这样的魔术表达式文字）：

```
func print(items: Any...) { … }

func assert(_ condition: @autoclosure () -> Bool,
            _ message: @autoclosure () -> String = String(),
            file: StaticString = #file,
            line: UInt = #line) { … }
```

但是，尽管在接受参数方面具有所有这种灵活性，但是您将遇到的大多数函数都是在隐式自变量上运行的。这些功能称为方法。

# 方法

方法是一种类型所包含的功能。方法自动提供对self的访问权限，从而使它们可以有效地捕获被称为隐式参数的实例。

```
struct Queue<Element> {
    private var elements: [Element] = []

    mutating func push(_ newElement: Element) {
        self.elements.append(newElement)
    }

    mutating func pop() -> Element? {
        guard !self.elements.isEmpty else { return nil }
        return self.elements.removeFirst()
    }
}
```

> Swift未来会更进一步，当成员访问的时候，可以省略`self.`，使已经隐含的`self.`变得更加隐含。

通过将所有语法组合在一起，可以使Swift代码表达性强，表达清晰，简洁：

```
var queue = Queue<Int>()
queue.push(1)
queue.push(2)
queue.pop() // 1
```

与Objective-C等更冗长的语言相比，编写Swift的经验非常好。很难想象会有任何Swift开发人员反对我们这里所说的“糖衣”。

但是就像16盎司的Surge罐头一样，某些东西的含糖量通常令人惊讶。事实证明，以前的例子远非无辜：

```
var queue = Queue<Int>() // desugars to `Queue<Int>.init()`
queue.push(1) // desugars to `Queue.push(&queue)(1)`
```

一直以来，我们所谓的对方法和初始化器的“直接”调用实际上是使用部分应用函数的函数的简写形式.

> 经常将部分应用程序和currying混为一谈。实际上，它们是不同但相关的概念。
  Swift的早期版本具有用于currying函数的专用语法，但事实证明它没有最初预期的有用，并且已被第二次Swift Evolution提议删除。
  ```
  // Swift <3:
  func curried(x: Int)(y: String) -> Float {
    return Float(x) + Float(y)!
  }

  // Swift >=3
  func curried(x: Int) -> (String) -> Float {
    return { (y: String) -> Float in
        return Float(x) + Float(y)!
    }
  }
  ```

考虑到这一点，让我们现在更笼统地研究Swift中的可调用类型。

  
# {Type, Instance, Member} ⨯ {Static, Dynamic}

自从分别在Swift 4.2和Swift 5中引入以来，许多开发人员一直难以保持@dynamicMemberLookup和@dynamicCallable的直觉-在Swift 5.2中引入callAsFunction变得更加困难。

如果您也感到困惑，我们认为下表可以帮助您解决问题：

|           | Static           | Dynamic                |
| --------- | ---------------  | ---------------------- |
| Type      | `init`           | N/A     |
| Instance  | `callAsFunction` | `@dynamicCallable`     |
| Member    | `func`           | `@dynamicMemberLookup` |

Swift一直都有静态的可调用类型和类型成员。新版Swift中的变化是实例现在可以调用，实例和成员都可以动态调用。

> 您可能已经注意到我们表中的空白处。确实，无法动态调用类型。实际上，除了调用初始化程序外，没有其他方法可以静态地调用类型，这可能是最好的。

让我们看看从静态可调用对象开始的实际含义。

# Static Callable

```
struct Static {
    init() {}

    func callAsFunction() {}

    static func function() {}
    func function() {}
}
```

可以通过以下方式静态调用此类型：

```
let instance = Static() // ❶ desugars to `Static.init()`

Static.function() // ❷ (no syntactic sugar!)
instance.function() // ❸ desugars to Static.function(instance)()

instance() // ❹ desugars to `Static.callAsFunction(instance)()`
```

❶ 调用静态类型将调用初始化程序

❷ 在Static类型上调用函数将调用相应的static函数成员，并将Static作为隐式的self参数传递。

❸ 在Static实例上调用函数将调用相应的函数成员，并将该实例作为隐式的self参数传递。

❹ 调用Static的实例将调用callAsFunction（）函数成员，并将该实例作为隐式的self参数传递。

> 为了完整起见，请注意以下几点：
+ 您还可以静态调用下标和变量成员（属性）。
+ 运算符提供了另一种调用静态成员函数的方法。
+ 枚举情况是……完全是另外一回事。

# Dynamic Callable

```
@dynamicCallable
@dynamicMemberLookup
struct Dynamic {
    func dynamicallyCall(withArguments args: [Int]) -> Void { () }
    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Void { () }

    static subscript(dynamicMember member: String) -> (Int) -> Void { { _ in } }
    subscript(dynamicMember member: String) -> (Int) -> Void { { _ in } }
}
```

可以通过几种不同的方式动态调用此类型：

```
let instance = Dynamic() // desugars to `Dynamic.init()`

instance(1) // ❶ desugars to `Dynamic.dynamicallyCall(instance)(withArguments: [1])`
instance(a: 1) // ❷ desugars to `Dynamic.dynamicallyCall(instance)(withKeywordArguments: ["a": 1])`

Dynamic.function(1) // ❸ desugars to `Dynamic[dynamicMember: "function"](1)`
instance.function(1) // ❹ desugars to `instance[dynamicMember: "function"](1)`
```

❶ 调用Dynamic的实例将调用dynamicCall（withArguments :)方法，并传递参数数组并将Dynamic作为隐式的self参数传递。

❷ 用至少一个带标签的参数调用Dynamic的实例将调用dynamicCall（withKeywordArguments :)方法，将参数传递给KeyValuePairs对象，并将Dynamic作为隐式的self参数传递。
 
❸ 在Dynamic类型上调用function会调用static dynamicMember下标，并以'function'作为键；在这里，我们称为返回的匿名闭包。

❹ 在Dynamic实例上调用function会调用dynamicMember下标，并将“ function”作为键；在这里，我们称为返回的匿名闭包。

# 通过声明属性的Dynamism

`@dynamicCallable`和`@dynamicMemberLookup`是声明属性，这意味着它们无法通过扩展名应用于现有声明。

因此，例如，您无法使用带有Ruby风格的自然语言访问器为Int增添趣味：

```
@dynamicMemberLookup // ⚠︎ Error: '@dynamicMemberLookup' attribute cannot be applied to this declaration
extension Int {
    static subscript(dynamicMember member: String) -> Int? {
        let string = member.replacingOccurrences(of: "_", with: "-")

        let formatter = NumberFormatter()
        formatter.numberStyle = .spellOut
        return formatter.number(from: string)?.intValue
    }
}

// ⚠︎ Error: Just to be super clear, this doesn't work
Int.forty_two // 42 (hypothetically, if we could apply `@dynamicMemberLookup` in an extension)
```

将此与`callAsFunction`进行比较，后者可以添加到扩展中的任何类型。

`@dynamicMemberLookup`，`@dynamicCallable`和`callAsFunction`还有很多要讨论的地方，我们希望在以后的文章中更详细地介绍它们。

# Swift ⨯ _______

添加到我们的“代码是什么样的”列表中：

> Code is like fan fiction.

有时要交付软件，您需要配对并“交付”不同的技术。

从某种意义上说，Swift的故事是现代计算机中最伟大的悲剧性恋情之一。 我们还能如何描述Objective-C牺牲自身以使Swift成为可能的方式？

在构建这些功能时，Swift取代了Python成为机器学习的“力量”。 理所当然的认为增量方法是最好的，实现这一目标的方法是允许Swift与Python像与Objective-C一样无缝地互操作。 从Swift 4.2开始，我们已经非常接近了。

```
import Python

let numpy = Python.import("numpy")
let zeros = numpy.ones([2, 4])
/* [[1, 1, 1, 1]
    [1, 1, 1, 1]] */
```

# Dynamism的外部影响

附加更改的承诺是，如果您不希望它们进行任何更改，它们不会更改任何内容。您可以继续编写Swift代码，而完全不了解本文中介绍的功能（到目前为止，我们大多数人都知道）。但请明确一点：没有免费的抽象。

经济学使用负外部性一词来描述决策产生的间接成本。尽管您不使用这些功能就不必付费，但是我们所有人都负担着更复杂的语言的负担，这种语言更加难以教授，学习，记录和推理。

从一开始就加入Swift的许多人对Swift Evolution感到厌倦。对于那些向外看的人来说，这是不可思议的，我们正在浪费时间在这种无关紧要的“糖”上，而不是像`async` / `await`那样真正能够移动针头的功能。

孤立地，这些建议中的每一个都是真正的周到且有用的。我们已经有机会使用其中一些。但是，当他们沉迷于情感的包裹中时，很难凭自己的技术优势来判断事物。

每个人都有自己的糖耐量，通常会根据他们的习惯来了解。意识到吊桥效应，老实说，我无法断定我是否失去联系，还是孩子错了……

# 翻译感受

讲实话，这篇文章，对我这个小菜鸟来说有点难了，所以建议还是看英文原文比较好，翻译只是锻炼我的英文能力。
