---
title: iOS 符号表(dSYM)
date: 2019-04-30 14:07:14
tags: iOS
categories: iOS
---

今天是个适合学习的日子，嘻嘻嘻QAQ

# 什么是符号表
它是内存地址与函数名，文件名，行号的映射表。大概长这样：
` <起始地址> <结束地址> <函数> [<文件名:行号>] `

符号表文件`.dSYM`实际上是从Mach-O文件中抽取调试信息而得到的文件目录，实际用于保存调试信息的文件夹是`DWARF`

如果利用这些二进制的地址信息来定位问题是不可能的，因此我们需要将这些二进制的地址信息还原成源代码种的函数以及行号，这时候就需要符号表了。

特别地，如果使用bugly来做crash上报管理，只需要将构建时的符号表上传到bugly，当应用crash时，bugly会将crash信息上报到bugly，然后会自动替我们将原始的crash的二进制堆栈信息还原成包含行号的源代码文件信息，我们就可以快速定位问题。（好想用bugly QAQ）

# 符号表如何产生呢
生成符号表的步骤是在处理pinfo.plst文件之后，最初生成的符号表并不是在我们看到的归档文件内部，而是放在构建的一个临时目录中，最后才拷贝到归档目录下的，大概是这样，不过这其实没那么重要，了解一下。
## 1.Xcode自动生成
Xcode会在编译工程或者归档时自动为我们生成.dSYM文件，当然我们也可以通过更改Xcode的若干项Build Settings来阻止它那么干。
## 2.手动生成
另一种方式是通过命令行从Mach-O文件中手工提取，比如：
```
$ /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/dsymutil /Users/hahaha/Library/Developer/Xcode/DerivedData/YourApp-cqvijavqbptjyhbwewgpdmzbmwzk/Build/Products/Debug-iphonesimulator/YourApp.app/YourApp -o YourApp.dSYM
```
该方式通过Xcode提供的工具`dsymutil`，从项目编译结果`.app`目录下的Mach-O文件中提取出调试符号表文件。实际上Xcode也是通过这种方式来生成符号表文件。

**注意：debug配置默认不会生成符号表，并且每次构建时都会产生不同的符号表，每个符号表都有一个唯一的uuid，和每次构建对应**

# 如何定位dSYM文件

dSYM文件其实是一个带后缀的文件夹形式的文件，内容如下所示:

```
yourAppName.app.dSYM/
└── Contents
    ├── Info.plist
    └── Resources
        └── DWARF
            └── yourAppName
```

真实的符号表文件其实是一个二进制文件，bugly提供了脚本将这个二进制文件转为文本形式的文件，文件的内容其实就是二进制地址对和源代码文件，行号以及函数名字的对应关系

# 如何查看dSYM文件的uuid

iOS App崩溃时会有此次构建的uuid信息，如果要将崩溃堆栈还原成对应的源代码文件信息，需要根据这个uuid找到对应的符号表的uuid，这样才能正确还原

总结下来有2种方式:**dwarfdump**，**atos** (网上教程很多，使用起来也都很简单)

# CrashLog相关
有了符号表文件，有了崩溃日志文件，在解析之前一定要确保二者的对应关系，否则就算按照下述步骤解析出内容也肯定是不准确的。二者的对应关系可以通过UUID来确定。
其中，崩溃日志中的地址和符号表中的地址不是完全一样的，需要分别计算.
`运行时堆栈地址 = 运行时起始地址 + 偏移量`
`符号表堆栈地址 = 符号表起始地址 + 偏移量`

计算出地址，再利用xcode工具，就能很方便的定位crash位置了

# 原理简介
## DWARF简介
`DWARF`（DebuggingWith Arbitrary Record Formats），是ELF和Mach-O等文件格式中用来存储和处理调试信息的标准格式，`.dSYM`中真正保存符号表数据的是`DWARF`文件。`DWARF`中不同的数据都保存在相应的`section`（节）中，ELF文件里所有的`section`名称都以`".debug_"`开头，如下表所示：

```
| Section Name         | Contents                                          |
| -------------------- | ------------------------------------------------  |
| .debug_abbrev        | Abbreviations used in the .debug_info section     |
| .debug_aranges       | A mapping between memory address and compilation  |
| .debug_frame         | Call Frame Information                            |
| .debug_info          | The core DWARF data containing DIEs               |
| .debug_line          | Line Number Program                               |
| .debug_loc           | Macro descriptions                                |
| .debug_macinfo       | A lookup table for global objects and functions   |
| .debug_pubnames      | A lookup table for global objects and functions   | 
| .debug_pubtypes      | A lookup table for global types                   |
| .debug_ranges        | Address ranges referenced by DIEs                 |
| .debug_str           | String table used by .debug_info                  |
```

Mach-O中关于section的命名和ELF稍有区别，把名称前的.换成了_，例如`.debug_info`变成了`_debug_info`

## section信息提取
保存在DAWARF中的信息是高度压缩的，可以通过dwarfdump命令从中提取出可读信息。前文所述的那些section中，定位CrashLog只需要用到`.debug_info`和`.debug_line`。由于解析出来的数据量较大，为了方便查看，就将其保存在文本中

```
.debug_info
$ dwarfdump -e --debug-info YourPath/YourApp.dSYM/Contents/Resources/DWARF > info-e.txt
.debug_line
$ dwarfdump -e --debug-line YourPath/YourApp.dSYM/Contents/Resources/DWARF > line-e.txt
```

## 解析崩溃地址
`.debug_info`中最基本的描述单元为DIE（Debug Information Entry），首先我们要根据符号表崩溃地址从`.debug_info`中取出包含这个地址的DIE单元。
从上述DIE中我们可以获取到这些信息:
```
崩溃所在源码文件
发生崩溃的方法
发生崩溃的方法在源文件中的行号
```

截止目前，我们可以获取到发生了崩溃的方法的相关信息，但要想确定崩溃发生的具体行号，还需要`.debug_line`的帮助。

`.debug_line`以一个方法为基本块，即方法中每一行对应的符号表地址。通过`.debug_info`得知崩溃发生的方法地址范围，通过起始地址去解析`.debug_line`得到的`line-e.txt`中直接搜索即可得到崩溃所在方法的`.debug_line`数据

`.debug_line`段的第一行内容标识了该方法的起始符号表地址，行号及方法所在文件路径，通过之前得到的崩溃地址，即可得知崩溃行。

至此我们已经根据崩溃地址解析出了崩溃发生位置的详细信息：

```
崩溃所在源码文件
发生崩溃的方法
发生崩溃的方法在源文件中的行号
崩溃发生在源文件中得行号
```
















