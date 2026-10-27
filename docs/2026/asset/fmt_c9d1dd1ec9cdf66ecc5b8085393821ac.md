
# [原创]提取嵌入文件中的 PE 文件

有时候会遇到某些程序把整个 PE 文件（就像整个 `.exe`, `.dll`, `.sys`, ...）嵌入到数据段中，在需要的时候才释放出来。

要提取这类程序中的文件，只有动态调试或者手工静态提取。

Exeinfo 本身带有提取功能，但是它的提取方法是找到一个有效的文件头之后就从文件头一直提取到整个文件的末尾。

这样虽然拨出了萝卜，但是带出了不少泥。

只好自己动手写一个程序了。

## 原理

标准的 PE 文件结构是：Dos头｜PE头｜区段表｜区段1,区段2,区段3,...

这样要提取出完整的文件，就是从Dos头开始，一直到最后一个区段的末尾。

## 思路

先把目标文件映射到内存，获取文件在内存中的指针，遍历整个文件内容。

把所有疑似 PE 文件的内容都保存出来。

相关代码如下：

把文件映射到内存之后，通过内存指针来判断是否是一个 PE 文件，通过简单的判断 DOS 头和 PE 头来确定。

```
// 是否疑似 PE 文件？

BOOL IsPeLike(LPVOID lpImageBase)
{
    PIMAGE_DOS_HEADER    pDosHeader;
    PIMAGE_NT_HEADERS    pNtHeader;

    pDosHeader    =    (PIMAGE_DOS_HEADER)lpImageBase;
    
    if (IMAGE_DOS_SIGNATURE != pDosHeader->e_magic)    return    FALSE;

    pNtHeader    =    (PIMAGE_NT_HEADERS)((DWORD)pDosHeader + pDosHeader->e_lfanew);
    // 如果偏移落在结构体范围之外，说明数据是错误的。跳过。
    if((DWORD)pDosHeader->e_lfanew > ((DWORD)lpImageBase + sizeof(IMAGE_DOS_HEADER) + sizeof(IMAGE_NT_HEADERS)))    return    FALSE;
    if ((DWORD)pNtHeader > ((DWORD)lpImageBase + sizeof(IMAGE_DOS_HEADER) + sizeof(IMAGE_NT_HEADERS)))    return    FALSE;

    if (IMAGE_NT_SIGNATURE != pNtHeader->Signature)    return    FALSE;

    return    TRUE;
}
```

如果通过判断，那么说明找到的数据可能是一个 PE 文件，接下来获取这个文件的真实大小，真实大小是从 DOS 头开始，直到最后一个区段末尾的总大小，不包含附加数据。

```
// 获取 PE 文件真实大小

DWORD WINAPI GetRealSize(LPVOID    lpImageBase)
{
    PIMAGE_DOS_HEADER    pDosHeader;
    PIMAGE_NT_HEADERS    pNtHeader;
    PIMAGE_FILE_HEADER    pFileHeader;
    PIMAGE_SECTION_HEADER    pSectionHeader;

    pDosHeader    =    (PIMAGE_DOS_HEADER)lpImageBase;
    pNtHeader    =    (PIMAGE_NT_HEADERS)((DWORD)pDosHeader + pDosHeader->e_lfanew);
    pSectionHeader    =    IMAGE_FIRST_SECTION(pNtHeader);
    pFileHeader    =    (PIMAGE_FILE_HEADER)&pNtHeader->FileHeader;
    pSectionHeader    +=    (pFileHeader->NumberOfSections - 1);    //定位到最后一个区块表的开头

    return (pSectionHeader->PointerToRawData + pSectionHeader->SizeOfRawData); // 文件实际大小 = 文件结尾偏移 = 最后一个区段的磁盘偏移 + 最后一个区段的磁盘大小
}
```

在保存到磁盘之前先判断一下 PE 的类型，根据判断结果来决定文件的扩展名，如果不能确定类型，则默认为 `.bin`。
