---
layout: default
title: MachO
nav_order: 5
parent: iOS
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


