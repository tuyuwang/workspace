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

