
笔者在一款基于 `LLVM` 编译器架构的 [`retdec`](https://bbs.kanxue.com/elink@b45K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2N6X3q4K6N6q4)9J5c8Y4u0W2N6r3c8W2j5H3`.`.) 开源反编译器工具的基础上，融合了 [`klee`](https://bbs.kanxue.com/elink@ba3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8L8r3g2W2i4K6u0r3K9$3I4W2k6b7`.`.) 符号执行工具，通过符号执行（`Symbolic Execution`）引擎动态模拟反编译后的 `LLVM` 的 `IR`（中间指令集）运行源程序的方法，插桩所有的对 `x86` 指令集的 `thiscall` 类型函数对 `this` 指针结构体（也就是 `rcx` 寄存器，简称 `this` 结构）偏移量引用，经行分析汇总后自动识别 `this` 结构体的具体内容，并自动集成导入 `ida` 工具辅助分析。

## 源码分析

### LLVM编译器简介

`LLVM` 命名最早源自于底层虚拟机（`Low Level Virtual Machine`）的缩写，由于命名带来的混乱，`LLVM` 就是该项目的全称。`LLVM` 核心库提供了与编译器相关的支持，可以作为多种语言编译器的后台来使用。能够进行程序语言的编译器优化、链接优化、在线编译优化、代码生成。`LLVM` 的项目是一个模块化和可重复使用的编译器和工具技术的集合。`LLVM` 是伊利诺伊大学的一个研究项目，提供一个现代化的，基于 `SSA`（`static single assignment` 静态单一赋值）的编译策略，能够同时支持静态和动态的任意编程语言的编译目标。自那时以来，已经成长为 `LLVM` 的主干项目，由不同的子项目组成，其中许多是正在生产中使用的各种商业和开源的项目，以及被广泛用于学术研究。本文介绍的几款相关工具都是基于 `LLVM` 编译器架构。

### Retdec源码分析

[`retdec`](https://bbs.kanxue.com/elink@811K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2N6X3q4K6N6q4)9J5c8Y4u0W2N6r3c8W2j5H3`.`.) 是一款基于 `LLVM` 编译器架构的支持多种机器码环境的跨平台反编译工具，输入为任意支持的二进制可执行文件，目标输出为一个可以再次编译的 `C` 语言的源码文件。

克隆 `git` 仓库后使用 [`Cmake GUI`](https://bbs.kanxue.com/elink@f93K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8h3q4C8k6g2)9J5k6h3!0J5k6#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3) 工具使用如下图配置将源工程及 `LLVM` 依赖工程转为 `Visual Studio 2017` 工程，笔者提供的工程已经是转换后的 `vs` 工程，可以直接在 `vs` 中打开编译。

![图1](https://bbs.kanxue.com/upload/attach/202204/609565_V2JMZSN5FXVNYS6.png)

![图2](https://bbs.kanxue.com/upload/attach/202204/609565_WNKRJT3WJD3HF9R.png)

![图3](https://bbs.kanxue.com/upload/attach/202204/609565_CSFS3BBETXMXPBF.png)

笔者的工程包含 2 个主要项目，一个是 `retdec-decompiler` 用于反编译输入二进制文件至 `C` 语言的源码输出文件，另外一个是 `IDA` 自动结构体识别插件，这 2 个都是依赖于 `llvm` 的独立项目。

`x86` 指令集的 [`__thiscall`](https://bbs.kanxue.com/elink@a70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6$3K9i4y4#2j5h3I4K6N6s2g2V1K9h3!0Q4x3V1k6$3K9i4y4#2j5h3I4Q4x3X3c8K6N6s2g2V1K9h3!0Q4x3X3b7J5x3o6p5J5i4K6u0r3k6h3D9^5N6r3E0X3j5Y4N6Q4x3U0S2$3i4K6y4p5N6Y4y4Q4x3X3f1I4x3e0l9\`.) 调用约定不同于 [`__stdcall`](https://bbs.kanxue.com/elink@77cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6$3K9i4y4#2j5h3I4K6N6s2g2V1K9h3!0Q4x3V1k6$3K9i4y4#2j5h3I4Q4x3X3c8K6N6s2g2V1K9h3!0Q4x3X3b7J5x3o6p5J5i4K6u0r3P5Y4S2C8x3s2c8%4z5e0y4Q4x3U0S2$3i4K6y4p5N6Y4y4Q4x3X3f1I4x3e0l9\`.) 参数从右向左被推入堆栈，而 `this` 指针由寄存器 `rcx` 传递而不是堆栈，也就是说 `rcx` 指向 `this` 结构体指针，一般是在类构造函数中申请的类结构体指针的地址，申请大小固定后不会改变。这个结构体由从派生类到基类的顺序叠加在一起，每个类的指针首地址指向一个 `vftable` 虚表函数结构体包含了当前类所有的包括重写虚函数在内的类成员函数数组，而这个 `vftable` 的地址减去一个 `sizeof(void*)` 大小后指向一个 `RTTICompleteObjectLocator` 结构体，它的相关结构体描述了这个类 `C++` 的所有多态性（运行时）相关信息，具体可以参考 [`dynamic_cast` 实现原理](https://bbs.kanxue.com/elink@a35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4m8S2M7%4y4A6L8$3&6Q4y4h3k6%4N6e0p5J5z5q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3K6R3#2x3e0p5&6y4e0N6Q4x3@1k6V1k6i4m8@1K9q4)9#2k6U0q4Q4x3X3b7\`.) 相关文章，相关结构体如下，`retdec` 原有代码分析 `x64` 的 `rtti` 结构体有问题，主要出在偏移量为 `int32` 类型是基于模块基址的偏移而不是绝对地址，这个问题笔者已修复。

```c
typedef struct TypeDescriptor
{
    void *    pVFTable; // Field overloaded by RTTI
    void *    spare; // reserved, possible for RTTI
    char    name[]; // The decorated name of the type; 0 terminated.类名称
} TypeDescriptor;

struct PMD
{
    ptrdiff_t mdisp; //member displacement vftable offset
    ptrdiff_t pdisp; //vftable displacement vftable offset
    ptrdiff_t vdisp; //displacement within vftable offset(for virtual base class)
};

typedef const struct _s_RTTIBaseClassDescriptor
{
    TypeDescriptor *pTypeDescriptor;
    DWORD                           numContainedBases;
    PMD                             where;
    DWORD                           attributes;
} _RTTIBaseClassDescriptor;

typedef const struct  _s_RTTIBaseClassArray
{
    _RTTIBaseClassDescriptor* arrayOfBaseClassDescriptors[3];
}_RTTIBaseClassArray;

typedef const struct _s_RTTIClassHierarchyDescriptor
{
    DWORD                           signature;
    DWORD                           attributes;
    DWORD                           numBaseClasses;
    _RTTIBaseClassArray *pBaseClassArray;
}_RTTIClassHierarchyDescriptor;

typedef const struct _s_RTTICompleteObjectLocator
{
    DWORD                           signature;
    DWORD                           offset; //vftbl相对this的偏移
    DWORD                           cdOffset; //constructor displacement
    TypeDescriptor *pTypeDescriptor;
    _RTTIClassHierarchyDescriptor *pClassDescriptor;
}_RTTICompleteObjectLocator;

typedef struct TypeDescriptor
{
#if defined(_WIN64) || defined(_RTTI) || defined(BUILDING_C1XX_FORCEINCLUDE)
    const void * pVFTable; // Field overloaded by RTTI
#else
    unsigned long    hash; // Hash value computed from type's decorated name
#endif
    void *    spare; // reserved, possible for RTTI
    char            name[]; // The decorated name of the type; 0 terminated.
} TypeDescriptor;
```

在缺少调试符号和 `dll` 导出函数名的情况下，一般最多只能通过解析二进制程序内的 `rtti` 结构体获得有限的可用信息，比如类名称和继承关系等，`ida` 默认会自动识别基于 `ms` 和 `gcc` 等编译器生成的 `rtti` 结构体，如果用户觉得不够全面可以使用 [`classinformer`](https://bbs.kanxue.com/elink@78fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2X3L8%4u0Y4k6g2)9J5k6h3&6W2N6q4)9J5c8Y4m8J5L8$3A6W2j5%4c8K6i4K6u0r3j5$3I4S2M7%4y4A6L8X3k6G2M7X3#2W2M7W2)9J5c8R3`.`.) 这款 `ida` 插件暴力搜索所有符合条件的 `rtti` 结构体，提供汇总分析功能。笔者工具中也提供一个功能将 `vftable` 所有的包含函数加入分析器统一分析，类似的功能还有对 `MFC` 窗口的 `AFX_MSGMAP_ENTRY` 结构体包含的相关目标函数分析。由于这些成员函数都是 `thiscall` 类型，所以 `rcx` 总是指向同一个派生类结构体，把这些函数根据 `vftable` 地址分类进行分析显然是个很好的办法。这些功能在工具中集成进了 `ida` 提供添加目标函数加入分析队列，具体使用方法见工具使用介绍节。

`LLVM` 整体架构，前端用的是 `clang`，广义的 `LLVM` 是指整个 `LLVM` 架构，一般狭义的 `LLVM` 指的是 `LLVM` 后端（包含代码优化和目标二进制代码生成），前端 `clang` 用于分析高级语言源代码产生后端的 `LLVM IR` 中间指令表示。`LLVM IR` 中间指令是一种“通用中间语言”，采用 `SSA` 形式，可以拥有无限多个虚拟寄存器且是抽象与高级语言和目标最终编译输出二进制文件架构约束无关，它通过提供类型信息可以用于进一步提供优化转化为目标代码生成也可以反过来转化成类似 `C` 的高级语言表示。编译为目标二进制代码生成的功能不在本文的讨论范围，文本主要讨论反编译过程和模拟 `LLVM IR` 中间指令执行分析获得 `this` 结构体的相关信息实现方式。

`retdec` 工具使用的反编译引擎是 [`capstone2llvmir`](https://bbs.kanxue.com/elink@e55K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9s2g2T1j5Y4W2E0j5h3N6Y4K9h3g2Q4x3V1k6U0j5i4m8K6N6r3!0F1k6e0u0D9L8s2k6E0K9i4t1\`.)，主要由 `retdec-decoder` 这个 `llvm Passe` 负责将源程序二进制汇编 `ASM` 代码转化成 `LLVM IR` 中间指令，在转换初始化之前从程序入口、调试信息和 `Vftables` 获取静态已知的跳转目标 `JumpTargets` 用于构造整个程序的 `LLVM IR Selection DAG`（`directed acyclic graph` 有向无环图），对后续的跳转目标进行指令集顺序上的调度优化转化为 `retdec` 的等价 `DAG` 的节点 `CFGNode` 及其组织架构。
