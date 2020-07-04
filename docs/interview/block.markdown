---
layout: default
title: block
nav_order: 8
parent: interview
---

## block在内存中的结构
block的结构可以在 Runtime 的开源代码 Objc4-706 中找到，它位于 Block-private.h 中, 在内存中是以一个结构体的形式存在的，大致的结构如下：

~~~

struct Block_layout {
  /**
  block在内存中也是类NSObject的结构体，
  结构体开始位置是一个isa指针
  */
  void *isa;
  
  volatile int32_t flags;
  int32_t reserved;
  
  /**
  真正的函数指针！！
  */
  void (*invoke)(void *, ...);
  struct Block_descriptor_1 *descriptor;
  // imported variables 捕获的外部参数
  // ...
}

~~~

block中的isa指针，根据实际情况会有三种不同的取值，来表示不同类型的block:

1、_NSConcreteStackBlock

    栈上的block，一般block创建时是在栈上分配了一个block结构体的空间，然后对其中的isa等变量赋值。

2、_NSConcreteMallocBlock

    堆上的block，当block被加入到GCD或者被对象持有时，将栈上的block复制到堆上，此时复制得到的block类型变为了_NSConcreteMallocBlock。

3、_NSConcreteGlobalBlock

    全局静态的block，当block不依赖于上下文环境，比如不持有block外的变量、只使用block内部的变量的时候，block的内存分配可以在编译期就完成，分配在全局的静态常量区。


### 对每种类型的block调用copy操作后是什么结果？
- __NSGlobalBlock __ 调用copy操作后，什么也不做
- __NSStackBlock __ 调用copy操作后，复制效果是：从栈复制到堆；副本存储位置是堆
- __NSMallocBlock __ 调用copy操作后，复制效果是：引用计数增加；副本存储位置是堆

### 在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上的几种情况？
1.block作为函数返回值时
2.将block赋值给__strong指针时
3.block作为Cocoa API中方法名含有usingBlock的方法参数时
4.block作为GCD API的方法参数时


总结:
- 在block内部使用的是将外部变量拷贝到堆中的（基本数据类型直接拷贝一份到堆中，对象类型只将在栈中的指针拷贝到堆中并且指针指向的地址不变）。
- __block修饰符的作用：是将block用到的变量，拷贝到堆中，并且将外部的变量本身地址也改变到堆中。
- __block不能解决循环引用，需要在block执行尾部将变量设置为nil
- __weak可以解决循环引用，block在捕获weakObj时，会对weakObj指向的对象进行弱引用。
- 使用__weak时，可在block开始用局部__strong变量持有，以免block执行期间对象被释放。
- strongSelf是block内部的一个局部变量，变量的作用域仅限于局部代码，而程序一旦跳出作用域，strongSelf就会被释放，这个临时产生的“循环引用”就会被自动打破

## 参考
- [iOS-Block 浅谈](https://www.jianshu.com/p/25a7ba546eac)
- [iOS-Block本质](https://www.jianshu.com/p/4e79e9a0dd82)

