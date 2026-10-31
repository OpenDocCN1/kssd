
# [原创]搜索输出表取得`GetProcAddress`地址C++版

看到某些病毒用到的技术从`Kernel32.dll`中取`GetProcAddress`地址，想半天没想明白，为什么要里面有一个`*4`呢？原来是`DWORD`长度的指针偏移，我晕！![](https://bbs.kanxue.com/view/img/face/1.gif)

自己用C++实现一下，加深一下理解，同时方便那些还有些不明白的同学，嘿嘿![](https://bbs.kanxue.com/view/img/face/12.gif)

这里直接用`GetModuleHandle`获取，方便的话也能直接从`PEB`中取得，但这里主要是了解病毒中取`GetProcAddress`地址的原理，所以直接偷懒了（下面是原码及运行结果）：

```
#include "stdafx.h"

#include <windows.h>

void main(){

        PIMAGE_DOS_HEADER pDosHeader;
        PIMAGE_NT_HEADERS pNtHeader;
        PIMAGE_EXPORT_DIRECTORY pExportDirectory;

        HMODULE hMod = GetModuleHandle("kernel32.dll");

        pDosHeader = (PIMAGE_DOS_HEADER)hMod;
        pNtHeader = (PIMAGE_NT_HEADERS)((PBYTE)hMod + pDosHeader->e_lfanew);
        pExportDirectory = PIMAGE_EXPORT_DIRECTORY(pNtHeader->OptionalHeader.DataDirectory[0].VirtualAddress + (PBYTE)hMod);

        PDWORD pAddressName = PDWORD((PBYTE)hMod+pExportDirectory->AddressOfNames); //函数名称表指针
        PWORD pAddressOfNameOrdinals = (PWORD)((PBYTE)hMod+pExportDirectory->AddressOfNameOrdinals); //函数名称序号表指针
        PDWORD pAddresOfFunction = (PDWORD)((PBYTE)hMod+pExportDirectory->AddressOfFunctions);  //函数地址表指针

        for (DWORD i = 0; i < pExportDirectory->NumberOfNames; i ++){

                PCHAR pFunc = (PCHAR)((PBYTE)hMod + *pAddressName++);
                printf("%s\r\n", pFunc);

                if (0 == strcmp(pFunc, "GetProcAddress"))
                        break;

                pAddressOfNameOrdinals++;  //ENT和函数名序号数组两个并行数组同时滑动指针(序号数组中的序号就对应函数名对应的函数地址的数组索引)
        }

        printf("function ordinal is: %02x and address is %08x ! \r\n",
                pExportDirectory->Base + *pAddressOfNameOrdinals,   //函数的序号＝函数名序号数组中的值+Base
                (PBYTE)hMod + pAddresOfFunction[*pAddressOfNameOrdinals]); //函数地址＝EAT[序号表中的值]

        DWORD x = (DWORD)GetProcAddress;  //直接获取的地址
        printf("directly aquire address is %08x\r\n", x);
}
```

![](https://bbs.kanxue.com/upload/attach/201003/89687_crjp778oijz6qe6.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`a.jpg`](javascript:void(0);)
  （71.72kb，311次下载）
