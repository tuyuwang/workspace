---
layout: default
title: bitCode
nav_order: 6
parent: iOS
---

## BitCode


什么是bitcode？

LLVM 是目前苹果采用的编译器工具链,Bitcode是LLVM编译器的中间代码的一种编码,LLVM的前端可以理解为C/C++/OC/Swift等编程语言,LLVM的后端可以理解为各个芯片平台上的汇编指令或者可执行机器指令数据,那么,BitCode就是位于这两者直接的中间码. 

LLVM的编译工作原理是前端负责把项目程序源代码翻译成Bitcode中间码,然后再根据不同目标机器芯片平台转换为相应的汇编指令以及翻译为机器码.这样设计就可以让LLVM成为了一个编译器架构,可以轻而易举的在LLVM架构之上发明新的语言(前端),以及在LLVM架构下面支持新的CPU(后端)指令输出,虽然Bitcode仅仅只是一个中间码不能在任何平台上运行,但是它可以转化为任何被支持的CPU架构,包括现在还没被发明的CPU架构,也就是说现在打开Bitcode功能提交一个App到应用商店,以后如果苹果新出了一款手机并CPU也是全新设计的,在苹果后台服务器一样可以从这个App的Bitcode开始编译转化为新CPU上的可执行程序,可供新手机用户下载运行这个App.

在新建项目中，使用bitcode进行archive进行打包出来的二进制文件与原始archive的二进制文件有所区别，如下：

在Load Commands中新增加了个段 __LLVM：
![bitcode](../../../images/iOS/bitcode-command.png)

在section中新增了__LLVM,__bundle: 
![bitcode](../../../images/iOS/bitcode-section.png)

对比其汇编代码，发现并没有受到bitcode影响：
![bitcode](../../../images/iOS/bitcode-assembly.png)

综合上述对比，发现在新的空项目中，bitcode并不会影响原始代码进行转汇编的流程，而是在原有的编译输出基础上，进行一次bitcode代码生成，并写入macho里。

所以可以知道使用bitcode的缺点：
- 造成项目编译速度变慢
- 苹果重新编译后，客户下载的是重新编译后的执行文件，原有的DSYM文件失效，线上崩溃无法解析
 