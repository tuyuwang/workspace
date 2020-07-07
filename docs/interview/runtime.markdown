---
layout: default
title: runtime
nav_order: 4
parent: interview
---

## runtime原理

~~~
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                       OBJC2_UNAVAILABLE;  // 父类
    const char *name                        OBJC2_UNAVAILABLE;  // 类名
    long version                            OBJC2_UNAVAILABLE;  // 类的版本信息，默认为0
    long info                               OBJC2_UNAVAILABLE;  // 类信息，供运行期使用的一些位标识
    long instance_size                      OBJC2_UNAVAILABLE;  // 该类的实例变量大小
    struct objc_ivar_list *ivars            OBJC2_UNAVAILABLE;  // 该类的成员变量链表
    struct objc_method_list **methodLists   OBJC2_UNAVAILABLE;  // 方法定义的链表
    struct objc_cache *cache                OBJC2_UNAVAILABLE;  // 方法缓存
    struct objc_protocol_list *protocols    OBJC2_UNAVAILABLE;  // 协议链表
#endif
} OBJC2_UNAVAILABLE;
~~~

在上面的objc_class结构体中，ivars是objc_ivar_list（成员变量列表）指针；methodLists是指向objc_method_list指针的指针。在Runtime中，objc_class结构体大小是固定的，不可能往这个结构体中添加数据，只能修改。所以ivars指向的是一个固定区域，只能修改成员变量值，不能增加成员变量个数。methodList是一个二维数组，所以可以修改*methodLists的值来增加成员方法，虽没办法扩展methodLists指向的内存区域，却可以改变这个内存区域的值（存储的是指针）。因此，可以动态添加方法，不能添加成员变量。

## runtime应用


## 分类实现原理

~~~
Category 是表示一个指向分类的结构体的指针，其定义如下：
typedef struct objc_category *Category;
struct objc_category {
  char *category_name                          OBJC2_UNAVAILABLE; // 分类名
  char *class_name                             OBJC2_UNAVAILABLE; // 分类所属的类名
  struct objc_method_list *instance_methods    OBJC2_UNAVAILABLE; // 实例方法列表
  struct objc_method_list *class_methods       OBJC2_UNAVAILABLE; // 类方法列表
  struct objc_protocol_list *protocols         OBJC2_UNAVAILABLE; // 分类所实现的协议列表
}
~~~

1、Category的实现原理，以及Category为什么只能加方法不能加属性?

分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。
Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。

分类中的属性在关联列表里

实现原理：
1、我们不主动引入 Category 的头文件，Category 中的方法都会被添加进主类中。我们可以通过 - performSelector: 等方式对 Category 中的相应方法进行调用：
a)将 Category 和它的主类（或元类）注册到哈希表中；
b)如果主类（或元类）已实现，那么重建它的方法列表。

2、在这里分了两种情况进行处理：
a)Category 中的实例方法和属性被整合到主类中；
b)类方法则被整合到元类中
c)对协议的处理比较特殊，Category 中的协议被同时整合到了主类和元类中。

3、最终都是通过调用 staticvoid remethodizeClass(Class cls) 函数来重新整理类的数据的。

分类特点：
- 分类是用于给原有类添加方法的,因为分类的结构体指针中，没有属性列表，只有方法列表。原则上讲它只能添加方法, 不能添加属性(成员变量),实际上可以通过其它方式添加属性
- 分类中的可以写@property, 但不会生成setter/getter方法, 也不会生成实现以及私有的成员变量，会编译通过，但是引用变量会报错
- 如果分类中有和原有类同名的方法, 会优先调用分类中的方法, 就是说会忽略原有类的方法，同名方法调用的优先级为 分类 > 本类 > 父类
- 如果多个分类中都有和原有类中同名的方法, 那么调用该方法的时候执行谁由编译器决定；编译器会执行最后一个参与编译的分类中的方法
- 同名分类方法生效取决于编译顺序

2、Category中有load方法吗？load方法是什么时候调用的？load方法能继承吗？
Category中有load方法，load方法在程序启动装载类信息的时候就会调用。load方法可以继承。调用子类的load方法之前，会先调用父类的load方法

3、load、initialize的区别，以及他们在catefory重写的时候的调用次序
区别在于调用方式和调用时刻
调用方式：load是根据函数地址直接调用，initialize是通过objc_msgSend调用
调用时刻：load是runtime加载类、分类的时候调用(只会调用1次)，initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）
调用顺序：先调用类的load方法，先编译那个类，就先调用load.在调用load之前会先调用父类的load方法。分类中load方法不会覆盖本类的load方法，先编译的分类优先调用load方法。initialize先初始化父类，之后再初始化子类。如果子类没有实现initialize,会调用父类的initialize,如果分类实现了initialize，就会覆盖类本身的initialize调用。

## 分类和扩展

①分类中原则上只能增加方法（能添加属性的的原因只是通过runtime解决无setter/getter的问题而已）

②类扩展不仅可以增加方法，还可以增加实例变量（或者属性），只是该实例变量默认是@private类型的（
用范围只能在自身类，而不是子类或其他地方）

③类扩展中声明的方法没被实现，编译器会报警，但是类别中的方法没被实现编译器是不会有任何警告的。这是因为类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中。

④类扩展不能像类别那样拥有独立的实现部分（@implementation部分），也就是说，类扩展所声明的方法必须依托对应类的实现部分来实现。

⑤定义在 .m 文件中的类扩展方法为私有的，定义在 .h 文件（头文件）中的类扩展方法为公有的。类扩展是在 .m 文件中声明私有方法的非常好的方式。



## 参考

- [iOS底层原理总结 - Category的本质](https://www.jianshu.com/p/fa66c8be42a2)
- [iOS-分类Category](https://www.jianshu.com/p/01911be8ce83)
