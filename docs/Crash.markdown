---
layout: default
title: Crash
nav_order: 14
---

## 崩溃原理



#### 线程回溯原理
程序发生崩溃时，获取到崩溃的线程，取出线程的thread_id，然后调用thread_get_state获取当前的执行状态，找出帧指针BP和栈指针SP。通过遍历取出BP和SP回溯堆栈。


## swift weak实现原理
## 寄存器
## 卡顿优化