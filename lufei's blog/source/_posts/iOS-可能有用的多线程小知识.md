---
title: iOS 可能有用的多线程小知识
date: 2020-10-29 15:03:50
tags: iOS
categories: iOS
---

# 前言

Grand Central Dispatch (简称 GCD) 是一套苹果提供的用来更方便的操作线程的 api。

基于 Grand Central Dispatch，我们可以了解一下，关于多线程的一些小知识。

# 一些概念

## Queues

中文可以翻译为「队列」，是一种先进先出（FIFO, First-in First-out）的数据结构。

## Serial vs Concurrent

这两个是反义词，主要是在描述当每项任务（task）被执行时，跟其他任务的关系。

+ Serial（照顺序执行的）

serial queues 的意思就是指这个队列里的任务是按照顺序执行的，一次执行一个，前一个执行完，才会执行下一个。

+ Concurrent（同时进行的）

也就是说 concurrent queues 代表这个队列里的任务会按照顺序「开始」执行，但是因为是 concurrent，不会等待上一个完了才执行下一个，因此每个任务执行结果的时间是不可预测的。

## Synchronous vs Asynchronous

+ Synchronous (同步)

一个 Synchronous 的 function 只有在里面的任务完成后才会回传值，也就是说，只有当前任务完成之后，当前线程才能继续接下来的任务，也就是平时所说的 Synchronous 的 function 会阻塞当前线程。

+ Asynchronous(异步)

相对于 Synchronous，asynchronous 的 function 会立刻回传值，asynchronous function 里的任务会按照顺序执行，但是 function 不会等待其他的任务执行完，它会马上回传值，因此 asynchronous function 不会造成所在的线程阻塞。

# 一些小知识

## 任务在普通队列里的时候

+ 只有 Async 才会开启新的线程

+ 在 Concurrent 的 Queue 下，用 Async 执行 Tasks 会开启多条线程

+ 在 Concurrent 的 Queue 下，用 Async 执行 Tasks 是速度最快的方式

## 任务在主队列的时候

+ 主队列的任务都会在主线程执行，不会开启新的线程

+ 主队列就算 Async，也不会开启新线程，而是会在主线程空闲的时候才会执行异步任务

+ 多个主队列异步的嵌套，会让异步任务在主线程的开始执行时间越来越迟

+ 多个主队列异步的任务，经过测试，它们的执行顺序，也是按照先进先出的原则来处理的

+ 最好不要在主线程异步做耗时任务，比较好的方式是开子线程操作，完成后回到主线程，比如下面这样：

```
DispatchQueue.global(qos: .default).async {
            
    /// 一些耗时操作
            
    /// 回到主线程
    DispatchQueue.main.async {
                
        /// 主线程操作
                
    }           
}
```

## 关于 Quality of service (QoS)

[QoS (QoSClass) ](https://developer.apple.com/documentation/dispatch/dispatchqos/qosclass) 是一个可以用来定义队列里的任务的执行顺序的 enum，如果没有定义也会有 default 值，当然主队列里的任务的执行优先级最高。

## 更简单的方式来获取主线程异步和子线程异步的队列

+ 主线程异步的队列

```
DispatchQueue.main.async {

}  
```

+ 子线程异步的队列

```
DispatchQueue.global().async {
            
}
```

# 参考

[Dispatch](https://developer.apple.com/documentation/DISPATCH)





