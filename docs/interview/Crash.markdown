---
layout: default
title: Crash
nav_order: 2
parent: interview
---

## 崩溃

### 捕获原理


### 堆栈回溯原理
程序发生崩溃时，获取到崩溃的线程，取出线程的thread_id，然后调用thread_get_state获取当前的执行状态，找出帧指针BP和栈指针SP。通过遍历取出BP和SP回溯堆栈。

### 上报


## 避免崩溃

### AvoidCrash原理

核心处理就是用@try catch，局限性：只能捕获OC层的崩溃，对于更底层signal/mach/c++类型崩溃无法避免

1、指定方法名和类对象，进行方法交换，通过try catch进行OC层的崩溃拦截，如：

- NSArray
- NSString
- NSDictionary
- NSAttributedString

以及其对应的可变类型。

还有NSObject的KVC方法：

- setValue:forKey:
- setValue:forKeyPath:
- setValue:forUndefinedKey:
- setValuesForKeysWithDictionary:

2、通过NSObject分类，	交换消息转发的方法：

- methodSignatureForSelector
- forwardInvocation：

指向一个白板NSObject子类，一个白板方法

~~~
- (NSMethodSignature *)avoidCrashMethodSignatureForSelector:(SEL)aSelector {
    
    NSMethodSignature *ms = [self avoidCrashMethodSignatureForSelector:aSelector];
    
    BOOL flag = NO;
    if (ms == nil) {
        for (NSString *classStr in noneSelClassStrings) {
            if ([self isKindOfClass:NSClassFromString(classStr)]) {
                ms = [AvoidCrashStubProxy instanceMethodSignatureForSelector:@selector(proxyMethod)];
                flag = YES;
                break;
            }
        }
    }
    if (flag == NO) {
        NSString *selfClass = NSStringFromClass([self class]);
        for (NSString *classStrPrefix in noneSelClassStringPrefixs) {
            if ([selfClass hasPrefix:classStrPrefix]) {
                ms = [AvoidCrashStubProxy instanceMethodSignatureForSelector:@selector(proxyMethod)];
            }
        }
    }
    return ms;
}

- (void)avoidCrashForwardInvocation:(NSInvocation *)anInvocation {
    
    @try {
        [self avoidCrashForwardInvocation:anInvocation];
        
    } @catch (NSException *exception) {
        NSString *defaultToDo = AvoidCrashDefaultIgnore;
        [AvoidCrash noteErrorWithException:exception defaultToDo:defaultToDo];
        
    } @finally {
        
    }
    
}
~~~

### QMUI原理

hook NSException的raise方法，进行方法拦截，区分处理吧，也是类似于@try catch，不同于@try catch到处写，这个方法可以统一处理崩溃

[QMUI_iOS](https://github.com/Tencent/QMUI_iOS/commit/cd406c75145e2dc42471e5785f72af6c2958fafa)

![hook](../../../images/Interview/NSExceptionHandler.jpeg)

![hook](../../../images/Interview/hook.jpeg)

### KSCrash

通过调用NSSetUncaughtExceptionHandler函数进行崩溃拦截

