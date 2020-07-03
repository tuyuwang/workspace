---
layout: default
title: clang
nav_order: 1
parent: llvm
---

只处理预编译(Only run the preprocessor)
> clang -E hello.c

生成block的arm64架构下的汇编，引入Foundation框架
> clang -arch arm64 -isysroot `xcrun --sdk iphoneos --show-sdk-path` block.m -framework Foundation -fobjc-arc

添加文件内容(-sectcreate segname sectname file)
>clang -arch arm64 -isysroot `xcrun --sdk iphoneos --show-sdk-path` block.m -framework Foundation -fobjc-arc -Wl,-sectcreate,__TEXT,__info_plist,/Users/mac/Desktop/test/RSSwizzle/RSSwizzle/Info.plist 

编译多个文件
-c : Only run preprocess, compile, and assemble steps
> clang -c Foo.m
> clang -c helloworld.m

-Wl : -Wl,<arg> Pass the comma separated arguments in <arg> to the linker
> clang helloworld.o Foo.o -Wl,`xcrun --show-sdk-path`/System/Library/Frameworks/Foundation.framework/Foundation

#### 编译器的处理过程分为几个阶段
> clang  -ccc-print-phases hello.m

~~~
0: input, "hello.m", objective-c
1: preprocessor, {0}, objective-c-cpp-output //预处理
2: compiler, {1}, assembler //编译程序
3: assembler, {2}, object //汇编程序
4: linker, {3}, image  // 链接
5: bind-arch, "x86_64", {4}, image
~~~

预处理: 
- 符号化 (Tokenization)
- 宏定义的展开
- #include 的展开

语法和语义分析:
- 将符号化后的内容转化为一棵解析树 (parse tree)
- 解析树做语义分析
- 输出一棵抽象语法树（Abstract Syntax Tree* (AST)）

生成代码和优化:
- 将 AST 转换为更低级的中间码 (LLVM IR)
- 对生成的中间码做优化
- 生成特定目标代码
- 输出汇编代码

汇编器:
- 将汇编代码转换为目标对象文件。

链接器:
- 将多个目标对象文件合并为一个可执行文件 (或者一个动态库)

详情:
预处理完成以后，每一个 .m 源文件里都有一堆的声明和定义。这些代码文本都会从 string 转化成特殊的标记流。

例如，下面是一段简单的 Objective-C hello word 程序：
~~~
int main() {
  NSLog(@"hello, %@", @"world");
  return 0;
}
~~~
利用 clang 命令 clang -Xclang -dump-tokens hello.m 来将上面代码的标记流导出：
~~~
int 'int'        [StartOfLine]  Loc=<hello.m:4:1>
identifier 'main'        [LeadingSpace] Loc=<hello.m:4:5>
l_paren '('             Loc=<hello.m:4:9>
r_paren ')'             Loc=<hello.m:4:10>
l_brace '{'      [LeadingSpace] Loc=<hello.m:4:12>
identifier 'NSLog'       [StartOfLine] [LeadingSpace]   Loc=<hello.m:5:3>
l_paren '('             Loc=<hello.m:5:8>
at '@'          Loc=<hello.m:5:9>
string_literal '"hello, %@"'            Loc=<hello.m:5:10>
comma ','               Loc=<hello.m:5:21>
at '@'   [LeadingSpace] Loc=<hello.m:5:23>
string_literal '"world"'                Loc=<hello.m:5:24>
r_paren ')'             Loc=<hello.m:5:31>
semi ';'                Loc=<hello.m:5:32>
return 'return'  [StartOfLine] [LeadingSpace]   Loc=<hello.m:6:3>
numeric_constant '0'     [LeadingSpace] Loc=<hello.m:6:10>
semi ';'                Loc=<hello.m:6:11>
r_brace '}'      [StartOfLine]  Loc=<hello.m:7:1>
eof ''          Loc=<hello.m:7:2>
~~~

解析:
接下来要说的东西比较有意思：之前生成的标记流将会被解析成一棵抽象语法树 (abstract syntax tree -- AST)。由于 Objective-C 是一门复杂的语言，因此解析的过程不简单。解析过后，源程序变成了一棵抽象语法树：一棵代表源程序的树。假设我们有一个程序 hello.m：
~~~
#import <Foundation/Foundation.h>

@interface World
- (void)hello;
@end

@implementation World
- (void)hello {
  NSLog(@"hello, world");
}
@end

int main() {
   World* world = [World new];
   [world hello];
}

~~~

当我们执行 clang 命令 clang -Xclang -ast-dump -fsyntax-only hello.m 之后，命令行中输出的结果如下所示：
~~~
@interface World- (void) hello;
@end
@implementation World
- (void) hello (CompoundStmt 0x10372ded0 <hello.m:8:15, line:10:1>
  (CallExpr 0x10372dea0 <line:9:3, col:24> 'void'
    (ImplicitCastExpr 0x10372de88 <col:3> 'void (*)(NSString *, ...)' <FunctionToPointerDecay>
      (DeclRefExpr 0x10372ddd8 <col:3> 'void (NSString *, ...)' Function 0x1023510d0 'NSLog' 'void (NSString *, ...)'))
    (ObjCStringLiteral 0x10372de38 <col:9, col:10> 'NSString *'
      (StringLiteral 0x10372de00 <col:10> 'char [13]' lvalue "hello, world"))))


@end
int main() (CompoundStmt 0x10372e118 <hello.m:13:12, line:16:1>
  (DeclStmt 0x10372e090 <line:14:4, col:30>
    0x10372dfe0 "World *world =
      (ImplicitCastExpr 0x10372e078 <col:19, col:29> 'World *' <BitCast>
        (ObjCMessageExpr 0x10372e048 <col:19, col:29> 'id':'id' selector=new class='World'))")
  (ObjCMessageExpr 0x10372e0e8 <line:15:4, col:16> 'void' selector=hello
    (ImplicitCastExpr 0x10372e0d0 <col:5> 'World *' <LValueToRValue>
      (DeclRefExpr 0x10372e0a8 <col:5> 'World *' lvalue Var 0x10372dfe0 'world' 'World *'))))
~~~

静态分析:
一旦编译器把源码生成了抽象语法树，编译器可以对这棵树做分析处理，以找出代码中的错误，比如类型检查：即检查程序中是否有类型错误。例如：如果代码中给某个对象发送了一个消息，编译器会检查这个对象是否实现了这个消息（函数、方法）。此外，clang 对整个程序还做了其它更高级的一些分析，以确保程序没有错误。

代码生成:
clang 完成代码的标记，解析和分析后，接着就会生成 LLVM 代码。下面继续看看hello.c：
~~~
#include <stdio.h>

int main() {
  printf("hello world\n");
  return 0;
}
~~~

要把这段代码编译成 LLVM 字节码（绝大多数情况下是二进制码格式），我们可以执行下面的命令：
> clang -O3 -emit-LLVM hello.c -c -o hello.bc

接着用另一个命令来查看刚刚生成的二进制文件：
> llvm-dis < hello.bc | less

输出如下：
~~~
; ModuleID = '<stdin>'
target datalayout = "e-p:64:64:64-i1:8:8-i8:8:8-i16:16:16-i32:32:32-i64:64:64-f32:32:32-f64:64:64-v64:64:64-v128:128:128-a0:0:64-s0:64:64-f80:128:128-n8:16:32:64-S128"
target triple = "x86_64-apple-macosx10.8.0"

@str = private unnamed_addr constant [12 x i8] c"hello world\00"

; Function Attrs: nounwind ssp uwtable
define i32 @main() #0 {
  %puts = tail call i32 @puts(i8* getelementptr inbounds ([12 x i8]* @str, i64 0, i64 0))
  ret i32 0
}

; Function Attrs: nounwind
declare i32 @puts(i8* nocapture) #1

attributes #0 = { nounwind ssp uwtable }
attributes #1 = { nounwind }
~~~


### 参考
- [编译器](https://objccn.io/issue-6-2/)
