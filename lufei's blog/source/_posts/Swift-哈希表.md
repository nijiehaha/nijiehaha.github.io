---
title: Swift 哈希表
date: 2020-04-23 08:54:25
tags: Swift
categories: Swift
---

# 哈希

哈希函数，又被称为「散列算法」，是一种从任何一种数据中创建小的数字“指纹”的方法。

「哈希函数」把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来。该函数将数据打乱混合，重新创建一个叫做哈希值（ hash values，hash codes，hash sums，或 hashes）的“指纹”。

## 哈希碰撞

「哈希函数」是单向的，如果两个哈希值是不相同的，那么这两个哈希值的原始输入也是不相同的，但另一方面，哈希函数的输入和输出不是唯一对应关系的，如果两个哈希值相同，两个输入值很可能是相同的，但也可能不同。

这种「不同输入值通过哈希函数却得到相同输出」的情况称为「哈希碰撞」。

## 哈希洪水攻击

利用「哈希碰撞」的不可避免，制作出大量同样散列值的字符串，提交给服务器制作哈希表

# 哈希表

若关键字为 k，则其值存放在 f(k) 的存储位置上。由此，不需比较便可直接取得所查记录。称这个对应关系 f 为哈希函数，按这个思想建立的表为**哈希表**。

哈希表允许您通过 “键” 存储和检索对象

哈希表用于实现一些结构，例如字典，映射和关联数组。 这些结构可以通过树或普通数组实现，但使用哈希表效率更高。

简单来说，哈希表是一个数组，刚开始为空，之后将「值」放入某个「键」下的哈希表时，它使用该「键」计算数组中的索引。

就像上面说的那样，通过「键」计算数组的索引，就是使用的哈希函数来计算的。

## Swift 中的 哈希表

Swift 中的 Dictionary 内部就是使用「哈希表」来实现的。

哈希表使用「键」并询问它的 hashValue 属性。 因此，键必须符合 Hashable 协议。

但是 hashValue 返回的数字很大，还有可能是负数，那么如何使用这个大数字呢，一个普遍的做法是取它的绝对值，之后以数组长度做模运算，这个值就是数组的索引。

使用哈希的方式使得字典变得更为效率。在哈希表中查找一个值，你必须取键的哈希值来获取数组的下标，接下来就是查找对应下标的数组元素了。所有的这些操作所消耗的时间都是固定的，因此，新增，查询和删除的时间复杂度都是 O(1)。

> 备注：由于无法预估数组的最终长度。因此，字典无法保证哈希表中元素的有序性。

一个标准的 Swift 哈希表 的基础结构大概是这样的：

```

public struct HashTable<Key: Hashable, Value> {
    private typealias Element = (key: Key, value: Value)
    private typealias Bucket = [Element]
    private var buckets: [Bucket]
    
    private(set) public var count = 0
    public var isEmpty: Bool {
        return count == 0
    }
    
    public init(capacity: Int) {
        assert(capacity > 0)
        buckets = Array<Bucket>(repeating: [], count: capacity)
    }
}

```

由于将 Key 约束为 Hashable，所以字典中的所有键都有哈希值，所以你的字典无需担心计算哈希值。哈希表中的，主要的数组命名为 buckets，并通过 init(capacity) 方法提供了固定的长度。另外可以通过 count 属性来获取存入哈希表中的元素个数。

接下来，我们该可以实现一些基本操作：
+ 插入新的元素
+ 查找某个元素
+ 更新已经存在的元素
+ 删除一个元素

此处就不多说了

# 参考

[Swift 使用 SipHash 哈希函数的一个实现](https://github.com/attaswift/SipHash/blob/master/SipHash/SipHasher.swift)

[Swift 中的字符串如何转换成哈希值](https://github.com/apple/swift/blob/111499d2bfc58dc12fcb9cd1ce1dda7978c995b7/stdlib/public/core/StringHashable.swift)

[哈希表](https://zh.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E8%A1%A8)

[哈希洪水攻击](https://www.zhihu.com/question/286529973/answer/676290355)

[散列函数](https://zh.wikipedia.org/wiki/%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B8)

[Swift Algorithm Club: Hash Tables](https://www.raywenderlich.com/206-swift-algorithm-club-hash-tables)

[Swift 算法 - 哈希表](https://szewei.me/2018/08/22/swift-algorithm-hash-table/)

[哈希](https://swifter.tips/hash/)












