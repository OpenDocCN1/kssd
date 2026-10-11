
# SEH 源码赏析之 C 篇

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-312.htm`](https://www.pediy.com/kssd/pediy09/pediy09-312.htm)

> SEH 结构化异常处理源码赏析(C 篇)
> 关键字: C SEH 结构化异常处理 `_try` `_finally` `_except` `_except_handler3` VC
> 工程使用工具: VC7.1.3088 IDA pro 4.9.0.863 cdb Windbg Editplus

## 1. 起因

C++程序员对 `try`、`catch`、`throw` 都应该很熟悉,能知道 VC 怎么实现它的人就不多了,不过网络世界使很多人知道了它与 SEH(structured exception handling）有密切关系,我也不例外,也是在若干年前从网络知道了 SEH,并且大致也知道 SEH 的流程.但是和多数人一样在我的实践也很少直接使用 SEH,对 SEH 也就仅限于网络上一些文章的介绍.曾经在用 Windbg 对某些软件作分析,我遇到了断点失效的情况,查找资料介绍是 SEH 中的 Handler 清除了调试寄存器,在分析 SEH 代码中由于 VC 没有 SEH 的源码,于是我产生了一种想法,详细完整地翻译 VC 的 SEH 的代码,一劳永逸的解决问题.C 和 C++的 SEH 有所不同,C++的要复杂些,我在此介绍的仅为 C 的 SEH 代码,也就是`_try`、`_finally`、`_except`、`_leave` 所产生的 SEH 代码,C++篇有时间的话我再作.我以前看过的资料大都以比较专业的语言介绍,在此我仅以我自己感觉比较通俗的语言介绍,希望能有更多的人能认识,认清 SEH.

## 2. SEH 术语

SEH 中术语虽然不多,但由于没有 SDK 的明确定义有时很难区别,各家说的表述也不太统一,因此为此文特定义如下术语,这些术语可能与其它文献有点冲突或细微差别,有的也是我自己的定义:

### A. SEH(structured exception handling)

在 C 语言中关键字是`_try`（`__try`）、`_finally`（`__finally`）、`_except`（`__except`）,而 C++使用的关键字是 `try`、`catch`.在以下的表述中所有的 `try` 均不再特别声明为 C 关键字,一律默认为 C 关键字.

### B. `EH3_List`

`_EH3_EXCEPTION_REGISTRATION` 链表,表头位于 `FS:[0]`,`0xFFFFFFFF` 为链表结束标志.编译器在编译一个函数时只要检测到含有`_try` 或`__try` 则为此函数生成一个`_EH3_EXCEPTION_REGISTRATION` 节点,并插入到表头.因为每个函数编译只生成一个节点,因此在一个函数中 C 和 C++的 SEH 不能同时存在,如果代码中同时有 `catch` 和 `except` 则不能通过编译就是此原因.

### C. `EH3_ScopeTable`

是一个由编译器在 `data section` 生成的一张表(数组),实质可看作是二叉树结构(可能有多个二叉树顺序存放),节点为`_SCOPETABLE_ENTRY` 类型,其中`_SCOPETABLE_ENTRY.ParentLevel` 是父节点在数组中的位置,`EH3_ScopeTable[0]`是根节点,`_SCOPETABLE_ENTRY.ParentLevel=0xFFFFFFFF`.由此可见 `ParentLevel` 很重要,是 SEH 判断 `try` 嵌套层次的唯一依据.编译器从函数入口点开始遍历 `try`,每遇到一个 `try` 生成一个节点`_SCOPETABLE_ENTRY`,并放在表最后,注意节点的先后与 `try` 的嵌套层次无关.

### D. filter handler

是异常发生后让用户决定是否认识此异常,通过修改异常语句的上下文环境(`CONTEXT`)可使应用程序能继续正常运行.其返回值有三.

*   `EXCEPTION_EXECUTE_HANDLER(1)`: 去执行 `exception handler`,然后进程终止,且不显示出错提示框.
*   `EXCEPTION_CONTINUE_SEARCH(0)`: 不执行 `exception handler`,显示出错提示框,进程终止或者进入调试器进行调试.
*   `EXCEPTION_CONTINUE_EXECUTION(-1)`: 不执行 `exception handler`,系统用 `CONTEXT` 重新设置 CPU 环境,进程继续执行,如果修改了 `EIP` 则从新的 `EIP` 开始执行,否则从原异常点开始执行.

### E. exception handler

是异常发生后检测到该异常无法被处理,而进程终止前提醒应用程序执行的收尾工作

### F. termination handler

如果 `try` 语句被过早终止,不管是正常离开或者是非正常离开,包括 `goto`、`leave` 及异常时均执行此 `handler`,具体执行过程可查 MSDN.

### G. 展开(unwind)

这个名词很让人费解,翻译也确实不好命名,我也沿用此名.展开的目的是执行 `finally` 对应的 `termination handler`,对照在下面的源代码中我们就能很容易理解 MSDN 关于 `finally` 的解释了.展开分为本地局部展开(`_local_unwind`)和全局展开(`_global_unwind`);本地展开:展开此 `try` 所在函数 `try` 嵌套关系并分别执行其 `finally` 对应的 `termination handler`;全局展开:当 `exception handler` 不在本 `try` 函数时,执行 `exception handler` 前需要先执行这之前的 `termination handler`,全局展开就是查找这些 `termination handler` 并执行它.需要说明的是全局展开不含本地展开.

## 3. SEH 数据结构

```c
typedef struct  // sizeof=0xC
{
    DWORD ParentLevel ;      // 当前 Handler 父层 TRY 在 EH3_ScopeTable 中的位置,根没有上一层,故值=-1
                             // 形成 TRY 层次的二叉树结构,与_EH3_EXCEPTION_REGISTRATION.TryLevel 物理意义一样
    DWORD FilterFunc;       // 非 NULL 则 HandlerFunc 是 exception handler,否则 termination handler
    DWORD HandlerFunc;      // exception handler or termination handler
} _SCOPETABLE_ENTRY;

typedef struct  // sizeof=0x10
{
    _EH3_EXCEPTION_REGISTRATION* pPrev;// 栈上一级 EH3_List 节点,=0xFFFFFFFF 则为最后一个节点
    EXCE_HANDLER ExceptionHandler;     // VC7.1 中统一指向_except_handler3
    _SCOPETABLE_ENTRY* pScopeTable;     // 指向一个_SCOPETABLE_ENTRY 数组,函数有 n 个 TRY,则数组有 n 个元素
                               // p[0]->Try0 为根,p[1]->Try1,p[2]->Try2...
    DWORD TryLevel;     // 指示当前指令在 Try 的层次级别,-1 未进入 TRY,进第一个 TRY 为 0,第二个为 1,...
                    // 但它与嵌套层次无关,在编译时确定,从函数代码开始处开始计数
} _EH3_EXCEPTION_REGISTRATION;

typedef struct  // sizeof=0x10还有待进一步分析其用处
{
    DWORD unKnown;    // 未知:被编译器赋值
    DWORD HandlerFunc;  // _SCOPETABLE_ENTRY.HandlerFunc
    DWORD firstPara;    // Try 所在函数第一个参数:crtMain!EBP+8
    DWORD TryEBP;    // Try 所在函数 EBP
}_NLGDestination;

// 以下在 MSDN 中均有定义,不再作解释
typedef struct _EXCEPTION_RECORD
{
    DWORD    ExceptionCode;
    DWORD ExceptionFlags;
    struct _EXCEPTION_RECORD *ExceptionRecord;
    PVOID ExceptionAddress;
    DWORD NumberParameters;
    ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
} EXCEPTION_RECORD,*PEXCEPTION_RECORD;

typedef struct _CONTEXT
{
    ...         // 依据 CPU 类型有不同定义,具体可见 winnt.h
} CONTEXT,*PCONTEXT;

typedef struct _EXCEPTION_POINTERS
{
    PEXCEPTION_RECORD ExceptionRecord;
    PCONTEXT ContextRecord;
} EXCEPTION_POINTERS,*PEXCEPTION_POINTERS;
```

## 4. SEH 汇编要点

### A. 函数中 `try` 的数据模型

VC 将会为有 `try` 函数在栈上首先建立三个变量:

```asm
EBP-18h:    SaveESP          // TRY 前保存当时 ESP,因此有:SaveESP<&Eh3Exception
EBP-14h:    pExceInfo        // GetExceptionPointers(),在 handler 中调用 filter 前赋值
EBP-10h:    Eh3Exception     // 一个_EH3_EXCEPTION_REGISTRATION,其大小刚好为 10h 哦
EBP+00h:    ebp             // 上一 frame 的 EBP
EBP+04h:    EIP             // CALL 返回地址
```

在这里我们应该注意到这个公式是成立的:

```
EBP=&Eh3Exception+10h
```

而在`_except_handler3` 中参数: `pEh3Exce` 刚好为`&Eh3Exception`,所以每当在`_except_handler3` 中要回调 `filter`、`exception`、`termination handler` 时,在这之前汇编中均有一句:

```asm
lea     ebp, [ebx+10h]    // ebx=_except_handler3!arg_pEh3Exception
```

明白了这点就不难明白如何在`_except_handler3` 中访问 `pExceInfo` 及 `SaveESP` 了!

### B. `try` 块的终止

异常发生,`goto`、`return`、`leave`、正常终止. `goto` 和 `return` 可能跨过其它 `try`,所以必须要展开到目的地所在的 `TryLevel`,但是 `leave` 关键字不会跨过其它 `try`,只是跳出自己这层 `try`,如果编译器检测到这层 `try` 有 `termination handler`,则用 `CALL xxx` 直接调用.当然也有例外,这也是 VC 聪明的地方,如果函数只有一个 `try`,则根本不用展开而直接使用 `CALL` 了,这种情况有时可见.这样就不难理解如下代码:

**`goto` 终止代码:**

```asm
// ---------------goto 跳出 try----------------------------------------
    goto try0;
push    0              或者  (仅一个 try 时)    CALL 00401070  // 直接调 termination handler
lea     eax, [ebp+var_Eh3Exce]               jmp loc_4013FB
push    eax
call    __local_unwind2          // 展开到 0(即第一个 try 内,goto 目的地肯定在第一个 try 内)
add     esp, 8
jmp     loc_4013FB
```

**`return` 终止代码:(无返回值)**

```asm
// ---------------return 跳出 try--------------------------------------
push        0FFFFFFFFh
lea         eax,[ebp-10h]
push        eax
call        __local_unwind2 (401786h)  // 展开到-1(即所有 try 外)
add         esp,8
return ;
jmp         $L19800 (401139h)
```

**`return` 终止代码:(有返回值)**

```asm
// ---------------return(带返回值)跳出 try----------------------------
mov         eax,dword ptr [i]
mov         dword ptr [ebp-100h],eax  // 返回值先被保存
push        0FFFFFFFFh
lea         ecx,[ebp-10h]
push        ecx
call        __local_unwind2 (4017D6h)  // 再展开,即使 finally 中改变了 i,也不会改变返回值!!!
add         esp,8
return i;
mov         eax,dword ptr [ebp-100h]  // 取保存的返回值来保存
jmp         $L19800 (40117Bh)      // 跳到复原先前 SEH 链处
```

**`leave` 和正常退出的代码:**

```asm
// ---------------leave 跳出 try---------------------------------------
if (x > 18)
004010FD  cmp         dword ptr [x],12h
00401101  jle         FunC+55h (401105h)
    __leave;
00401103  jmp         FunC+66h (401116h) // 直接调用 termination handler
    printf("%s Try!\n",fun);
00401105  mov         eax,dword ptr [fun]
00401108  push        eax
00401109  push        offset string "%s Try!\n" (410130h)
0040110E  call        printf (401650h)
00401113  add         esp,8
00401116  mov         dword ptr [ebp-4],0FFFFFFFFh // 退出 try
```



#### C. `_except_handler3` 执行两次原因

实际上理解了展开就能理解它,举例解释如下:

`EH3_List` 如: `FS[0]->E1->E2->E3->E4`, 在 `E4` 中发生异常, 依次搜索 `E1, E2, E3`, 最后在 `E4` 中找到能处理此异常的 `filter handler`, 这样 `E4, E3, E2, E1` 的 `_except_handler3` 均执行了一次, 这是第一次.

第二次: 在执行 `E4` 的 `exception handler` 前要先调用全局展开(`_global_unwind2`)以运行 `E1, E2, E3` 的 `termination handler`。在全局展开(`_global_unwind2`)中将为 `EXCEPTION_RECORD.ExceptionFlags` 增加标志 `_EH_UNWINDING(2)`, 再依次调用它们的 `_except_handler3`, 这就是 `E1, E2, E3` 的 `_except_handler3` 的第二次调用, 但 `E4` 只有一次.

#### 5. VC7.1 下的 SEH 的 C 代码

```c
// 源码基本以汇编为蓝本,没有进行优化,主要是为了方便大家与汇编对照阅读
#define MAX_PAGES 0x10

int ValidateEH3RN(_EH3_EXCEPTION_REGISTRATION* pEh3Exce)
{
    // 1.验证_EH3_EXCEPTION_REGISTRATION.pScopeTable 的合法性:4 字节对齐,且不在栈空间内,因为它是编译器生成的全局变量
    _SCOPETABLE_ENTRY* pScopeTable=pEh3Exce->pScopeTable;
    if (((DWORD)pScopeTable & 0x3) == 0)
        return 0;
    NT_TIB *pTeb = (NT_TIB*)FS[0x18];
    DWORD nStackLimit = pTeb->StackLimit;
    if (pScopeTable >= pTeb->StackLimit && pScopeTable < pTeb->StackBase)
        return 0;// pScopeTable 在栈内肯定不合法
    
    // 2.判断二叉树 pScopeTable 是否合法,并统计 exception handler 数量
    if (pEh3Exce->TryLevel == -1)
        return 1;// 表示语句不在 try 中
        
    DWORD nFilters,count;
    nFilters=count=0;
    for(;nFilters <= pEh3Exce->TryLevel; nFilters++)
    {// LOOP1:验证已进入的 TRY 的 SCOPETABLE_ENTRY 是否都合法
        if(pScopeTable[nFilters].ParentLevel!=-1 && pScopeTable[nFilters].ParentLevel>=nFilters)
            return 0;// EnclosingLevel 在二叉树中不合法:不是根结点且 ParentLevel>=nFilters
                    // ParentLevel>=nFilters 不合法原因:二叉树存贮不可能儿子先存
        if (pScopeTable[nFilters].FilterFunc == NULL)
            continue;// termination handler 不统计
        count++;
    }
    if (count != 0)
    {// 有异常处理,验证 saveESP
        PVOID saveESP =0;// 运行时真实值 = crtMain!saveESP=crtMain!(ebp-18h);
        if (saveESP < nStackLimit || saveESP >= pEh3Exce)
            return 0;// 不在栈内,或不在 TRY 函数内,saveESP 为 TRY 函数的栈顶
    }

    // 3.找 g_rgValidPages 中是否已经有 pScopeTable 的页基址
    static int g_nValidPages=0;// g_rgValidPages 数组有效元素个数
    static int g_rgValidPages[MAX_PAGES]={0}; // MaxPages=0x10
    DWORD nPageBase = (DWORD)pScopeTable & 0xfffff000;// 取 pScopeTable 所在页基址
    int nFind=0;
    if(g_nValidPages > 0)
    {
        do
        {// LOOP2
            if(nPageBase==g_rgValidPages[nFind])
                goto Finded;// 找到
            nFind++;
        }
        while(nFind<g_nValidPages);
    }

    // 4.没有过在 g_rgValidPages 找到 nPageBase 则判断 pScopeTable 是否在合法的 EXE 映像内
    MEMORY_BASIC_INFORMATION memInfo;
    if (VirtualQuery(pScopeTable,&memInfo,sizeof(memInfo)) == 0)
        return -1;
    if (memInfo.Type != 0x01000000)// MEM_IMAGE
        return -1;
    DWORD protect = 0xCC;// PAGE_READWRITE|PAGE_WRITECOPY|PAGE_EXECUTE_READWRITE|PAGE_EXECUTE_WRITECOPY;
    if (memInfo.Protect & protect)
    {// 有指定属性
        IMAGE_DOS_HEADER* pDosHeader = (IMAGE_DOS_HEADER*)memInfo.AllocationBase;
        if (pDosHeader->e_magic != 'ZM')
            return -1;  // 非法 DOS 头,pScopeTable 不为编译器分配的空间
        IMAGE_NT_HEADERS* pPeHeader = (IMAGE_NT_HEADERS*)((char*)pDosHeader+pDosHeader->e_lfanew);
        if (pPeHeader->Signature != 'ZM')
            return -1;  // 非法 PE 头,pScopeTable 不为编译器分配的空间
        IMAGE_OPTIONAL_HEADER32* pOptHeader = &pPeHeader->OptionalHeader;
        if (pOptHeader->Magic != 0x10b)
            return -1; // 非 WIN32 EXE
        DWORD rvaScope = (DWORD)pScopeTable-(DWORD)pDosHeader; // 计算 pScopeTable 的 RAV
        if (pPeHeader->FileHeader.NumberOfSections <=0 )
            return -1;
        IMAGE_SECTION_HEADER* pSection = (IMAGE_SECTION_HEADER*)(pPeHeader+1);
        if (rvaScope >= pSection->VirtualAddress && rvaScope < pSection->VirtualAddress+pSection->Misc.VirtualSize)
        {// rvaScope 在代码节内
            if (pSection->Characteristics & 0x80000000)// 0x80000000=IMG_SCN_MEM_WRITE
                return 0;
        }
    }

    // 5.对新验证的 nPageBse 插入到数组中
    static int g_nLock=0; // 1:上锁,0:解锁
    if(InterlockedExchange(&g_nLock,1)!=0)// 写线程锁
        return 1;// 其它线程已经进入,则不能再进入
    int k=g_nValidPages;
    if(k > 0)
    {
        do
        {// LOOP4:判断其它线程是否写入这个页基址
            if(nPageBase==g_rgValidPages[k-1])
                break;// 找到,k>0
            k--;
        }
        while(k>0);
    }
    if (k==0)
    {// 没有找到,nPageBase 插入到 g_rgValidPages 头
        int pages = 0x0F;
        if(g_nValidPages <= pages)
            pages = g_nValidPages;
        k=0;
        if(pages >= 0)
        {
            do
            {// LOOP5
                int temp=g_rgValidPages[k];
                g_rgValidPages[k]=nPageBase;
                nPageBase=temp;
                k++;
            }
            while(k<=pages);
        }
        if (g_nValidPages<0x10h)
            g_nValidPages++;
    }
    InterlockedExchange(&g_nLock,0);// 解锁
    return 1;

    // 6.找到的 nPageBase 移到头
    //   但前面找到的结果也可能被其它线程改变位置甚至推出数组,但无论如何这个 nPageBase 合法可不再验证
Fined:
    if (nFind <= 0)// 相当于 if(nFind == 0)
        return 1;// nPageBase 已经在头,可直接返回
    if(InterlockedExchange(&g_nLock,1)!=0)// 写线程锁
        return 1;// 其它线程已经进入,则不能再进入
    if(g_rgValidPages[nFind] != nPageBase)
    {// 再次对找到的 pos 进行比较,因为其它线程可能又修改了这个元素的值
        nFind = g_nValidPages-1;
        if (nFind>=0)
        {
            while(nFind>=0)
            {// LOOP3
                if (g_rgValidPages[nFind]==nPageBase)
                    break;
                nFind--;
            }
            if (nFind>=0)
                goto End1;
        }
        // 没找到,新增加
        if(g_nValidPages < 0x10)
            g_nValidPages++;
        nFind = g_nValidPages-1;
        goto end2;
    }
    else
        goto end2;
end1:
    if (nFind != 0)
    {
end2:
        if (nFind >= 0)
        {
            for(int j = 0;j<=nFind;j++)
            {// LOOP6:g_rgValidPages 中找到的 nPageBase 移到头或新 nPageBase 插入头,其余每个元素向后推
                int temp=g_rgValidPages[j];
                g_rgValidPages[j]=nPageBase;
                nPageBase=temp;
            }
        }
    }
    InterlockedExchange(&g_nLock,0);// 解线程锁
    return 1;
}

void _global_unwind2(_EH3_EXCEPTION_REGISTRATION*pEh3Exce)
{// 对调用 RtlUnwind 的封装
    RtlUnwind(pEh3Exce,offset exit,0,0);
exit:
    return;
}

int _UnwindHandler(EXCEPTION_RECORD*pExceRec,_EH3_EXCEPTION_REGISTRATION*pEh3Exce,
            void*,_EH3_EXCEPTION_REGISTRATION** ppEh3Exce)
{
    if(pExceRec->ExceptionFlags && 6 == 0)
        return 1;// ExceptionContinueSearch
    *ppEh3Exce = pEh3Exce;
    return 3;// ExceptionCollidedUnwind
}

// NLG == "non-local-goto"
_NLGDestination g_NLGDestination;// 此全局变量我至始至终未见到任何其它 EXE 处或者 DLL 使用,通知是何意义暂无知!!!!!!!

void _NLG_Notify1(int x)// --------------------------CallSettingFrame 调用
{// g_NLGDestination 全局分配变量:0x19930520
    g_NLGDestination.dwInCode = EBP+8;// Try 函数中第一个参数???
    g_NLGDestination.HandlerFunc = EAX;// 调用前通过 EAX 传入
    g_NLGDestination.TryEBP = EBP;// Try 中的 EBP???
}

void _NLG_Notify(int x)// --------------------------_except_handler3,_local_unwind2 调用
{// 传入参数未用
```



```
// g_NLGDestination.unKnown 全局分配变量:0x19930520

g_NLGDestination.dwInCode = EBP+8;// Try 函数中第一个参数,如:FunB(i,j)!i,crtMain!ebp+8

g_NLGDestination.HandlerFunc = EAX;// 调用前通过 EAX 传入:pScopeTable.HandlerFunc

g_NLGDestination.TryEBP = EBP;// Try 中的 EBP

}

// nToLevel:展开到此为止(不含 nToLevel),举例:在 goto 中 nToLevel 由目标所在 try 决定,因为系统规定跳入 TRY 是不合法的,因此
// goto 不能跳入其它 TRY(嵌套的上层 TRY 是合法的,很明显平级层是不合法的)层,这时编译是通不过的.

void _local_unwind2(_EH3_EXCEPTION_REGISTRATION* pEh3Exce,int nToLevel)
{// 用 ESP,未用 EBP,之前的 EBP 传入内调的_NLG_Notify,__try 中的 goto 将用此函数展开得到其 handler
    _EH3_EXCEPTION_REGISTRATION  eh3Unwind;

    eh3Unwind.TryLevel = (int)pEh3Exce;

    eh3Unwind.pScopeTable = (_SCOPETABLE_ENTRY*)-2;

    eh3Unwind.ExceptionHandler = (EXCE_HANDLER)_UnwindHandler;

    eh3Unwind.pPrev = (_EH3_EXCEPTION_REGISTRATION*)FS[0];

    FS[0]=ESP;

    int nLevel;//

    while(1)
    {
        _SCOPETABLE_ENTRY *pScope=pEh3Exce->pScopeTable;

        ESI = pEh3Exce->TryLevel;

        if (ESI == -1)
            break;

        if (ESI == nToLevel)
            break;

        nLevel=pScope[ESI].ParentLevel;

        pEh3Exce->TryLevel=nLevel;

        if (pScope[ESI].FilterFunc != NULL) // 有 FilterFunc 则是 exception handler
            continue;

        EAX=pScope[ESI].HandlerFunc;     // 无 FilterFunc 则是 termination handler

        _NLG_Notify(0x101);      // 无返回值

        CALL(EAX);           // 这就是展开所求得的 Handler,可以是 termination handler,exception Handler
    }
    FS[0]=(DWORD)eh3Unwind.pPrev;
}

// _except_handler3 执行两次原因:
// 举例:EH3_List 如 FS[0]->E1->E2->E3->E4,在 E4 中发生异常,依次搜索 E1,E2,E3,最后在 E4 中找到能处理此异常的 filter handler,这样 E4,E3,E2,E1 的_except_handler3 均执行了一次,这是第一次.
//      第二次:在执行 E4 的 exception handler 前要先调用全局展开(_global_unwind2)以运行 E1,E2,E3 的 termination handler
// 在全局展开(_global_unwind2)中将为 EXCEPTION_RECORD.ExceptionFlags 增加标志_EH_UNWINDING(2),再依次调用它们的
// _except_handler3,这就是 E1,E2,E3 的_except_handler3 的第二次调用,但 E4 只有一次.

    int _except_handler3(EXCEPTION_RECORD* pExceRec,_EH3_EXCEPTION_REGISTRATION* pEh3Exce,
                CONTEXT* pContextRecord/*,void * DispatcherContext*/)
    {
        if ((pExceRec->ExceptionFlags & 6) == 0)
        {// 无_EH_UNWINDING and _EH_EXIT_UNWIND
            EXCEPTION_POINTERS ePointer;
            ePointer.ExceptionRecord = pExceRec;
            ePointer.ContextRecord = pContextRecord;
            // pEh3Exce=&EhRecord
            // lea     EAX, [EBP+var_ExcePointers]
            // mov     [ebx-4], EAX ;EBX=pEh3Exce
            *((DWORD*)pEh3Exce-1)=(DWORD)&ePointer;  // 给_XcptFilter 的参数 pExceptPointers 赋值
            if (ValidateEH3RN(pEh3Exce))
            {
                _SCOPETABLE_ENTRY* pScopeTable=pEh3Exce->pScopeTable;
                int i = pEh3Exce->TryLevel;
                while(1)
                {
                    if (i==-1)
                        return 1;// handler 拒绝处理这个异常
                    EAX=pScopeTable[i].FilterFunc;
                    if(EAX!=NULL)
                    {// 属于 exception handler
                        EBP=(DWORD)pEh3Exce+0x10;// 恢复原 TRY 基址 pEh3Exce=&crtMain!EhRecord,sizeof(EhRecord)=0x10
                        CALL(EAX);// 没有参数,ePointer 已传过去
                        if(EAX != 0)
                        {
                            if ((int)EAX < 0)
                                return 0;        // 如果在 filter 中返回值<0,此返回后会引起代码为
                                                // EXCEPTION_NONCONTINUABLE_EXCEPTION(0xC0000025)的异常,反复如此栈溢出
                            _global_unwind2(pEh3Exce);  // 调用 handler 前,展开前级函数中的 termination handler
                            EBP=(DWORD)pEh3Exce+0x10;  // 恢复原 TRY 基址 pEh3Exce=&crtMain!EhRecord,sizeof(EhRecord)=0x10
                            _local_unwind2(pEh3Exce,i);  // 此函数内不直接用 EBP,但暗传 EBP 给可能执行的_NLG_Notify
                                                // 并展开本函数中的 termination handler
                            EAX=pScopeTable[i].HandlerFunc;
                            _NLG_Notify(1);// 此函数内要用 EAX
                            pEh3Exce->TryLevel=pScopeTable[i].ParentLevel;// 修改节点所在层次使 EIP 在 TRY 中位置表示正确
                            EAX = pScopeTable[i].HandlerFunc;
                            CALL(EAX);// 进入异常处理,不再返回此函数(内部可接受异常并修改环境,继续执行)
                        }
                    }
                    i=pScopeTable[i].ParentLevel;
                }
                return 0;
            }
            else
            {
                pExceRec->ExceptionFlags |= 8;// 置 pEh3Exce 无效标志,它不属于此 handler
                return 1;// handler 拒绝处理这个异常,系统转调上一级
            }
        }
        else
        {// 有_EH_UNWINDING or _EH_EXIT_UNWIND
            EBP=(DWORD)pEh3Exce+0x10;// 恢复原 TRY 基址 pEh3Exce=&crtMain!EhRecord,sizeof(EhRecord)=0x10
            _local_unwind2(pEh3Exce,-1);
        }
        return 1;
    }
```

## 6. 结语

SEH 代码还有很多，它们有许多包含在 `NTDLL.DLL` 中，比如 `RtlUnwind` 我曾经非常想翻译它，但至今我也还未完成，因此我也就没有粘出来。实际上从我以上粘出的初步代码，我们应该能看出 SEH 并不复杂，翻译 DLL 中的相关代码也并不困难，我估计我们唯一无法看见的代码就是中断处理的起始部分，把它想像成一个黑匣子就成了。附件有我的 `IDA` 注释说明，注释由于是一个动态过程，特别是起初的注释可能有错，我没有精力再去一一阅读更正，希望大家理解，一并奉献上，希望对大伙有用。

在这里，我只是作了 SEH 源码的一部分翻译工作，在翻译过程中我也在网上查看了很多关于 SEH 方面的文章，在此我不再一一列出，一并致谢这些无私奉献的网友和同行们！看了此文相信再看其它 SEH 专著就应该容易多了！

今天就是我们的嫦娥飞天的日子，谨以此文预祝她能回宫成功！顺便也希望此文是我下一个工作的起始点！

2007.10.24 于北京

`4dd466d0e1e0cc123afbe504da2462ca`
