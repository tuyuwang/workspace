---
layout: default
title: Pass
nav_order: 3
parent: LLVM
---

## 自定义Pass

[参考opt Pass](https://llvm.org/docs/WritingAnLLVMPass.html#running-a-pass-with-opt)

上述文档有点滞后，按照文档上操作基本没问题，但是在llvm-project中已有Hello的Pass，所以完全按照文档上做会编译失败，需要进行改名。

下面进行编写一个DDPass

1、在llvm-project/llvm/lib中创建文件夹DDPass
~~~
cd llvm-project/llvm/lib
mkdir DDPass
~~~

2、在DDPass目录下创建CMakeLists.txt和源文件DDPass.cpp
~~~
cd llvm-project/llvm/lib/DDPass

touch CMakeLists.txt
touch DDPass.cpp
~~~

3、然后编写DDPass/CMakeLists.txt文件内容
~~~
add_llvm_library( LLVMDDPass MODULE
  DDPass.cpp

  PLUGIN_TOOL
  opt
  )
~~~

4、编写DDPass/DDPass.cpp
~~~ c++
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"

#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"

using namespace llvm;

namespace {
struct DDLog : public FunctionPass {
  static char ID;
  DDLog() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override {
    errs() << "DDLog: ";
    errs().write_escaped(F.getName()) << '\n';
    return false;
  }
}; // end of struct DDLog
}  // end of anonymous namespace

char DDLog::ID = 0;
static RegisterPass<DDLog> X("ddlog", "DDLog Pass",
                             false /* Only looks at CFG */,
                             false /* Analysis Pass */);

static RegisterStandardPasses Y(
    PassManagerBuilder::EP_EarlyAsPossible,
    [](const PassManagerBuilder &Builder,
       legacy::PassManagerBase &PM) { PM.add(new DDLog()); });
~~~

5、在llvm-project/llvm/lib/CMakeLists.txt中添加
> add_subdirectory(DDPass)

6、在llvm-project/build目录下，重新编译
>  cmake -G Xcode -DLLVM_ENABLE_PROJECTS=clang ../llvm

7、打开llvm-project/build/LLVM.xcodeproj

8、添加Scheme DDPass，进行编译，并编译opt

9、测试
> Debug/bin/opt --load /Users/tuyw/llvm/llvm-project/build/Debug/lib/LLVMDDPass.dylib  -ddlog ../../test/hello.bc > /dev/null 

> -time-passes 查看执行时间

10、clang加载pass
>clang -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -Xclang -load -Xclang /path/to/Pass.dylib

> clang -Xclang -load -Xclang mypass.so code.c

~~~
clang -c -emit-llvm code.c
$ opt -load myhello.so -myhello code.bc
~~~

## xcode中clang加载pass
1、

> build setting -> other C flags -> -Xclang -load -Xclang path/to/xxx1.dylib -Xclang -load -Xclang path/to/xxx2.dylib

2、

> build setting -> + 号 -> CC 设置clang路径 -> CXX 设置clang++路径
~~~
/Users/tuyw/llvm/llvm-project-llvmorg-8.0.0/build/Debug/bin/clang
~~~

需要这样注册LLVM Pass
~~~
char Hello::ID = 0;
static void registerHello(const PassManagerBuilder &, legacy::PassManagerBase &PM)
{
  PM.add(new Hello());
}

static RegisterStandardPasses RegisterStringEncryption(PassManagerBuilder::EP_EnabledOnOptLevel0, registerHello);

~~~

报错时调整EP_EnabledOnOptLevel0这个枚举


Pass可选参数
~~~
// c flag use: -mllvm -hello
if (getenv("hello")) {
    // do something
}
~~~