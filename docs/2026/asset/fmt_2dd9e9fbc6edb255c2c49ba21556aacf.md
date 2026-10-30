
```cpp
#include <iostream>
#include <fstream>
#include "pin.H"
using std::cerr;
using std::endl;
using std::ios;
using std::ofstream;
using std::string;
ofstream OutFile;
// 记录运行的指令的数量，设置为静态变量方便编译器优化docount函数
static UINT64 icount = 0;
// 在每个块之前调用该函数
// 对calls使用fast linkage
VOID PIN_FAST_ANALYSIS_CALL docount(ADDRINT c) { icount += c; }
// Pin在遇到一个新块时调用，插入对docount 函数的调用
VOID Trace(TRACE trace, VOID* v)
{
// 检查trace中的每个基本块
for (BBL bbl = TRACE_BblHead(trace); BBL_Valid(bbl); bbl = BBL_Next(bbl))
{
// 对每个bbl插入对docount函数的调用，将指令数量作为参数传递
// IPOINT_ANYWHERE参数允许Pin在bbl内部任意位置插入call以获取最好的性能
// 对call使用fast linkage
BBL_InsertCall(bbl, IPOINT_ANYWHERE, AFUNPTR(docount), IARG_FAST_ANALYSIS_CALL, IARG_UINT32, BBL_NumIns(bbl), IARG_END);
}
}
KNOB< string > KnobOutputFile(KNOB_MODE_WRITEONCE, "pintool", "o", "inscount.out", "specify output file name");
// 程序退出时调用
VOID Fini(INT32 code, VOID* v)
{
OutFile.setf(ios::showbase);
OutFile << "Count " << icount << endl;
OutFile.close();
}
/* ===================================================================== */
/* Print Help Message */
/* ===================================================================== */
INT32 Usage()
{
cerr << "This tool counts the number of dynamic instructions executed" << endl;
cerr << endl << KNOB_BASE::StringKnobSummary() << endl;
return -1;
}
/* ===================================================================== */
/* Main */
/* ===================================================================== */
int main(int argc, char* argv[])
{
// 初始化Pin
if (PIN_Init(argc, argv)) return Usage();
OutFile.open(KnobOutputFile.Value().c_str());
// 注册插桩函数Trace
TRACE_AddInstrumentFunction(Trace, 0);
// 注册Fini函数
PIN_AddFiniFunction(Fini, 0);
// 开始执行，不返回
PIN_StartProgram();
return 0;
}
```

这里`IPOINT`是一个枚举类型，决定了分析call被插入到什么地方。插入的对象可以是`INS`、`BBL`、`TRACE`、`RTN`，其完整可用的值如下：

- `IPOINT_BEFORE`：在插桩对象的第一条指令之前插入call，总是有效
- `IPOINT_AFTER`：在插桩对象的最后一条指令的失败路径处插入call
- 如果是routine（`RTN`），在所有返回路径处插桩
- 如果是instruction（`INS`），仅在`INS_IsValidForIpointAfter()`函数为真的情况下适用
- 如果是`BBL`，仅在`BBL_HasFallThrough()`函数为真的情况下适用
- 如果是`TRACE`，仅在`TRACE_HasFallThrough()`函数为真的情况下适用
- `IPOINT_ANYWHERE`：在插桩对象的任意位置插入call，不适用`INS_InsertCall()`和`INS_InsertThenCall()`函数
- `IPOINT_TAKEN_BRANCH`：在插桩对象的控制流的执行边界处插入call，仅适用于`INS_IsValidForIpointTakenBranch()`返回真的情况。

### 7.5 Using Fast Call Linkages

对于一些比较“小”的函数来说，对函数的调用开销有时与函数自身的运算开销基本相同，因此一些编译器会提供一些调用链接优化机制来降低开销。例如，IA-32下的`gcc`有一个在寄存器中传递参数的`regparm`属性。

Pin中有一定数量的备用链接，使用`PIN_FAST_ANALYSIS_CALL`来声明分析函数即可使用，而插桩函数`InsertCall`则需要使用`IARG_FAST_ANALYSIS_CALL`。如果二者只更改了一个，那么就可能出现传参错误。例如前面给出的源码例子就使用了fast call linkages：

```cpp
... ...
// 对分析函数使用fast linkage
VOID PIN_FAST_ANALYSIS_CALL docount(ADDRINT c) { icount += c; }
VOID Trace(TRACE trace, VOID* v)
{
// 检查trace中的每个基本块
for (BBL bbl = TRACE_BblHead(trace); BBL_Valid(bbl); bbl = BBL_Next(bbl))
{
// 对插桩函数使用fast linkage
BBL_InsertCall(bbl, IPOINT_ANYWHERE, AFUNPTR(docount), IARG_FAST_ANALYSIS_CALL, IARG_UINT32, BBL_NumIns(bbl), IARG_END);
}
}
... ...
```

在对比较复杂的大型函数使用该方法时，效果并不明显，但不会造成性能的下降。

第二个调用链接优化是消除帧指针。如果使用`gcc`，则推荐加上`-fomit-frame-pointer`选项。Pin官方的标准Pintool的`makefile`包括该选项。与`PIN_FAST_ANALYSIS_CALL`一样，该选项对“小”函数的效果比较明显。需要注意的是，debugger会根据帧指针来显示堆栈回溯情况，所以如果想调试Pintool的话，就不要设置该选项。如果使用标准的Pintool的`makefile`来进行变异，则可以通过修改`OPT`选项来进行改变：

```shell
make OPT=-O0
```

### 7.6 Rewriting Conditional Analysis Code to Help Pin Inline

Pin通过自动内联没有控制流变化的分析routine来提升插桩性能。但是有很多分析routine是有控制流的，最典型的就是有一个简单的“if-then”的条件语句，它只会执行少量的分析代码，并“then”部分只执行一次。为了将这类的语句转换为常规的没有控制流变化的语句，Pin提供了一些插桩API来重写分析routine。下面是一个重写的例子：

例如我们当前想要实现的一个分析routine的代码如下：

```cpp
// IP-sampling分析routine实现:
VOID IpSample(VOID* ip)
{
--icount;
if (icount == 0)
{
fprintf(trace, "%p\n", ip);
icount = N + rand() % M;
}
}
```

在原始的`IpSample()`函数中有一个明显的条件语句，会存在控制流的变化。如何消除该条件控制流的存在呢？

可以看到分析routine内部其实可以拆解为2部分功能：`icount`的自减和“if”语句，那么可以使用两个单独的函数实现。而且，前者比后者的执行频率要更高。拆解后的代码如下：

```cpp
/*
* IP-sampling分析routine实现:
*
* VOID IpSample(VOID* ip)
* {
* --icount;
* if (icount == 0)
* {
* fprintf(trace, "%p\n", ip);
* icount = N + rand() % M;
* }
* }
*/
// 计算icount
ADDRINT CountDown()
{
--icount;
return (icount == 0);
}
// 打印当前指令的IP并且icount被重置为N和N+M中的一个随机数
VOID PrintIp(VOID* ip)
{
fprintf(trace, "%p\n", ip);
// 准备下次计算
icount = N + rand() % M;
}
```

一个完整的实现消除控制流变化的代码如下：

```cpp
/* source/tools/ManualExamples/isampling.cpp */
#include <stdio.h>
#include <stdlib.h>
#include "pin.H"
FILE* trace;
const INT32 N = 100000;
const INT32 M = 50000;
INT32 icount = N;
/*
* IP-sampling分析routine实现:
*
* VOID IpSample(VOID* ip)
* {
* --icount;
* if (icount == 0)
* {
* fprintf(trace, "%p\n", ip);
* icount = N + rand() % M;
* }
* }
*/
// 计算icount
ADDRINT CountDown()
{
--icount;
return (icount == 0);
}
// 打印当前指令的IP并且icount被重置为N和N+M中的一个随机数
VOID PrintIp(VOID* ip)
{
fprintf(trace, "%p\n", ip);
// 准备下次计算
icount = N + rand() % M;
}
VOID Instruction(INS ins, VOID* v)
{
// 每条指令执行后都会调用CountDown()
INS_InsertIfCall(ins, IPOINT_BEFORE, (AFUNPTR)CountDown, IARG_END);
// 只有当CountDown返回非0值时才会调用PrintIp()
INS_InsertThenCall(ins, IPOINT_BEFORE, (AFUNPTR)PrintIp, IARG_INST_PTR, IARG_END);
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
PIN_ERROR("This Pintool samples the IPs of instruction executed\n" + KNOB_BASE::StringKnobSummary() + "\n");
return -1;
}
/* ===================================================================== */
/* Main */
/* ===================================================================== */
int main(int argc, char* argv[])
{
trace = fopen("isampling.out", "w");
if (PIN_Init(argc, argv)) return Usage();
INS_AddInstrumentFunction(Instruction, 0);
PIN_StartProgram();
return 0;
}
```

使用条件插桩API `INS_InsertIfCall()`和`INS_InsertThenCall()`来告诉Pin只有当`CountDown()`执行结果非0时，才执行`PrintIp()`。这样一来，`CountDown()`函数就可以内联在Pin中，对于没有内联的`PrintIp()`则只有在满足条件时才会执行一次。

`INS_InsertThenCall()`插进去的函数只有在`INS_InsertIfCall()`插进去的函数返回非0值时才会执行。这个功能可以说是一个十分巧妙的功能。

## 8. Building Your Own Tool

在开发自己的Pintool时，可以copy一份`example`目录，然后在`makefile.rules`文件中添加上自己的tool，可以以最简单的`MyPinTool`为模版。

### 8.1 Building a Tool From Within the Kit Directory Tree

如果直接修改`MyPinTool`，并且没有特殊的编译需求，则直接使用默认配置就好。如果要新增tool或者需要指定特殊的构建标志，则需要修改`makeifile.rules`文件。

构建`YourTool.so`(源文件为`YourTool.cpp`)：

```shell
make obj-intel64/YourTool.so
```

如果想编译成IA-32架构，则使用`obj-ia32`替换`obj-intel64`即可。

### 8.2 Building a Tool Out of the Kit Directory Tree

copy文件夹`MyPinTool`到指定位置子，然后编辑`makefile.rules`文件。

```shell
make PIN_ROOT=<path to Pin kit> obj-intel64/YourTool.so
```

要更改将创建工具的目录，可以从命令行覆盖`OBJDIR`变量：

```shell
make PIN_ROOT=<path to Pin kit> OBJDIR=<path to output dir> <path to output dir>/YourTool.so
```

## 9. Pin's makefile Infrastructure

### 9.1 The Config Directory

目录`source/tools/Config`中存放了`make`配置的基本文件，不要轻易修改这些文件，可以基于其中的模版文件进行更新。

![image-20211012171957115](upload/attach/202110/779730_K3PRSSZEU89PR9G.jpg)

下面对其中的几个关键文件进行说明：

- `makefile.config`：在include链中第一个应该include的文件。它保存了用户可用的所有相关标识和变量的文档，此外还包括特定于OS的配置文件。
- `unix.vars`：该文件包含`makefile`使用的一些架构变量和实用程序的Unix定义。
- `makefile.default.rules`：该文件包含默认的`make`目标、测试用例和构建规则。

### 9.2 The Test Directories

`source/tools`目录下的每个测试性质的目录中都包含`makefile`链中的两个文件：

![image-20211012172558700](upload/attach/202110/779730_U6GGDWYX9S73C23.jpg)

- `makefile`：运行`make`时调用，不要修改。其中保存了`makefile`链的所有相关配置文件的包含指令，属于通用文件，在所有的测试目录中都是相同的。
- `makefile.rules`：目录特定文件，不同测试目录，文件内容不同。它保存了当前目录的逻辑，应该在目录中构建和运行的所有工具、应用程序和测试等都在该文件中进行定义。

### 9.3 Adding Tests, Tools and Applications to the `makefile`

下面介绍如何通过`makefile`构建二进制程序并运行测试。以下描述的变量都在`makefile.rules`文件的"Test targets"部分进行描述：

- `TOOL_ROOTS`：定义工具名称，不带文件扩展名，具体的文件扩展名将由`make`自动添加，例如`YourTools.so`；
- `APP_ROOTS`：定义应用程序，不带文件扩展名，具体的文件扩展名将由`make`自动添加，例如`YourApp.exe`；
- `TEST_ROOTS`：定义测试，不要加`.test`后缀，`make`会自动添加，例如`YourTest.test`。
