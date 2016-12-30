---
layout: post
title: "一些常见的面试题"
date: 2016-12-21 10:00:00.000000000 +08:00
tags: iOS开发面试题
---

[TOC]

# 前言

人挪死, 树挪活. 恰逢最近准备挪下窝, 虽然没有开始投递简历, 但是也看了些面试的内容, 做个总结, 权当为了以后复习节省时间.

# 面试题

## 基础

### 属性的修饰关键字

关于修饰属性的关键字可以分为以下几部分:

* 关于线程安全: atomic, nonatomic
* 关于读写权限: readwrite, readonly
* 关于属性内存: weak, strong, copy, assign, retain
* 关于对象为空: nullable, nonnull

#### nonatomic和atomic

atomic: 原子性, 默认修饰属性, 多线程安全, 能保证在别的线程来访问这个属性之前, 先执行完当前流程.

nonatomic: 非原子性, 禁止多线程安全, 相比atomic效率略高, 如有两个线程访问同一个属性, 会出现无法预料的结果.

atomic和nonatomic最主要的区别就是atomic会使编译器在setter/getter方法中自动生成一个互斥锁, 不会出现某一个线程执行完setter/getter方法之前另一个线程开始执行setter/getter的情况, 可以保证数据的完整性.

atomic的setter/getter方法实现

```objective-c
- (void)setAppVersion:(NSString *)appVersion
{
     @synchronized (self)
     {
          if (_appVersion != appVersion)
          {
               [_appVersion release];
               _appVersion = [appVersion retain];
          }
     }
}

- (NSString *)appVersion
{
    @synchronized (self)
    {
        return [[_appVersion retain] autorelease];
    }
}
```

nonatomic的setter/getter方法实现:

```objective-c
- (void)setAppVersion:(NSString *)appVersion
{
    if (_appVersion != appVersion)
    {
      	 [_appVersion release];
         _appVersion = [appVersion retain];
    }
}

- (NSString *)appVersion
{
    return _appVersion;
}
```

#### readwrite和readonly

readwrite: 可读可写权限, 默认修饰属性, 编译器自动生成setter和getter方法, 外部可以通过setter和getter方法对属性进行读写操作.

readonly: 只读权限, 编译器只自动生成getter方法而不生成setter方法, 外部只可以通过getter方法对属性进行读取操作.

readonly的属性只可以在该类内部中重写setter方法进行赋值, 如果我们想要改变这个值有两种解决办法:

* 开放一个给该属性赋值的方法供外部使用, 在这个方法的实现中调用该属性的setter方法.

* 可以使用KVC进行赋值操作, 如

  ```objective-c
  // people.name = @"Charles";	
  [people setValue:@"Charles" forKey:NSStringFromSelector(@selector(name))];
  ```

* 可以通过runtime的object_setIvar()方法进行赋值

  ```objective-c
  unsigned int count = 0;

  // 获取类中的所有成员属性
  Ivar *ivarList = class_copyIvarList(modelClass, &count);
      
  for (int i = 0; i < count; i++)
  {
    // 根据角标, 从数组取出对应的成员属性
    Ivar ivar = ivarList[i];
          
    // 获取成员属性名
    NSString *name = [NSString stringWithUTF8String:ivar_getName(ivar)];
    
    if ([name isEqualToString:@"_name"])
    {
      object_setIvar(people, ivar, @"Charles");
      break;
    }
  }
  ```

#### weak, strong, copy, assign, retain

weak: 表示的是一个弱引用, 这个引用不会增加对象的引用计数, 并且在所指向的对象被释放之后, weak指针会被自动置为nil, 使用weak关键字能有效的防止野指针的出现. 一般delegate属性会使用weak进行修饰.

strong: 表示的是一个强引用, 对象的引用计数+1.

copy: 赋值时将值拷贝到另一个内存空间, 引用计数为1, 不会影响原来内存空间的值, 适用于NSString, NSArray等不可变对象.

assign: 默认修饰属性, setter方法直接赋值, 不进行任何retain操作, 不改变移用计数. 该方法只会针对CGFloat, NSInteger等或int, float, double, char等.

retain: MRC下独有, 适用于Objective-C对象的成员变量. 每次retain时, 移用计数+1.

* weak和strong的区别

  weak是弱引用, strong是强引用. 当有strong类型的指针指向一个对象时, 该对象不会被释放. 当对象不再有strong类型的指针指向它时, 无论是否有weak类型的指针指向它, 它都会被从内存中释放, 同时指向该对象的weak类型的指针也会自动指向nil. 而strong类型的指针只能手动指向nil.

* weak和assign的区别

  weak和assign修饰指针时都是弱引用, 区别就是当weak类型的指针指向的对象被释放后, 该指针会自动指向nil, 而assign类型的指针不会自动指向nil, 这样就容易出现野指针的问题.

* strong和copy的区别

  strong是生成一个指针指向原来的内存空间, copy是生成一个新的内存空间, 内存空间中存放的内容和原来内存空间的内容一致, 修改copy生成的内存空间的内容不会改变原来的内存空间的内容.

* copy和mutableCopy

  当对可变类型的对象进行操作时, 都为深拷贝, 即拷贝的是该对象内存空间中存储的内容. 当对不可变类型的对象进行操作时, 如果目标对象为可变对象时, 则为深拷贝, 如果目标对象为不可变对象时, 则为浅拷贝, 即拷贝的只是该对象的指针.

#### nullable和nonnull

nullable: 可以为空, 表示该属性的值可以是NULL或nil.

nonnull: 禁止为空, 表示该属性必须有值.

nullable和nonnull是苹果在Xcode6.3中引入的新特性: nullability annotations. 为了方便制定多个属性是不可以为空, 苹果提供了一对宏, 处于这对宏中间的属性都不可以为空.

```objective-c
#define NS_ASSUME_NONNULL_BEGIN _Pragma("clang assume_nonnull begin")
#define NS_ASSUME_NONNULL_END   _Pragma("clang assume_nonnull end")
```

### 几个别的关键字

#### const

const关键字修饰常量, 苹果推荐使用const常量而不是用宏来定义常量, 因为, 宏只是在预编译阶段进行单纯的替换, 不会进行类型等的检查, 而const修饰的变量会在编译阶段进行检查. 大量使用宏会使预编译时间变长

const只是用来修饰右边的变量, 被const修饰的变量是只读的.

NSString const *str, NSString * const str, const NSString *str的区别:

NSString const *str和const NSString *str相同, const修饰的是 *str, 指str指针指向的内容是常量;  而NSString * const str, const修饰的是str, 指str指针是常量.

#### static和extern

static修饰局部变量时, 可以延长局部变量的生命周期, 直到程序结束才会销毁, 只会初始化一次, 生成一个内存. static修饰全局变量时, 可以避免重复定义全局变量.

extern只是用来获取全局变量的值, 不能用于定义变量. extern是先在当前文件查找有没有全局变量, 没有才会去其他文件查找.

## UI

### UIView和CALayer之间的关系

* UIView显示在屏幕上归功于CALayer, 通过调用drawRect方法来渲染自身的内容, 调节CALayer属性可以调整UIView的外观, UIView继承自UIResponder, CALayer不可以响应用户事件
* UIView是iOS系统中界面元素的基础, 所有的界面元素都继承自它。它内部是由Core Animation来实现的，它真正的绘图部分，是由一个叫CALayer(Core Animation Layer)的类来管理。UIView本身，更像是一个CALayer的管理器，访问它的根绘图和坐标有关的属性，如frame，bounds等，实际上内部都是访问它所在CALayer的相关属性
* UIView有个layer属性，可以返回它的主CALayer实例，UIView有一个layerClass方法，返回主layer所使用的类，UIView的子类，可以通过重载这个方法，来让UIView使用不同的CALayer来显示

## block

## 网络

## 多线程

## 运行时

### 运行时方法

#### 获取方法地址

```objective-c
// 获取description方法地址
Method description = class_getClassMethod(self, @selector(description));
```

#### 交换方法

交换方法实际上调用method_exchangeImplementations()方法, 交换两个方法的地址, 从而实现交换实现方法

```objective-c
// 获取description方法地址
Method description = class_getClassMethod(self, @selector(description));

// 获取csDescription方法地址
Method csDescription = class_getClassMethod(self, @selector(csDescription));

// 交换方法地址，相当于交换实现方式
method_exchangeImplementations(description, csDescription);
```

### 通过运行时实现让category支持属性

```objective-c
@interface NSObject (test)

@property (nonatomic, copy) NSString *name;

@end


@implementation NSObject (test)
  
// 定义关联的key
static const char *key = "name";

- (NSString *)name
{
    // 根据关联的key，获取关联的值。
    return objc_getAssociatedObject(self, key);
}

- (void)setName:(NSString *)name
{
    // 第一个参数：给哪个对象添加关联
    // 第二个参数：关联的key，通过这个key获取
    // 第三个参数：关联的value
    // 第四个参数:关联的策略
    objc_setAssociatedObject(self, key, name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```



## 设计模式

## 数据结构

## 算法