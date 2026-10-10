

# 挫败反跟踪

> 原文：[`www.pediy.com/kssd/pediy06/pediy6435.htm`](https://www.pediy.com/kssd/pediy06/pediy6435.htm)

不久就要进入总复习阶段，如果我不幸没能保送入高中，可能有一段时间不能来论坛了。这里利用我倒数第二个休息日介绍一些常见反跟踪技术，望能避免一些新出道的 Cracker 作重复劳动，高手就不要在此费神了。我的文笔不是很好，还望诸位看官海涵。   
  
一、TEB/TIB   
  
    TEB(Thread Environment Block)  在 Windows 9x 系列中被称为 TIB(Thread Information Block)，它记录了线程的重要信息，且每一个线程皆对应一个 TEB 结构。 Matt Pietrek 已列出了它的结构（摘自 Matt Pietrek 的 Under The Hood - MSJ 1996）：   
  
//===========================================================    
// file: TIB.H    
// Author: Matt Pietrek    
// From: Microsoft Systems Journal "Under the Hood", May 1996    
//===========================================================    
#pragma pack(1)    
  
typedef struct \_EXCEPTION\_REGISTRATION\_RECORD    
{    
  struct \_EXCEPTION\_REGISTRATION\_RECORD \* pNext;    
  FARPROC                                pfnHandler;    
} EXCEPTION\_REGISTRATION\_RECORD, \*PEXCEPTION\_REGISTRATION\_RECORD;    
  
typedef struct \_TIB    
{    
PEXCEPTION\_REGISTRATION\_RECORD pvExcept; // 00h Head of exception record list    
PVOID  pvStackUserTop;        // 04h Top of user stack    
PVOID  pvStackUserBase;        // 08h Base of user stack    
  
union                          // 0Ch (NT/Win95 differences)    
{    
  struct  // Win95 fields    
  {    
      WORD    pvTDB;          // 0Ch TDB    
      WORD    pvThunkSS;      // 0Eh SS selector used for thunking to 16 bits    
      DWORD  unknown1;      // 10h    
  } WIN95;    
  
  struct  // WinNT fields    
  {    
      PVOID SubSystemTib;    // 0Ch    
      ULONG FiberData;        // 10h    
  } WINNT;    
} TIB\_UNION1;    
  
PVOID  pvArbitrary;            // 14h Available for application use    
struct \_tib \*ptibSelf;          // 18h Linear address of TIB structure    
  
union                          // 1Ch (NT/Win95 differences)    
{    
  struct  // Win95 fields    
  {    
      WORD    TIBFlags;          // 1Ch    
      WORD    Win16MutexCount;    // 1Eh    
      DWORD  DebugContext;      // 20h    
      DWORD  pCurrentPriority;  // 24h    
      DWORD  pvQueue;            // 28h Message Queue selector    
  } WIN95;    
  
  struct  // WinNT fields    
  {    
      DWORD unknown1;            // 1Ch    
      DWORD processID;            // 20h    
      DWORD threadID;            // 24h    
      DWORD unknown2;            // 28h    
  } WINNT;    
} TIB\_UNION2;    
  
PVOID\*  pvTLSArray;            // 2Ch Thread Local Storage array    
  
union                          // 30h (NT/Win95 differences)    
{    
  struct  // Win95 fields    
  {    
      PVOID\*  pProcess;      // 30h Pointer to owning process database    
  } WIN95;    
} TIB\_UNION3;    
  
} TIB, \*PTIB;    
#pragma pack()    
  
    结构很复杂，但多数与我们无关，我们只关心这些：   
  
      DWORD  DebugContext;      // 20h    
      PVOID\*  pProcess;      // 30h Pointer to owning process database        
  
    曾经看过 ProcDump 代码的人一定记得 DebugContext ，ProDump 就是通过将其置零隐藏自身的。   
    而 pProcess “拥有者”，当用用户模式调试器如 OllyDbg 调试程序时，调试器会将程序作为一个子线程进行跟踪，在这种情况下，被调试程序的“拥有者”就是调试器，也就是说，其 TEB 的 30h 偏移所指内容定不为 0 ，程序利用这一点，判断 30h 偏移指向的内容，来判断是否有调试器跟踪。若看过 IsDebuggerPresent 函数的代码，会发现它也是利用 pProcess 工作的：   
  
    mov eax, fs:\[18\]; 取 TEB 结构线性偏移地址   
    mov eax, dword ptr \[eax+30h\]; 取 pProcess 中的值   
    movzx eax, byte ptr \[eax+2h\]   
        ret   
  
    调试时遇到任何访问 FS 段 20h / 30h 偏移地址的代码就要小心了，他们十有八九是要检测调试器的，不想被发现的话将其所用寄存器的值置零或者在其判断语句之后反转 ZF 标志位吧。   
  
二、SEH   
  
    Asprotect / tElock 等流行保护程序无一例外使用了 SEH ，关于 SEH ，我不想对它扫盲，如果不了解，请参阅 Hume 的 SEH 教程。   
    SEH 不但可以改变流程干扰跟踪，还可以通过修改 Context 来做一些“不良勾当”。这里是 Context 结构定义：       
  
CONTEXT            STRUC    
cx\_ContextFlags    DD ?    
  
;CONTEXT\_DEBUG\_REGISTERS    
cx\_Dr0                DD ?        ;04    
cx\_Dr1                DD ?        ;08    
cx\_Dr2                DD ?        ;0C    
cx\_Dr3                DD ?        ;10    
cx\_Dr6                DD ?        ;14    
cx\_Dr7                DD ?        ;18    
  
;CONTEXT\_FLOATING\_POINT    
cx\_ControlWord            DD ?    
cx\_StatusWord            DD ?    
cx\_TagWord                DD ?    
cx\_ErrorOffset            DD ?    
cx\_ErrorSelector        DD ?    
cx\_DataOffset            DD ?    
cx\_DataSelector        DD ?    
SIZE\_OF\_80387\_REGISTERS        EQU 80    
cx\_RegisterArea        DB SIZE\_OF\_80387\_REGISTERS DUP (?)    
cx\_Cr0NpxState            DD ?    
  
;CONTEXT\_SEGMENTS    
cx\_SegGs            DD ?        ;8C    
cx\_SegFs            DD ?        ;90    
cx\_SegEs            DD ?        ;94    
cx\_SegDs            DD ?        ;98    
  
;CONTEXT\_INTEGER    
cx\_Edi                DD ?        ;9C    
cx\_Esi                DD ?        ;A0    
cx\_Ebx                DD ?        ;A4    
cx\_Edx                DD ?        ;A8    
cx\_Ecx                DD ?        ;AC    
cx\_Eax                DD ?        ;B0    
  
;CONTEXT\_CONTROL    
cx\_Ebp                DD ?        ;B4    
cx\_Eip                DD ?        ;B8    
cx\_SegCs            DD ?        ;BC    
cx\_EFlags            DD ?        ;C0    
cx\_Esp                DD ?        ;C4    
cx\_SegSs            DD ?        ;C8    
\_CONTEXT            ENDS    
;size of CONTEXT is 0CCH bytes   
  
     对我们有帮助的是：   
  
cx\_Dr0                DD ?        ;04    
cx\_Dr1                DD ?        ;08    
cx\_Dr2                DD ?        ;0C    
cx\_Dr3                DD ?        ;10    
cx\_Dr6                DD ?        ;14    
cx\_Dr7                DD ?        ;18    
  
cx\_Eip                DD ?        ;B8    
        
    Dr0、Dr1、Dr2、Dr3 是调试断点寄存器，而 Dr6、Dr7 是调试控制寄存器，一般的 SEH Handler 会将 Dr0~6 清零，将 Dr7 置 155h 使之前所下断点失效，这种情形在壳中极为常见，关于细节，请参阅有关调试寄存器的文献资料，中英文都有。   
  
    Eip 不必多说，如果 SEH Handler 修改了此值，系统就会从其开始继续执行程序，如果直接在其设断会节省很多时间，而盲目在系统中不断返回常常会迷失在系统中，因此——   
  
    调试中如果获知指向 Context 的寄存器，那么它的 04h/08h/0Ch/10h/14h/18h 偏移将访问调试寄存器，用尽手段恢复它们吧。   
    对于利用 SEH Handler 修改流程的程序，见到写入 Context 之 0B8h 偏移的代码，不要犹豫在那里下断，可以省去很多时间进行进一步跟踪。   
  
    SEH Hanlder 还有一些特殊的用法，例如：   
       
    SehHandler:   
             xor eax, eax   
             mov esp, fs:\[eax\]; 恢复堆栈   
             pop fs:\[eax\]; 恢复旧的 SEH Hanlder   
             jmp somewhere   
  
    这种情况将不通过系统而直接继续执行代码。另外 Windows 9x 系列中还可以通过 SEH 技巧夺取 RIng 0 权限直接操作 DrX 寄存器，可以参考 ljtt 《怎样利用 SEH 技巧进入 Ring0 级》一文。   
  
（作者 linson，未完待续  ）  
    
 4dd466d0e1e0cc123afbe504da2462ca