---
layout: post
title: "一些常见的面试题[持续更新]"
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

### 修饰变量常量的关键字

#### const

const关键字修饰常量, 苹果推荐使用const常量而不是用宏来定义常量, 因为, 宏只是在预编译阶段进行单纯的替换, 不会进行类型等的检查, 而const修饰的变量会在编译阶段进行检查. 大量使用宏会使预编译时间变长

const只是用来修饰右边的变量, 被const修饰的变量是只读的.

NSString const *str, NSString * const str, const NSString *str的区别:

NSString const *str和const NSString *str相同, const修饰的是 *str, 指str指针指向的内容是常量;  而NSString * const str, const修饰的是str, 指str指针是常量.

#### static和extern

static修饰局部变量时, 可以延长局部变量的生命周期, 直到程序结束才会销毁, 只会初始化一次, 生成一个内存. static修饰全局变量时, 可以避免重复定义全局变量.

extern只是用来获取全局变量的值, 不能用于定义变量. extern是先在当前文件查找有没有全局变量, 没有才会去其他文件查找.

### 其他

#### isa

isa是一个Class类型的指针, 每个实例对象都有个isa指针指向对象的类, 而Class里也有个isa的指针, 指向metaClass(元类). 元类保存了类方法的列表. 当类方法被调用时, 先会从本身查找类方法的实现, 如果没有, 元类会向他父类查找该方法. 元类也是类, 也有isa指针, 最终指向的是一个rootmetaClass(根元类), 根元类的isa指针指向它本身, 这样形成了一个封闭的内循环.

每一个对象本质上都是一个的实例, 其中类定义了成员变量和成员方法的列表, 对象通过对象的isa指针指向类.

每一个类本质上都是一个对象, 类其实是元类(metaClass)的实例, 元类定义了类方法的列表, 类通过类的isa指针指向元类.

所有的元类最终继承于一个根元类, 根元类isa指针指向本身, 行程一个封闭的内循环.

#### Class

Class是一个objc_class结构体类型的指针

```objective-c
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    // 父类, 如果该类已经是最顶层的根类, 那么它喂NULL 
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    // 该类的实例变量大小
    long instance_size                                       OBJC2_UNAVAILABLE;
    // 成员变量数组 
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */
```

#### id

id是一个objc_object结构体类型的指针

#### SEL

SEL是一个objc_selector结构体类型的指针, Objective-C在编译时会根据每个一个方法的名字, 参数序列生成一个唯一的整型标识, 这个标识就是SEL. 不同的类的实例对象执行相同的selector时, 会在各自的方法列表中去根据selector去寻找自己对应的IMP.

#### IMP

IMP实际上是一个函数指针, 指向方法实现的首地址. 

```objective-c
// 第一个参数是指向self的指针(如果是实例方法, 则是类实例的内存地址, 如果是类方法, 则是指向元类的指针)
// 第二个参数是方法选择器(selector)
// 接下来是方法的实际参数列表
id (*IMP)(id, SEL, ...)
```

通过取得IMP, 可以跳过runtime的消息传递机制, 直接执行IMP指向的函数实现, 这样省去了runtime消息传递过程中所做的一系列查找操作, 会比直接向对象发送消息高效一些.

#### Method

Method是一个objc_method结构体类型的指针

```objective-c
struct objc_method {
    // 方法名
    SEL method_name                	    OBJC2_UNAVAILABLE;  
    char *method_types                  OBJC2_UNAVAILABLE;
    // 方法实现
    IMP method_imp                      OBJC2_UNAVAILABLE;
}
```



## UI

### UIView和CALayer之间的关系

* UIView显示在屏幕上归功于CALayer, 通过调用drawRect方法来渲染自身的内容, 调节CALayer属性可以调整UIView的外观, UIView继承自UIResponder, CALayer不可以响应用户事件
* UIView是iOS系统中界面元素的基础, 所有的界面元素都继承自它。它内部是由Core Animation来实现的，它真正的绘图部分，是由一个叫CALayer(Core Animation Layer)的类来管理。UIView本身，更像是一个CALayer的管理器，访问它的根绘图和坐标有关的属性，如frame，bounds等，实际上内部都是访问它所在CALayer的相关属性
* UIView有个layer属性，可以返回它的主CALayer实例，UIView有一个layerClass方法，返回主layer所使用的类，UIView的子类，可以通过重载这个方法，来让UIView使用不同的CALayer来显示

## block

## 网络

## 多线程

## 运行时

### 运行时机制

#### 运行时概述

是一套比较底层的纯C语言API, 在我们平时编写的Objective-C代码中, 程序运行过程时, 其实都是转成了runtime的C语言代码.

##### 方法的动态绑定

在Objective-C中, 消息直到运行时才绑定到方法实现上, 编译器会将消息表达式[receiver message]转化为一个消息函数的调用, 这个函数将消息接收者和方法名作为其基础参数

```objective-c
objc_msgSend(receiver, selector, arg1, arg2, ....);
```

这个函数完成了动态绑定的所有事情:

* 首先按照receiver的类型找到selector对应的实现方法
* 调用方法的实现, 并将接收者对象及方法的所有参数传给它
* 将方法实现返回的值作为它自己的返回值

objc_msgSend通过对象的isa指针获取到类的objc_class结构体, 然后在方法分发表methodLists里查找selector, 如果没有则通过super_class找到其父类, 并在父类的方法分发表里面查找selector, 依次继续寻找, 一旦定位到selector, 函数就会获取到实现的入口, 并传入相应的参数来执行方法的具体实现. 如果最后没有定位到selector则会进行消息转发.

##### 消息转发

* 动态方法解析

  对象在接收到未知的消息时, 首先会调用

  ```objective-c
  // 解决类方法
  + (BOOL)resolveClassMethod:(SEL)sel OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

  // 解决实例方法
  + (BOOL)resolveInstanceMethod:(SEL)sel OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
  ```

  在这个方法中, 我们有机会为该未知消息新增一个已经实现了的处理方法


* 备用接收者

  如果在上一步无法处理消息, runtime会继续调用

  ```objective-c
  - (id)forwardingTargetForSelector:(SEL)aSelector OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
  ```

  如果一个对象实现了这个方法, 并返回了一个非空的结果, 则这个对象会作为消息的新接收者, 消息会被分发到这个对象, 这个对象不能是self自身, 否则就出现无限循环. 如果没有指定相应的对象来处理aSelector, 则应该调用父类的实现来返回结果

* 完整消息转发

  如果在上一步还不能处理未知消息, 则唯一能做的就是启用完整的消息转发机制了. 此时会调用

  ```objective-c
  // 1.定位可以相应封装在anInvocation中的消息对象, 这个对象不需要能处理所有未知消息
  // 2.使用anInvocation作为参数, 将消息发送到选中的对象. anInvocation将会保留调用结果, 运行时系统会提取这一结果并将其发送到消息的原始发送者
  - (void)forwardInvocation:(NSInvocation *)anInvocation OBJC_SWIFT_UNAVAILABLE("");
  ```

  运行时系统会在这一步给消息接收者最后一次机会将消息转发给其他对象, 对象会创建一个表示消息的NSInvocation对象, 把与没有处理的消息有关的全部细节都封装在anInvocation中, 包括selector, target和arguments. 我们可以在forwardInvocation: 方法中选择将消息转发给其他对象.

  必须重写为selector进行方法签名

  ```objective-c
  - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector OBJC_SWIFT_UNAVAILABLE("");
  ```

  消息转发机制使用从这个方法中获取的信息来创建NSInvocation对象, 因此必须重写这个方法, 为给定的selector提供一个合适的方法签名

  ```objective-c
  - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector 
  {
      NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
   
      if (!signature) 
      {
          if ([RuntimeHelper instancesRespondToSelector:aSelector]) 
          {
              signature = [RuntimeHelper instanceMethodSignatureForSelector:aSelector];
          }
      }
   
      return signature;
  }
   
  - (void)forwardInvocation:(NSInvocation *)anInvocation 
  {
      if ([RuntimeHelper instancesRespondToSelector:anInvocation.selector]) 
      {
          [anInvocation invokeWithTarget:_helper];
      }
  }
  ```

  NSObject的forwardInvocation: 方法只是简单调用了

  ```objective-c
  - (void)doesNotRecognizeSelector:(SEL)aSelector;
  ```

  它不会转发任何消息, 如果不在上面三个步骤中处理未知消息, 则会引发一个异常.

##### 消息转发与多重继承

通过forwardingTargetForSelector: 和 forwardInvocation: 这两个方法我们可以允许一个对象与其他对象建立关系, 以处理某些未知消息, 而表面上仍是该对象在处理消息. 通过这种关系, 可以实现多重继承的某些特性, 让对象可以继承其他对象的特性来处理一些事情.

不过, 这两者有一个重要的区别: 多重继承将不同的功能集成到一个对象中, 它会让对象变得过大, 而消息转发将功能分解到独立的小的对象中, 并通过某种方式将这些对象连接起来, 并做相应的消息转发

#### 运行时的应用

通过相关方法获取对象或者类的isa指针

* 动态创建一个类(比如KVO的底层实现)
* 动态为某个类添加属性/方法, 修改属性值/方法
* 遍历一个类的所有属性/所有方法

#### 相关方法

* 添加

  * 添加方法:

    ```objective-c
    OBJC_EXPORT BOOL class_addMethod(Class cls, SEL name, IMP imp, 
                                     const char *types) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
    ```

  * 添加实例变量:

    ```objective-c
    OBJC_EXPORT BOOL class_addIvar(Class cls, const char *name, size_t size, 
                                   uint8_t alignment, const char *types) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
    ```

  * 添加属性:

    ```objective-c
    OBJC_EXPORT BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)
        OBJC_AVAILABLE(10.7, 4.3, 9.0, 1.0);
    ```

  * 添加协议:

    ```objective-c
    OBJC_EXPORT BOOL class_addProtocol(Class cls, Protocol *protocol) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
    ```

* 获取

  * 获取方法:

    ```objective-c
    OBJC_EXPORT Method class_getClassMethod(Class cls, SEL name)
        OBJC_AVAILABLE(10.0, 2.0, 9.0, 1.0);
    ```

  * 获取方法信息:

    ```objective-c
    // 获取方法名
    OBJC_EXPORT SEL method_getName(Method m) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

    // 通过引用返回方法的返回值类型字符串
    OBJC_EXPORT void method_getReturnType(Method m, char *dst, size_t dst_len) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

    // 通过引用返回方法指定位置参数的类型字符串
    OBJC_EXPORT void method_getArgumentType(Method m, unsigned int index, 
                                            char *dst, size_t dst_len) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

    // 返回指定方法的方法描述结构体
    OBJC_EXPORT struct objc_method_description *method_getDescription(Method m) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

    // 获取描述方法参数和返回值类型的字符串
    OBJC_EXPORT const char *method_getTypeEncoding(Method m) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

    // 返回方法的实现
    OBJC_EXPORT IMP method_getImplementation(Method m) 
        OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);

    // 返回方法的参数的个数
    OBJC_EXPORT unsigned int method_getNumberOfArguments(Method m)
        OBJC_AVAILABLE(10.0, 2.0, 9.0, 1.0);

    OBJC_EXPORT unsigned method_getArgumentInfo(struct objc_method *m, int arg, const char **type, int *offset) OBJC2_UNAVAILABLE;

    OBJC_EXPORT unsigned int method_getSizeOfArguments(Method m) OBJC2_UNAVAILABLE;
    ```

  * 获取属性列表:

    ```objective-c

    ```

  * 获取属性信息:

    ```objective-c

    ```

  * 获取类信息:

    ````objective-c

    ````

  * 获取类属性列表:

    ```objective-c

    ```

* ​

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