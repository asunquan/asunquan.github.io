---
layout: post
title: "[转改]面试题: 按层遍历二叉树的节点(Objective-C版)"
date: 2017-01-09 16:00:00.000000000 +08:00
tags: iOS开发面试题
---

## 题目

**给你一棵二叉树, 请按层输出其的节点值, 即: 从上到下, 从左到右的顺序. 例如, 如果给你如下一棵二叉树: **

```objective-c
	3
   /  \   
  9   20
     /  \
    15   7
```

输出结果应该是:

```objective-c
[
  [3],
  [9, 20],
  [15, 7]  
]
```

## 分析及答案

* 二叉树节点定义

  采用单链表的形式, 只从根节点指向子节点, 不保存父节点

  ```objective-c
  @interface BinaryTreeNode : NSObject

  /**
   该节点的值
   */
  @property (nonatomic, assign) NSInteger value;

  /**
   该节点的左节点
   */
  @property (nonatomic, strong) BinaryTreeNode *leftNode;

  /**
   该节点的右节点
   */
  @property (nonatomic, strong) BinaryTreeNode *rightNode;

  @end
  ```


* 本题出自LeetCode第102题, 是一个典型的有关遍历的题目. 为了按层遍历, 我们需要使用队列, 来将每一层的节点先保存下来, 然后再依次处理.

  因为我们不但需要按层来遍历, 还需要按层来输出结果, 所以我在代码中使用了两个队列，分别名为queueArray和 nextLevel，用于保存不同层的节点

  最终，整个算法逻辑是：

  1. 判断输入参数是否是为空。
  2. 将根节点加入到队列queueArray中。
  3. 如果queueArray不为空，则：
  4. 3.1 将queueArray加入到结果 ans 中。
  5. 3.2 遍历queueArray的左子节点和右子节点，将其加入到 nextLevel 中。
  6. 3.3 将 nextLevel 赋值给 level，重复第 3 步的判断。
  7. 将 ans 中的节点换成节点的值，返回结果。

  ```objective-c
  + (NSArray *)levelorderTraversalTree:(BinaryTreeNode *)rootNode
  {
      if (!rootNode)
      {
          return nil;
      }
      
      NSMutableArray *result = [NSMutableArray array];
      NSMutableArray *queueArray = [NSMutableArray array];
      
      // 插入根节点
      [queueArray addObject:rootNode];
      
      while (queueArray.count)
      {
          BinaryTreeNode *node = queueArray.firstObject;
          
          // 先进先出, 移除节点
          [queueArray removeObjectAtIndex:0];
          
          // 插入左节点
          if (node.leftNode)
          {
              [queueArray addObject:node.leftNode];
          }
          
          // 插入右节点
          if (node.rightNode)
          {
              [queueArray addObject:node.rightNode];
          }
      }
      
      return result;
  }
  ```


