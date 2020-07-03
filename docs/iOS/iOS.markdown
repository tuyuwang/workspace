---
layout: default
title: iOS
nav_order: 14
permalink: /docs/iOS
has_children: true
---

#### constructor / destructor
加上这两个属性的函数会在分别在可执行文件（或 shared library）load 和 unload 时被调用，可以理解为在 main() 函数调用前和 return 后执行

若有多个 constructor 且想控制优先级的话，可以写成 attribute((constructor(101)))，里面的数字越小优先级越高，1 ~ 100 为系统保留。

- [__attribute__((constructor))用法解析](https://www.jianshu.com/p/dd425b9dc9db)
- [CaptainHook浅析](https://www.jianshu.com/p/8834b4ce8781)

#### weak的实现原理

那么 runtime 如何实现 weak 变量的自动置nil？

runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

#### copy与block的原理


#### copy关键字的作用