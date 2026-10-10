
# 汇编跟踪与分析

```asm
f287beb1 33f6            xor     esi,esi
f287beb3 ff159cc087f2    call    dword ptr ds:[0F287C09Ch]
f287beb9 8bc6            mov     eax,esi
```

`lkd> u`

```asm
f287bebb 5e              pop     esi
f287bebc c20800          ret     8
f287bebf 688fbe87f2      push    0F287BE8Fh
f287bec4 e807f6ffff      call    f287b4d0
f287bec9 8d1440          lea     edx,[eax+eax*2]
f287becc 56              push    esi
f287becd 8b44240c        mov     eax,dword ptr [esp+0Ch]
f287bed1 50              push    eax
```

`lkd> u f287b2a0`

```asm
f287b2a0 8b442408        mov     eax,dword ptr [esp+8]
f287b2a4 8b542404        mov     edx,dword ptr [esp+4]
f287b2a8 56              push    esi
f287b2a9 8d4c240c        lea     ecx,[esp+0Ch]
f287b2ad 50              push    eax
f287b2ae 51              push    ecx
f287b2af 52              push    edx
f287b2b0 e8fbfdffff      call    f287b0b0
```

`lkd> u f287b0b0`

```asm
f287b0b0 55              push    ebp
f287b0b1 8bec            mov     ebp,esp
f287b0b3 83ec24          sub     esp,24h
f287b0b6 64a104000000    mov     eax,dword ptr fs:[00000004h]
f287b0bc 8945f8          mov     dword ptr [ebp-8],eax
f287b0bf 64a108000000    mov     eax,dword ptr fs:[00000008h]
f287b0c5 8945fc          mov     dword ptr [ebp-4],eax
f287b0c8 8b4508          mov     eax,dword ptr [ebp+8]
```

`lkd> u`

```asm
f287b0cb 8b4d0c          mov     ecx,dword ptr [ebp+0Ch]
f287b0ce 8b50fc          mov     edx,dword ptr [eax-4]
f287b0d1 8b45fc          mov     eax,dword ptr [ebp-4]
f287b0d4 3bd0            cmp     edx,eax
f287b0d6 8911            mov     dword ptr [ecx],edx
f287b0d8 7210            jb      f287b0ea
f287b0da 3b55f8          cmp     edx,dword ptr [ebp-8]
f287b0dd 730b            jae     f287b0ea
```

`lkd> u`

```asm
f287b0df b8e7030000      mov     eax,3E7h
f287b0e4 8be5            mov     esp,ebp
f287b0e6 5d              pop     ebp
f287b0e7 c20c00          ret     0Ch
f287b0ea 8b4510          mov     eax,dword ptr [ebp+10h]
f287b0ed 3d00000068      cmp     eax,68000000h
f287b0f2 0f829e000000    jb      f287b196
f287b0f8 3d00000065      cmp     eax,65000000h
```

其中 `f287b0d4` 处的几条比较指令可能就是判断溢出与否的关键，上面有两条指令也特别值得注意：

```asm
mov     eax,dword ptr fs:[00000004h]
mov     eax,dword ptr fs:[00000008h]
```

## TEB 数据结构

```c
typedef struct _TEB {              // Size: 0xF88
/*000*/  NT_TIB NtTib;
/*01C*/  VOID *EnvironmentPointer;
/*020*/  CLIENT_ID ClientId;       // PROCESS id, THREAD id
/*028*/  HANDLE ActiveRpcHandle;
/*02C*/  VOID *ThreadLocalStoragePointer;
/*030*/  PEB *ProcessEnvironmentBlock;  // PEB
/*034*/  ULONG LastErrorValue;
/*038*/  ULONG CountOfOwnedCriticalSections;
/*03C*/  ULONG CsrClientThread;
/*040*/  ULONG Win32ThreadInfo;
/*044*/  UCHAR Win32ClientInfo[0x7C];
/*0C0*/  ULONG WOW32Reserved;
/*0C4*/  ULONG CurrentLocale;
/*0C8*/  ULONG FpSoftwareStatusRegister;
/*0CC*/  UCHAR SystemReserved1[0xD8];   // ExitStack ???
/*1A4*/  ULONG Spare1;
/*1A8*/  ULONG ExceptionCode;
/*1AC*/  UCHAR SpareBytes1[0x28];
/*1D4*/  UCHAR SystemReserved2[0x28];
/*1FC*/  UCHAR GdiTebBatch[0x4E0];
/*6DC*/  ULONG gdiRgn;
/*6E0*/  ULONG gdiPen;
/*6E4*/  ULONG gdiBrush;
/*6E8*/  CLIENT_ID RealClientId;
/*6F0*/  ULONG GdiCachedProcessHandle;
/*6F4*/  ULONG GdiClientPID;
/*6F8*/  ULONG GdiClientTID;
/*6FC*/  ULONG GdiThreadLocalInfo;
/*700*/  UCHAR UserReserved[0x14];
/*714*/  UCHAR glDispatchTable[0x460];
/*B74*/  UCHAR glReserved1[0x68];
/*BDC*/  ULONG glReserved2;
/*BE0*/  ULONG glSectionInfo;
/*BE4*/  ULONG glSection;
/*BE8*/  ULONG glTable;
/*BEC*/  ULONG glCurrentRC;
/*BF0*/  ULONG glContext;
/*BF4*/  ULONG LastStatusValue;
/*BF8*/  LARGE_INTEGER StaticUnicodeString;
/*C00*/  UCHAR StaticUnicodeBuffer[0x20C];
/*E0C*/  ULONG DeallocationStack;
/*E10*/  UCHAR TlsSlots[0x100];
/*F10*/  LARGE_INTEGER TlsLinks;
/*F18*/  ULONG Vdm;
/*F1C*/  ULONG ReservedForNtRpc;
/*F20*/  LARGE_INTEGER DbgSsReserved;
/*F28*/  ULONG HardErrorsAreDisabled;
/*F2C*/  UCHAR Instrumentation[0x40];
/*F6C*/  ULONG WinSockData;
/*F70*/  ULONG GdiBatchCount;
/*F74*/  ULONG Spare2;
/*F78*/  ULONG Spare3;
/*F7C*/  ULONG Spare4;
/*F80*/  ULONG ReservedForOle;
/*F84*/  ULONG WaitingOnLoaderLock;
} TEB, *PTEB;
```

```c
typedef struct _NT_TIB {
    struct _EXCEPTION_REGISTRATION_RECORD *ExceptionList; // 00h Head of exception
              // record list
    PVOID StackBase;                    // 04h
    PVOID StackLimit;                   // 08h
    PVOID SubSystemTib;                 // 0Ch
    union {                             // 10h
        PVOID FiberData;                // for TIB
        ULONG Version;                  // for TEB
    };
    PVOID ArbitraryUserPointer;         // 14h Available
          // for application use
    struct _NT_TIB *Self;               // 18h Linear address
          // of TEB structure
} NT_TIB;
typedef NT_TIB *PNT_TIB;
```

`Fs:[4]` 和 `Fs:[8]` 分别是当前线程的栈基址和栈顶，现在思路就比较明朗了，卡巴就是通过检测 `GetProcAddress` 等关键 API 的返回地址是否处于栈中来判定栈溢出的发生。

## 现有检测模式的不足

既然知道了他的检测模式，那么突破就是轻而易举的事了，有两种思路：

修改 TEB 中 `Fs:[4]` 和 `Fs:[8]` 的值来使卡巴认为返回地址不在栈中.就是上面 Shellcode 中在 `call` 之前注释掉的部分 `mov dword ptr fs:[4],0x0012FFFF` 和 `mov dword ptr fs:[8],0x0012FFFF`。在测试的时候发现使用这一方法能使 `GetProcAddress` 绕过卡巴的检测，但是 `LoadLibrary` 系列却不行，百思不得其解，若有高人知道，请指教（可能 TEB 跟线程有关，`LoadLibrary` 涉及到多线程方面的一些问题）。

第二种方法就是在内存中找一个相对固定又可写可执行的地址写入 `push 返回地址 ret` 这样一系列的指令，然后再将函数返回地址指向其来绕过卡巴的检测。具体的实现如下：在原来调用 API 的地方，如

```asm
call   dword ptr [edi-4]           // LoadLibraryA
```

使用如下方法来代替:

```asm
mov    edx,dword ptr [edi-4]            // LoadLibraryA
call   gcall
```

其中 `gcall` 代码如下：

```asm
// ============ 绕过缓冲区溢出检查的 call ============
//
// 输入参数:
// edx 函数地址
// 0x7C884000
gcall:
    pop    eax //将真正的返回地址保存到 eax
    mov    ecx,0x7C884000
    push   ecx
    mov    byte  ptr [ecx],0x68   //push 指令
    mov    dword ptr [ecx+1],eax  //写入地址
    mov    byte  ptr [ecx+5],0xC3  //写入 ret 指令
    jmp    edx //执行真正的函数
```

`0x7C884000` 是 `Kernel32` 数据段的地址，当然，可以用其它等价的地址替换。再次运行 `shellcode`，卡巴没有任何反应，如图 2：

![](img/17a4f7679ca952652d83bd3e33a8eeb5.jpg)

## 针对性的改进

在卡巴现有的检测中还是有一个 Bug，就是 TEB 中的栈基址和栈顶数据是不可信的，应当在初始化时保存，而不应每次都去重新获取，当然这种检测机制本身就是不可靠的，需要改进，这就是本文要提出的基于栈指纹检测缓冲区溢出，说白了就是利用特征码+API Hook 来更可靠的检测栈溢出的发生，防止迂回绕过我们的检测，因为有一点是肯定的：栈溢出发生时 Shellcode 是在栈中的，这个想法也是借鉴自反病毒的概念。微软公司在 VC7 开始提供了一个 `/GS` 编译选项来防止栈溢出带来的危害，但是事实证明还是能被绕过，例如：覆盖 SEH。笔者认为安全产品做的越前端就越不容易被绕过，比如拦截 Shellcode 的行为肯定要比拦截溢出的发生有效的多。

笔者利用 `Detour` 库做了一个检测的模型，代码如下：

```cpp
// dll.cpp : 定义 DLL 应用程序的入口点。
//
#include <windows.h>
#include "detours.h"
#include <dbt.h>

DETOUR_TRAMPOLINE(HMODULE WINAPI fLoadLibraryA(LPCTSTR lpFileName),LoadLibraryA);

HMODULE WINAPI MyLoadLibraryA(LPCTSTR lpFileName)
{
  DWORD stackbase,stacklimit,retaddr;

  __asm{
    mov eax,dword ptr [esp+0x1c]
    mov retaddr,eax
    mov eax,dword ptr fs:[4]
    mov stackbase,eax
    mov eax,dword ptr fs:[8]
    mov stacklimit,eax
  }

  if ( retaddr < stacklimit && retaddr > stackbase )
  {
overflow:
    MessageBox(0,"BufferOverflow Detected!","gyzy",MB_ICONINFORMATION);
    return NULL;
  }

  //0x64 0xA1 0x30 0x00 0x00 0x00
  __asm{
```

