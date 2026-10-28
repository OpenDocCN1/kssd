
`80510ff2 81e900000040    sub     ecx,40000000h  ;pte=((VA>>12)<<3) &0x7FFFF8 +0xc0000000; (-0x40000000 means +0xc0000000)`

`80510ff8 8b11            mov     edx,dword ptr [ecx] ;PTE context`

`80510ffa 8945fc          mov     dword ptr [ebp-4],eax`

`80510ffd 53              push    ebx`

`80510ffe 8bc2            mov     eax,edx`

`80511000 33db            xor     ebx,ebx`

`80511002 83e001          and     eax,1 ; pte 页面存在标志 Present位`

`80511005 0bc3            or      eax,ebx`

`80511007 5b              pop     ebx`

`80511008 7427            je      nt!MmIsAddressValid+0x91 (80511031);无效位跳转`

`8051100a 23d7            and     edx,edi ;edx=pte,edi=80h,取Page size位，Page size位 = 1，表示LargePage`

`8051100c 6a00            push    0`

`8051100e 8955f8          mov     dword ptr [ebp-8],edx ;暂存`

`80511011 58              pop     eax`

`80511012 7421            je      nt!MmIsAddressValid+0x95 (80511035) ;not LargePage,valid`

`80511014 85c0            test    eax,eax                             ; LargePage`

`80511016 751d            jne     nt!MmIsAddressValid+0x95 (80511035)`

`80511018 23ce            and     ecx,esi  ;ecx=pte,esi=3FF8h 3FF8H = 11 1111 1111 1 000 ,表示取高11位 2M分页的情况`

`8051101a 8b89000060c0    mov     ecx,dword ptr [ecx-3FA00000h] ; 0 - 3FA00000 = c0600000h ;ecx = PDE Context`

`80511020 b881000000      mov     eax,81h ;判断Page size位和Present位`

`80511025 23c8            and     ecx,eax`

`80511027 33d2            xor     edx,edx`

`80511029 3bc8            cmp     ecx,eax`

`8051102b 7508            jne     nt!MmIsAddressValid+0x95 (80511035)`

`8051102d 85d2            test    edx,edx ;not valid`

`8051102f 7504            jne     nt!MmIsAddressValid+0x95 (80511035)`

`80511031 32c0            xor     al,al`

`80511033 eb02            jmp     nt!MmIsAddressValid+0x97 (80511037)`

`80511035 b001            mov     al,1`

`80511037 5f              pop     edi`

`80511038 5e              pop     esi`

`80511039 c9              leave`

`8051103a c20400          ret     4`

还原为c的代码为：

```c
BOOLEAN MmIsAddressValid ( PVOID VirtualAddress )
{
    BYTE PresentSign = 0x1;
        BYTE PageSizeSign = 0x80;
        BYTE PresentAndPageSizeSign = 0x81;
        PVOID lVirtualAddress;
        ULONG PDE,PDEContext;
        ULONG PTE,PTEContext;
        lVirtualAddress = VirtualAddress;
        PDE = (((ULONG)lVirtualAddress>>21)<<3) & 0x3FF8 + 0xC0600000;
        PDEContext = (ULONG)\*(PVOID)PDE;
        if(!(PDEContext & PresentSign))
                return FALSE;
        if(PDEContext & PageSizeSign)
                return TRUE;
    PTE = (((ULONG)lVirtualAddress>>12)<<3) & 0x7FFFF8 + 0xC0000000;
        PTEContext = (ULONG)\*(PVOID)PTE;
        if(!(PTEContext & PresentSign ))
                return FALSE;
        if(!(PTEContext & PageSizeSign))
                return TRUE;
        else
        {
         PDE = (ULONG)PTE & 0x3ff8  + 0xc0600000;
                 PDEContext = (ULONG)\*(PVOID)PDE;
                 if(PDEContext & PresentAndPageSizeSign)
                         return TRUE;
        }
    return FALSE;
}
```

**现在PAE模式，PDE,PTE地址的计算公式是：**

```
int   PDE = ((lVirtualAddress>>21)<<3) & 0x3FF8 + 0xC0600000;
int   PTE = ((lVirtualAddress>>12)<<3) & 0x7FFFF8 + 0xC0000000;
```

以上`VA`表示`virtual address`

看看本机`pte`和`pde`的基地址

```
lkd> !pte
               VA 00000000
PDE at 00000000C0600000    PTE at 00000000C0000000
contains 000000004D5C1067  contains 0000000000000000
pfn 4d5c1 ---DA--UWEV
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [15.JPG](javascript:void(0);)

  （41.79kb，4019次下载）

- [16.JPG](javascript:void(0);)

  （34.57kb，4011次下载）

- [17.JPG](javascript:void(0);)

  （31.91kb，4176次下载）

- [18.JPG](javascript:void(0);)

  （26.54kb，4013次下载）

- [19.JPG](javascript:void(0);)

  （58.93kb，4015次下载）

- [20.JPG](javascript:void(0);)

  （38.31kb，4001次下载）

- [21.JPG](javascript:void(0);)

  （52.14kb，3993次下载）

- [22.JPG](javascript:void(0);)

  （38.96kb，3976次下载）
