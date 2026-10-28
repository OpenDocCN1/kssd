
```c
DWORD Convert(wchar_t* lpBase, DWORD dwAddr, BOOL bFile2RVA)
{
    DWORD dwRet = -1;
    //读取该文件的信息（文件内存对齐方式以及区块数量，并将区块表指针指向区块表第一个区块头）
    PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)lpBase;
    //pDosHeader->e_lfanew，LONG AddressOfNewExeHeader定位到4550的50
    PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)((unsigned long)lpBase + pDosHeader->e_lfanew);
    //转换为long是操作实际物理地址值,再从unsigned long转换为指针
    //printf("%x\n",*pNtHeader);//显示此时是不是4550
    //printf("NumberOfSection:%x\n",*(pNtHeader+4));

    DWORD dwMemAlign = pNtHeader->OptionalHeader.SectionAlignment;
    DWORD dwFileAlign = pNtHeader->OptionalHeader.FileAlignment;
    int dwSecNum = pNtHeader->FileHeader.NumberOfSections;

    PIMAGE_SECTION_HEADER pSecHeader = (PIMAGE_SECTION_HEADER)((char*)lpBase + pDosHeader->e_lfanew + sizeof(IMAGE_NT_HEADERS));
    //用同样的方法定义到了段头部
    DWORD dwHeaderSize = 0;

    if (!bFile2RVA) // 内存偏移转换为文件偏移
    {
        //看需要转移的偏移是否在PE头内，如果在则两个偏移相同
        dwHeaderSize = pNtHeader->OptionalHeader.SizeOfHeaders;//0x400
        if (dwAddr <= dwHeaderSize)//0x400
        {
            delete lpBase;
            lpBase = NULL;
            return dwAddr;
        }
        else //不在PE头里，查看该地址在哪个区块中
        {
            for (int i = 0; i < dwSecNum; i++)//0x7200十进制29184
            {
                DWORD dwSecSize = pSecHeader[i].Misc.VirtualSize;
                if ((dwAddr >= pSecHeader[i].VirtualAddress) && (dwAddr <= pSecHeader[i].VirtualAddress + dwSecSize))
                    //文件偏移 = 该区块的文件偏移 + （该偏移 - 该区块的内存偏移）
                    dwRet = pSecHeader[i].PointerToRawData + dwAddr - pSecHeader[i].VirtualAddress;
            }
        }
    }
    else // 文件偏移转换为内存偏移
    {
        dwHeaderSize = pNtHeader->OptionalHeader.SizeOfHeaders;
        //看需要转移的偏移是否在PE头内，如果在则两个偏移相同
        if (dwAddr <= dwHeaderSize)
        {
            delete lpBase;
            lpBase = NULL;
            return dwAddr;
        }
        else//不在PE头里，查看该地址在哪个区块中
        {
            for (int i = 0; i < dwSecNum; i++)
            {
                DWORD dwSecSize = pSecHeader[i].Misc.VirtualSize;
                if ((dwAddr >= pSecHeader[i].PointerToRawData) && (dwAddr <= pSecHeader[i].PointerToRawData + dwSecSize))
                    //内存偏移 = 该区块的内存偏移 + （该偏移 - 该区块的文件偏移）
                    dwRet = pSecHeader[i].VirtualAddress + dwAddr - pSecHeader[i].PointerToRawData;
            }
        }
    }
    //delete lpBase;
    //lpBase = NULL;
    return dwRet;
}
```

首先对于感染型病毒，我们要弄清楚的几点：

1. 感染哪种类型的文件（exe?dll?sys?com?）
2. 感染GUI还是CUI？（这个问题和1一样，只是容易被忽略）
3. 入口点有没有被篡改？假如被篡改如何找到入口点？
4. 附加恶意代码数据有几种方式？

带着这几种问题进行动态调试可以事半功倍。

## Virut本变种感染方式

1. 感染`.rsrc`区段并在尾部追加病毒代码
2. 修改入口点到资源段的病毒代码处

## 感染型病毒修复流程

1. 定位被感染区段，找到入口点地址。
2. 入口点修复（真实的入口点可能需要执行一段病毒代码才出现，具体入口点地址何时暴露，需要动态调试分析）
3. 删除附加的病毒代码数据
4. PE信息修复（包括被感染区段大小，区段数量（一般减1），`SizeOfImage`）

```c
if (((PIMAGE_DOS_HEADER)pFile)->e_magic != IMAGE_DOS_SIGNATURE)
    return -1;//不是DOS头，返回
DWORD dwNewPos = (DWORD)pFile + ((PIMAGE_DOS_HEADER)pFile)->e_lfanew;
PIMAGE_NT_HEADERS32 pNTHeader = (PIMAGE_NT_HEADERS32)(dwNewPos);
if (pNTHeader->Signature != IMAGE_NT_SIGNATURE)
    return -1;//不是NT头，说明不是PE文件，返回
PIMAGE_FILE_HEADER pFileHeader = &(pNTHeader->FileHeader);
PIMAGE_OPTIONAL_HEADER32 pOptionalHeader = &(pNTHeader->OptionalHeader);
PIMAGE_SECTION_HEADER pSectionHeader = IMAGE_FIRST_SECTION(pNTHeader);
```

首先解析PE结构，拿到各个信息点的指针，定位到区段信息指针`PIMAGE_SECTION_HEADER`。具体PE结构的知识这里不再说。

```c
while (j < pFileHeader->NumberOfSections)
{
    //pSectionHeader = pSectionHeader+pFileHeader->NumberOfSections-1;//倒着输出
    WCHAR Tag[8] = { 0 };//区段名字是个 BYTE [8]类名的字符串，需要转换成一个宽字符
    MultiByteToWideChar(CP_ACP, 0, (LPCCH)pSectionHeader->Name, 8, Tag, 8);//转换为宽字符存储在Tag数组中
    DWORD buffer1 = (DWORD)pSectionHeader - (DWORD)pFile;//!!!找到感染区段的物理偏移
    if (!strcmp(".rsrc", (LPCCH)pSectionHeader->Name))
    {
        RSRCSIZE = pSectionHeader->SizeOfRawData;
        RSRC = pSectionHeader->PointerToRawData;

        //还需要获取Misc中的虚拟地址大小
    }
}
```

遍历区段信息表，找到被感染的`.rsrc`资源段，为后续抹除掉`.rsrc`段尾部数据做好准备。但在抹除掉附加数据之前，必须先在病毒代码里找到被修改的入口点地址。

```c
SetFilePointer(hFile, FOAEntry, 0, FILE_BEGIN);
//从入口点处向后遍历1000字节寻找81 C5  add esp,xxx
ReadFile(hFile, pEntry, BYTELENGTH, &haveread, NULL);

for (i = 0; i < BYTELENGTH; i++)
{
    if (pEntry[i] == 0xE8)
    {
        printf("\n第一个call:0x%x", i + 5);
        break;
    }
}
RealEntry = i + 5 + pOptionalHeader->AddressOfEntryPoint;
//实际计算时必须用RVA地址！！！！！！！！！！！！

SetFilePointer(hFile, RSRC, 0, FILE_BEGIN);
ReadFile(hFile, prsrc, RSRCSIZE, &haveread, NULL); //读取整个资源节

for (i = 0; i < RSRCSIZE; i++)
{
    if (prsrc[i] == 0x81 && prsrc[i + 1] == 0xC5)
    {
        printf("\n%x", prsrc[i + 5]);
        if (prsrc[i + 5] == 0xFF)
        {
            for (m = 0; m < 4; m++)
                printf("\n%x", prsrc[i + 2 + m]);
            for (m = 3; m > -1; m--)
                RandomKey = prsrc[i + 2 + m] + RandomKey * 0x100;
            break;
        }
    }
}

printf("\nRandomKey:%x", RandomKey);
RealEntry += RandomKey;
printf("\n原始文件入口点:%x", RealEntry);
pOptionalHeader->AddressOfEntryPoint = RealEntry; //入口点修复
```

上面代码主要目的是修复入口点信息。因为Virut感染后的文件起初加载的入口点被修改到了资源节，通过动态调试发现真正的入口点`RealEntry`是第一个`call`的相对偏移再加上一个有特征的随机数。最后把`RealEntry`写入`pOptionalHeader->AddressOfEntryPoint`信息处修复原来的PE入口。
