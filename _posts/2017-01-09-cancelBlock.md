---
layout: post
title: "[转]面试题: 创建一个可以被取消执行的block"
date: 2017-01-09 16:00:00.000000000 +08:00
tags: iOS开发面试题
---

## 题目

**我们知道block默认是不能被取消掉的, 请你封装一个可以被取消执行的block wrapper类, 定义如下**

```objective-c
typedef void(^Block)();

@interface CancelableObject : NSObject

- (id)initWithBlock:(Block)block;
- (void)start;
- (void)cancel;

@end
```

## 分析及答案

* 方法一: 创建一个类, 将要执行的block封装起来, 然后类的内部有一个\_isCancel变量, 在执行的时候, 检查这个变量, 如果\_isCancel被设置成了YES, 则退出执行.

  ```objective-c
  @implementation CancelableObject 

  {
      BOOL _isCanceled;
      Block _block;
  }

  - (id)initWithBlock:(Block)block 
  {
      self = [super init];
      if (self != nil) 
      {
          _isCanceled = NO;
          _block = block;
      }
      return self;
  }

  - (void)start 
  {
      __weak typeof(self) weakSelf = self;
      dispatch_async(dispatch_get_global_queue(0, 0), 
         ^{
          if (weakSelf) 
          {
             typeof(self) strongSelf = weakSelf;
             if (!strongSelf->_isCanceled) 
             {
                 (strongSelf->_block)();
             }
          }
      });
  }

  - (void)cancel 
  {
      _isCanceled = YES;
  }

  @end
  ```

* 方法二: 将要执行的block直接放到执行队列中, 但是让其在执行前检查检查另一个isCanceled的变量, 然后把这个变量的修改实现在另一个block方法中.

  ```objective-c
  typedef void (^CancelableBlock)();
  typedef void (^Block)();

  + (CancelableBlock)dispatch_async_with_cancelable:(Block)block 
  {
      __block BOOL isCanceled = NO;
      CancelableBlock cb = ^() {
          isCanceled = YES;
      };

      dispatch_async(dispatch_get_global_queue(0, 0), ^{
          if (!isCanceled) 
          {
              block();
          }
      });

      return cb;
  }
  ```

以上两种方法都只能在 block 执行前有效, 如果要在 block 执行中有效, 只能让 block 在执行中, 有一个机制来定期检查外部的变量是否有变化, 而要做到这一点, 需要改 block 执行中的代码. 在本例中, 如果 block 执行中的代码是通过参数传递进来的话, 似乎并没有什么办法可以修改它了.


