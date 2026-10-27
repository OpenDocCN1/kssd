
### `BOOL CPE::SetShellReloc(LPBYTE pShellBuf, DWORD hShell)`

```
BOOL CPE::SetShellReloc(LPBYTE pShellBuf, DWORD hShell)

{

  typedef struct _TYPEOFFSET

  {

    WORD offset : 12;      //偏移值

    WORD Type  : 4;      //重定位属性(方式)

  }TYPEOFFSET, *PTYPEOFFSET;

  //1.获取被加壳PE文件的重定位目录表指针信息

  PIMAGE_DATA_DIRECTORY pPERelocDir =

    &(m_pNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC]);

  

  //2.获取Shell的重定位表指针信息

  PIMAGE_DOS_HEADER    pShellDosHeader = (PIMAGE_DOS_HEADER)pShellBuf;

  PIMAGE_NT_HEADERS    pShellNtHeader = (PIMAGE_NT_HEADERS)(pShellBuf + pShellDosHeader->e_lfanew);

  PIMAGE_DATA_DIRECTORY  pShellRelocDir =

    &(pShellNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC]);

  PIMAGE_BASE_RELOCATION  pShellReloc = 

    (PIMAGE_BASE_RELOCATION)((DWORD)pShellBuf + pShellRelocDir->VirtualAddress);

  

  //3.还原修复重定位信息

  //由于Shell.dll是通过LoadLibrary加载的，所以系统会对其进行一次重定位

  //我们需要把Shell.dll的重定位信息恢复到系统没加载前的样子，然后在写入被加壳文件的末尾

  PTYPEOFFSET pTypeOffset = (PTYPEOFFSET)(pShellReloc + 1);

  DWORD dwNumber = (pShellReloc->SizeOfBlock - 8) / 2;

  for (DWORD i = 0; i < dwNumber; i++)

  {

    if (*(PWORD)(&pTypeOffset[i]) == NULL)

      break;

    //RVA

    DWORD dwRVA =pTypeOffset[i].offset + pShellReloc->VirtualAddress;

    //FAR地址（LordPE中这样标注）

    //***新的重定位地址=重定位后的地址-加载时的镜像基址+新的镜像基址+代码基址(PE文件镜像大小)

    DWORD AddrOfNeedReloc =  *(PDWORD)((DWORD)pShellBuf + dwRVA);

    *(PDWORD)((DWORD)pShellBuf + dwRVA) 

      = AddrOfNeedReloc - pShellNtHeader->OptionalHeader.ImageBase + m_dwImageBase + m_dwImageSize;

  }

  //3.1修改Shell重定位表中.text的RVA

  pShellReloc->VirtualAddress += m_dwImageSize;

  //4.修改PE重定位目录指针，指向Shell的重定位表信息

  pPERelocDir->Size = pShellRelocDir->Size;

  pPERelocDir->VirtualAddress = pShellRelocDir->VirtualAddress + m_dwImageSize;

  return TRUE;

}
```

#### 4.3.修改被加壳程序的OEP，指向Shell

这样就可以让程序运行时从Shell部分开始执行，那么这个地址改如何确定呢？还记得之前我们的Shell部分所导出的那个结构体么？对，其第一个变量就是Shell部分启动函数的地址，让OEP执行这个地址即可。

#### 4.4.合并PE文件和Shell的代码到新的缓冲区

目前我们内存中有两个缓冲区，一个是原PE程序的缓冲区，另一个是Shell部分的缓冲区，我们要做的就是重新申请一块连续的空间，大小为这两个缓冲区的大小，然后那他们拷贝进去。内存中这样处理是没问题的，这时如果保存这个缓冲区的话就是两个文件的结合体，但逻辑上并没有如此简单，因为你从物理上多出来一个Shell区段，但逻辑上并没有变，也就是系统并不认识这个新加的Shell区段，这时候你就又需要去修改PE文件信息了，这次需要修改的是区段表中的信息，涉及到新增一个区段目录表信息，并正确设置该区段的起始地址、大小等信息，而且在设置这些信息的时候还要考虑到文件对齐的问题，源代码中则是`MergeBuf()`函数实现了这两个缓冲区的合并，并添加区段信息：

```
void CPE::MergeBuf(LPBYTE pFileBuf, DWORD pFileBufSize,

  LPBYTE pShellBuf, DWORD pShellBufSize, 

  LPBYTE& pFinalBuf, DWORD& pFinalBufSize)

{

  //获取最后一个区段的信息

  PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)pFileBuf;

  PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)(pFileBuf + pDosHeader->e_lfanew);

  PIMAGE_SECTION_HEADER pSectionHeader = IMAGE_FIRST_SECTION(pNtHeader);

  PIMAGE_SECTION_HEADER pLastSection =

    &pSectionHeader[pNtHeader->FileHeader.NumberOfSections - 1];

  //1.修改区段数量

  pNtHeader->FileHeader.NumberOfSections += 1;

  //2.编辑区段表头结构体信息

  PIMAGE_SECTION_HEADER AddSectionHeader =

    &pSectionHeader[pNtHeader->FileHeader.NumberOfSections - 1];

  memcpy_s(AddSectionHeader->Name, 8, ".cyxvc", 7);

  //VOffset(1000对齐)

  DWORD dwTemp = 0;

  dwTemp = (pLastSection->Misc.VirtualSize / m_dwMemAlign) * m_dwMemAlign;

  if (pLastSection->Misc.VirtualSize % m_dwMemAlign)

  {

    dwTemp += 0x1000;

  }

  AddSectionHeader->VirtualAddress = pLastSection->VirtualAddress + dwTemp;

  //Vsize（实际添加的大小）

  AddSectionHeader->Misc.VirtualSize = pShellBufSize;

  //ROffset（旧文件的末尾）

  AddSectionHeader->PointerToRawData = pFileBufSize;

  //RSize(200对齐)

  dwTemp = (pShellBufSize / m_dwFileAlign) * m_dwFileAlign;

  if (pShellBufSize % m_dwFileAlign)

  {

    dwTemp += m_dwFileAlign;

  }

  AddSectionHeader->SizeOfRawData = dwTemp;

  //区段属性标志（可读可写可执行）

  AddSectionHeader->Characteristics = 0XE0000040;

  //3.修改PE头文件大小属性，增加文件大小

  dwTemp = (pShellBufSize / m_dwMemAlign) * m_dwMemAlign;

  if (pShellBufSize % m_dwMemAlign)

  {

    dwTemp += m_dwMemAlign;

  }

  pNtHeader->OptionalHeader.SizeOfImage += dwTemp;

  //4.申请合并所需要的空间

  pFinalBuf = new BYTE[pFileBufSize + dwTemp];

  pFinalBufSize = pFileBufSize + dwTemp;

  memset(pFinalBuf, 0, pFileBufSize + dwTemp);

  memcpy_s(pFinalBuf, pFileBufSize, pFileBuf, pFileBufSize);

  memcpy_s(pFinalBuf + pFileBufSize, dwTemp, pShellBuf, dwTemp);

}
```

### 5.保存文件

保存文件就是保存上个函数中所合并的缓冲区，由于我是直接从从内存中dump出来的，其分布也是以内存对齐大小所对齐的，所以保存成文件的时候我顺便修改了文件对齐大小，同内存对齐大小相同；同时将数据目录表中不必要的信息摸掉：

```
BOOL CPACK::SaveFinalFile(LPBYTE pFinalBuf, DWORD pFinalBufSize, CString strFilePath)

{

  //修正区段信息中 文件对齐大小（文件对齐大小同内存对齐大小）

  PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)pFinalBuf;

  PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)(pFinalBuf + pDosHeader->e_lfanew);

  PIMAGE_SECTION_HEADER pSectionHeader = IMAGE_FIRST_SECTION(pNtHeader);

  for (DWORD i = 0; i < pNtHeader->FileHeader.NumberOfSections; i++, pSectionHeader++)

  {

    pSectionHeader->PointerToRawData = pSectionHeader->VirtualAddress;

  }

  //清除不需要的目录表信息

  //只留输出表，重定位表，资源表

  DWORD dwCount = 15;

  for (DWORD i = 0; i < dwCount; i++)

  {

    if (i != IMAGE_DIRECTORY_ENTRY_EXPORT && 

      i != IMAGE_DIRECTORY_ENTRY_RESOURCE &&

      i != IMAGE_DIRECTORY_ENTRY_BASERELOC )

    {

      pNtHeader->OptionalHeader.DataDirectory[i].VirtualAddress = 0;

      pNtHeader->OptionalHeader.DataDirectory[i].Size = 0;

    }

  }

  //获取保存路径

  TCHAR strOutputPath[MAX_PATH] = { 0 };

  LPWSTR strSuffix = PathFindExtension(strFilePath);

  wcsncpy_s(strOutputPath, MAX_PATH, strFilePath, wcslen(strFilePath));

  PathRemoveExtension(strOutputPath);

  wcscat_s(strOutputPath, MAX_PATH, L"_cyxvc");

  wcscat_s(strOutputPath, MAX_PATH, strSuffix);

  //保存文件

  HANDLE hNewFile = CreateFile(

    strOutputPath,

    GENERIC_READ | GENERIC_WRITE,

    0,

    NULL,

    CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

  if (hNewFile == INVALID_HANDLE_VALUE)

  {

    MessageBox(NULL, _T("保存文件失败！"), _T("提示"), MB_OK);

    return FALSE;

  }

  DWORD WriteSize = 0;

  BOOL bRes = WriteFile(hNewFile, pFinalBuf, pFinalBufSize, &WriteSize, NULL);

  if (bRes)

  {

    CloseHandle(hNewFile);

    return TRUE;

  }

  else

  {

    CloseHandle(hNewFile);

    MessageBox(NULL, _T("保存文件失败！"), _T("提示"), MB_OK);

    return FALSE;

  }

}
```

### 6.释放资源

万事大吉以后不要忘记擦屁股，养成良好的习惯，释放掉资源，避免资源泄露（虽然这个小程序也用不了多少内存吧哈哈！）

```
  delete[] objPE.m_pFileBuf;

  delete[] pShellBuf;

  delete[] pFinalBuf;

  objPE.InitValue();
```

至此，第一部分(`Pack`)的功能已经完成，剩下的就是Shell部分的编写了！

## 第二部分：外壳程序(`Shell.dll`)的编写

壳的Shell部分，网上很多人在用汇编编写，汇编的优势是不需要进行重定位，移植性好，用C++的话需要重定位，而这个问题在第一部分已经解决了，再配合上两部分的数据交换，Shell部分用C++写起来也变得如鱼得水，代码的可读性有了很大的提高（好吧，我承认，其实是我不会用汇编写...）。

外壳程序部分的流程(`此流程在Shell工程中的Start()函数中实现`)：

1. 获取Shell部分所用到的函数
2. 解密代码段
3. 修复原程序的重定位信息
4. 修复原程序的导入表(`IAT`)信息
5. 跳到程序入口点，将控制权交还给程序

### 具体实现

#### 1.获取Shell部分所用到的函数

有很多壳为了隐藏自己的行为，不让别人看出它用到了哪些函数，直接没有导入表，不让PE加载器为其导入函数。而是直接自己获取所用到的函数，此项目也是用的这种方法，通过代码实现导入Shell部分所需要的函数。

无论一个PE文件是否有导入表，系统都会为其加载两个模块，`ntdll.dll`和`Kernel32.dll`，那么我们就从这两个模块入手，目标是获取`GetProcAddress()`这个函数，而这个函数位于`Kernel32.dll`中，那么我们首先要做的就是获取`Kernel32.dll`的加载基址，常用的方法有三种：①通过特征匹配的暴力搜索。②利用系统的SEH机制找到`Kernel32.dll`的加载基址。③通过线程环境块TEB的信息逐步找到`Kernel32.dll`的加载基址。我在这里用的是第三种，详细的代码请见Shell部分的`MyGetProcAddress()`函数，第三种方法代码简介，但存在兼容性问题，因为XP和Win7下获取基址时的代码略有出入，所以如果你为了兼容性考虑，请改用其他两种方法。

在获取了`Kernel32.dll`的加载基址以后，就可以通过遍历`Kernel32.dll`的导出表来搜索`GetProcAddress()`，从而获取`GetProcAddress()`函数的函数地址：
