---
layout: post
title: "[转]面试题: 什么时候再block中不需要使用weakSelf"
date: 2016-12-01 20:00:00.000000000 +08:00
tags: iOS开发面试题
---

## 题目

**我们知道, 在使用 block 的时候, 为了避免产生循环引用, 通常需要使用 weakSelf 与 strongSelf, 写下面这样的代码：**

```objective-c
__weak typeof(self) weakSelf = self;
[self doSomeBlockJob:^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (strongSelf) {
        ...
    }
}];
```

**那么请问: 什么时候在 block 里面用 self, 不需要使用 weak self ?**

## 分析及答案

当 block 本身不被 self 持有, 而被别的对象持有, 同时不产生循环引用的时候, 就不需要使用 weak self 了. 最常见的代码就是 UIView 的动画代码, 我们在使用 UIView 的 animateWithDuration:animations 方法 做动画的时候, 并不需要使用 weak self, 因为引用持有关系是:

- UIView 的某个负责动画的对象持有了 block
- block 持有了 self

因为 self 并不持有 block, 所以就没有循环引用产生, 因为就不需要使用 weak self 了

```objective-c
[UIView animateWithDuration:0.2 animations:^{
    self.alpha = 1;
}];
```

当动画结束时, UIView 会结束持有这个 block, 如果没有别的对象持有 block 的话, block 对象就会释放掉, 从而 block 会释放掉对于 self 的持有. 整个内存引用关系被解除.

## 题目

**那么请问: 为什么 block 里面还需要写一个 strongSelf, 如果不写会怎么样?**

## 分析及答案

在 block 中先写一个 strongSelf 其实是为了避免 block 的执行过程中, 突然出现 self 被释放的尴尬情况. 通常情况下, 如果不这么做的话, 还是很容易出现一些奇怪的逻辑, 甚至闪退.

我们以 AFNetworking 中 AFNetworkReachabilityManager.m 的一段代码举例:

```objective-c
__weak __typeof(self)weakSelf = self;
AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
    __strong __typeof(weakSelf)strongSelf = weakSelf;

    strongSelf.networkReachabilityStatus = status;
    if (strongSelf.networkReachabilityStatusBlock) {
        strongSelf.networkReachabilityStatusBlock(status);
    }

};
```

如果没有 strongSelf 的那行代码, 那么后面的每一行代码执行时, self 都可能被释放掉了, 这样很可能造成逻辑异常. 

特别是当我们正在执行 strongSelf.networkReachabilityStatusBlock(status); 这个 block 闭包时, 如果这个 block 执行到一半时 self 释放, 那么多半情况下会 Crash.


这里有一篇文章详细解释了这个问题: [I finally figured out weakSelf and strongSelf](https://dhoerl.wordpress.com/2013/04/23/i-finally-figured-out-weakself-and-strongself/)

