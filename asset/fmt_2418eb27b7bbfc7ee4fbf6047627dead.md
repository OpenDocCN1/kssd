
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
