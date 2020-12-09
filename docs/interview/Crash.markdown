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

根据崩溃线程的thread_t获取线程的寄存器，获取pc寄存器，即当前执行函数的入口地址，获取lr寄存器，即当前执行函数的返回地址，也就是上一个函数的地址。然后获取fp，即函数栈帧，通过遍历栈帧回溯堆栈。栈帧是一个单向链表。

~~~C
/** Represents an entry in a frame list.
 * This is modeled after the various i386/x64 frame walkers in the xnu source,
 * and seems to work fine in ARM as well. I haven't included the args pointer
 * since it's not needed in this context.
 */

/*
 
 comment by @SecondDog
 
 try to explain why it is work in arm64:
 
 as we know ,arm stack layout is like this:
 so the pre FP should be *(current FP - 32)
 
 -------------  <---------- current FP
 | PC         |
 -------------  <---------- current FP - 8
 | LR         |
 -------------  <---------- current FP - 16
 | SP         |
 -------------  <---------- current FP - 24
 | FP(pre)    |
 -------------  <---------- current FP - 32
 
 the struct FrameEntry is defined like this
 typedef struct FrameEntry
 {
 struct FrameEntry* previous; <----- this pointer is 8 byte in arm64
 uintptr_t return_address;  <------ this value is also 8 byte
 } FrameEntry; <----- 16byte total
 
 
 but the arm64 call stack is like this.
 -------------
 | LR(x30)     |
 -------------
 | pre FP(x29) |
 -------------  <-----Current FP
 
 so copy 16 byte data to the FrameEntry is just fit in arm64,so it works fine in arm64
 
 */

typedef struct FrameEntry
{
    /** The previous frame in the list. */
    struct FrameEntry* previous;
    
    /** The instruction address. */
    uintptr_t return_address;
} FrameEntry;


typedef struct
{
    const struct KSMachineContext* machineContext;
    int maxStackDepth;
    FrameEntry currentFrame;
    uintptr_t instructionAddress;
    uintptr_t linkRegister;
    bool isPastFramePointer;
} MachineContextCursor;

static bool advanceCursor(KSStackCursor *cursor)
{
    MachineContextCursor* context = (MachineContextCursor*)cursor->context;
    uintptr_t nextAddress = 0;
    
    if(cursor->state.currentDepth >= context->maxStackDepth)
    {
        cursor->state.hasGivenUp = true;
        return false;
    }
    
    if(context->instructionAddress == 0 && cursor->state.currentDepth == 0)
    {
        context->instructionAddress = kscpu_instructionAddress(context->machineContext);
        nextAddress = context->instructionAddress;
        goto successfulExit;
    }
    
    if(context->linkRegister == 0 && !context->isPastFramePointer)
    {
        // Link register, if available, is the second address in the trace.
        context->linkRegister = kscpu_linkRegister(context->machineContext);
        if(context->linkRegister != 0)
        {
            nextAddress = context->linkRegister;
            goto successfulExit;
        }
    }

    if(context->currentFrame.previous == NULL)
    {
        if(context->isPastFramePointer)
        {
            return false;
        }
        context->currentFrame.previous = (struct FrameEntry*)kscpu_framePointer(context->machineContext);
        context->isPastFramePointer = true;
    }

    if(!ksmem_copySafely(context->currentFrame.previous, &context->currentFrame, sizeof(context->currentFrame)))
    {
        return false;
    }
    if(context->currentFrame.previous == 0 || context->currentFrame.return_address == 0)
    {
        return false;
    }

    nextAddress = context->currentFrame.return_address;
    
successfulExit:
    cursor->stackEntry.address = kscpu_normaliseInstructionPointer(nextAddress);
    cursor->state.currentDepth++;
    return true;
}
~~~

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

