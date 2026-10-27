
```


;Microsoft Windows XP Professional Service Pack 3 [ver 5.1.2600]

;ntkrnlpa.exe

;MD5   : E7E72F0935D0F224768126B49CF2A9E8



.text:00430CD2                         _MmGetPhysicalAddress@4 proc near       ; CODE XREF: IoSetDumpRange(x,x,x,x)+22p

.text:00430CD2                                                                 ; IoSetDumpRange(x,x,x,x)+62p ...

.text:00430CD2

.text:00430CD2                         BaseAddress     = dword ptr  8

.text:00430CD2

.text:00430CD2 8B FF                                   mov     edi, edi

.text:00430CD4 55                                      push    ebp

.text:00430CD5 8B EC                                   mov     ebp, esp

.text:00430CD7 53                                      push    ebx

.text:00430CD8 56                                      push    esi

.text:00430CD9 57                                      push    edi

.text:00430CDA 8B 7D 08                                mov     edi, [ebp+BaseAddress]

.text:00430CDD 8B CF                                   mov     ecx, edi

.text:00430CDF C1 E9 12                                shr     ecx, 12h

.text:00430CE2 81 E1 F8 3F 00 00                       and     ecx, 3FF8h

.text:00430CE8 8B 81 00 00 60 C0                       mov     eax, [ecx+0C0600000h]

.text:00430CEE 8B 89 04 00 60 C0                       mov     ecx, [ecx+0C0600004h]

.text:00430CF4 BE 81 00 00 00                          mov     esi, 81h

.text:00430CF9 8B D0                                   mov     edx, eax

.text:00430CFB 23 D6                                   and     edx, esi

.text:00430CFD 33 DB                                   xor     ebx, ebx

.text:00430CFF 3B D6                                   cmp     edx, esi

.text:00430D01 75 1F                                   jnz     short loc_430D22

.text:00430D03 85 DB                                   test    ebx, ebx

.text:00430D05 75 1B                                   jnz     short loc_430D22

.text:00430D07 0F AC C8 0C                             shrd    eax, ecx, 0Ch

.text:00430D0B C1 E9 0C                                shr     ecx, 0Ch

.text:00430D0E 8B CF                                   mov     ecx, edi

.text:00430D10 C1 E9 0C                                shr     ecx, 0Ch

.text:00430D13 25 FF FF FF 03                          and     eax, 3FFFFFFh

.text:00430D18 81 E1 FF 01 00 00                       and     ecx, 1FFh

.text:00430D1E 03 C1                                   add     eax, ecx

.text:00430D20 EB 3F                                   jmp     short loc_430D61

.text:00430D22                         ; ---------------------------------------------------------------------------

.text:00430D22

.text:00430D22                         loc_430D22:                             ; CODE XREF: MmGetPhysicalAddress(x)+2Fj

.text:00430D22                                                                 ; MmGetPhysicalAddress(x)+33j

.text:00430D22 83 E0 01                                and     eax, 1

.text:00430D25 33 C9                                   xor     ecx, ecx

.text:00430D27 0B C1                                   or      eax, ecx

.text:00430D29 74 24                                   jz      short loc_430D4F

.text:00430D2B 8B CF                                   mov     ecx, edi

.text:00430D2D C1 E9 09                                shr     ecx, 9

.text:00430D30 81 E1 F8 FF 7F 00                       and     ecx, 7FFFF8h

.text:00430D36 8B 91 04 00 00 C0                       mov     edx, [ecx+0C0000004h]

.text:00430D3C 81 E9 00 00 00 40                       sub     ecx, -0C0000000h

.text:00430D42 8B 01                                   mov     eax, [ecx]

.text:00430D44 8B C8                                   mov     ecx, eax

.text:00430D46 83 E1 01                                and     ecx, 1

.text:00430D49 33 F6                                   xor     esi, esi

.text:00430D4B 0B CE                                   or      ecx, esi

.text:00430D4D 75 06                                   jnz     short loc_430D55

.text:00430D4F

.text:00430D4F                         loc_430D4F:                             ; CODE XREF: MmGetPhysicalAddress(x)+57j

.text:00430D4F 33 C0                                   xor     eax, eax

.text:00430D51 33 D2                                   xor     edx, edx

.text:00430D53 EB 1F                                   jmp     short loc_430D74

.text:00430D55                         ; ---------------------------------------------------------------------------

.text:00430D55

.text:00430D55                         loc_430D55:                             ; CODE XREF: MmGetPhysicalAddress(x)+7Bj

.text:00430D55 0F AC D0 0C                             shrd    eax, edx, 0Ch

.text:00430D59 C1 EA 0C                                shr     edx, 0Ch

.text:00430D5C 25 FF FF FF 03                          and     eax, 3FFFFFFh

.text:00430D61

.text:00430D61                         loc_430D61:                             ; CODE XREF: MmGetPhysicalAddress(x)+4Ej

.text:00430D61 33 C9                                   xor     ecx, ecx

.text:00430D63 0F A4 C1 0C                             shld    ecx, eax, 0Ch

.text:00430D67 C1 E0 0C                                shl     eax, 0Ch

.text:00430D6A 81 E7 FF 0F 00 00                       and     edi, 0FFFh

.text:00430D70 03 C7                                   add     eax, edi

.text:00430D72 8B D1                                   mov     edx, ecx

.text:00430D74

.text:00430D74                         loc_430D74:                             ; CODE XREF: MmGetPhysicalAddress(x)+81j

.text:00430D74 5F                                      pop     edi

.text:00430D75 5E                                      pop     esi

.text:00430D76 5B                                      pop     ebx

.text:00430D77 5D                                      pop     ebp

.text:00430D78 C2 04 00                                retn    4

.text:00430D78                         _MmGetPhysicalAddress@4 endp



```

　　一看之下，有点晕。

　　没关系，有前面清楚的`PTE`结构指引，这个函数就比较容易理解了。

　　以下就是它的C++实现：

```


struct VirtAddr1

  {

  unsigned int PageOFF:12;

  unsigned int PageTable:20;

  };



struct VirtAddr2

  {

  unsigned int PageOFF:12;

  unsigned int PageTableIndex:9;

  unsigned int PageLargeIndex:11;

  };



struct VirtAddr3

  {

  unsigned int PageOFF:12;

  unsigned int PageTableIndex:9;

  unsigned int PageDirIndex:9;

  unsigned int PageDirPoint:2;

  };



MMPTE_HARDWARE* const PDE_BASE = (MMPTE_HARDWARE*)0xC0600000;  //页目录表

MMPTE_HARDWARE* const PTE_BASE = (MMPTE_HARDWARE*)0xC0000000;  //进程页表



PHYSICAL_ADDRESS GetPhysicalAddress(const unsigned long VirtualAddress)

  {

  const VirtAddr1* VA1 = (const VirtAddr1*)&VirtualAddress;

  const VirtAddr2* VA2 = (const VirtAddr2*)&VirtualAddress;

  MMPTE_HARDWARE PTE = *(PDE_BASE + VA2->PageLargeIndex);

  PHYSICAL_ADDRESS PA = {0,0};

  if(!(PTE.Valid))return PA;

  if(PTE.LargePage)

    {

    PTE.PageFrameNumber += VA2->PageTableIndex;

    }

  else

    {

    PTE = *(PTE_BASE + VA1->PageTable);

    if(!(PTE.Valid))return PA;

    }

  PTE.reserved1 = 0;

  PA.HighPart = PTE.HighPart;

  PA.LowPart = PTE.LowPart;

  ((VirtAddr1*)(&PA.LowPart))->PageOFF = VA1->PageOFF;

  return PA;

  }

```

　　到这里应该可以结束了，不过我又想看看win2003是不是有啥不一样呢？
