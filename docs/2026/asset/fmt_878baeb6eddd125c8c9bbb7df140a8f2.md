
```cpp
#include <stdio.h>
#include "pin.H"
#include <iostream>
#include <fstream>
using std::cerr;
using std::endl;
std::ofstream\* out = 0;
//=======================================================
// Analysis routines
//=======================================================
// 从内存转移到寄存器中
ADDRINT DoLoad(REG reg, ADDRINT\* addr)
{
\*out << "Emulate loading from addr " << addr << " to " << REG\_StringShort(reg) << endl;
ADDRINT value;
PIN\_SafeCopy(&value, addr, sizeof(ADDRINT));
return value;
}
//=======================================================
// Instrumentation routines
//=======================================================
VOID EmulateLoad(INS ins, VOID\* v)
{
// Find the instructions that move a value from memory to a register
if (INS\_Opcode(ins) == XED\_ICLASS\_MOV && INS\_IsMemoryRead(ins) && INS\_OperandIsReg(ins, 0) && INS\_OperandIsMemory(ins, 1))
{
// op0 <- \*op1
INS\_InsertCall(ins, IPOINT\_BEFORE, AFUNPTR(DoLoad), IARG\_UINT32, REG(INS\_OperandReg(ins, 0)), IARG\_MEMORYREAD\_EA,IARG\_RETURN\_REGS, INS\_OperandReg(ins, 0), IARG\_END);
// Delete the instruction
INS\_Delete(ins);
}
}
/\* ===================================================================== \*/
/\* Print Help Message \*/
/\* ===================================================================== \*/
INT32 Usage()
{
cerr << "This tool demonstrates the use of SafeCopy" << endl;
cerr << endl << KNOB\_BASE::StringKnobSummary() << endl;
return -1;
}
/\* ===================================================================== \*/
/\* Main \*/
/\* ===================================================================== \*/
int main(int argc, char\* argv[])
{
// Write to a file since cout and cerr maybe closed by the application
out = new std::ofstream("safecopy.out");
// 初始化Pin，初始化符号
if (PIN\_Init(argc, argv)) return Usage();
PIN\_InitSymbols();
// 注册EmulateLoad函数以进行插桩
INS\_AddInstrumentFunction(EmulateLoad, 0);
// 开始执行，不返回
PIN\_StartProgram();
return 0;
}
```

### 9. Order of Instrumentation

Pin提供了多种方式来控制`analysis call`的执行顺序，主要取决于插入点`IPOINT`和调用顺序`CALL_ORDER`。

执行和查看输出：

```shell
$ ../../../pin -t obj-ia32/invocation.so -- obj-ia32/little\_malloc
$ head invocation.out
After: IP = 0x64bc5e
Before: IP = 0x64bc5e
Taken: IP = 0x63a12e
After: IP = 0x64bc5e
Before: IP = 0x64bc5e
Taken: IP = 0x641c76
After: IP = 0x641ca6
After: IP = 0x64bc5e
Before: IP = 0x64bc5e
Taken: IP = 0x648b02
```

源码`source/tools/ManualExamples/invocation.cpp`：

```cpp
#include "pin.H"
#include <iostream>
#include <fstream>
using std::cerr;
using std::dec;
using std::endl;
using std::hex;
using std::ios;
using std::ofstream;
using std::string;
KNOB< string > KnobOutputFile(KNOB\_MODE\_WRITEONCE, "pintool", "o", "invocation.out", "specify output file name");
ofstream OutFile;
/\*
\* Analysis routines
\*/
VOID Taken(const CONTEXT\* ctxt)
{
ADDRINT TakenIP = (ADDRINT)PIN\_GetContextReg(ctxt, REG\_INST\_PTR);
OutFile << "Taken: IP = " << hex << TakenIP << dec << endl;
}
VOID Before(CONTEXT\* ctxt)
{
ADDRINT BeforeIP = (ADDRINT)PIN\_GetContextReg(ctxt, REG\_INST\_PTR);
OutFile << "Before: IP = " << hex << BeforeIP << dec << endl;
}
VOID After(CONTEXT\* ctxt)
{
ADDRINT AfterIP = (ADDRINT)PIN\_GetContextReg(ctxt, REG\_INST\_PTR);
OutFile << "After: IP = " << hex << AfterIP << dec << endl;
}
/\*
\* Instrumentation routines
\*/
VOID ImageLoad(IMG img, VOID\* v)
{
for (SEC sec = IMG\_SecHead(img); SEC\_Valid(sec); sec = SEC\_Next(sec))
{
// RTN\_InsertCall()和INS\_InsertCall()谁先出现谁先执行
// 在下面的代码中，IPOINT\_AFTER在IPOINT\_BEFORE之前执行。
for (RTN rtn = SEC\_RtnHead(sec); RTN\_Valid(rtn); rtn = RTN\_Next(rtn))
{
// 打开RTN.
RTN\_Open(rtn);
// IPOINT\_AFTER通过在一个routine中对每个return指令插桩实现。
// Pin会尝试查找所有的return指令，成不成功则是另外一回事（有点可爱23333）。
RTN\_InsertCall(rtn, IPOINT\_AFTER, (AFUNPTR)After, IARG\_CONTEXT, IARG\_END);
// 检查routine中的每条指令
for (INS ins = RTN\_InsHead(rtn); INS\_Valid(ins); ins = INS\_Next(ins))
{
if (INS\_IsRet(ins))
{
// 插桩每条return指令
// IPOINT\_TAKEN\_BRANCH总是最后使用
INS\_InsertCall(ins, IPOINT\_BEFORE, (AFUNPTR)Before, IARG\_CONTEXT, IARG\_END);
INS\_InsertCall(ins, IPOINT\_TAKEN\_BRANCH, (AFUNPTR)Taken, IARG\_CONTEXT, IARG\_END);
}
}
// 关闭RTN.
RTN\_Close(rtn);
}
}
}
VOID Fini(INT32 code, VOID\* v) { OutFile.close(); }
/\* ===================================================================== \*/
/\* Print Help Message \*/
/\* ===================================================================== \*/
INT32 Usage()
{
cerr << "This is the invocation pintool" << endl;
cerr << endl << KNOB\_BASE::StringKnobSummary() << endl;
return -1;
}
/\* ===================================================================== \*/
/\* Main \*/
/\* ===================================================================== \*/
int main(int argc, char\* argv[])
{
// 初始化
if (PIN\_Init(argc, argv)) return Usage();
PIN\_InitSymbols();
// 注册ImageLoad函数
IMG\_AddInstrumentFunction(ImageLoad, 0);
PIN\_AddFiniFunction(Fini, 0);
// 写入到文件
OutFile.open(KnobOutputFile.Value().c\_str());
OutFile.setf(ios::showbase);
// 开始执行，无返回
PIN\_StartProgram();
return 0;
}
```

### 10. Finding the Value of Function Arguments

功能：使用`RTN_InsertCall()`查看函数参数

执行和查看输出：

```shell
$ ../../../pin -t obj-intel64/malloctrace.so -- /bin/cp makefile obj-intel64/malloctrace.so.makefile.copy
$ cat malloctrace.out
malloc(0x5a1)
returns 0x7f87d8ce2190
malloc(0x4a1)
returns 0x7f87d8ce2740
malloc(0x10)
returns 0x7f87d8ce2bf0
malloc(0x9d)
returns 0x7f87d8ce2c00
malloc(0x28)
returns 0x7f87d8ce2ca0
malloc(0x140)
returns 0x7f87d8ce2cd0
malloc(0x26)
returns 0x7f87d8ce2e10
free(0)
malloc(0x4b0)
returns 0x7f87c4428000
malloc(0x26)
returns 0x7f87c44284b0
malloc(0x22)
returns 0x7f87c44284e0
free(0)
... ...
```

源码`source/tools/ManualExamples/malloctrace.cpp`:
