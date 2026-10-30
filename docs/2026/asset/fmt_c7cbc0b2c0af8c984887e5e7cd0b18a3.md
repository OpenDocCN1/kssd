
原理：在调用分析程序时，Pin 允许传递指令指针、寄存器当前值、内存操作的有效地址、常量等数据给分析程序。完整的可传递的参数的类型如下：`IARG_TYPE`。将指令计数程序中的参数更改为`INS_InsertCall`来传递即将执行的指令的地址，将`docount`更改为`printip`来打印指令的地址，最后将输出写入到文件`itrace.out`中。

源码`source/tools/ManualExamples/itrace.cpp`：

```cpp
#include <stdio.h>
#include "pin.H"
FILE* trace;
// 在每条指令执行前都会被调用，打印出当前指令的地址
VOID printip(VOID* ip) { fprintf(trace, "%p\n", ip); }
// 遇到一条新指令调用一次
VOID Instruction(INS ins, VOID* v)
{
// 在每条指令前插入对 printip 函数的调用，并传递 ip 参数
INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)printip, IARG_INST_PTR, IARG_END);
}
// 结束函数
VOID Fini(INT32 code, VOID* v)
{
fprintf(trace, "#eof\n");
fclose(trace);
}
/* ===================================================================== */
/* Print Help Message */
/* ===================================================================== */
INT32 Usage()
{
PIN_ERROR("This Pintool prints the IPs of every instruction executed\n" + KNOB_BASE::StringKnobSummary() + "\n");
return -1;
}
/* ===================================================================== */
/* Main */
/* ===================================================================== */
int main(int argc, char* argv[])
{
trace = fopen("itrace.out", "w");
// 初始化
if (PIN_Init(argc, argv)) return Usage();
// 桩指令注册
INS_AddInstrumentFunction(Instruction, 0);
// 结束逻辑注册
PIN_AddFiniFunction(Fini, 0);
// 开始执行，不返回
PIN_StartProgram();
return 0;
}
```

### 4. Memory Reference Trace（指令插桩）

功能：内存引用追踪（只对读写内存的指令插桩）

运行和查看输出：

```shell
$ ../../../pin -t obj-intel64/pinatrace.so -- /bin/ls
Makefile atrace.o imageload.o inscount0.o itrace.out
Makefile.example atrace.out imageload.out itrace proccount
atrace imageload inscount0 itrace.o proccount.o
$ head pinatrace.out
0x40001ee0: R 0xbfffe798
0x40001efd: W 0xbfffe7d4
0x40001f09: W 0xbfffe7d8
0x40001f20: W 0xbfffe864
0x40001f20: W 0xbfffe868
0x40001f20: W 0xbfffe86c
0x40001f20: W 0xbfffe870
0x40001f20: W 0xbfffe874
0x40001f20: W 0xbfffe878
0x40001f20: W 0xbfffe87c
$
```

原理：Pin 中包含可以对指令进行分类和检查功能的 API，通过调用该 API 可以实现对某一类功能的函数的追踪。

源码`source/tools/ManualExamples/itrace.cpp`：

```cpp
/*
 * This file contains an ISA-portable PIN tool for tracing memory accesses.
 */
#include <stdio.h>
#include "pin.H"
FILE* trace;
// 打印地址读的指令的地址
VOID RecordMemRead(VOID* ip, VOID* addr) { fprintf(trace, "%p: R %p\n", ip, addr); }
// 打印地址写的指令的地址
VOID RecordMemWrite(VOID* ip, VOID* addr) { fprintf(trace, "%p: W %p\n", ip, addr); }
// 使用谓词函数调用来检测内存访问
// 每个读和写的指令都会调用
VOID Instruction(INS ins, VOID* v)
{
// 获取指令中的内存操作数计数
UINT32 memOperands = INS_MemoryOperandCount(ins);
// 遍历指令中的每个内存操作数
for (UINT32 memOp = 0; memOp < memOperands; memOp++)
{
// 如果是内存读
if (INS_MemoryOperandIsRead(ins, memOp))
{
INS_InsertPredicatedCall(ins, IPOINT_BEFORE, (AFUNPTR)RecordMemRead, IARG_INST_PTR, IARG_MEMORYOP_EA, memOp,
IARG_END);
}
// 在某些架构下，内存操作数可以同时用作读和写，例如 IA-32 的 %eax，这种情况下只记录一次
// 如果是写
if (INS_MemoryOperandIsWritten(ins, memOp))
{
INS_InsertPredicatedCall(ins, IPOINT_BEFORE, (AFUNPTR)RecordMemWrite, IARG_INST_PTR, IARG_MEMORYOP_EA, memOp,
IARG_END);
}
}
}
VOID Fini(INT32 code, VOID* v)
{
fprintf(trace, "#eof\n");
fclose(trace);
}
/* ===================================================================== */
/* Print Help Message */
/* ===================================================================== */
INT32 Usage()
{
PIN_ERROR("This Pintool prints a trace of memory addresses\n" + KNOB_BASE::StringKnobSummary() + "\n");
return -1;
}
/* ===================================================================== */
/* Main */
/* ===================================================================== */
int main(int argc, char* argv[])
{
if (PIN_Init(argc, argv)) return Usage();
trace = fopen("pinatrace.out", "w");
// 注册桩函数
INS_AddInstrumentFunction(Instruction, 0);
// 注册结束函数
PIN_AddFiniFunction(Fini, 0);
// 开始，不返回
PIN_StartProgram();
return 0;
}
```

### 5. Detecting the loading and Unloading of Images（image插桩）

功能：在 image 加载和卸载时打印信息到 trace 文件中。

执行和查看输出：

```shell
$ ../../../pin -t obj-intel64/imageload.so -- /bin/ls
Makefile atrace.o imageload.o inscount0.o proccount
Makefile.example atrace.out imageload.out itrace proccount.o
atrace imageload inscount0 itrace.o trace.out
$ cat imageload.out
Loading /bin/ls
Loading /lib/ld-linux.so.2
Loading /lib/libtermcap.so.2
Loading /lib/i686/libc.so.6
Unloading /bin/ls
Unloading /lib/ld-linux.so.2
Unloading /lib/libtermcap.so.2
Unloading /lib/i686/libc.so.6
$
```

原理：本质上没有对 image 文件进行插桩。

源码`source/tools/ManualExamples/imageload.cpp`：

```cpp
//
// This tool prints a trace of image load and unload events
//
#include "pin.H"
#include <iostream>
#include <fstream>
#include <stdlib.h>
using std::endl;
using std::ofstream;
using std::string;
KNOB< string > KnobOutputFile(KNOB_MODE_WRITEONCE, "pintool", "o", "imageload.out", "specify file name");
ofstream TraceFile;
// Pin 在 image 加载时调用该函数，在该例中没有进行插桩
VOID ImageLoad(IMG img, VOID* v) { TraceFile << "Loading " << IMG_Name(img) << ", Image id = " << IMG_Id(img) << endl; }
// Pin 在 image 卸载时调用该函数，对于将要卸载的image无法进行插桩
VOID ImageUnload(IMG img, VOID* v) { TraceFile << "Unloading " << IMG_Name(img) << endl; }
// This function is called when the application exits
// It closes the output file.
VOID Fini(INT32 code, VOID* v)
{
if (TraceFile.is_open())
{
TraceFile.close();
}
}
/* ===================================================================== */
/* Print Help Message */
/* ===================================================================== */
INT32 Usage()
{
PIN_ERROR("This tool prints a log of image load and unload events\n" + KNOB_BASE::StringKnobSummary() + "\n");
return -1;
}
/* ===================================================================== */
/* Main */
/* ===================================================================== */
int main(int argc, char* argv[])
{
// 符号初始化
PIN_InitSymbols();
// pin 初始化
if (PIN_Init(argc, argv)) return Usage();
TraceFile.open(KnobOutputFile.Value().c_str());
// 注册加载桩函数
IMG_AddInstrumentFunction(ImageLoad, 0);
// 注册卸载桩函数
IMG_AddUnloadFunction(ImageUnload, 0);
// 注册退出函数
PIN_AddFiniFunction(Fini, 0);
// 开始执行，无返回
PIN_StartProgram();
return 0;
}
```

### 6. More Efficient Instruction Counting（trace 插桩）

功能：计算 BBL （单入口单出口）数量

执行和查看输出：

```shell
$ ../../../pin -t obj-intel64/inscount1.so -o inscount.out -- /bin/ls
Makefile atrace.o imageload.out itrace proccount
Makefile.example imageload inscount0 itrace.o proccount.o
atrace imageload.o inscount0.o itrace.out
$ cat inscount.out
Count 707208
```

原理：在每个 BBL 进行插桩来替代在每个指令进行插桩，在进行计数时，以 bbl 为单位，每次增加每个 bbl 中的指令数量。

源码`source/tools/ManualExamples/inscount1.cpp`：
