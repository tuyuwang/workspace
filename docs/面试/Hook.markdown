---
layout: default
title: Hook
nav_order: 2
parent: 面试
---

## Aspect

原理: 通过替换原方法为消息转发指针,强制走消息转发流程.并且hook消息转发流程中的forwardInvocationfor方法进行统一处理实现.

缺点: 子父类同时hook同一方法时,难以区分是子类调用还是父类调用