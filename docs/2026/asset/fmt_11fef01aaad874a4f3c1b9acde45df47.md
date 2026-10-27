
# [翻译]VS2019官方文档_64位异常处理Part1

最近在学习64位逆向, 于是想深入理解一下异常处理的流程与机制, 查询了官方文档之后发现并没有中文版, 微软官方提供的文档中也只有google翻译自动生成的文章. 于是就站在google机翻的肩膀上, 翻译了异常处理的数据结构介绍部分.

有一些地方理解的并不清晰, 如果翻译得有问题希望指正

前一段为原文, 后一段为对应的翻译

同时上传了PDF版, 后续如果有修改会再次上传

**x64 exception handling**

---

文档来源

- <mark class="encrypted">5f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3y4H3M7q4)9J5c8X3u0#2K9h3I4V1i4K6u0r3k6i4S2U0k6i4m8@1K9h3!0F1i4K6u0V1K9r3q4F1k6r3I4A6L8X3N6Q4x3X3c8^5y4U0c8Q4x3@1k6$3K9h3g2%4i4K6y4p5N6Y4y4Q4x3X3b7J5x3o6p5&6</mark>
- <mark class="encrypted">f82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3y4H3M7q4)9J5c8X3y4H3M7q4)9J5c8X3y4H3M7q4)9J5k6r3g2^5j5$3g2H3N6r3W2G2L8W2)9J5k6r3S2S2L8X3c8D9K9h3&6Y4i4K6y4r3N6X3W2W2N6#2)9K6c8s2k6K6i4K6u0V1x3U0l9I4z5b7`.`.</mark>

An overview of structured exception handling and C++ exception handling coding conventions and behavior on the x64. For general information on exception handling, see . <a href="elink@1d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3y4H3M7q4)9J5c8X3y4H3M7q4)9J5c8X3g2^5j5$3g2H3N6r3W2G2L8W2)9J5k6r3S2S2L8X3c8D9K9h3&6Y4i4K6u0V1K9h3&6Q4x3X3c8$3K9i4y4#2j5h3I4Q4x3X3c8U0M7s2m8Q4x3@1k6$3K9h3g2%4i4K6y4p5N6Y4y4Q4x3X3b7J5x3o6p5&6">Exception Handling in Visual C++</a>

**异常处理结构化与C++异常处理在64位系统下的代码行为与调用约定的概述. 对于异常处理的更多介绍, 参考<a href="elink@3dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3y4H3M7q4)9J5c8X3y4H3M7q4)9J5c8X3g2^5j5$3g2H3N6r3W2G2L8W2)9J5k6r3S2S2L8X3c8D9K9h3&6Y4i4K6u0V1K9h3&6Q4x3X3c8$3K9i4y4#2j5h3I4Q4x3X3c8U0M7s2m8Q4x3@1k6$3K9h3g2%4i4K6y4p5N6Y4y4Q4x3X3b7J5x3o6p5&6">Exception Handling in Visual C++</a>**

---

## C++ 中的try-catch 异常Example

The following example throws a `char *` exception, but does not contain a handler designated to catch exceptions of type `char *`. The call to `set_terminate` instructs `terminate` to call `term_func`.

**下面的例子抛出一个`char *`类型的异常, 但是不包含对应的句柄去处理该类异常. `set_terminate` 函数命令 `terminate` 去调用 `term_func` 函数**

```C++
// exceptions_Unhandled_Exceptions.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;
void term_func() {
cout << "term_func was called by terminate." << endl;
exit( -1 );
}
int main() {
try
{
set_terminate( term_func );
throw "Out of memory!"; // 没有对应的catch句柄对应该异常
}
catch( int )
{
cout << "Integer exception raised." << endl;
}
return 0;
}
```

---

## Unwind data for exception handling, debugger support **支持调试器的异常处理的数据展开**

Several data structures are required for exception handling and debugging support.

**为了处理异常并支持调试器, 本段将涉及到一下几个数据结构**

### struct `RUNTIME_FUNCTION`

Table-based exception handling requires a table entry for all functions that allocate stack space or call another function (for example, nonleaf functions). Function table entries have the format:

**函数中, 基于表格的处理异常方式都需要一个表项, 包含所有被置于栈空间中的函数 与 所有调用其他函数的函数 (例如, 非叶函数). 函数表项有如下的布局:**

| 类型 | 说明 | 翻译 |
| --- | --- | --- |
| `ULONG` | `Function start address` | 函数起始地址 |
| `ULONG` | `Function end address` | 函数结束地址 |
| `ULONG` | `Unwind info address` | 展开信息的地址 |

The `RUNTIME_FUNCTION` structure must be `DWORD` aligned in memory. All addresses are image relative, that is, they are 32-bit offsets from the starting address of the image that contains the function table entry. These entries are sorted, and put in the `.pdata` section of a `PE32+` image. For dynamically generated functions [JIT compilers], the runtime to support these functions must either use `RtlInstallFunctionTableCallback` or `RtlAddFunctionTable` to provide this information to the operating system. Failure to do so will result in unreliable exception handling and debugging of processes.

**`RUNTIME_FUNCTION` 结构体在内存中必须为四字节对齐. 所有地址为 整个文件映射到内存中后的地址存储 (`RVA`方式存储), 也就是说, 从包含函数表项的内存地址处, 第32位偏移开始计算. 这些表项被依次排序, 并放置入`PE32+`头部内存的`.pdata`段中. 对于动态生成的函数[JIT 编译器], 想要运行时支持这些函数必须 使用 `RtlInstallFunctionTableCallback` 或 `RtlAddFunctionTable` 这两个函数, 将该信息添加进操作系统. 如果不这么做, 将导致不可靠的异常处理与进程调试.**

---

### struct `UNWIND_INFO`

The unwind data info structure is used to record the effects a function has on the stack pointer and where the nonvolatile registers are saved on the stack:

**展开的数据信息结构被用来记录一个函数对栈指针的影响, 与寄存器保存在堆栈中的位置**

| 类型 | 说明 | 翻译 |
| --- | --- | --- |
| `UBYTE`: 3 | `Version` | 版本 |
| `UBYTE`: 5 | `Flags` | 标志位 |
| `UBYTE` | `Size of prolog` | <a href="elink@069K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3y4H3M7q4)9J5c8X3u0#2K9h3I4V1i4K6u0r3M7s2u0G2L8r3!0Y4i4K6u0V1j5h3&6V1i4K6u0V1k6i4m8A6L8r3!0Y4i4K6y4r3N6X3W2W2N6#2)9K6c8s2k6K6i4K6u0V1x3U0l9I4z5b7`.`.">Prolog</a>的大小 |
| `UBYTE` | `Count of unwind codes` | 展开码的大小 |
| `UBYTE`: 4 | `Frame Register` | 帧寄存器 |
| `UBYTE`: 4 | `Frame Register offset (scaled)` | 帧寄存器偏移(比例) |
| `USHORT` * n | `Unwind codes array` | 展开码的数组 |
| `variable` | `Can either be of form (1) or (2) below` | 可以是下方的(1)或(2) |

#### (1) Exception Handler **异常处理**

| 类型 | 说明 | 翻译 |
| --- | --- | --- |
| `ULONG` | `Address of exception handler` | 异常处理的地址 |
| `variable` | `Language-specific handler data (optional)` | 特定编程语言的异常处理数据(可选) |

#### (2) Chained Unwind Info **链式展开信息**

| 类型 | 说明 | 翻译 |
| --- | --- | --- |
| `ULONG` | `Function start address` | 函数起始地址 |
| `ULONG` | `Function end address` | 函数结束地址 |
| `ULONG` | `Unwind info address` | 展开信息的地址 |

The `UNWIND_INFO` structure must be `DWORD` aligned in memory. Here's what each field means:

**`UNWIND_INFO`结构体在内存中必须为四字节对齐, 以下是各成员的含义:**

- `Version`: Version number of the unwind data, currently 1. / 展开数据的版本号, 当前为1.

- `Flags`: Three flags are currently defined. / 三个标志位现在被定义为:

| 类型 | 说明 | 翻译 |
| --- | --- | --- |
| `UNW_FLAG_EHANDLER` | The function has an exception handler that should be called when looking for functions that need to examine exceptions. | 当寻找需要检查异常的函数时, 函数需要调用该异常处理程序 |
| `UNW_FLAG_UHANDLER` | The function has a termination handler that should be called when unwinding an exception. | 当展开一个异常时，函数应调用该终止处理程序。 |
| `UNW_FLAG_CHAININFO` | This unwind info structure is not the primary one for the procedure. Instead, the chained unwind info entry is the contents of a previous `RUNTIME_FUNCTION` entry. For informations, see Chained unwind info structures. If this flag is set, then the `UNW_FLAG_EHANDLER` and `UNW_FLAG_UHANDLER` flags must be cleared. Also, the frame register and fixed-stack allocation fields must have the same values as in the primary unwind info. | 此展开信息结构不是主要过程。 链式展开信息项是上一个`RUNTIME_FUNCTION`结构项的内容。 有关信息，请参阅链式展开信息结构。 如果此标志被设置，那么 `UNW_FLAG_EHANDLER` 和 `UNW_FLAG_UHANDLER` 标志必须被清除。 此外，帧寄存器和固定堆栈分配字段必须与主展开信息中具有相同的信息。 |

- `Size of prolog`: Length of the function prolog in bytes. / 以字节为单位的 该函数的Prolog的长度.

- `Count of unwind codes`: The number of slots in the unwind codes array. Some unwind codes, for example, `UWOP_SAVE_NONVOL`, require more than one slot in the array. / 展开码数组中存储的元素个数. 一些展开的代码, 例如`UWOP_SAVE_NONVOL`, 需要在数组中占据多于一个位置的大小.

- `Frame register`: If nonzero, then the function uses a frame pointer (FP), and this field is the number of the nonvolatile register used as the frame pointer, using the same encoding for the operation info field of `UNWIND_CODE` nodes. / 如果不为零, 那么该函数使用帧指针(FP), 该字段是非易失性寄存器作为帧指针的个数, 与`UNWIND_CODE`节的操作信息段使用相同的编码.

- `Frame register offset (scaled)`: If the frame register field is nonzero, this field is the scaled offset from `RSP` that is applied to the `FP` register when it's established. The actual `FP` register is set to `RSP + 16 * this number`, allowing offsets from 0 to 240. This offset permits pointing the `FP` register into the middle of the local stack allocation for dynamic stack frames, allowing better code density through shorter instructions (more instructions can use the 8-bit signed offset form). / 帧寄存器的偏移(比例)
