
# [原创]扩展任意节大小

> 原文：[`www.pediy.com/kssd/pediy10/84932.html`](https://www.pediy.com/kssd/pediy10/84932.html)

如题。

很久以前就写这么个东西。

然后一直懒觉得费力不讨好就没写。

上个星期?上上个星期开始写。写了两个下午就没劲了。

然后一直到昨天断断续续的写完了..

代码。

额- -~

就是修正一些地址什么的吧.~

好像没什么好讲的 哈哈

C++语言: `ExpandSection`

```cpp
BOOL CPatch::ExpandSection( PBYTE pImage, int ExpandSize, int SectionID)
{
    if ( !InitPeHelp( pImage))
    {
        return false;
    }

    int iExpandVsize;    //对齐后需要扩展的 Vsize
    int iExpandRsize;    //对齐后需要扩展的 Rsize
    PBYTE pNewImageBase;    //指向扩展后的文件镜像
    int iNewImageSize;    //新文件的 ImageSize

    iExpandVsize = ( ( ExpandSize > pNtHeader->OptionalHeader.SectionAlignment ? ExpandSize/pNtHeader->OptionalHeader.SectionAlignment : 0 )
        + ( ExpandSize % pNtHeader->OptionalHeader.SectionAlignment != 0 ? 1 : 0) ) * pNtHeader->OptionalHeader.SectionAlignment;
    iExpandVsize += iExpandVsize == 0 ? pNtHeader->OptionalHeader.SectionAlignment : 0;

    iExpandRsize = ( ( ExpandSize > pNtHeader->OptionalHeader.FileAlignment ? ExpandSize/pNtHeader->OptionalHeader.FileAlignment : 0)
        + ( ExpandSize % pNtHeader->OptionalHeader.FileAlignment !=0 ? 1 : 0) ) * pNtHeader->OptionalHeader.FileAlignment;
    iExpandRsize += iExpandRsize == 0? pNtHeader->OptionalHeader.FileAlignment : 0;

    printf( "\r\niExpandVsize:%08X,iExpandRsize:%08X\r\n", iExpandVsize, iExpandVsize);

    iNewImageSize = pNtHeader->OptionalHeader.SizeOfImage  + iExpandVsize;
    pNewImageBase = (PBYTE)VirtualAlloc( NULL, iNewImageSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    memcpy( pNewImageBase, pImageBase, pNtHeader->OptionalHeader.SizeOfHeaders );    //Copy 文件头

    //调整新文件头指针
    PIMAGE_DOS_HEADER pNewDosHeader = (PIMAGE_DOS_HEADER)pNewImageBase;
    printf( "pNewDosHeader:%p\r\n", pNewDosHeader);

    PIMAGE_NT_HEADERS pNewNtHeader = (PIMAGE_NT_HEADERS)(pNewImageBase + pNewDosHeader->e_lfanew);
    printf( "pNewNtHeader:%p\r\n", pNewNtHeader);

    PIMAGE_SECTION_HEADER pNewSectionHeader = (PIMAGE_SECTION_HEADER)((PBYTE)pNewNtHeader + sizeof(IMAGE_NT_HEADERS));
    printf( "pNewSectionHeader:%p\r\n", pNewSectionHeader);

    PIMAGE_SECTION_HEADER pTheAddSectionHeader = pNewSectionHeader + SectionID - 1;
    int iTheBreakRVA = pTheAddSectionHeader->VirtualAddress + pTheAddSectionHeader->Misc.VirtualSize;    //在这个 RVA 后添加
    int iTheBreakOffset = pTheAddSectionHeader->PointerToRawData + pTheAddSectionHeader->SizeOfRawData;    //在这个 Offset 后添加

    //所有在 BreakPoint 后面的参数(偏移 or RVA)都要加上 iExpandVSize or iExpandRSize
    pNewNtHeader->FileHeader.PointerToSymbolTable += pNewNtHeader->FileHeader.PointerToSymbolTable >= iTheBreakRVA
        ? iExpandVsize : 0;    // 指向调试符号,RVA
    pNewNtHeader->OptionalHeader.AddressOfEntryPoint += pNewNtHeader->OptionalHeader.AddressOfEntryPoint >= iTheBreakRVA
        ? iExpandVsize : 0;    //指向文件入口点,RVA
    pNewNtHeader->OptionalHeader.BaseOfCode += pNewNtHeader->OptionalHeader.BaseOfCode >= iTheBreakRVA
        ? iExpandVsize : 0;    //代码基址,RVA
    pNewNtHeader->OptionalHeader.BaseOfData += pNewNtHeader->OptionalHeader.BaseOfData >= iTheBreakRVA
        ? iExpandVsize : 0; //数据基址,RVA
    pNewNtHeader->OptionalHeader.SizeOfImage += iExpandVsize;    //镜像大小 ,得加
    //pNtHeader->OptionalHeader.SizeOfCode 不知道咋搞,不管他, 反正不管他也没事
    //这个先不管, 到后面再去修改得了...
    //for ( int i = 0; i < 16; i++)    //DataDirectory ,RVA, 一共有 16 项
    //{
    [I]//    pNewNtHeader->OptionalHeader.DataDirectory[i].VirtualAddress += pNewNtHeader->OptionalHeader.DataDirectory[i].VirtualAddress > iTheBreakRVA && pNewNtHeader->OptionalHeader.DataDirectory*.VirtualAddress != 0
    //        ? iExpandVsize : 0;
    //}

    //修改第 SectionID 个节表的参数(Vsize 和 RSize)
    pTheAddSectionHeader->SizeOfRawData += iExpandRsize;
    pTheAddSectionHeader->Misc.VirtualSize += iExpandVsize;

    //修改第 SectionID 后的节表.
    for ( int i = SectionID; i < pNewNtHeader->FileHeader.NumberOfSections; i++)
    {
        PIMAGE_SECTION_HEADER pNowSecHeader = pNewSectionHeader + i;
        pNowSecHeader->PointerToLinenumbers += pNowSecHeader->PointerToLinenumbers > 0
            ? iExpandVsize : 0;    //这个不确定是 RVA 还是 Roffset 估摸着是 RVA 吧
        pNowSecHeader->PointerToRawData += pNowSecHeader->PointerToRawData > 0
            ? iExpandRsize : 0;    //Offset
        pNowSecHeader->PointerToRelocations += pNowSecHeader->PointerToRelocations > 0
            ? iExpandVsize : 0;    //不确定 , 猜 RVA
        pNowSecHeader->VirtualAddress += pNowSecHeader->VirtualAddress > 0
            ? iExpandVsize : 0;    //RVA
    }

    //Copy 各个节
    for ( int i = 0; i < pNewNtHeader->FileHeader.NumberOfSections; i++)
    {
        PIMAGE_SECTION_HEADER pNowSecHeader = pNewSectionHeader + i;
        PIMAGE_SECTION_HEADER pSrcSecHeader = pSectionHeader + i;
        memcpy( pNowSecHeader->VirtualAddress + pNewImageBase, pSrcSecHeader->VirtualAddress + pImageBase, pSrcSecHeader->SizeOfRawData);
    }

    //EAT
    if ( pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress )
    {
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress +=
            pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress >= iTheBreakRVA
            ? iExpandVsize : 0;
        PIMAGE_EXPORT_DIRECTORY pNewExportDirectory = (PIMAGE_EXPORT_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);
        pNewExportDirectory->AddressOfFunctions +=
            pNewExportDirectory->AddressOfFunctions >= iTheBreakRVA
            ? iExpandVsize : 0;
        pNewExportDirectory->AddressOfNameOrdinals +=
            pNewExportDirectory->AddressOfNameOrdinals >= iTheBreakRVA
            ? iExpandVsize : 0;
        pNewExportDirectory->AddressOfNames +=
            pNewExportDirectory->AddressOfNames >= iTheBreakRVA
            ? iExpandVsize : 0;
        pNewExportDirectory->Base +=
            pNewExportDirectory->Base >= iTheBreakRVA
            ? iExpandVsize : 0;

        PDWORD pNames = PDWORD( pNewImageBase + pNewExportDirectory->AddressOfNames);
        for ( int i = 0; i < pNewExportDirectory->NumberOfNames; i++)
        {
            //__asm int 3;
            pNames[i] += pNames[i] >= iTheBreakRVA ? iExpandVsize : 0;
            [I]//printf( "\r\nExportAPI \r\nName :%s \r\nRVA %X\r\n", pNames[i] + pNewImageBase, pNames*);
        }

        PDWORD pFuntions = PDWORD( pNewImageBase + pNewExportDirectory->AddressOfFunctions);
        for ( int i = 0; i < pNewExportDirectory->NumberOfFunctions; i++)
        {
            pFuntions[i] += pFuntions[i] >= iTheBreakRVA ? iExpandVsize : 0;
            [I]//printf( "\r\nExprtAPI RVA: %X\r\niTheBreakRVA:%08X\r\n", pFuntions*, iTheBreakRVA );
        }
    }

    //Import Table
    if ( pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress)
    {
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress +=
            pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress >= iTheBreakRVA
            ? iExpandVsize : 0;
```



```
PIMAGE_IMPORT_DESCRIPTOR pNewImpDesciptor = (PIMAGE_IMPORT_DESCRIPTOR)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);
while (pNewImpDesciptor->FirstThunk)
{
    pNewImpDesciptor->FirstThunk += pNewImpDesciptor->FirstThunk >= iTheBreakRVA ? iExpandVsize : 0;
    pNewImpDesciptor->Name += pNewImpDesciptor->Name >= iTheBreakRVA ? iExpandVsize : 0;
    pNewImpDesciptor->OriginalFirstThunk += pNewImpDesciptor->OriginalFirstThunk >= iTheBreakRVA ? iExpandVsize : 0;
    //printf("Import Dll Name : %s\r\n", pNewImageBase + pNewImpDesciptor->Name);
    PIMAGE_THUNK_DATA pOriginalThunk = (PIMAGE_THUNK_DATA)(pNewImageBase + pNewImpDesciptor->OriginalFirstThunk);
    while (pOriginalThunk->u1.Function)
    {
        pOriginalThunk->u1.AddressOfData += pOriginalThunk->u1.AddressOfData >= iTheBreakRVA ? iExpandVsize : 0;
        //printf("\r\nImportAPI Name : %s", pNewImageBase + pOriginalThunk->u1.AddressOfData + 2);
        pOriginalThunk++;
    }
    //这个应该不要填充~ 这里由系统填充的
    //PIMAGE_THUNK_DATA pThunk = (PIMAGE_THUNK_DATA)(pNewImageBase + pNewImpDesciptor->FirstThunk);
    //while (pThunk->u1.Function)
    //{
    //    __asm int 3;
    //    pThunk->u1.Function += pThunk->u1.Function >= iTheBreakRVA ? iTheBreakRVA : 0;
    //    pThunk++;
    //}
    pNewImpDesciptor++;
}
}

//资源
if (pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_RESOURCE].VirtualAddress)
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_RESOURCE].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_RESOURCE].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
    PIMAGE_RESOURCE_DIRECTORY pTopResDir = (PIMAGE_RESOURCE_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_RESOURCE].VirtualAddress);
    MoveTheResource((PBYTE)pTopResDir, pTopResDir, iTheBreakRVA, iExpandVsize);
}

//IMAGE_DIRECTORY_ENTRY_SECURITY 异常表?不处理吧.貌似...
//IMAGE_DIRECTORY_ENTRY_SECURITY.貌似是绑定证书 事实上改了也没用 0 0
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_SECURITY].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_SECURITY].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
}

//调试目录
if (pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DEBUG].VirtualAddress)
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DEBUG].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DEBUG].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
    int nCount = pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DEBUG].Size / sizeof(IMAGE_DEBUG_DIRECTORY);
    PIMAGE_DEBUG_DIRECTORY pDbgDir = (PIMAGE_DEBUG_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DEBUG].VirtualAddress);
    while (nCount--)
    {
        pDbgDir->AddressOfRawData += pDbgDir->AddressOfRawData >= iTheBreakRVA ? iExpandVsize : 0;
        pDbgDir->PointerToRawData += pDbgDir->PointerToRawData >= iTheBreakOffset ? iExpandRsize : 0;
    }
}

//IMAGE_DIRECTORY_ENTRY_ARCHITECTURE //不晓得
//IMAGE_DIRECTORY_ENTRY_GLOBALPTR 全局指针
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_GLOBALPTR].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_GLOBALPTR].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
}

//IMAGE_DIRECTORY_ENTRY_TLS tls table
//就这样吧~没有看到适合测试这个的文件,貌似 Delphi 写的东西.
// - -! DLL 貌似没有 TLS? 哦 是 LoadLibrary 跟 __declspec(thread)不和谐
//反正懒得管了
if (pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress)
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
    PIMAGE_TLS_DIRECTORY pTlsDir = (PIMAGE_TLS_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress);
    pTlsDir->AddressOfCallBacks += pTlsDir->AddressOfCallBacks >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;
    pTlsDir->AddressOfIndex += pTlsDir->AddressOfIndex >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;
    pTlsDir->StartAddressOfRawData += pTlsDir->StartAddressOfRawData >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;
    pTlsDir->EndAddressOfRawData += pTlsDir->EndAddressOfRawData >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;
    PDWORD pCallBacks = (PDWORD)(pNewImageBase + pTlsDir->AddressOfCallBacks - pNtHeader->OptionalHeader.ImageBase); // - -! 突然来个什么 Va.
    for (int i = 0; pCallBacks[i]; i++)
    {
        pCallBacks[i] += pCallBacks[i] >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0; //这里应该也是 VA 吧~不过懒得管他了
    }
    PDWORD pAddressIndexs = (PDWORD)(pNewImageBase + pTlsDir->AddressOfIndex - pNtHeader->OptionalHeader.ImageBase); //VAVAVA~~~~~~
    for (int i = 0; pAddressIndexs[i]; i++)
    {
        pAddressIndexs[i] += pAddressIndexs[i] >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0; //这里应该也是 VA 吧~不过懒得管他了
    }
}

//IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG //这个结构里面没有地址.
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
}

//IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT //这个东西还算好搞吧
if (pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress)
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
    //PIMAGE_BOUND_IMPORT_DESCRIPTOR
    //这个里面的地址都是相对 pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress 的地址,所以..高级编译器编译的不用管
}

//IMAGE_DIRECTORY_ENTRY_IAT
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IAT].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IAT].VirtualAddress >= iTheBreakRVA ? iExpandVsize : 0;
}

//IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT
//反正我不懂这个结构.
```



### 延时加载目录 (`Delay Import Directory`)

```c
/**
    ->Delay Import Directory
    1. DelayImportDescriptor:
    grAttrs:          0x00000001  (dlattrRva)
    DLLName (R)VA:    0x00066160  ("MSIMG32.dll")
    Hmod (R)VA:       0x00069CE0
    IAT (R)VA:        0x00069000
    INT (R)VA:        0x0006618C
    BoundIAT (R)VA:   0x00000000
    UnloadIAT (R)VA:  0x00000000
    TimeDateStamp:    0x00000000  (GMT: Thu Jan 01 00:00:00 1970)
    */
```

定义结构体：

```c
typedef struct _IMAGE_DELAY_IMPORT_DESCRIPTOR {
    DWORD grAttrs;
    DWORD DLLName;
    DWORD Hmod;
    DWORD IAT;
    DWORD INT;
    DWORD BoundIAT;
    DWORD UnloadIAT;
    DWORD TimeDateStamp;
} IMAGE_DELAY_IMPORT_DESCRIPTOR, *PIMAGE_DELAY_IMPORT_DESCRIPTOR;
```

处理逻辑：

```c
if ( pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT].VirtualAddress )
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT].VirtualAddress >= iTheBreakRVA
        ? iExpandVsize : 0;
    PIMAGE_DELAY_IMPORT_DESCRIPTOR pDelayDes = (PIMAGE_DELAY_IMPORT_DESCRIPTOR)( pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT].VirtualAddress);
    int iTheBreakVA = iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase;
    int nCount = pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT].Size / sizeof(IMAGE_DELAY_IMPORT_DESCRIPTOR);
    while ( nCount--)
    {
        pDelayDes[nCount].BoundIAT += pDelayDes[nCount].BoundIAT >= iTheBreakVA ? iExpandVsize : 0;
        pDelayDes[nCount].DLLName += pDelayDes[nCount].DLLName >= iTheBreakVA ? iExpandVsize : 0;
        pDelayDes[nCount].Hmod += pDelayDes[nCount].Hmod >= iTheBreakVA ? iExpandVsize : 0;    //!!
        pDelayDes[nCount].IAT += pDelayDes[nCount].IAT >= iTheBreakVA ? iExpandVsize : 0;
        pDelayDes[nCount].INT += pDelayDes[nCount].INT >= iTheBreakVA ? iExpandVsize : 0;
        pDelayDes[nCount].UnloadIAT += pDelayDes[nCount].UnloadIAT >= iTheBreakVA ? iExpandVsize : 0;
    }
}
```

#### COM

```c
//COM
//IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR
//
if ( pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].VirtualAddress )
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].VirtualAddress >= iTheBreakRVA
        ? iExpandVsize : 0;
    //日了..不支持 COM 了 !
}
```

#### 重定位

```c
// - -! 终于完了~~
// 我晕..我真是白痴..还有重定位..
if ( !pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress )    //如果没有重定位表,就无法修正代码里的绝对引用 - -~
{
    goto Failed;
}

//IMAGE_DIRECTORY_ENTRY_BASERELOC
{
    pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress +=
        pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress >= iTheBreakRVA
        ? iExpandVsize : 0;
    PIMAGE_BASE_RELOCATION pBaseRelocal = (PIMAGE_BASE_RELOCATION)( pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);
    int iCount = pNewNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size / sizeof(IMAGE_BASE_RELOCATION);
    while ( pBaseRelocal->VirtualAddress && iCount)
    {
        typedef struct {
            WORD Offset:12;
            WORD Type:4;
        } WORD_RELOCAL, *PWORD_RELOCAL;
        if ( pBaseRelocal->VirtualAddress >= iTheBreakRVA)
        {
            pBaseRelocal->VirtualAddress += iExpandVsize;
            PWORD_RELOCAL pRelocalWord = (PWORD_RELOCAL)((PBYTE)pBaseRelocal + sizeof(IMAGE_BASE_RELOCATION));
            for ( int i = 0; i < pBaseRelocal->SizeOfBlock / sizeof(WORD_RELOCAL); i++)
            {
                *(PDWORD)(pNewImageBase + pBaseRelocal->VirtualAddress + pRelocalWord[i].Offset) +=
                    pRelocalWord[i].Type == IMAGE_REL_BASED_HIGHLOW && pRelocalWord[i].Offset
                    ? iExpandVsize : 0;
            }
        }
        pBaseRelocal = (PIMAGE_BASE_RELOCATION)( (PBYTE)pBaseRelocal + pBaseRelocal->SizeOfBlock);
    }
}

return true;

Failed:
    //释放内存啥的 - -~
    return false;
}
```
