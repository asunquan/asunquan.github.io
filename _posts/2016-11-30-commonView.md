---
layout: post
title: "[转]面试题: 寻找最近公共View"
date: 2016-11-30 20:00:00.000000000 +08:00
tags: iOS开发面试题
---

## 前言

最近在巧叔的微信公众号上看到了巧叔开始更新了iOS开发面试题系列, 特此转载学习. 多谢巧叔. 闲话少说下面进入正题.

## 题目

**找出两个 UIView 的最近的公共 View, 如果不存在, 则输出 nil**

## 分析及答案

**这其实是数据结构里面的找最近公共祖先的问题**

一个 UIViewController 中的所有 view 之间的关系其实可以看成一颗树, UIViewController 的 view 变量是这颗树的根节点, 其它的 view 都是根节点的直接或间接子节点.

所以我们可以通过 view 的 superview 属性, 一直找到根节点. 需要注意的是, 在代码中, 我们还需要考虑各种非法输入, 如果输入了 nil, 则也需要处理, 避免异常. 以下是找到指定 view 到根 view 的路径代码:

```objective-c
+ (NSArray *)superViews:(UIView *)view 
{
    if (view == nil) 
    {
        return @[];
    }
  
    NSMutableArray *result = [NSMutableArray array];
    while (view != nil) 
    {
        [result addObject:view];
        view = view.superview;
    }
    return [result copy];
}
```

然后对于两个 view A 和 view B, 我们可以得到两个路径, 而本题中我们要找的是这里面最近的一个公共节点.

一个简单直接的办法: 拿第一个路径中的所有节点, 去第二个节点中查找. 

假设路径的平均长度是 N, 因为每个节点都要找 N 次, 一共有 N 个节点, 所以这个办法的时间复杂度是 O ( N ^ 2 ) 

```objective-c
+ (UIView *)commonView_1:(UIView *)viewA andView:(UIView *)viewB 
{
    NSArray *arr1 = [self superViews:viewA];
    NSArray *arr2 = [self superViews:viewB];
  
    for (NSUInteger i = 0; i < arr1.count; ++i) 
    {
        UIView *targetView = arr1[i];
        for (NSUInteger j = 0; j < arr2.count; ++j) 
        {
            if (targetView == arr2[j]) 
            {
                return targetView;
            }
        }
    }
    return nil;
}
```

一个改进的办法: 我们将一个路径中的所有点先放进 NSSet 中. 因为 NSSet 的内部实现是一个 hash 表, 所以查找元素的时间复杂度变成了 O (1), 我们一共有 N 个节点, 所以总时间复杂度优化到了 O (N).

```objective-c
+ (UIView *)commonView_2:(UIView *)viewA andView:(UIView *)viewB 
{
    NSArray *arr1 = [self superViews:viewA];
    NSArray *arr2 = [self superViews:viewB];
    NSSet *set = [NSSet setWithArray:arr2];
  
    for (NSUInteger i = 0; i < arr1.count; ++i) 
    {
        UIView *targetView = arr1[i];
        if ([set containsObject:targetView]) 
        {
            return targetView;
        }
    }
    return nil;
}
```

除了使用 NSSet 外, 我们还可以使用类似归并排序的思想.

用两个指针, 分别指向两个路径的根节点, 然后从根节点开始, 找第一个不同的节点, 第一个不同节点的上一个公共节点, 就是我们的答案.

代码如下

```objective-c
/* O(N) Solution */
+ (UIView *)commonView_3:(UIView *)viewA andView:(UIView *)viewB 
{
    NSArray *arr1 = [self superViews:viewA];
    NSArray *arr2 = [self superViews:viewB];
    NSInteger p1 = arr1.count - 1;
    NSInteger p2 = arr2.count - 1;
    UIView *answer = nil;
    
    while (p1 >= 0 && p2 >= 0) 
    {
        if (arr1[p1] == arr2[p2]) 
        {
            answer = arr1[p1];
        }
        p1--;
        p2--;
    }
    return answer;
}
```

我们还可以使用 UIView 的 isDescendant 方法来简化我们的代码, 不过这样写的话, 时间复杂度应该也是 O ( N ^ 2 ) 的. 

lexrus 提供了如下的 Swift 版本的代码：

```swift
// without flatMap
extension UIView {    
func commonSuperview(of view: UIView) -> UIView? {        
    if let s = superview {            
       if view.isDescendant(of: s) {                
                 return s
            } else {                
                 return s.commonSuperview(of: view)
            }
        }        
       return nil
    }
}
```

特别地, 如果我们利用 Optinal 的 flatMap 方法, 可以将上面的代码简化得更短, 基本上算是一行代码搞定. 

```swift
extension UIView {
    func commonSuperview(of view: UIView) -> UIView? {
        return superview.flatMap { 
           view.isDescendant(of: $0) ? 
             $0 : $0.commonSuperview(of: view) 
        }
    }
}
```


