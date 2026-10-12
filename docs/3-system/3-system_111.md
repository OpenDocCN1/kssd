
# LOADEXE 之 C 版本

> 原文：[`www.pediy.com/kssd/pediy10/83215.html`](https://www.pediy.com/kssd/pediy10/83215.html)

今天课堂上钱老师讲 PE 文件结构，并用汇编加载一个 EXE 然后运行。我尝试用 VC 的也写了个。

用 VC 实现这个功能的最大麻烦是，如何让程序运行起来的 `0X401000` 到 `0X0404000` 这段内存空出，用来放被加载程序的各个区段。

首先在编译器 `LINK` 选项中设置 `BASEADDR` 的地址。我先把这个地址设置的高点。默认应该是 `0X400000`。我把这个地址设为 `0X500000`，然后用 `VirautlAlloc` 函数申请 `0X401000` 开头的 `0X3000` 大小的内存。现在的问题是：无论我怎么设置 `BASEADDR`，`0X401000` 这段内存都会被系统资源占用，`VirautlAlloc` 失败。郁闷。

只好定义了一个大的数组，作为全局变量。然后慢慢调整 `BASEADDR`，使得程序执行的时候，这个数组可以包含 `0X401000` 这段内存。下面给出程序实现。

```c
BYTE g_bImageBsae[0xe000];

void CLoadExeDlg::OnButton1()

{
  // TODO: Add your control notification handler code here
  UpdateData();
  void *pFileAddr;//文件隐射基地址
  IMAGE_DOS_HEADER *pDosHead;
  IMAGE_OPTIONAL_HEADER *pOPHead;
  IMAGE_FILE_HEADER  *pFileHead;
  DWORD dwSecNum;
  IMAGE_SECTION_HEADER *pSecInfo;
  IMAGE_IMPORT_DESCRIPTOR *pIID;
  LPTHREAD_START_ROUTINE OEP;
  void *pImageBase = (void*)0x400000;
  HANDLE hFile = ::CreateFile(m_strFileName,GENERIC_READ,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
  if (hFile==INVALID_HANDLE_VALUE)
  {
    AfxMessageBox("Can't open file");
    return;
  }
  HANDLE hMaping = CreateFileMapping(hFile,NULL,PAGE_READONLY,0,0,NULL);
  pFileAddr = MapViewOfFile(hMaping,FILE_MAP_READ,0,0,0);

  pDosHead = (IMAGE_DOS_HEADER*)pFileAddr;
  pFileHead = (IMAGE_FILE_HEADER*)(pDosHead->e_lfanew+4 +(DWORD)pFileAddr);
  pOPHead = (IMAGE_OPTIONAL_HEADER*)((DWORD)pFileHead +
           sizeof(_IMAGE_FILE_HEADER));
  OEP = (LPTHREAD_START_ROUTINE)(pOPHead->AddressOfEntryPoint + (DWORD)pImageBase);
  dwSecNum = pFileHead->NumberOfSections;
  pSecInfo = new IMAGE_SECTION_HEADER[dwSecNum];
  //描述表首地址
  DWORD SecInfoStart;
  SecInfoStart = (DWORD)pOPHead + pFileHead->SizeOfOptionalHeader ;
  //获取区信息表
  memcpy((void*)pSecInfo, (void*)SecInfoStart,
    sizeof(IMAGE_SECTION_HEADER)*dwSecNum);
  //把区搬到 0X400000 为基地址的地方。
  for (DWORD i = 0; i < dwSecNum; i++)
  {
    DWORD dwSecImageAddr = pSecInfo[i].VirtualAddress+(DWORD)pImageBase;
    DWORD dwSecFileAddr  = pSecInfo[i].PointerToRawData + (DWORD)pFileAddr;
    memcpy((void*)dwSecImageAddr,(void*)dwSecFileAddr,pSecInfo[i].Misc.VirtualSize);
  }
  pIID = (IMAGE_IMPORT_DESCRIPTOR*)((pOPHead->DataDirectory[1].VirtualAddress) + (DWORD)pImageBase);
  //解析 INT 表并填入 IAT。
  while (pIID->Name != 0)
  {
    char *pDllName;
    char *pFacName;
    HMODULE hDll;
      IMAGE_THUNK_DATA32 *pIDT,*pIAT;
    PIMAGE_IMPORT_BY_NAME *pImByName;
    pDllName = (char*)((DWORD)(pIID->Name) + (DWORD)pImageBase);
    hDll = LoadLibrary((LPCSTR)pDllName);
    pIDT = (IMAGE_THUNK_DATA32*)(pIID->OriginalFirstThunk + (DWORD)pImageBase);
    pIAT = (IMAGE_THUNK_DATA32*)(pIID->FirstThunk + (DWORD)pImageBase);
    while (*(DWORD*)pIDT != 0)
    {
      pImByName = (PIMAGE_IMPORT_BY_NAME*)((DWORD)(pIDT->u1.AddressOfData)+ (DWORD)pImageBase);
      pFacName =(char*)pImByName+2;
      pIAT->u1.Function = (PDWORD)GetProcAddress(hDll,pFacName);
      pIDT++ ;
    }
    pIID++;
  }
  DWORD dwThreadID;
  HANDLE hThread = ::CreateThread(NULL,NULL,OEP,NULL,NULL,&dwThreadID);
  ::WaitForSingleObject(hThread,INFINITE);
  delete []pSecInfo;
}
```

上边代码在 `VC6.0` 下编译通过。编译器警告：在 95 系统中无法运行。

这个方法是在是太笨，如果你们有好的方法，一定告知我。

类型转换是在是看的晕。这么点代码竟然折腾了我一个下午。看来要努力啦![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")
