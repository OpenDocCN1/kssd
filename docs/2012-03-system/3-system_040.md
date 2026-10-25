
# 琐碎的发现，`ImageRvaToVa` 函数分析！

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-313.htm`](https://www.pediy.com/kssd/pediy09/pediy09-313.htm)

> ![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑") ![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑") ![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑") ![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

> 最近,写了东西的时候用到了一个函数感觉很不错!MS 就是好嘛!`ImageRvaToVa` 就是把物理地址转换为虚拟地址的函数!可是最近到网络上找了好长时间也没发现有多少人用它!也许是我 RP 问题,我想不会吧!

> ![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸") ![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸") ![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")

> 写代码的时候,忽然要用到虚拟地址转换物理地址的函数,我顺手在 VC 里输入了 `ImageVaToRva`,惯例嘛!怎么来的怎么回去!编译了好多遍就是没这个函数,不会吧!我觉得 MS 应该不会忘记写了的!怎么回的来,回不去了呢?找遍了 `IMAGEHLP.H` 这个文件也没找到,晕倒!开始发现 MS 真不是玩意,才想起来很多代码中有 `RvaToOffset` 或 `OffsetToRva` 自写的函数,真没想到 MS 真的没做,找了找发现非要拿区段信息才行,我看到很多朋友都说函数有问题,我也看了是乎有点问题,不相信!MS 它也就拿区段干活!

> 拿 OD 看了一下!傻了!MS 还真的是拿区段干活!在 `ImageRvaToVa` 中有 `ImageRvaToSection` 函数!

> 为了让我的 `ImageVaToRva` 重出江湖,就分析了一下:

```
//-------------------------------------------------------------------------------------------

76C67554  >  8BFF            mov     edi, edi
76C67556    55              push    ebp
76C67557    8BEC            mov     ebp, esp
76C67559    56              push    esi                     ;保存 ESI
76C6755A    8B75 14         mov     esi, dword ptr [ebp+14]    ;将区段结构指针放入 ESI
76C6755D    85F6            test    esi, esi                  ;如果 ESI 是否为 0
76C6755F    57              push    edi                     ;保存 EDI
76C67560    8B7D 10         mov     edi, dword ptr [ebp+10]    ;EDI=要转换值
76C67563    74 16           je      short 76C6757B              ;ESI 为 0,直接转换
76C67565    8B0E            mov     ecx, dword ptr [esi]       ;不为空,将结构地址放入 ECX
76C67567    85C9            test    ecx, ecx                  ;ECX 为空直接转换
76C67569    74 10           je      short 76C6757B
76C6756B    8B41 0C         mov     eax, dword ptr [ecx+C]
76C6756E    3BF8            cmp     edi, eax
76C67570    72 09           jb      short 76C6757B
76C67572    8B51 10         mov     edx, dword ptr [ecx+10]
76C67575    03D0            add     edx, eax
76C67577    3BFA            cmp     edi, edx
76C67579    72 0E           jb      short 76C67589
76C6757B    57              push    edi                     ;要转换值
76C6757C    FF75 0C         push    dword ptr [ebp+C]        ;BASE 基地址
76C6757F    FF75 08         push    dword ptr [ebp+8]        ;NT 结构指针
76C67582    E8 8AFFFFFF     call    ImageRvaToSection
76C67587    8BC8            mov     ecx, eax
76C67589    85C9            test    ecx, ecx                  ;如果为 0 就跳出去
76C6758B    74 13           je      short 76C675A0
76C6758D    85F6            test    esi, esi
76C6758F    74 02           je      short 76C67593
76C67591    890E            mov     dword ptr [esi], ecx        ;保存区段地址
76C67593    8B41 14         mov     eax, dword ptr [ecx+14]     ;EAX=物理地址
76C67596    2B41 0C         sub     eax, dword ptr [ecx+C]      ;EAX=物理地址-虚拟地址
76C67599    0345 0C         add     eax, dword ptr [ebp+C]      ;EAX=EAX+基地址 BASE
76C6759C    03C7            add     eax, edi                    ;EAX=EAX+要转换的值
76C6759E    EB 02           jmp     short 76C675A2
76C675A0    33C0            xor     eax, eax
76C675A2    5F              pop     edi
76C675A3    5E              pop     esi
76C675A4    5D              pop     ebp
76C675A5    C2 1000         retn    10
```

> `ImageRvaToSection` 如下:

```
//--------------------------------------------------------------------
76C67511  >  8BFF            mov     edi, edi
76C67513    55              push    ebp
76C67514    8BEC            mov     ebp, esp
76C67516    8B4D 08         mov     ecx, dword ptr [ebp+8]      ;NT 结构地址
76C67519    0FB741 14       movzx   eax, word ptr [ecx+14]      ;EAX=SizeOfOptionalHeader
76C6751D    8D4408 18       lea     eax, dword ptr [eax+ecx+18] ;EAX=区段开始地址
76C67521    0FB749 06       movzx   ecx, word ptr [ecx+6]       ;获得区段总数为 6
76C67525    56              push    esi
76C67526    33F6            xor     esi, esi
76C67528    85C9            test    ecx, ecx
76C6752A    57              push    edi
76C6752B    76 1A           jbe     short 76C67547              ;如果区段数小于等于 0,那么就跳出去
76C6752D    8B50 0C         mov     edx, dword ptr [eax+C]      ;EDX=区段虚拟地址
76C67530    3955 10         cmp     dword ptr [ebp+10], edx     ;与 RVA 比较如果大于就跳
76C67533    72 0A           jb      short 76C6753F
76C67535    8B78 10         mov     edi, dword ptr [eax+10]     ;不大于,将物理区段大小放入 EDI
76C67538    03FA            add     edi, edx                    ;EDI=物理区段大小+虚拟区段地址
76C6753A    397D 10         cmp     dword ptr [ebp+10], edi     ;EDI 是否大于 RVA 值,如果大于就跳出去
76C6753D    72 0A           jb      short 76C67549
76C6753F    83C0 28         add     eax, 28                     ;不大于计算下一区段
76C67542    46              inc     esi
76C67543    3BF1            cmp     esi, ecx
76C67545  ^ 72 E6           jb      short 76C6752D
76C67547    33C0            xor     eax, eax
76C67549    5F              pop     edi
76C6754A    5E              pop     esi
76C6754B    5D              pop     ebp
76C6754C    C2 0C00         retn    0C
```

> 大致分析了,如上,我自己效仿了个函数,把虚拟地址转换为物理地址的:

```c
LPVOID CPNExeInfo::ImageVaToRva(PIMAGE_NT_HEADERS _nt, LPVOID Base, ULONG va)
{
    DWORD dwVa = va;
    PIMAGE_SECTION_HEADER begin = (PIMAGE_SECTION_HEADER)((DWORD)_nt + _nt->FileHeader.SizeOfOptionalHeader + 0x18);

    for(int i = 0; i < _nt->FileHeader.NumberOfSections; i++, begin++)
    {
        DWORD dwSectionVa = begin->PointerToRawData + (DWORD)Base;
        DWORD dwSectionSize = begin->SizeOfRawData;

        if(dwVa >= dwSectionVa && dwVa < (dwSectionSize + dwSectionVa))
        {
            dwVa = dwVa - (DWORD)Base;
            dwVa = dwVa + begin->VirtualAddress;
            dwVa = dwVa - begin->PointerToRawData;
            return (LPVOID)dwVa;
        } else if(dwVa < dwSectionVa)
        {
            //如果计算值大于前一区段的最大值，而后一区段地址大于计算值
            //则假设在前一区段:
            dwVa = dwVa - (DWORD)Base;
            begin--;
            dwVa = dwVa + begin->VirtualAddress;
            dwVa = dwVa - begin->PointerToRawData;
        }
    }
    return NULL;
}
```

> 有什么不对的地方大家给意见一下!来给大家解解闷!...........

> 叫法容易引起误会,`RVA` 为相对虚拟地址,`VA` 为虚拟地址，
> 如果是 `RVA2VA`，直接 `RVA + ImgBase` 就行了。
> 看意思应该是 `Rva2FilePos` 吧(`FilePos` 即为文件偏移)。
> ms 提供这个函数有 bug 的，以前测试某些壳时出现过(不记得是 FSG 还是什么)。
>
> 区段是可以乱序的
>
> 可以不是从小到大排列
>
> 虚拟地址转换物理地址，叫法好像有点不妥吧，咋看还以为是内存管理里提到的物理地址……![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心")

`4dd466d0e1e0cc123afbe504da2462ca`
