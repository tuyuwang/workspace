---
layout: default
title: llvm
nav_order: 4
permalink: /docs/llvm
has_children: true
---

## LLVM

下载:
> git clone https://github.com/llvm/llvm-project.git

安装CMake:

https://cmake.org/download/

> git clone https://gitee.com/brightxiaohan/CMake.git
> ./bootstrap && make && sudo make install

~~~
mkdir build
cd build

 cmake -DLLVM_ENABLE_PROJECTS=clang -G "Unix Makefiles" ../llvm
 make

 
~~~