---
title: 【译】Xcode Build Configuration Files
date: 2020-03-04 09:50:09
tags: 翻译计划
categories: 翻译计划
copyright: false
---

# 原文

[Xcode Build Configuration Files](https://nshipster.com/xcconfig/)

# 正文

软件开发最佳实践严格[规定](https://12factor.net/config)配置应与代码分隔开。但是，Apple平台上的开发人员经常难以将这些准则与Xcode的项目繁重的工作流程相提并论。

了解每个项目设置的功能以及它们之间如何交互是一项需要花费数年时间才能磨练的技能。但是事实上，关于这些的信息都隐藏在Xcode的图形交互中，这对我们了解这些没有帮助。

点击项目编辑页面的“Build Settings”标签，你会很吃惊的看到几百个关于projects, targets, 和configurations的构建设置分布在各个层级里，这还不包括其他六个选项卡。

<img src="https://nshipster.com/assets/xcconfig-project-build-settings--dark-e07ddeb7133a1c8247b7d53f9db7c0be6dd7728dfa4b58c8a6ec3ebbec4ea596.png">

幸运的是，有一种更好的方法来管理这些配置，而无需通过点击一系列迷宫般的选项卡和显示箭头。

本周，我们将向您展示如何使用基于文本的`xcconfig`文件从Xcode外部化构建设置，以使您的项目更加紧凑，易懂且功能强大。

> 请访问[XcodeBuildSettings.com](https://xcodebuildsettings.com/)，以获取最新版本的Xcode支持的每个构建设置的完整参考。

[Xcode构建配置文件](https://help.apple.com/xcode/mac/8.3/#/dev745c5c974)，通常以其`xcconfig`文件扩展名而闻名，它允许在不使用Xcode的情况下声明和管理应用程序的构建设置。它们是纯文本格式，这意味着它们对源代码控制系统更加友好，并且可以使用任何编辑器进行修改。

从根本上讲，每个配置文件都由一系列键值分配组成，这些键值分配具有以下语法：

```
[BUILD_SETTING_NAME] = [value]
```

例如，要为项目指定Swift语言版本，您可以指定`SWIFT_VERSION`构建设置，如下所示：

```
SWIFT_VERSION = 5.0
```

> 根据[posix标准](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap08.html#tag_08)，环境变量的名称仅由大写字母，数字和下划线`（_）`组成，这是我喜欢称之为`SCREAMING_SNAKE_CASE`的约定

乍一看，`xcconfig`文件以简单的换行符分隔的语法与`.env`文件极为相似。但是，Xcode构建配置文件不仅仅让人眼前一亮。看哪！

# 持有已存在的值

要附加而不是替换现有定义，请使用`$（inherited）`变量，如下所示：

```
BUILD_SETTING_NAME = $(inherited)additional_value
```

通常，您这样做是为了建立值列表，例如编译器在其中搜索框架以查找包含的头文件（FRAMEWORK_SEARCH_PATHS`）的路径：

```
FRAMEWORK_SEARCH_PATHS = $(inherited) $(PROJECT_DIR)
```

Xcode按以下顺序分配继承的值（从最低到最高优先级）：

+ 平台默认值
+ Xcode项目xcconfig文件
+ Xcode项目文件构建设置
+ Target xcconfig 文件
+ Target Build Settings

> 空格用于分隔字符串和路径列表中的项目。要指定包含空格的项目，必须用引号`（“）`引起来。

# 引用值

您可以使用以下语法将其他设置中的值替换为其声明名称：

```
BUILD_SETTING_NAME = $(ANOTHER_BUILD_SETTING_NAME)
```

替代可用于根据现有值定义新变量，或内联以动态建立新值。

```
OBJROOT = $(SYMROOT)
CONFIGURATION_BUILD_DIR = $(BUILD_DIR)/$(CONFIGURATION)-$(PLATFORM_NAME)
```

# 为引用的构建设置设置回退值

```
$(BUILD_SETTING_NAME:default=value)
```

# 条件化构建设置

您可以根据以下语法根据其SDK（`sdk`），体系结构（`arch`）和/或配置（`config`）来对构建设置进行条件化：

```
BUILD_SETTING_NAME[sdk=sdk] = value_for_specified_sdk
BUILD_SETTING_NAME[arch=architecture] = value_for_specified_architecture
BUILD_SETTING_NAME[config=configuration] = value_for_specified_configuration
```

给定相同构建设置的多个定义之间的选择，编译器将根据具体情况进行解析。

```
BUILD_SETTING_NAME[sdk=sdk][arch=architecture] = value_for_specified_sdk_and_architectures
BUILD_SETTING_NAME[sdk=*][arch=architecture] = value_for_all_other_sdks_with_specified_architecture
```

例如，您可以指定以下构建设置以通过仅针对活动体系结构进行编译来加快本地构建：

```
ONLY_ACTIVE_ARCH[config=Debug][sdk=*][arch=*] = YES
```

# 包括来自其他配置文件的构建设置

一个构建配置文件可以包含来自其他配置文件的设置，使用与此功能所基于的等效C指令相同的`#include`语法:

```
#include "path/to/File.xcconfig"
```

正如我们将在本文后面看到的，您可以利用这一点以非常强大的方式构建构建设置的级联列表.

> 通常，当编译器遇到不能解析的#include指令时，它会引发一个错误。但是xcconfig文件也支持#include?指令，如果找不到文件，也不会报错。<br><br>在很多情况下，您都不希望文件的存在或不存在来更改编译时行为;毕竟，构建在可预测的时候是最好的。但是你可以把它作为一个钩子来使用可选的开发工具，比如Reveal，它需要以下配置:<br>
```
# Reveal.xcconfig
OTHER_LDFLAGS = $(inherited) -weak_framework RevealServer
FRAMEWORK_SEARCH_PATHS = $(inherited) /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries
```

# 创建构建配置文件

创建一个构建配置文件,选择“File > New File…”菜单项(⌘n),向下滚动到部分标记为“Other”,并选择配置文件模板。接下来，将其保存在项目目录中的某个位置，确保将其添加到所需的目标

<img src="https://nshipster.com/assets/xcconfig-new-file--dark-634ea6d9c8b32c825cf6e9edc66807943d5650a5bfc2cc081d0fe4d37c49c9ad.png">

一旦创建了`xcconfig`文件，就可以将其分配给与其相关的目标的一个或多个构建配置。

<img src="https://nshipster.com/assets/xcconfig-project-configurations--dark-61210a47703d2f987efe90262006132f06415f2d15c68f7d719c728188995343.png">

> 构建配置文件不应该包含在任何项目目标中。如果您在应用程序的`.ipa`存档中发现任何.`xcconfig`文件，请确保它们不是任何目标的成员，也不会出现在任何“复制包资源”构建阶段。

现在我们已经介绍了使用Xcode构建配置文件的基础知识，接下来让我们看几个示例，看看如何使用它们来管理开发、阶段和生产环境。

# 自定义内部构建的应用程序名称和图标

开发iOS应用程序通常需要在模拟器和测试设备(以及应用程序商店的最新版本，以供参考)上调整各种内部构建。

您可以使用`xcconfig`文件简化工作，这些文件为每个配置分配一个不同的名称和应用程序图标。

```
// Development.xcconfig
PRODUCT_NAME = $(inherited) α
ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon-Alpha

//////////////////////////////////////////////////

// Staging.xcconfig
PRODUCT_NAME = $(inherited) β
ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon-Beta
```

# 跨不同环境管理常量

如果您的后端开发人员根据前面提到的[12 Factor App](https://12factor.net/config)原则进行配置，那么他们将为开发、阶段和生产环境提供独立的端点。

在iOS上，管理这些环境最常见的方法可能是将条件编译语句与诸如`DEBUG`之类的构建设置一起使用。

```
import Foundation

#if DEBUG
let apiBaseURL = URL(string: "https://api.staging.example.com")!
#else
let apiBaseURL = URL(string: "https://api.example.com")!
#endif
```

这就完成了工作，但是与代码/配置分离的标准相冲突。

另一种方法采用这些特定于环境的值，并将它们放在它们所属的位置—`xcconfig`文件中。

```
// Development.xcconfig
API_BASE_URL = api.staging.example.com

//////////////////////////////////////////

// Production.xcconfig
API_BASE_URL = api.example.com
```

> `xcconfig`文件将序列`//`作为一个注释分隔符，不管它是否用引号括起来。如果您尝试使用反斜杠`\/\/`进行转义，那么这些反斜杠将按字面意思显示，并且必须从结果值中删除。这在指定每个环境的URL常量时特别不方便。<br><br>如果您不希望处理这种不幸的行为，可以在代码中省略模式并在前面加上`https://`。(您使用的是`https`…对吗?)

然而，要以编程方式获取这些值，我们需要采取一个额外的步骤:

# 从Swift访问构建设置

Xcode项目文件，`xcconfig`文件和环境变量定义的构建设置仅在构建时可用。当您运行编译的应用程序时，周围的上下文均不可用。 （为此感谢上帝！）

但是，请稍等一会儿-您还记得在其他选项卡之一中之前看到过一些构建设置吗？信息，是吗？

碰巧的是，“信息”标签实际上只是目标的`Info.plist`文件的精美呈现。在构建时，该`Info.plist`文件将根据提供的构建设置进行编译，并复制到生成的应用程序包中。因此，通过添加对$（API_BASE_URL）的引用，您可以通过`Foundation Bundle API`的`infoDictionary`属性访问这些设置的值。整洁！

<img src="https://nshipster.com/assets/xcconfig-project-info-plist--dark-de7dfd79b9d23bd111cbab3b268626b74b6f5a70c9bc53e3a0466aa2e5b36df6.png">

按照这种方法，我们可以做如下事情:

```
import Foundation

enum Configuration {
    enum Error: Swift.Error {
        case missingKey, invalidValue
    }

    static func value<T>(for key: String) throws -> T where T: LosslessStringConvertible {
        guard let object = Bundle.main.object(forInfoDictionaryKey:key) else {
            throw Error.missingKey
        }

        switch object {
        case let value as T:
            return value
        case let string as String:
            guard let value = T(string) else { fallthrough }
            return value
        default:
            throw Error.invalidValue
        }
    }
}

enum API {
    static var baseURL: URL {
        return try! URL(string: "https://" + Configuration.value(for: "API_BASE_URL"))!
    }
}
```

当从调用站点查看时，我们发现这种方法与我们的最佳实践完美地协调在一起——没有一个硬编码的常量在视线中!

```
let url = URL(string: path, relativeTo: API.baseURL)!
var request = URLRequest(url: url)
request.httpMethod = method
```

> 不要使用`xcconfig`文件来存储API密匙或其他凭证之类的秘密，更多信息请参考我们关于iOS上的秘密管理的文章。

Xcode项目是单一的、脆弱的和不透明的。它们是团队成员之间协作的摩擦来源，通常也是工作的累赘。

幸运的是，`xcconfig`文件花了很长时间来解决这些问题。将配置从Xcode移到`xcconfig`文件中会带来很多好处，并且提供了一种方法，可以使您的项目与Xcode的细节保持一定距离，也离开了需要cupertino批准的“快乐路径”。
