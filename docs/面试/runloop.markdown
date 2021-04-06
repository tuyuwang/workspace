---
layout: default
title: runloop
nav_order: 3
parent: 面试
---

## runloop原理
对于RunLoop而言最核心的事情是保证线程在没有消息的时候休眠，在有消息时唤醒，以提高程序性能。RunLoop这个机制是依靠系统内核来完成的(苹果操作系统核心组件Darwin中Mach)

RunLoop通过mach_msg()函数接收、发送消息。它的本质是调用函数mach_msg_trap(),相当于是一个系统调用，会触发内核状态切换。在用户态调用mach_msg_trap()时会切换到内核态；内核态中内核实现的mach_msg()函数会完成实际的工作。
即基于port的source1，监听端口，端口有消息就会触发回调；而source0，要手动标记为待处理和手动唤醒RunLoop

大致逻辑为:

1、通知观察者RunLoop即将启动。

2、通知观察者即将要处理Timer事件。

3、通知观察者即将要处理source0事件。

4、处理source0事件。

5、如果基于端口的源(source1)准备好并处于等待状态，进入步骤9.

6、通知观察者线程即将进入休眠状态。

7、将线程置于休眠状态，由用户态切换到内核态，直到下面的任一事件发生才唤醒线程。

- 一个基于port的source1的事件
- 一个Timer到时间了
- RunLoop自身的超时时间到了
- 被其他调用者手动唤醒


8、通知观察者线程将被唤醒

9、处理唤醒时收到的事件。

- 如果用户定义的定时器启动，处理定时器事件并重启RunLoop。进入步骤2.
- 如果输入源启动，传递相应的消息。
- 如果RunLoop被显示唤醒而且时间还没超时，重启RunLoop.进入步骤2


10、通知观察者RunLoop结束。

## runloop与线程间关系
线程和RunLoop是一一对应的，其映射关系是保存在一个全局的Dictionary里，自己创建的线程默认是没有开启RunLoop的

## 如何创建一个常驻线程

1、为当前线程开启一个RunLoop（第一次调用[NSRunLoop currentRunLoop]
方法时实际是会先创建一个RunLoop）

2、向当前RunLoop中添加一个Port/Source等维持RunLoop的事件循环(如果RunLoop的mode中一个item都没有，RunLoop会退出)

3、启动该RunLoop
~~~
@autoreleasepool{
    NSRunLoop *runloop = [NSRunLoop currentRunLoop];
    [[NSRunLoop currentRunLoop] addPort: [NSMachPort port] forMode: NSDefaultRunLoopMode];
    [runloop run];
}
~~~

## 怎样保证子线程数据回来更新UI的时候不打断用户的滑动操作？
当我们在子线程请求数据的同时滑动浏览当前页面，如果数据请求成功要切回主线程更新UI,那么就会影响当前正在滑动的体验。
我们就可以将更新UI事件放在主线程的NSDefaultRunLoopMode上执行即可，这样就会等用户不再滑动页面，主线程RunLoop由UITrackingRunLoopMode切换到NSDefaultRunLoopMode时再去更新UI.
~~~
[self performSelectorOnMainThread:@selector(reloadData) withObject: nil waitUntilDone: NO modes: @(NSDefaultRunLoopMode)];
~~~

## PerformSelector的实现原理
当调用NSObject的performSelecter:afterDelay:后，实际上其内部会创建一个 Timer 并添加到当前线程的runloop中。所以如果当前线程没有runloop，则这个方法会失效。

## Runloop的Mode
关于Mode首先要知道一个RunLoop对象中可能包含多个Model,且每次调用Runloop的主函数时，只能指定其中一个Mode.切换Mode，需要重新指定一个Mode。主要是为了分隔开不同的Source、Timer、Observer，让他们之间互不影响。

当RunLoop运行在Mode1上时，是无法接受处理Mode2或Mode3上的Source、Timer、Observer事件的

总共是有五种CFRunLoopMode:
- KCFRunLoopDefaultMode: 默认模式，主线程是在这个运行模式下运行
- UITrackingRunLoopMode: 跟踪用户交互事件(用于ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode影响)
- UIInitializationRunLoopMode: 在刚启动App时进入的第一个Mode,启动完成后就不再使用
- GSEventReceiveRunLoopMode: 接受系统内部事件，通常用不到
- KCFRunLoopCommonModes: 伪模式，不是一种真正的运行模式，是同步Source/Timer/Observer到多个Mode中的一种解决方案

## 自动释放池的实现原理

//图片

magic 用来检验AutoreleasePoolPage的结构是否完整
next 指向最新添加的autoreleased对象的下一个位置，初始化时指向begin()
thread 指向当前线程
parent 指向父节点，第一个结点的parent值为nil，双向链表上一个结点
child 指向子节点，最后一个结点的child值为nil,双向链表下一个节点
depth 代表深度，从0开始，往后递增1
hiwat 代表 high water mark

每一个自动释放池都是由一系列的AutoreleasePoolPage组成的，并且每一个AutoreleasePool的大小都是4096字节(16进制0x1000)

另外，当next==begin()时，表示AutoreleasePoolPage为空，当next==end()时，表示AutoreleasePoolPage已满

总结：

1、@autorelease展开来其实就是objc_autoreleasePoolPush和objc_autoreleasePoolPop，但是这两个函数也是封装的底层一个对象AutoreleasePoolPage，实际对应的是AutoreleasePoolPage::push和AutoreleasePoolPage::pop

2、autoreleasepool本身并没有内部结构，而是一种通过AutoreleasePoolPage为节点的双向链表结构

3、根据AutoreleasePoolPage双向链表结构，可以看到当调用objc_autoreleasePoolPush的时候实际上除了初始化poolpage对象属性之外，还会插入一个POOL_SENTINEL哨兵，用来区分不同autoreleasepool之间包裹的对象。

4、当对象调用autorelease方法时，会将实际对象插入AutoreleasePoolPage的栈中，通过next指针移动

5、autoreleasePoolPage的结构字段上面有介绍，其中每个双向链表的node节点也就是poolpage对象内存大小为4096，除了基础属性之外，外插一个POOL_SENTINEL，每出现一个@autorelease就会有一个哨兵，剩下的通过begin和end来标识是否存储满，满了就会重新创建一个poolpage来链接链表，按照这个套路，出现一个PoolPush就创建一个哨兵，出现一个对象的autorelease，就增加一个实际的对象，满了就创建新的链表节点这样衍生下去

6、AutoreleasePoolPage::pop那么当调用pop的时候，会传入需要drain的哨兵节点，遍历该内存地址上所有的对象，直到遇到对应的哨兵，然后释放栈中遍历到的对象，每删除一页就修正双向链表的指针。

7、ARC下，直接调用上面的方法，整个线程都被自动释放池双向链表管理，Push创建的时候插入哨兵对象，当我们在内部写代码的时候，会自动添加Autorelease,对象会加入到哨兵节点之间，加入到next指针上，一个个往后移，满了4096就换下一个poolPage对象节点来存储，出了释放池，会调用pop，传入自动释放池的哨兵给pop,然后遍历哨兵内存地址之后的所有对象执行release,最后把next指针移到目标哨兵。

8、runloop这里就不介绍了，可以翻看另外的博客，App启动的时候会在runloop里面注册两个观察者和一个回调函数，第一个Observe观察到entry即将进入loop的时候，会调用_objc_autoreleasePoolPush()创建自动释放池，优先级最高，保证在所有回调的方法之前。第二个Observe观察到即将进入休眠或者退出的时候，当监听到beforewaiting的时候，调用_objc_autoreleasePoolPop()和_objc_autoreleasePoolPush()释放旧的创建新的，当监听到Exit的时候调用_objc_autoreleasePoolPop释放Pool,这里的Obser优先级最低，发生在所有回调函数之后。

类方法创建的对象，在ARC下会自动调用autorelease

子线程需要开启runloop才能使用自动释放池

只有在自动释放池中调用了对象的autorelease方法,这个对象才会被存储到这个自动释放池之中.
如果只是将对象的创建代码写在自动释放之中,而没有调用对象的autorelease方法.是不会将这个对象存储到这个自动释放池之中的.

当自动释放池结束的时候.仅仅是对存储在自动释放池中的对象发送1条release消息 而不是销毁对象。

如果在自动释放池中,调用了存储到自动释放中的对象的release方法.在自动释放池结束的时候,还会再调用对的release方法.这个时候就有有可能会造成野指针操作.

## 利用runloop解释一下页面的渲染过程
当我们调用[UIView setNeedsDisplay]时，会先标记当前的layer，当runloop即将进入休眠时，会调用[CALayer display]，进入到真正的绘制工作，CALayer内部会创建一个Backing Store，用来获取图形上下文，接下来会判断这个layer是否有代理，如果有的话，会调用代理方法执行回调。如果没有代理，那么会调用[CALayer drawIncontext:]方法进行系统绘制，最终CALayer都会将位图提交到 Backing Store，最后提交给 GPUh绘制。

## RunLoop的数据结构
NSRunLoop是CFRunLoop的封装：
- CFRunLoop: RunLoop对象
- CFRunLoopMode: 运行模式
- CFRunLoopSource: 输入源/事件源
- CFRunLoopTimer: 定时源
- CFRunLoopObserver: 观察者

CFRunLoopSource:
- source0: 用户触发事件。需要手动唤醒线程，将当前线程从内核态切换到用户态
- source1: 基于port,包含一个mach_port和一个回调，可监听系统端口和通过内核和其他线程发送的消息，能主动唤醒runloop，接收分发系统事件。具备唤醒线程的能力。

CFRunLoopTimer:
基于时间的触发器，基本上说的就是NSTimer。在预设的时间点唤醒RunLoop执行回调。因为它是基于 RunLoop 的，因此它不是实时的(就是NSTimer是不准确的。 因为RunLoop只负责分发源的消息。如果 线程当前正在处理繁重的任务，就有可能导致 Timer 本次延时，或者少执行一次)。

CFRunLoopObserver:
监听以下事件：
- KCFRunLoopEntry: RunLoop 准备启动
- KCFRunLoopBforeTimers: RunLoop 将要处理一些 Timer 相关事件
- KCFRunLoopBeforeSources: RunLoop 将要处理一些 Source 事件
- KCFRunLoopBeforeWaiting: RunLoop 将要进行休眠状态,即将由用户态切换到内核态
- KCFRunLoopAfterWaiting: RunLoop 被唤醒，即从内核态切换到用户态后
- KCFRunLoopExit: RunLoop 退出
- KCFRunLoopAllActivities: 监听所有状态


