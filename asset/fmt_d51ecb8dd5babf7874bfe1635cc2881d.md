
`80510fe6 81e1f8ff7f00    and     ecx,7FFFF8h ;7FFFF8h = 0111 1111 1111 1111 1111 1 000,20 位,第 31 到第 12 位,即前面 20 位`

`80510fec 8b81040000c0    mov     eax,dword ptr [ecx-3FFFFFFCh];  `

`80510ff2 81e900000040    sub     ecx,40000000h  ;pte=((VA>>12)<<3) &0x7FFFF8 +0xc0000000; (-0x40000000 means +0xc0000000)  `

`80510ff8 8b11            mov     edx,dword ptr [ecx] ;PTE context`

`80510ffa 8945fc         mov     dword ptr [ebp-4],eax`

`80510ffd 53             push    ebx`

`80510ffe 8bc2           mov     eax,edx`

`80511000 33db          xor     ebx,ebx`

`80511002 83e001        and     eax,1 ; pte 页面存在标志 Present 位`

`80511005 0bc3          or      eax,ebx`

`80511007 5b            pop     ebx`

`80511008 7427          je      nt!MmIsAddressValid+0x91 (80511031);无效位跳转`

`8051100a 23d7          and     edx,edi ;edx=pte,edi=80h,取 Page size 位，Page size 位 = 1，表示 LargePage`

`8051100c 6a00          push    0`

`8051100e 8955f8        mov     dword ptr [ebp-8],edx ;暂存`

`80511011 58            pop     eax`

`80511012 7421          je      nt!MmIsAddressValid+0x95 (80511035) ;not LargePage,valid`

`80511014 85c0          test    eax,eax                         ; LargePage`

`80511016 751d          jne     nt!MmIsAddressValid+0x95 (80511035)`

`80511018 23ce          and     ecx,esi  ;ecx=pte,esi=3FF8h 3FF8H = 11 1111 1111 1 000 ,表示取高 11 位 2M 分页的情况`

`8051101a 8b89000060c0  mov     ecx,dword ptr [ecx-3FA00000h] ; 0 - 3FA00000 = c0600000h ;ecx = PDE Context`

`80511020 b881000000    mov     eax,81h ;判断 Page size 位和 Present 位`

`80511025 23c8          and     ecx,eax`

`80511027 33d2          xor     edx,edx`

`80511029 3bc8          cmp     ecx,eax     `

`8051102b 7508          jne     nt!MmIsAddressValid+0x95 (80511035)`

`8051102d 85d2          test    edx,edx ;not valid`

`8051102f 7504          jne     nt!MmIsAddressValid+0x95 (80511035)`

`80511031 32c0          xor     al,al`

`80511033 eb02          jmp     nt!MmIsAddressValid+0x97 (80511037)`

`80511035 b001          mov     al,1`

`80511037 5f            pop     edi`

`80511038 5e            pop     esi`

`80511039 c9            leave`

`8051103a c20400        ret     4`

还原为 c 的代码为：

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

  PDEContext = (ULONG)*(PVOID)PDE;

  if(!(PDEContext & PresentSign))

    return FALSE;

  if(PDEContext & PageSizeSign)

    return TRUE;

    PTE = (((ULONG)lVirtualAddress>>12)<<3) & 0x7FFFF8 + 0xC0000000;

  PTEContext = (ULONG)*(PVOID)PTE;

  if(!(PTEContext & PresentSign ))

    return FALSE;

  if(!(PTEContext & PageSizeSign))

    return TRUE;

  else

  {

         PDE = (ULONG)PTE & 0x3ff8  + 0xc0600000;

     PDEContext = (ULONG)*(PVOID)PDE;

     if(PDEContext & PresentAndPageSizeSign)

       return TRUE;

  }

    return FALSE;

}
```

现在 PAE 模式，`PDE`, `PTE` 地址的计算公式是：

`int PDE = ((lVirtualAddress>>21)<<3) & 0x3FF8 + 0xC0600000;`

`int PTE = ((lVirtualAddress>>12)<<3) & 0x7FFFF8 + 0xC0000000;`

以上 `VA` 表示 `virtual address`

看看本机 `pte` 和 `pde` 的基地址

`lkd> !pte`

```
               VA 00000000
PDE at 00000000C0600000    PTE at 00000000C0000000
contains 000000004D5C1067  contains 0000000000000000
pfn 4d5c1 ---DA--UWEV
```
