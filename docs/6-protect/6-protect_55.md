

# 逆向 MmGetPhysicalAddress 并正向

> 原文：[`www.pediy.com/kssd/pediy11/125037.html`](https://www.pediy.com/kssd/pediy11/125037.html)

　　这两天想做点坏事，也没啥，就是 PTE 欺骗啦。  
　　其中涉及了 VirtualAddress(VA)转化 PhysicalAddress(PA)的生成原理。  
　　于是网上找了些相关资料，很努力地看。  
　　但看来看去，怎么和 MmGetPhysicalAddress 逆向结果不大一样啊。  
　　这才发现，大部分现存的资料都很旧了。虽然应用原理还是差不多，但是已经不能被我们伸手党使用了。  
　　于是决定逆向一下 MmGetPhysicalAddress，再正向它，尝试 C++实现之。  
\----\_HARDWARE\_PTE 结构  

代码:

```
lkd> dt \_hardware\_pte
nt!\_HARDWARE\_PTE
   +0x000 Valid            : Pos 0, 1 Bit
   +0x000 Write            : Pos 1, 1 Bit
   +0x000 Owner            : Pos 2, 1 Bit
   +0x000 WriteThrough     : Pos 3, 1 Bit
   +0x000 CacheDisable     : Pos 4, 1 Bit
   +0x000 Accessed         : Pos 5, 1 Bit
   +0x000 Dirty            : Pos 6, 1 Bit
   +0x000 LargePage        : Pos 7, 1 Bit
   +0x000 Global           : Pos 8, 1 Bit
   +0x000 CopyOnWrite      : Pos 9, 1 Bit
   +0x000 Prototype        : Pos 10, 1 Bit
   +0x000 reserved0        : Pos 11, 1 Bit
   +0x000 PageFrameNumber  : Pos 12, 26 Bits
   +0x000 reserved1        : Pos 38, 26 Bits
   +0x000 LowPart          : Uint4B
   +0x004 HighPart         : Uint4B

```

由此，可以列出如下结构：  
代码:

```
typedef struct \_HARDWARE\_PTE\_X86PAE
  {
  union
    {
    struct
      {
      ULONGLONG Valid:1;
      ULONGLONG Write:1;
      ULONGLONG Owner:1;
      ULONGLONG WriteThrough:1;
      ULONGLONG CacheDisable:1;
      ULONGLONG Accessed:1;
      ULONGLONG Dirty:1;
      ULONGLONG LargePage:1;
      ULONGLONG Global:1;
      ULONGLONG CopyOnWrite:1;
      ULONGLONG Prototype:1;
      ULONGLONG reserved0:1;
      ULONGLONG PageFrameNumber:26;
      ULONGLONG reserved1:26;
      };
    struct
      {
      ULONG LowPart;
      ULONG HighPart;
      };
    };
  } HARDWARE\_PTE\_X86PAE, \*PHARDWARE\_PTE\_X86PAE,MMPTE\_HARDWARE, \*PMMPTE\_HARDWARE,\_MMPTE\_HARDWARE,HARDWARE\_PTE,\*PHARDWARE\_PTE;

```

　　注意，由于 PAE 模式下，PTE 结构突破 32 位，所以结构体使用位域的时候，需要用\_\_int64 来做(2 byte)。  
　　之前我做 PTE 结构时，使用 ulong 来做位域，无论是否强制结构体对齐，结果编译器硬要把这个结构识别为 3 byte。汗死。  
　　直到后面 xIkUg 大哥提供旧式结构体后，我才发现之间的差异。  
　　不过 xIkUg 大哥提供的 PTE 结构指明 PageFrameNumber:24;很奇怪。我在 XP 和 2003 上列表的结构都指明 PageFrameNumber:26。所以，在实际应用时，大家还是用 windbg 看看自己的 pte 结构好了。免得出现意外。  
  
\----MmGetPhysicalAddress 的 IDA 结果  
代码:

```
;Microsoft Windows XP Professional Service Pack 3 \[ver 5.1.2600\]
;ntkrnlpa.exe
;MD5   : E7E72F0935D0F224768126B49CF2A9E8

.text:00430CD2                         \_MmGetPhysicalAddress@4 proc near       ; CODE XREF: IoSetDumpRange(x,x,x,x)+22p
.text:00430CD2                                                                 ; IoSetDumpRange(x,x,x,x)+62p ...
.text:00430CD2
.text:00430CD2                         BaseAddress     = dword ptr  8
.text:00430CD2
.text:00430CD2 8B FF                                   mov     edi, edi
.text:00430CD4 55                                      push    ebp
.text:00430CD5 8B EC                                   mov     ebp, esp
.text:00430CD7 53                                      push    ebx
.text:00430CD8 56                                      push    esi
.text:00430CD9 57                                      push    edi
.text:00430CDA 8B 7D 08                                mov     edi, \[ebp+BaseAddress\]
.text:00430CDD 8B CF                                   mov     ecx, edi
.text:00430CDF C1 E9 12                                shr     ecx, 12h
.text:00430CE2 81 E1 F8 3F 00 00                       and     ecx, 3FF8h
.text:00430CE8 8B 81 00 00 60 C0                       mov     eax, \[ecx+0C0600000h\]
.text:00430CEE 8B 89 04 00 60 C0                       mov     ecx, \[ecx+0C0600004h\]
.text:00430CF4 BE 81 00 00 00                          mov     esi, 81h
.text:00430CF9 8B D0                                   mov     edx, eax
.text:00430CFB 23 D6                                   and     edx, esi
.text:00430CFD 33 DB                                   xor     ebx, ebx
.text:00430CFF 3B D6                                   cmp     edx, esi
.text:00430D01 75 1F                                   jnz     short loc\_430D22
.text:00430D03 85 DB                                   test    ebx, ebx
.text:00430D05 75 1B                                   jnz     short loc\_430D22
.text:00430D07 0F AC C8 0C                             shrd    eax, ecx, 0Ch
.text:00430D0B C1 E9 0C                                shr     ecx, 0Ch
.text:00430D0E 8B CF                                   mov     ecx, edi
.text:00430D10 C1 E9 0C                                shr     ecx, 0Ch
.text:00430D13 25 FF FF FF 03                          and     eax, 3FFFFFFh
.text:00430D18 81 E1 FF 01 00 00                       and     ecx, 1FFh
.text:00430D1E 03 C1                                   add     eax, ecx
.text:00430D20 EB 3F                                   jmp     short loc\_430D61
.text:00430D22                         ; ---------------------------------------------------------------------------
.text:00430D22
.text:00430D22                         loc\_430D22:                             ; CODE XREF: MmGetPhysicalAddress(x)+2Fj
.text:00430D22                                                                 ; MmGetPhysicalAddress(x)+33j
.text:00430D22 83 E0 01                                and     eax, 1
.text:00430D25 33 C9                                   xor     ecx, ecx
.text:00430D27 0B C1                                   or      eax, ecx
.text:00430D29 74 24                                   jz      short loc\_430D4F
.text:00430D2B 8B CF                                   mov     ecx, edi
.text:00430D2D C1 E9 09                                shr     ecx, 9
.text:00430D30 81 E1 F8 FF 7F 00                       and     ecx, 7FFFF8h
.text:00430D36 8B 91 04 00 00 C0                       mov     edx, \[ecx+0C0000004h\]
.text:00430D3C 81 E9 00 00 00 40                       sub     ecx, -0C0000000h
.text:00430D42 8B 01                                   mov     eax, \[ecx\]
.text:00430D44 8B C8                                   mov     ecx, eax
.text:00430D46 83 E1 01                                and     ecx, 1
.text:00430D49 33 F6                                   xor     esi, esi
.text:00430D4B 0B CE                                   or      ecx, esi
.text:00430D4D 75 06                                   jnz     short loc\_430D55
.text:00430D4F
.text:00430D4F                         loc\_430D4F:                             ; CODE XREF: MmGetPhysicalAddress(x)+57j
.text:00430D4F 33 C0                                   xor     eax, eax
.text:00430D51 33 D2                                   xor     edx, edx
.text:00430D53 EB 1F                                   jmp     short loc\_430D74
.text:00430D55                         ; ---------------------------------------------------------------------------
.text:00430D55
.text:00430D55                         loc\_430D55:                             ; CODE XREF: MmGetPhysicalAddress(x)+7Bj
.text:00430D55 0F AC D0 0C                             shrd    eax, edx, 0Ch
.text:00430D59 C1 EA 0C                                shr     edx, 0Ch
.text:00430D5C 25 FF FF FF 03                          and     eax, 3FFFFFFh
.text:00430D61
.text:00430D61                         loc\_430D61:                             ; CODE XREF: MmGetPhysicalAddress(x)+4Ej
.text:00430D61 33 C9                                   xor     ecx, ecx
.text:00430D63 0F A4 C1 0C                             shld    ecx, eax, 0Ch
.text:00430D67 C1 E0 0C                                shl     eax, 0Ch
.text:00430D6A 81 E7 FF 0F 00 00                       and     edi, 0FFFh
.text:00430D70 03 C7                                   add     eax, edi
.text:00430D72 8B D1                                   mov     edx, ecx
.text:00430D74
.text:00430D74                         loc\_430D74:                             ; CODE XREF: MmGetPhysicalAddress(x)+81j
.text:00430D74 5F                                      pop     edi
.text:00430D75 5E                                      pop     esi
.text:00430D76 5B                                      pop     ebx
.text:00430D77 5D                                      pop     ebp
.text:00430D78 C2 04 00                                retn    4
.text:00430D78                         \_MmGetPhysicalAddress@4 endp

```

　　一看之下，有点晕。  
　　没关系，有前面清楚的 PTE 结构指引，这个函数就比较容易理解了。  
　　以下就是它的 C++实现：  
代码:

```
struct VirtAddr1
  {
  unsigned int PageOFF:12;
  unsigned int PageTable:20;
  };

struct VirtAddr2
  {
  unsigned int PageOFF:12;
  unsigned int PageTableIndex:9;
  unsigned int PageLargeIndex:11;
  };

struct VirtAddr3
  {
  unsigned int PageOFF:12;
  unsigned int PageTableIndex:9;
  unsigned int PageDirIndex:9;
  unsigned int PageDirPoint:2;
  };

MMPTE\_HARDWARE\* const PDE\_BASE = (MMPTE\_HARDWARE\*)0xC0600000;  //页目录表
MMPTE\_HARDWARE\* const PTE\_BASE = (MMPTE\_HARDWARE\*)0xC0000000;  //进程页表

PHYSICAL\_ADDRESS GetPhysicalAddress(const unsigned long VirtualAddress)
  {
  const VirtAddr1\* VA1 = (const VirtAddr1\*)&VirtualAddress;
  const VirtAddr2\* VA2 = (const VirtAddr2\*)&VirtualAddress;
  MMPTE\_HARDWARE PTE = \*(PDE\_BASE + VA2->PageLargeIndex);
  PHYSICAL\_ADDRESS PA = {0,0};
  if(!(PTE.Valid))return PA;
  if(PTE.LargePage)
    {
    PTE.PageFrameNumber += VA2->PageTableIndex;
    }
  else
    {
    PTE = \*(PTE\_BASE + VA1->PageTable);
    if(!(PTE.Valid))return PA;
    }
  PTE.reserved1 = 0;
  PA.HighPart = PTE.HighPart;
  PA.LowPart = PTE.LowPart;
  ((VirtAddr1\*)(&PA.LowPart))->PageOFF = VA1->PageOFF;
  return PA;
  }

```

　　到这里应该可以结束了，不过我又想看看 win2003 是不是有啥不一样呢？  
代码:

```
;Microsoft Windows Server 2003 Enterprise Edition Service Pack 2 \[ver 5.2.3790\]
;ntkrnlpa.exe
;MD5   : 20A327D41B0A5659AAFD0AA1AD98F433

.text:00439DE0                         ; \_\_int64 \_\_stdcall MmGetPhysicalAddress(void \*BaseAddress)
.text:00439DE0                                         public \_MmGetPhysicalAddress@4
.text:00439DE0                         \_MmGetPhysicalAddress@4 proc near       ; CODE XREF: IoSetDumpRange(x,x,x,x)+1Ep
.text:00439DE0                                                                 ; IoSetDumpRange(x,x,x,x)+5Dp ...
.text:00439DE0
.text:00439DE0                         var\_8           = dword ptr -8
.text:00439DE0                         BaseAddress     = dword ptr  8
.text:00439DE0
.text:00439DE0 8B FF                                   mov     edi, edi
.text:00439DE2 55                                      push    ebp
.text:00439DE3 8B EC                                   mov     ebp, esp
.text:00439DE5 51                                      push    ecx
.text:00439DE6 51                                      push    ecx
.text:00439DE7 53                                      push    ebx
.text:00439DE8 8B 5D 08                                mov     ebx, \[ebp+BaseAddress\]
.text:00439DEB 8B C3                                   mov     eax, ebx
.text:00439DED C1 E8 12                                shr     eax, 12h
.text:00439DF0 25 F8 3F 00 00                          and     eax, 3FF8h
.text:00439DF5 56                                      push    esi
.text:00439DF6 8D 88 00 00 60 C0                       lea     ecx, \[eax+0C0600000h\]
.text:00439DFC 8B 31                                   mov     esi, \[ecx\]
.text:00439DFE 8B 51 04                                mov     edx, \[ecx+4\]
.text:00439E01 57                                      push    edi
.text:00439E02 BF 81 00 00 00                          mov     edi, 81h
.text:00439E07 89 75 F8                                mov     \[ebp+var\_8\], esi
.text:00439E0A 23 F7                                   and     esi, edi
.text:00439E0C 33 C0                                   xor     eax, eax
.text:00439E0E 3B F7                                   cmp     esi, edi
.text:00439E10 75 09                                   jnz     short loc\_439E1B
.text:00439E12 85 C0                                   test    eax, eax
.text:00439E14 75 05                                   jnz     short loc\_439E1B
.text:00439E16 8B 45 F8                                mov     eax, \[ebp+var\_8\]
.text:00439E19 EB 23                                   jmp     short loc\_439E3E
.text:00439E1B                         ; ---------------------------------------------------------------------------
.text:00439E1B
.text:00439E1B                         loc\_439E1B:                             ; CODE XREF: MmGetPhysicalAddress(x)+30j
.text:00439E1B                                                                 ; MmGetPhysicalAddress(x)+34j
.text:00439E1B 8B 01                                   mov     eax, \[ecx\]
.text:00439E1D 8B 51 04                                mov     edx, \[ecx+4\]
.text:00439E20 8B C8                                   mov     ecx, eax
.text:00439E22 83 E1 01                                and     ecx, 1
.text:00439E25 33 F6                                   xor     esi, esi
.text:00439E27 0B CE                                   or      ecx, esi
.text:00439E29 74 4F                                   jz      short loc\_439E7A
.text:00439E2B 8B C8                                   mov     ecx, eax
.text:00439E2D 81 E1 80 00 00 00                       and     ecx, 80h
.text:00439E33 56                                      push    esi
.text:00439E34 89 4D F8                                mov     \[ebp+var\_8\], ecx
.text:00439E37 59                                      pop     ecx
.text:00439E38 74 1C                                   jz      short loc\_439E56
.text:00439E3A 85 C9                                   test    ecx, ecx
.text:00439E3C 75 18                                   jnz     short loc\_439E56
.text:00439E3E
.text:00439E3E                         loc\_439E3E:                             ; CODE XREF: MmGetPhysicalAddress(x)+39j
.text:00439E3E 0F AC D0 0C                             shrd    eax, edx, 0Ch
.text:00439E42 8B CB                                   mov     ecx, ebx
.text:00439E44 C1 E9 0C                                shr     ecx, 0Ch
.text:00439E47 25 FF FF FF 03                          and     eax, 3FFFFFFh
.text:00439E4C 81 E1 FF 01 00 00                       and     ecx, 1FFh
.text:00439E52 03 C1                                   add     eax, ecx
.text:00439E54 EB 33                                   jmp     short loc\_439E89
.text:00439E56                         ; ---------------------------------------------------------------------------
.text:00439E56
.text:00439E56                         loc\_439E56:                             ; CODE XREF: MmGetPhysicalAddress(x)+58j
.text:00439E56                                                                 ; MmGetPhysicalAddress(x)+5Cj
.text:00439E56 8B CB                                   mov     ecx, ebx
.text:00439E58 C1 E9 09                                shr     ecx, 9
.text:00439E5B 81 E1 F8 FF 7F 00                       and     ecx, 7FFFF8h
.text:00439E61 8B 91 04 00 00 C0                       mov     edx, \[ecx+0C0000004h\]
.text:00439E67 81 E9 00 00 00 40                       sub     ecx, -0C0000000h
.text:00439E6D 8B 01                                   mov     eax, \[ecx\]
.text:00439E6F 8B C8                                   mov     ecx, eax
.text:00439E71 83 E1 01                                and     ecx, 1
.text:00439E74 33 F6                                   xor     esi, esi
.text:00439E76 0B CE                                   or      ecx, esi
.text:00439E78 75 06                                   jnz     short loc\_439E80
.text:00439E7A
.text:00439E7A                         loc\_439E7A:                             ; CODE XREF: MmGetPhysicalAddress(x)+49j
.text:00439E7A 33 C0                                   xor     eax, eax
.text:00439E7C 33 D2                                   xor     edx, edx
.text:00439E7E EB 1F                                   jmp     short loc\_439E9F
.text:00439E80                         ; ---------------------------------------------------------------------------
.text:00439E80
.text:00439E80                         loc\_439E80:                             ; CODE XREF: MmGetPhysicalAddress(x)+98j
.text:00439E80 0F AC D0 0C                             shrd    eax, edx, 0Ch
.text:00439E84 25 FF FF FF 03                          and     eax, 3FFFFFFh
.text:00439E89
.text:00439E89                         loc\_439E89:                             ; CODE XREF: MmGetPhysicalAddress(x)+74j
.text:00439E89 33 C9                                   xor     ecx, ecx
.text:00439E8B 0F A4 C1 0C                             shld    ecx, eax, 0Ch
.text:00439E8F C1 EA 0C                                shr     edx, 0Ch
.text:00439E92 C1 E0 0C                                shl     eax, 0Ch
.text:00439E95 81 E3 FF 0F 00 00                       and     ebx, 0FFFh
.text:00439E9B 03 C3                                   add     eax, ebx
.text:00439E9D 8B D1                                   mov     edx, ecx
.text:00439E9F
.text:00439E9F                         loc\_439E9F:                             ; CODE XREF: MmGetPhysicalAddress(x)+9Ej
.text:00439E9F 5F                                      pop     edi
.text:00439EA0 5E                                      pop     esi
.text:00439EA1 5B                                      pop     ebx
.text:00439EA2 C9                                      leave
.text:00439EA3 C2 04 00                                retn    4
.text:00439EA3                         \_MmGetPhysicalAddress@4 endp

```

　　粗看起来，是不一样了，但其实是一样的。不知道为什么多了一些不必要的判断与跳转。  
　　另外就是 Vista 的 MiGetPhysicalAddress，计算过程都差不多，但多了更多的判断与跳转，还有一些无实际意义的计算，这个我很不明白是为什么。如果有牛人知道，请告知一下。  
　　代码有点长，不贴了。看过 Vista 的 MiGetPhysicalAddress，其中显示，reserved0 这个位被使用了，但只做了一个判断，没有实际意义。  
　　再者，XP 与 2003 的 MmGetVirtualForPhysical 结果完全一样。而 Vista 多了一句“and     eax, 0FFFFFFFCh”，也就是取到 PTE 放弃最后两位。但是想来想去，好像也没什么实际意义。  
  
　　编译器开完全优化，编译出来的结果如下：  
代码:

```
.text:000117B0                         ; \_LARGE\_INTEGER \_\_stdcall GetPhysicalAddress(const unsigned int VirtualAddress)
.text:000117B0                         ?GetPhysicalAddress@@YG?AT\_LARGE\_INTEGER@@K@Z proc near
.text:000117B0                                                                 ; CODE XREF: Refresh(void \*)+10p
.text:000117B0
.text:000117B0                         VirtualAddress  = dword ptr  4
.text:000117B0
.text:000117B0 53                                      push    ebx
.text:000117B1 8B 5C 24 08                             mov     ebx, \[esp+4+VirtualAddress\]
.text:000117B5 55                                      push    ebp
.text:000117B6 8B C3                                   mov     eax, ebx
.text:000117B8 C1 E8 15                                shr     eax, 15h
.text:000117BB 56                                      push    esi
.text:000117BC 8B 34 C5 00 00 60 C0                    mov     esi, ds:0C0600000h\[eax\*8\]
.text:000117C3 8B CE                                   mov     ecx, esi
.text:000117C5 57                                      push    edi
.text:000117C6 8B 3C C5 04 00 60 C0                    mov     edi, ds:0C0600004h\[eax\*8\]
.text:000117CD 83 E1 01                                and     ecx, 1
.text:000117D0 33 ED                                   xor     ebp, ebp
.text:000117D2 33 C0                                   xor     eax, eax
.text:000117D4 33 D2                                   xor     edx, edx
.text:000117D6 0B CD                                   or      ecx, ebp
.text:000117D8 74 61                                   jz      short loc\_1183B
.text:000117DA 8B CE                                   mov     ecx, esi
.text:000117DC 81 E1 80 00 00 00                       and     ecx, 80h
.text:000117E2 0B CD                                   or      ecx, ebp
.text:000117E4 74 27                                   jz      short loc\_1180D
.text:000117E6 8B C3                                   mov     eax, ebx
.text:000117E8 C1 E8 0C                                shr     eax, 0Ch
.text:000117EB 25 FF 01 00 00                          and     eax, 1FFh
.text:000117F0 B9 00 10 00 00                          mov     ecx, 1000h
.text:000117F5 F7 E1                                   mul     ecx
.text:000117F7 03 C6                                   add     eax, esi
.text:000117F9 13 D7                                   adc     edx, edi
.text:000117FB 33 C6                                   xor     eax, esi
.text:000117FD 33 D7                                   xor     edx, edi
.text:000117FF 25 00 F0 FF FF                          and     eax, 0FFFFF000h
.text:00011804 83 E2 3F                                and     edx, 3Fh
.text:00011807 33 F0                                   xor     esi, eax
.text:00011809 33 FA                                   xor     edi, edx
.text:0001180B EB 1E                                   jmp     short loc\_1182B
.text:0001180D                         ; ---------------------------------------------------------------------------
.text:0001180D
.text:0001180D                         loc\_1180D:                              ; CODE XREF: GetPhysicalAddress(ulong)+34j
.text:0001180D 8B CB                                   mov     ecx, ebx
.text:0001180F C1 E9 0C                                shr     ecx, 0Ch
.text:00011812 8B 34 CD 00 00 00 C0                    mov     esi, ds:0C0000000h\[ecx\*8\]
.text:00011819 8B 3C CD 04 00 00 C0                    mov     edi, ds:0C0000004h\[ecx\*8\]
.text:00011820 8B CE                                   mov     ecx, esi
.text:00011822 83 E1 01                                and     ecx, 1
.text:00011825 33 ED                                   xor     ebp, ebp
.text:00011827 0B CD                                   or      ecx, ebp
.text:00011829 74 10                                   jz      short loc\_1183B
.text:0001182B
.text:0001182B                         loc\_1182B:                              ; CODE XREF: GetPhysicalAddress(ulong)+5Bj
.text:0001182B 8B C6                                   mov     eax, esi
.text:0001182D 33 C3                                   xor     eax, ebx
.text:0001182F 83 E7 3F                                and     edi, 3Fh
.text:00011832 25 FF 0F 00 00                          and     eax, 0FFFh
.text:00011837 33 C6                                   xor     eax, esi
.text:00011839 8B D7                                   mov     edx, edi
.text:0001183B
.text:0001183B                         loc\_1183B:                              ; CODE XREF: GetPhysicalAddress(ulong)+28j
.text:0001183B                                                                 ; GetPhysicalAddress(ulong)+79j
.text:0001183B 5F                                      pop     edi
.text:0001183C 5E                                      pop     esi
.text:0001183D 5D                                      pop     ebp
.text:0001183E 5B                                      pop     ebx
.text:0001183F C2 04 00                                retn    4
.text:0001183F                         ?GetPhysicalAddress@@YG?AT\_LARGE\_INTEGER@@K@Z endp

```

　　看起来比系统的小了一些。  
　　怎么说呢，光看 MmGetPhysicalAddress 实在是晕。但由 C 实现出来，原理异常清晰。  
　　同时，解析过 XP、2003、Vista 以后，发现它们没有太大改动，那么，我们可以放心使用自己实现的 GetPhysicalAddress 喽。  
  
　　最后感谢 xIkUg。  
　　胡乱写写，小小心得而已，欢迎拍砖。