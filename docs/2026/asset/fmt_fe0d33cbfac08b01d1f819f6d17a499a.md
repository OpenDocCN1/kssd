
```cpp
/*! @file
 * This file contains an ISA-portable PIN tool for tracing instructions
 */

#include <iostream>
#include <fstream>
#include <map>
#include <unistd.h>
#include "pin.H"

using std::cerr;
using std::endl;
using std::map;
using std::pair;
using std::string;

/* ================================================================ */
/* Commandline Switches */
/* ================================================================ */

KNOB<string> KnobOutputFile(KNOB_MODE_WRITEONCE, "pintool", "o", "edgcnt.out", "specify trace file name");
KNOB<INT32> KnobFilterByHighNibble(KNOB_MODE_WRITEONCE, "pintool", "f", "-1", "only instrument instructions with a code address matching the filter");
KNOB<BOOL> KnobPid(KNOB_MODE_WRITEONCE, "pintool", "i", "0", "append pid to output");

/* ================================================================ */
/* Print Help Message */
/* ================================================================ */

static INT32 Usage()
{
    cerr << "This pin tool collects an edge profile for an application\n";
    cerr << "The edge profile is partial as it only considers control flow changes (taken\n";
    cerr << "branch edges, etc.). It is the left to the profile consumer to compute the missing counts.\n";
    cerr << "\n";

    cerr << "The pin tool *does* keep track of edges from indirect jumps within, out of, and into\n";
    cerr << "the application. Traps to the OS a recorded with a target of -1.\n";

    cerr << KNOB_BASE::StringKnobSummary() << endl;
    return -1;
}

/* ================================================================ */
/* Global Variables */
/* ================================================================ */

class COUNTER
{
public:
    UINT64 _count; // 边界到达的次数，计数器

    COUNTER() : _count(0) {}
};

typedef enum
{
    ETYPE_INVALID,
    ETYPE_CALL,
    ETYPE_ICALL,
    ETYPE_BRANCH,
    ETYPE_IBRANCH,
    ETYPE_RETURN,
    ETYPE_SYSCALL,
    ETYPE_LAST
} ETYPE;

class EDGE
{
public:
    ADDRINT _src;
    ADDRINT _dst;
    ADDRINT _next_ins;
    ETYPE _type; // 必须为整数形式

    EDGE(ADDRINT s, ADDRINT d, ADDRINT n, ETYPE t) : _src(s), _dst(d), _next_ins(n), _type(t) {}

    bool operator<(const EDGE& edge) const { return _src < edge._src || (_src == edge._src && _dst < edge._dst); }
};

string StringFromEtype(ETYPE etype)
{
    switch (etype)
    {
        case ETYPE_CALL: return "C";
        case ETYPE_ICALL: return "c";
        case ETYPE_BRANCH: return "B";
        case ETYPE_IBRANCH: return "b";
        case ETYPE_RETURN: return "r";
        case ETYPE_SYSCALL: return "s";
        default: ASSERTX(0); return "INVALID";
    }
}

typedef map<EDGE, COUNTER*> EDG_HASH_SET;

static EDG_HASH_SET EdgeSet;

/* ================================================================ */
/*! 对于已经进行过插桩的Edge，重用entry；否则创建一个新的。 */
static COUNTER* Lookup(EDGE edge) // 查找边界
{
    COUNTER*& ref = EdgeSet[edge];

    if (ref == 0)
    {
        ref = new COUNTER();
    }

    return ref;
}

/* ================================================================ */
// 分析routine代码
VOID docount(COUNTER* pedg) { pedg->_count++; }

/* ================================================================ */
// 对于间接控制流，我们不知道边界，所以需要进行查找。
VOID docount2(ADDRINT src, ADDRINT dst, ADDRINT n, ETYPE type, INT32 taken)
{
    if (!taken) return;
    COUNTER* pedg = Lookup(EDGE(src, dst, n, type));
    pedg->_count++;
}

/* ================================================================ */
VOID Instruction(INS ins, void* v) // 插桩函数
{
    if (INS_IsDirectControlFlow(ins)) // 如果是直接控制流（ins为控制流指令，目标地址由指令指针或立即数指定）
    {
        ETYPE type = INS_IsCall(ins) ? ETYPE_CALL : ETYPE_BRANCH; // 判断是否为call指令，是则返回ETYPE_CALL

        // 静态目标可以在这里进行一次映射
        // 参数分别为当前指令地址、当前指令目标地址、下一指令地址、指令类型
        COUNTER* pedg = Lookup(EDGE(INS_Address(ins), INS_DirectControlFlowTargetAddress(ins), INS_NextAddress(ins), type));

        // 插桩
        INS_InsertCall(ins, IPOINT_TAKEN_BRANCH, (AFUNPTR)docount, IARG_ADDRINT, pedg, IARG_END);
    }
    else if (INS_IsIndirectControlFlow(ins)) // 如果是间接控制流（ins为控制流指令，且目标地址通过内存或寄存器提供）
    {
        ETYPE type = ETYPE_IBRANCH; // 直接指定类型为间接控制流

        if (INS_IsRet(ins)) // 是否为ret或iret
        {
            type = ETYPE_RETURN;
        }
        else if (INS_IsCall(ins))
        {
            type = ETYPE_ICALL;
        }

        // 进行插桩
        INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)docount2, IARG_INST_PTR, IARG_BRANCH_TARGET_ADDR, IARG_ADDRINT, INS_NextAddress(ins), IARG_UINT32, type, IARG_BRANCH_TAKEN, IARG_END);
    }
    else if (INS_IsSyscall(ins)) // 如果是syscall指令
    {
        COUNTER* pedg = Lookup(EDGE(INS_Address(ins), ADDRINT(~0), INS_NextAddress(ins), ETYPE_SYSCALL));

        INS_InsertPredicatedCall(ins, IPOINT_BEFORE, (AFUNPTR)docount, IARG_ADDRINT, pedg, IARG_END);
    }
}

/* ================================================================ */
inline INT32 AddressHighNibble(ADDRINT addr) { return 0xf & (addr >> (sizeof(ADDRINT) * 8 - 4)); }

/* ================================================================ */
static std::ofstream* out = 0;

VOID Fini(int n, void* v) // 程序结束时的处理函数
{
    const INT32 nibble = KnobFilterByHighNibble.Value();

    *out << "EDGCOUNT        4.0         0\n"; // profile header, no md5sum
    UINT32 count = 0;

    for (EDG_HASH_SET::const_iterator it = EdgeSet.begin(); it != EdgeSet.end(); it++)
    {
        const pair<EDGE, COUNTER*> tuple = *it;

        // skip inter shared lib edges
        if (nibble >= 0 && nibble != AddressHighNibble(tuple.first._dst) && nibble != AddressHighNibble(tuple.first._src))
        {
            continue;
        }

        if (tuple.second->_count == 0) continue;
        count++;
    }

    *out << "EDGs " << count << endl;
    *out << "# src          dst        type    count     next-ins\n";
    *out << "DATA:START" << endl;

    for (EDG_HASH_SET::const_iterator it = EdgeSet.begin(); it != EdgeSet.end(); it++)
    {
        const pair<EDGE, COUNTER*> tuple = *it;

        // skip inter shared lib edges
        if (nibble >= 0 && nibble != AddressHighNibble(tuple.first._dst) && nibble != AddressHighNibble(tuple.first._src))
        {
            continue;
        }

        if (tuple.second->_count == 0) continue;

        *out << StringFromAddrint(tuple.first._src) << " " << StringFromAddrint(tuple.first._dst) << " "
             << StringFromEtype(tuple.first._type) << " " << decstr(tuple.second->_count, 12) << " "
             << StringFromAddrint(tuple.first._next_ins) << endl;
    }

    *out << "DATA:END" << endl;
    *out << "## eof\n";
    out->close();
}

/* ================================================================ */
/* Main */
/* ================================================================ */

int main(int argc, char* argv[])
{
    if (PIN_Init(argc, argv)) // 初始化
    {
        return Usage();
    }

    string filename = KnobOutputFile.Value(); // 输出文件
    if (KnobPid)
    {
        filename += "." + decstr(getpid());
    }

    out = new std::ofstream(filename.c_str());

    INS_AddInstrumentFunction(Instruction, 0); // 注册插桩函数
    PIN_AddFiniFunction(Fini, 0); // 注册Fini函数

    // 开始执行，不返回
    PIN_StartProgram();

    return 0;
}

/* ================================================================ */
/* eof */
/* ================================================================ */
```
