---
layout: default
title: runtime
nav_order: 4
parent: 面试
---

## runtime原理


## runtime应用


## 分类实现原理

1、Category的实现原理，以及Category为什么只能加方法不能加属性?

分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。
Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。

分类中的属性在关联列表里

2、Category中有load方法吗？load方法是什么时候调用的？load方法能继承吗？
Category中有load方法，load方法在程序启动装载类信息的时候就会调用。load方法可以继承。调用子类的load方法之前，会先调用父类的load方法

3、load、initialize的区别，以及他们在catefory重写的时候的调用次序
区别在于调用方式和调用时刻
调用方式：load是根据函数地址直接调用，initialize是通过objc_msgSend调用
调用时刻：load是runtime加载类、分类的时候调用(只会调用1次)，initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）
调用顺序：先调用类的load方法，先编译那个类，就先调用load.在调用load之前会先调用父类的load方法。分类中load方法不会覆盖本类的load方法，先编译的分类优先调用load方法。initialize先初始化父类，之后再初始化子类。如果子类没有实现initialize,会调用父类的initialize,如果分类实现了initialize，就会覆盖类本身的initialize调用。




## 参考

- [iOS底层原理总结 - Category的本质](https://www.jianshu.com/p/fa66c8be42a2)
- 
