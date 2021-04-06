---
layout: default
title: Otool
nav_order: 2
parent: LLVM
---
查看指定区内容
> otool -v -s __TEXT __cstring a.out

输出代码段内容
> otool -tV a.out

查看链接的动态库
> otool -L a.out