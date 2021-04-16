---
layout: default
title: StringEncryption
nav_order: 4
parent: LLVM
---

[自己动手实现基于llvm的字符串加密](https://iosre.com/t/llvm/10610)

## 基础知识

- 每个源码文件对应一个翻译单元，LLVM体系中每个Module对应一个翻译单元
- Module是LLVM中间表示的最外层封装，所有的函数，变量，元数据等等全部封装在对应的Module中
- 几乎所有跟LLVM中间表示有关的数据类型都继承自llvm:Value下文中所有的值指代都是该类所有子类的统称
- 字符串在LLVM中间表示里用一个常数数组实现
- 每个函数由一个或多个基本块组成，每个基本块有一个结束指令用于改变控制流。调用其他函数，跳转至其他基本块，返回void或其他值的指令都属于此列
- LLVM的函数分为declare和definition两种。declare指的是实现在当前翻译单元外的函数,definition反之。
- IRBuilder是LLVM提供的方便指令插入的助手模版类

- IR中的字符串形式：字符串常量和字符串数组都是ConstantDataSequential(CDS)
- 所有的字符串字面量，都是i8/i16数组
- 匿名的字符串常量为@.str(n) = private unnamed_addr constant
- 具名的字符串数组为@name = global 或 @name = internal constant
- 空的字符串字面量为private unnamed_addr constant[1 x i8] zeroinitializer

~~~
//创建字符串常量
	llvm::Constant *strConst1 = llvm::ConstantDataArray::getString(context,
			"exception_name");
	llvm::Value *globalVar1 = new llvm::GlobalVariable(*module,
			strConst1->getType(), true, llvm::GlobalValue::PrivateLinkage,
			strConst1, "globalVar1");
	
	//ir长这样
	@globalVar1 = private constant [15 x i8] c"exception_name\00"

————————————————
原文链接：https://blog.csdn.net/qq_42570601/article/details/108007986
~~~

## 实现原理

搜索所有函数内对全局变量的调用，判断是否为常数数组，如是则在函数的起始位置插入解密函数，在函数的结尾插入加密函数。

[上海交通大学密码与计算机安全实验室的实现方案](http://mayuyu.io/2017/12/26/StringEncryption/)

## 步骤

Module包含所有Function，Function包含n个BasicBlock，BasicBlock包含n个Instruction


1. 首先定义测试字符串，模拟所有使用字符串的场景
2. 生成IR，进行分析字符串在IR中的存在形式
3. 根据分析结果获取所有字符串并编写Pass
4. 对字符串进行加解密
5. 删除指令块中的字符串明文部分


## 参考
- [对目前开源OLLVM的字符串加密混淆总结](https://bbs.pediy.com/thread-263107.htm)

