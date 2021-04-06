---
layout: default
title: MachO
nav_order: 5
parent: Objective-C
grand_parent: 语言
---

## MachO

[iOS逆向(5)-不知MachO怎敢说自己懂DYLD](https://www.jianshu.com/p/95896fb96a03)

结构:

- Header
- Load Commands
- Data


相关定义: mach-o/loader.h


Header: 包含该二进制文件的一般信息，字节顺序、架构类型、加载指令数量、大小等
~~~C
/*
 * The 64-bit mach header appears at the very beginning of object files for
 * 64-bit architectures.
 */
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};

/* Constant for the magic field of the mach_header_64 (64-bit architectures) */
#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
~~~


Load Commands：加载的指令

- LC_SEGMENT_64	将文件中（32位或64位）的段映射到进程地址空间中
- LC_DYLD_INFO_ONLY	动态链接相关信息
- LC_SYMTAB	符号地址
- LC_DYSYMTAB	动态符号表地址
- LC_LOAD_DYLINKER	使用谁加载，我们使用dyld
- LC_UUID	文件的UUID
- LC_VERSION_MIN_MACOSX	支持最低的操作系统版本
- LC_SOURCE_VERSION	源代码版本
- LC_MAIN	设置程序主线程的入口地址和栈大小
- LC_LOAD_DYLIB	依赖库的路径，包含三方库
- LC_FUNCTION_STARTS	函数起始地址表
- LC_CODE_SIGNATURE	代码签名
- LC_LOAD_DYLIB 该字段标记了所有动态库的地址


Data: 通常是对象文件中最大的部分，包含Segement的具体数据，如静态C字符串，带参数/不带参数的OC方法，C函数

- Section64(__TEXT,__text) 代码段，以汇编的形式
- Section64(__TEXT,__stubs) 符号桩
- Section64(__TEXT,__cstring) 静态字符串
- Section64(__TEXT,__methodname) 方法名
- Section64(__TEXT,__classname) 类名
- Section64(__TEXT,__methodtype) 方法签名？
-

macho加载流程:

- 配置环境变量
- 加载共享缓存库
- 实例化主程序
- 加载动态链接库
- 加载Load和特定的C++的构造函数方法
- 寻找APP的main函数并调用


Mach-O文件头mach-header:

- magic: 0xFEEDFACE表示32位二进制，0xFEEDFACF代表64位二进制
- cputype: CPU类型，定义在<mach/machine.h>头文件中
- cpusubtype: CPU子类型
- filetypes: 文件类型(可执行文件、库文件、核心转储文件、内核扩展等)
- ncmds: 用于加载器的加载命令的条数
- sizeofncmds: 加载命令的大小
- flags: 动态链接器(dyld)的标志
- Reserved: 极限64位，保留给未来使用

Mach-O文件类型:
定义在<mach-o/loader.h>

 - MH-OBJECT(1): 可重定位的目标文件（编译器对源代码编译得到的中间结果，也可以是32位的内核扩展）
 - MH-EXECUTABLE(2): 可执行二进制文件
 - MH-CORE(4): 核心转储文件
 - MH-DYLIB(6): 动态库
 - MH-DYLINKER(7): 动态链接器
 - MH-BUNDLE(8): 插件，非独立的二进制文件，要加载至其他二进制才能发挥作用。
 - MH-DSYM(10): 辅助的符号文件以及调试信息
 - MH-KEXT-BUNDLE(11): 内核扩展

Mach-O头文件标志:

- MH_NOUNDEFS: 表示目标文件没有带有未定义的符号。这些目标文件大部分都是静态二进制文件，没有进一步的链接依赖关系。
- MH_SPLITSEGS: 表示目标文件中的只读段和可读写段区分开了
- MH_TWOLEVEL: 两层名称绑定
- MH_PROCEFLAT: 扁平名称空间绑定
- MH_WEAK_DEFINES: 二进制文件使用了(导出了)弱符号
- MH_BINDS_TO_WEAK: 二进制文件链接了弱符号
- MH_ALLOW_STACK_EXECUTION: 允许栈可执行。只有可执行文件可以用这个标志，但是通常不建议使用。
- MP_PIE: 对可执行的文件类型启用地址空间布局随机化。
- MH_NO_HEAP_EXECUTION: 将堆标记为不可执行。可用于防止heap spray攻击技术。

heap spray技术: 使用这种技术的黑客盲目的用shellcode覆盖大量的堆空间，然后投机地跳转到这些地址中，企图能够跳转到自己的代码并执行。

MH_ALLOW_STACK_EXECUTION和MH_NO_HEAP_EXECUTION这两个标志都用于防止某些数据的执行。通过将数据所在的内存页面标记为不可执行，一般情况下可以防止黑客进行代码注入，因为黑客不能方便的执行数据段中的代码。如果试图执行数据段中的代码，则会引发一个硬件异常，进程会终止，让程序崩溃，从而避免执行注入的代码。

#### 加载命令

Mach-O头文件中包含了非常详细的指令，这些指令在被调用时清晰的指导了如何设置并加载二进制数据。这些指令，紧跟在mach_header之后。每条命令都采用类型-长度-值的格式:32位的cmd值(表示类型).32位的cmdsize值(32位二进制为4的倍数，64位二进制为8的倍数)，以及命令本身(由cmdsize指定的任意长度)。有一些命令是由内核加载器(定义在bsd/kern/mach_loader.c文件中)直接使用的，其他命令是由动态链接器处理的。

由内核处理的Mach-O加载命令:

- 0x01 LC_SEGMENT: 将文件中的32位的段映射到进程地址空间中。
- 0x19 LC_SEGMENT_64: 64位同上。
- 0x0E LC_LOAD_DYLINKER: 调用dyld(/usr/lib/dyld)。
- 0x1B LC_UUID: 一个唯一的128位ID,这个ID匹配一个二进制文件以及对应的符号。
- 0x04 LC_THREAD: 开启一个Mach线程，但是不分配堆栈。
- 0x05 LC_UNIXTHREAD: 开启一个UNIX线程(初始化栈布局和寄存器)。
- 0x1D LC_CODE_SIGNATURE: 代码签名
- 0x21 LC_ENCRYPTION_INFO: 加密的二进制文件

LC_CODE_SIGNATURE包含了Mach-O二进制文件的代码签名，如果这个签名和代码本身不匹配(或者如果在iOS上这个命令不存在)，那么内核会立即给进程发送一个SIGKILL信号将进程杀死。

从Mountain Lion开始新命令LC_MAIN替代了LC_UNIXTHREAD命令,LC_MAIN: 设置主程序的入口点地址和栈大小。

###### LC_SEGMENT以及进程虚拟内存设置

LC_SEGMENT命令是最主要的加载命令，这条命令指导内核如何设置新运行的进程的内存空间。这些段直接从Mach-O文件二进制文件加载到内存中。

LC_SEGMENT参数:

- segname: load_segment
- vmaddr: 所描述的段的虚拟物理地址
- vmsize: 为这个段分配的虚拟内存大小
- fileoff: 表示这个段在文件中的偏移量
- filesize: 表示这个段在文件中占用的字节数
- maxport: 段的页面所需要的最高内存保护，用八进制表示4=r, 2=w, 1=x
- initport: 段的页面最初始的内存保护
- nsects: 段中的区数量
- flags: 杂项标志位

对于每一段，将文件中相应的内存加载到内存中: 从偏移量位fileoff处加载filesize字节到虚拟内存地址vmaddr处的vmsize字节。每一个段的页面都根据initport进行初始化，initport指定了如何通过读/写/执行初始化页面的保护等级。段的保护设置可以动态改变，但不能超过maxport。

段的类型:

- __PAGEZERO： 空指针陷阱
- _TEXT: 程序代码
- _DATA: 程序数据
- _LINKEDIT: 链接器使用的符号和其他表

段中包含的区类型:

- __text: 主程序代码
- __stubs、__stub_helper: 用于动态链接的桩
- __cstring: 程序中硬编码的C语言字符串
- __const: 用const修饰的常量变量以及硬编码的常量
- __TEXT.__objc_methodname: OC方法名称
- __TEXT.__objc_methodtype: OC方法类型
- __TEXT.__objc_classname: OC类名称
- __DATA.__objc_classlist: OC类列表
- __DATA.__objc_protolist: OC原型
- __DATA.__objc_imginfo: OC镜像信息
- __DATA.__objc_const: OC常量
- __DATA.__objc_selfrefs: OC自引用(this)
- __DATA.__objc_protorefs: OC原型引用
- __DATA.__objc_superrefs: OC超类引用
- __DATA.__cfstring: 程序中使用的Core Foundation字符串(CFStringRefs)
- __DATA.__bss: BSS

###### 启动时库的加载

动态链接器是内核执行LC_DYLINKER加载命令时启动的，通常情况下，使用的是/usr/lib/dyld作为动态链接器，不过这条加载命令可以指定任何程序作为参数。链接器会查找进程中所有的符号和库的关系，然后解决这些关系。这个过程必须是递归地完成，因为通常情况下库还会依赖于其他的库。

dyld是一个用户态的进程。[dyld](http://www.opensource.apple.com/source/dyld)

由dyld处理的加载命令:

- 0x02 LC_SYMTAB: 符号表
- 0x0B LC_DSYMTAB: 符号表。符号表和字符串表是由这些命令中指定的偏移量分别提供的
- 0x0C LC_LOAD_DYLIB: 加载额外的动态库
- 0x20 LC_LAZY_LOAD_DYLIB: 懒加载的方式加载额外的动态库
- 0x0D LC_ID_DYLIB: 只在dylib中寻找。指定了dylib的ID、时间戳、版本和兼容版本
- 0x1F LC_REEXPORT_DTLIB: 只在动态库中寻找。允许一个库将其他库的符号作为自己的符号重新导出
- 0x24 LC_VERSION_MIN_IPHONEOS: 这个二进制文件的最低操作系统版本
- 0x25 LC_VERSION_MIN_MACOSX
- 0x26 LC_FUNCTION_STARTS: 压缩的函数起始地址表
- 0x2A LC_SOURCE_VERSION: 构建这个二进制文件使用的源代码版本


如果二进制文件中使用了外部定义的函数和符号，那么在它们的文本段中会有一个名为__stubs(桩)的区，在这个区中存放的是这些本地未定义符号的占位符。编译器生成代码时会创建对符号桩区的调用，链接器在运行时会解决对桩的调用。链接器解决的方法是在被调用的地址处放置一条JMP指令。JMP指令将控制权转交给真实的函数体，但是不会以任何方式修改栈。因此，真实的函数可以正常返回，就好像直接调用一样。

dyld有函数拦截的特性，DYLD_INTERPOSE宏定义允许一个库将其函数实现替换为另一个函数的实现。
~~~
//DYLD_INTERPOSE宏定义，dyld的include/mach-o/dyld-interposing.h

#if !defined(_DYLD_INTERPOSING_H_)
#define _DYLD_INTERPOSING_H_

/*
*   Example:
*   static
*   int
*   my_open(const char* path, int flags, mode_t mode)
*   {
*       int value;
*       // 在open之前做一些事情
*       value = open(path, flags, mode);
*       // 在open之后做一些事情
*       return value;
*   }
*   DYLD_INTERPOSE(my_open, open)
*/

#define DYLD_INTERPOSE(_replacement, _replacee) \
    __attribute__((used)) static struct { const void* replacement; const void* replacee;}
 _interpose_##_replacee \
            __attribute__((section ("__DATA,interpose"))) = { (const void*)(unsigned long)&_replacement, (const void*)(unsigned long)&_replacee };
~~~

dyld的函数拦截功能提供了一个新的__DATA区，名为__interpose,在这个区中依次列出了替换的函数和被替换的函数。

dyld可高度定制，可以通过改变环境变量改变行为，在程序加载时可设置以下环境变量:

- DYLD_FORCE_FLAT_NAMESPACE: 禁用库的二进制名称空间(为了INSERT)。否则，符号名称中还会包含库的名称
- DYLD_IGNORE_PREBINDING: 为性能测试的目的，禁用预先绑定
- DYLD_IMAGE_SUFFIX: 查找带有这个后缀的库。通常设置为_debug或_profile，这样的话就会加载libSystem.B_debug.dylib或libSystem.B_profile.dylib,而不是libSystem
- DYLD_INSERT_LIBRARTIES: 在程序加载时强制插入一个或多个库
- DYLD_LIBRARY_PATH
- DYLD_FALLBACK_LIBRARY_PATH: DYLD_LIBRARY_PATH失败时使用
- DYLD_FRAMEWORK_PATH: 同DYLD_LIBRARY_PATH，但是用于框架
- DYLD_FALLBACK_FRAMEWORK_PATH: DYLD_FRAMEWORK_PATH失败时使用

> DYDL_INSERT_LIBRARIES=xxxx.dylib ls //强制插入到ls中

此外，以下变量控制dyld的调试打印选项:

- DYLD_PRINT_APIS: 导出dyld的API调用
- DYLD_PRINT_BINGINGS: 导出符号绑定
- DYLD_PRINT_ENV: 导出初始的环境变量
- DYLD_PRINT_INITIALIZERS: 导出库的初始(入口点)调用
- DYLD_PRINT_LIBRARIES: 在加载库时打印出库
- DYLD_PRINT_LIBRARIES_POST_LAUNCH: 显示加载之后动态加载的库
- DYLD_PRINT_SEGMENTS: 导出段映射
- DYLD_PRINT_STATISTICS: 显示运行时的统计信息

###### 进程地址空间

用户态的一个优点在于虚拟内存的隔离。进程独享一个私有的地址空间。和所有标准的C语言程序一样，OS X中的可执行文件也有一个标准的入口，默认名称为main。常见的做法是将main的实现为NSApplicationMain()的包装，然后通过NSApplicationMain()进入Objectiive-C的编程模型。

地址空间布局随机化(ASLR): 进程每一次启动时，地址空间都会被简单的随机化---只是偏移，而不是搅乱。基本的布局---程序文本、数据和库---仍然是一样的。

内存空间分为以下几个段:

- __PAGEZERO: 在32位的系统中，这是内存中单独的一个页面(4KB),而且这个页面的所有访问权限都被撤销了。在64位系统上，这个段对应了一个完整的32位地址空间(4GB)。这个段有助于捕捉空指针引用(因为空指针实际上就是0)，或捕捉将整数当作指针引用(因为32位平台下的4095以下的值，以及64位下4GB以下的值都在这个范围内)，由于这个范围内的所有访问权限都被撤销了，所以这个范围内的任何解引用操作都会引发来自MMU的硬件页错误，进而产生一个内核可以捕捉的陷阱。内核将这个陷阱转换为C++异常或表示总线错误的POSIX信号(SIGBUS)  
- __TEXT: 这个段存放的是程序代码。和其他所有操作系统一样，文本段被设置为r-x，这样不仅可以防止二进制代码在内存中被修改，还可以通过共享的这个只读段优化内存的使用。通过这种方式，同一个程序的多个实例可以仅使用一份__TEXT副本。文本段通常包含多个区，实际代码在_text区中。文本段还可以包含其他只读数据，如常量和硬编码字符串
- __LINKEDIT: 由dyld使用，这个区包含了字符串表、符号表以及其他数据
- __IMPORT: 用于i386的二进制文件的导入表
- __DATA: 用于可读、可写的数据
- __MALLOC_TINY: 用于小于一个页面大小的内存分配
- __MALLOC_SMALL: 用于几个页面大小的内存分配
- __MALLOC_LARGE: 用于1MB以上大小的内存分配


物理内存页面的生命周期包含以下几个状态:
- Free(空闲): 物理页面没有被任何虚拟内存页面使用。如果需要的话，可以立即回收
- Active(活跃): 物理页面当前正用于一个虚拟内存页面，而且最近页面被引用过。这个页面不太可能被交换出去，除非不再有任何非活跃的物理页面存在了。如过这个页面近期不被引用，则会被置于非活跃状态
- Inactive(非活跃): 物理页面当前正用于一个虚拟内存页面，但是最近没有被任何进程引用过。这个页面有可能在需要时被交换出去。另外，如果这个页面在任何时刻被引用了，那么会被重新置于活跃状态
- Speculative(投机): 页面被投机映射。通常产生这个状态的原因是针对可能的内存需求做了一次猜测分配，但是还没有处于活跃状态(也不是非活跃状态，因为有可能很快就会被访问)
- Wired down(联动): 物理页面当前正用于一个虚拟内存页面，但是不能被交换出去，不论引用状态如何

![image](./物理页面的状态迁移)

###### 可执行文件的运行过程

- 解析mach-o文件
- 设置运行环境参数
- - 文本段VM映射参数
- - 加载命令
- - 动态库信息
- - 符号表地址信息
- - 动态符号表地址信息
- - 常亮字符串表地址信息
- - 动态库加载信息
- - 符号函数地址
- - 依赖动态库信息
- - 动态链接器地址信息
- 根据动态库加载信息，把桩占位符，填写为指定用_nL_symbol_ptr的汇编指令
- 根据LC_MAIN的entry point调用指定entry offset偏移地址
- 执行entry offset相关二进制(逻辑是按照汇编指令，进行运行)
- 第一次运行到动态库函数时，进行一次懒加载动态绑定，并且动态链接器自动修改_la_symbol_ptr区的地址，指向动态库对应符号地址
- 第二次运行到动态库函数时，直接jmp到指定的符号地址

系统很多动态库都是共有的，所以XOS做了共享库的缓存优化，只要有相关进程使用过相关动态库，在另一进程，动态链接器在填桩时，直接会把桩_la_symbol_ptr区的地址，指向动态库对应符号的地址


###### iOS单个app最大内存占用限制

http://stackoverflow.com/questions/5887248/iOS-app-maximum-memory-budget

device: (crash amount/total amount/percentage of total)

iPad1: 127MB/256MB/49%
iPad2: 275MB/512MB/53%
iPad3: 645MB/1024MB/62%
iPad4: 585MB/1024MB/57% (iOS 8.1)
iPad Mini 1st Generation: 297MB/512MB/58%
iPad Mini retina: 696MB/1024MB/68% (iOS 7.1)
iPad Air: 697MB/1024MB/68%
iPad Air 2: 1195MB/2048MB/58% (iOS 8.x)
iPad Air 2: 1383MB/2048MB/68% (iOS 10.2.1)
iPad Pro 9.7": 1395MB/1971MB/71% (iOS 10.0.2 (14A456))
iPad Pro 12.9: 3064MB/3981MB/77% (iOS 9.3.2)
iPod touch 4th gen: 130MB/256MB/51% (iOS 6.1.1)
iPod touch 5th gen: 286MB/512MB/56% (iOS 7.0)
iPhone4: 325MB/512MB/63%
iPhone4s: 286MB/512MB/56%
iPhone5: 645MB/1024MB/62%
iPhone5s: 646MB/1024MB/63%
iPhone6: 645MB/1024MB/62% (iOS 8.x)
iPhone6+: 645MB/1024MB/62% (iOS 8.x)
iPhone6s: 1396MB/2048MB/68% (iOS 9.2)
iPhone6s+: 1392MB/2048MB/68% (iOS 10.2.1)
iPhoneSE: 1395MB/2048MB/69% (iOS 9.3)
iPhone7: 1395/2048MB/68% (iOS 10.2)。    
最低存储容量为32G,另外两个容量是128G和256G.
iPhone7+: 2040MB/3072MB/66% (iOS 10.2.1)      别名：A1661
存储有32GB/128GB以及256GB可选
iPhone8: 1364/1990MB/70% (iOS 12.1)
iPhone X: 1392/2785/50% (iOS 11.2.1)
iPhone XS: 2040/3754/54% (iOS 12.1)
iPhone XS Max: 2039/3735/55% (iOS 12.1)
iPhone XR: 1792/2813/63% (iOS 12.1)
iPhone 11: 2068/3844/54% (iOS 13.1.3)
iPhone 11 Pro Max: 2067/3740/55% (iOS 13.2.3)


###### Xcode 可执行文件生成过程

~~~
MBP:testClang yan$ clang -ccc-print-phases main.m
0: input, "main.m", objective-c
1: preprocessor, {0}, objective-c-cpp-output
2: compiler, {1}, ir
3: backend, {2}, assembler
4: assembler, {3}, object
5: linker, {4}, image
6: bind-arch, "x86_64", {5}, image
MBP:testClang yan$ 
~~~

源码->预处理 -> OC,C++混编->编译，生成中间代码->链接生成image可执行文件

###### 编译过程：

1.预处理

主要处理文件中以#开头的预编译命令 比如宏定义替换，引入的头文件进行内容替换

2.词法分析

将代码分解成一个一个独立的词法符合（Token）

3.语法分析

将词法分析的结果生成对应的抽象语法树，并验证语法的正确性

4.语义分析

对整个语句进行判别是否有问题

5.生成中间代码

编译器前端的产物，生成与机器无关的中间代码。

- Class/Meta Class/Protocol/Category 内存结构生成，并放在指定的section中。

- Method/Ivar/Property内存结构生成

- 组成method_list/iva_list/property_list并填入Class

- 为每个ivar合成偏移量

- 将语法树中的ObjCMessageExpr翻译成相应版本的objc_msgSend等等

- 根据修饰符strong、weak、copy、atomic合成@property自动实现的setter/getter

- 生成block_layout的数据结构

- block数据结构捕获相应的变量

- 分析对象引用关系，将objc_storeStrong/objc_storeWeak等ARC代码插入

- 自动实现调用【super dealloc】

- 为每个拥有ivar的class合成.cxx_destructor方法来自动释放类的成员变量

- Bitcode 生成字节码

6.目标代码生成与优化

Optimizer对前端产物进行优化，再交给后端根据不同的机器指令集生成对应的机器代码，生成对应的Mach-O文件

7.对Mach-O，静态文件等进行link，生成可执行文件，写入对应的.app包里

8.拷贝项目中的资源文件到对应的.app包里

9.编译xib和storyboard

10.link xib和storyboard的产物

11.如果有自定义的脚本，执行脚本（Build Phase 里的Run Script 可以设置自定义的脚本）

12.处理info.plist文件（具体处理什么，不太清楚）

13.生成DSYM文件

14.如果有混编swift，就会将swift的标准库拷贝到对应的.app包里（所以混编oc和swift会增大包体积）

15.代码签名

出于安全考虑，为了防止代码被篡改，最后的可执行文件会被codeSign进行签名。app运行的时候会去校验签名

###### 点击APP图标启动过程

内核先加载主程序

Load dylibs

dyld（动态链接器）启动，然后dyld开始递归加载所有的动态库（通过ImageLoader去加载）。

动态库有哪些好处？

- 代码共用 很多程序都动态链接了这些lib，但是内存和磁盘上仅有一份

- 易于维护 可以随时进行更新替换

- 减少了程序的可执行文件体积（运行时候才被加载）

Rebase

ASLR，地址空间布局随机化，因为动态库是被加载到随机地址上的，所以要对所有库内部进行地址指针修正

Bind

修改库对外的指针

Objc Runtime

bind操作结束后，会读取二进制文件的DATA段，找到与objc相关的信息，注册objc类，确保selector的唯一性，读取protocol以及category的信息

Initializers初始化

调用objc类的+load函数

调用c++中带有constructor标记的函数

执行main函数
