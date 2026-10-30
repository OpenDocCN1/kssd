
### 6. Instruction Count（指令计数）

功能：统计程序执行的动态指令总数。

执行和检查输出：

```shell
$ ../../../pin -t obj-intel64/inscount0.so -- /bin/ls
$ cat inscount.out
Count 120280
```

源码`source/tools/ManualExamples/inscount0.cpp`：

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

// 保存指令的运行次数，设置为静态变量以帮助编译器优化 docount
static UINT64 icount = 0;

// 在每个 block 前都会被调用
VOID docount(UINT32 c) { icount += c; }

// Pin 在遇到一个新的block 时进行调用，插入对 docount 函数的调用
VOID Trace(TRACE trace, VOID* v)
{
    // 访问 trace 中的每个 bbl
    for (BBL bbl = TRACE_BblHead(trace); BBL_Valid(bbl); bbl = BBL_Next(bbl))
    {
        // 在每个 bbl 前插入对 docount 函数的调用，传入指令数量
        BBL_InsertCall(bbl, IPOINT_BEFORE, (AFUNPTR)docount, IARG_UINT32, BBL_NumIns(bbl), IARG_END);
    }
}

KNOB<string> KnobOutputFile(KNOB_MODE_WRITEONCE, "pintool", "o", "inscount.out", "specify output file name");

// 退出函数
VOID Fini(INT32 code, VOID* v)
{
    // 写入到文件中，程序可能会关闭 cout 和 cerr
    OutFile.setf(ios::showbase);
    OutFile << "Count " << icount << endl;
    OutFile.close();
}

/* ===================================================================== */
/* Print Help Message                                                    */
/* ===================================================================== */
INT32 Usage()
{
    cerr << "This tool counts the number of dynamic instructions executed" << endl;
    cerr << endl << KNOB_BASE::StringKnobSummary() << endl;
    return -1;
}

/* ===================================================================== */
/* Main                                                                  */
/* ===================================================================== */
int main(int argc, char* argv[])
{
    // 初始化 pin
    if (PIN_Init(argc, argv)) return Usage();

    OutFile.open(KnobOutputFile.Value().c_str());

    // 注册插桩函数
    TRACE_AddInstrumentFunction(Trace, 0);

    // 注册退出函数
    PIN_AddFiniFunction(Fini, 0);

    // 开始执行，不返回
    PIN_StartProgram();

    return 0;
}
```

### 7. Procedure Instruction Count（routine插桩）

功能：计算一个 procedure 被调用的次数，以及每个 procedure 中执行的命令总数。

执行和检查输出：

```shell
$ ../../../pin -t obj-intel64/proccount.so -- /bin/grep proccount.cpp Makefile
proccount_SOURCES = proccount.cpp
$ head proccount.out
Procedure                      Image            Address         Calls  Instructions
_fini                           libc.so.6       0x40144d00          1            21
__deregister_frame_info         libc.so.6       0x40143f60          2            70
__register_frame_info           libc.so.6       0x40143df0          2            62
fde_merge                       libc.so.6       0x40143870          0             8
__init_misc                     libc.so.6       0x40115824          1            85
__getclktck                     libc.so.6       0x401157f4          0             2
munmap                          libc.so.6       0x40112ca0          1             9
mmap                            libc.so.6       0x40112bb0          1            23
getpagesize                     libc.so.6       0x4010f934          2            26
$
```

源码`source/tools/ManualExamples/proccount.cpp`：

```cpp
//
// This tool counts the number of times a routine is executed and
// the number of instructions executed in a routine
//
#include <fstream>
#include <iomanip>
#include <iostream>
#include <string.h>
#include "pin.H"

using std::cerr;
using std::dec;
using std::endl;
using std::hex;
using std::ofstream;
using std::setw;
using std::string;

ofstream outFile;

// 保存 procedure 的指令数
typedef struct RtnCount
{
    string _name;
    string _image;
    ADDRINT _address;
    RTN _rtn;
    UINT64 _rtnCount;
    UINT64 _icount;
    struct RtnCount* _next;
} RTN_COUNT;

// 每个 procedure 的指令数的链表
RTN_COUNT* RtnList = 0;

// 每条指令执行前调用
VOID docount(UINT64* counter) { (*counter)++; }

const char* StripPath(const char* path)
{
    const char* file = strrchr(path, '/');
    if (file)
        return file + 1;
    else
        return path;
}

// Pin 在一个新的 rtn 执行时调用该函数
VOID Routine(RTN rtn, VOID* v)
{
    // 对该routine设置一个计数器
    RTN_COUNT* rc = new RTN_COUNT;

    // image unloaded 时， RTN 数据消失，所以在此处直接保存，后续 fini 中还要使用
    rc->_name = RTN_Name(rtn);
    rc->_image = StripPath(IMG_Name(SEC_Img(RTN_Sec(rtn))).c_str());
    rc->_address = RTN_Address(rtn);
    rc->_icount = 0;
    rc->_rtnCount = 0;

    // 添加到routines列表
    rc->_next = RtnList;
    RtnList = rc;

    RTN_Open(rtn);

    // 在routine入口处插入一个call，增加call计数
    RTN_InsertCall(rtn, IPOINT_BEFORE, (AFUNPTR)docount, IARG_PTR, &(rc->_rtnCount), IARG_END);

    // 对于routine中的每条指令
    for (INS ins = RTN_InsHead(rtn); INS_Valid(ins); ins = INS_Next(ins))
    {
        // 插入对docount函数的调用，增加该rtn中的指令计数
        INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)docount, IARG_PTR, &(rc->_icount), IARG_END);
    }

    RTN_Close(rtn);
}

// 退出函数，打印每个procedure的名字和计数
VOID Fini(INT32 code, VOID* v)
{
    outFile << setw(23) << "Procedure"
            << " " << setw(15) << "Image"
            << " " << setw(18) << "Address"
            << " " << setw(12) << "Calls"
            << " " << setw(12) << "Instructions" << endl;

    for (RTN_COUNT* rc = RtnList; rc; rc = rc->_next)
    {
        if (rc->_icount > 0)
            outFile << setw(23) << rc->_name << " " << setw(15) << rc->_image << " " << setw(18) << hex << rc->_address << dec
                    << " " << setw(12) << rc->_rtnCount << " " << setw(12) << rc->_icount << endl;
    }
}

/* ===================================================================== */
/* Print Help Message                                                    */
/* ===================================================================== */
INT32 Usage()
{
    cerr << "This Pintool counts the number of times a routine is executed" << endl;
    cerr << "and the number of instructions executed in a routine" << endl;
    cerr << endl << KNOB_BASE::StringKnobSummary() << endl;
    return -1;
}

/* ===================================================================== */
/* Main                                                                  */
/* ===================================================================== */
int main(int argc, char* argv[])
{
    PIN_InitSymbols();

    outFile.open("proccount.out");

    if (PIN_Init(argc, argv)) return Usage();

    // 注册桩函数
    RTN_AddInstrumentFunction(Routine, 0);

    // 注册程序退出时的 fini函数
    PIN_AddFiniFunction(Fini, 0);

    // 开始执行，不返回
    PIN_StartProgram();

    return 0;
}
```

下面是一些Pin的功能性特征说明样例。

### 8. Using PIN_SafeCopy()

功能：从源内存区域复制指定数量的字节数到目的内存区域。即使源或目的区域不可访问，此函数也可保证安全返回给caller。此外，该API还可以读写程序的内存数据。

执行和查看输出：

```shell
$ ../../../pin -t obj-ia32/safecopy.so -- /bin/cp makefile obj-ia32/safecopy.so.makefile.copy
$ head safecopy.out
Emulate loading from addr 0xbff0057c to ebx
Emulate loading from addr 0x64ffd4 to eax
Emulate loading from addr 0xbff00598 to esi
Emulate loading from addr 0x6501c8 to edi
Emulate loading from addr 0x64ff14 to edx
Emulate loading from addr 0x64ff1c to edx
Emulate loading from addr 0x64ff24 to edx
Emulate loading from addr 0x64ff2c to edx
Emulate loading from addr 0x64ff34 to edx
Emulate loading from addr 0x64ff3c to edx
```

源码`source/tools/ManualExamples/safecopy.cpp`:
