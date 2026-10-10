

# LoadLibrary C 版本

> 原文：[`www.pediy.com/kssd/pediy10/83447.html`](https://www.pediy.com/kssd/pediy10/83447.html)

继续前边 2 贴，现把 LOADLIBRARY 也一并贴上。  
  
HMODULE \_\_stdcall MyLoadLibraryA(char \*szDllName)  
{  
  void \*pFileAddr;//文件隐射基地址  
  void \*pImageBase;//加载 DLL 的基地址，作为返回数  
  IMAGE\_DOS\_HEADER \*pDosHead;  
  IMAGE\_OPTIONAL\_HEADER \*pOPHead;  
  IMAGE\_FILE\_HEADER  \*pFileHead;  
  DWORD dwSecNum;  
  IMAGE\_SECTION\_HEADER \*pSecInfo;  
  IMAGE\_IMPORT\_DESCRIPTOR \*pIID;  
  LPTHREAD\_START\_ROUTINE OEP;  
  IMAGE\_BASE\_RELOCATION \*pIBR;  
  DWORD dwDllRVA;           //实际装载地址与建议装载地址差。  
  HANDLE hFile = ::CreateFile(szDllName,GENERIC\_READ,0,NULL,OPEN\_EXISTING,  
    FILE\_ATTRIBUTE\_NORMAL,NULL);  
  if (hFile==INVALID\_HANDLE\_VALUE)  
  {  
    AfxMessageBox("Can't open file");  
    return 0;  
  }  
  HANDLE hMaping = CreateFileMapping(hFile,NULL,PAGE\_READONLY,0,0,NULL);  
  pFileAddr = MapViewOfFile(hMaping,FILE\_MAP\_READ,0,0,0);  
      
  pDosHead = (IMAGE\_DOS\_HEADER\*)pFileAddr;  
  pFileHead = (IMAGE\_FILE\_HEADER\*)(pDosHead->e\_lfanew+4 +(DWORD)pFileAddr);  
    
  pOPHead = (IMAGE\_OPTIONAL\_HEADER\*)((DWORD)pFileHead +   
    sizeof(\_IMAGE\_FILE\_HEADER));  
  
  //申请内存，建立 DLL 内存空间，并 COPY DLL  
  pImageBase = VirtualAlloc(NULL,pOPHead->SizeOfImage,MEM\_COMMIT  
    ,PAGE\_EXECUTE\_READWRITE);  
  if (pImageBase == NULL)  
  {  
    AfxMessageBox("分配空间失败");  
    return 0;  
  }  
  dwDllRVA = (DWORD)pImageBase - pOPHead->ImageBase;  
  pSecInfo = (IMAGE\_SECTION\_HEADER\*)((DWORD)pOPHead + pFileHead->SizeOfOptionalHeader) ;  
  dwSecNum = pFileHead->NumberOfSections;  
  //COPY 文件头  
  DWORD dwFileSize = pOPHead->SizeOfHeaders;  
  memcpy(pImageBase,pFileAddr,dwFileSize);  
    
  //COPY 区段  
  for (DWORD i = 0; i < dwSecNum; i++)  
  {  
    void \*des,\*sour;  
    des = (void\*)(pSecInfo->VirtualAddress + (DWORD)pImageBase);  
    sour = (void\*)(pSecInfo->PointerToRawData + (DWORD)pFileAddr);  
    memcpy(des,sour,pSecInfo->Misc.VirtualSize);  
    pSecInfo++;      
  }  
  OEP = (LPTHREAD\_START\_ROUTINE)(pOPHead->AddressOfEntryPoint +   
    (DWORD)pImageBase);  
  
    //  修正重定位表。  
  pIBR = (IMAGE\_BASE\_RELOCATION\*)(pOPHead->DataDirectory\[5\].VirtualAddress + (DWORD)pImageBase);  
  while (pIBR->VirtualAddress)  
  {  
    WORD \*pTypeOffset = (WORD\*)((DWORD)pIBR+ 8);  
  
    for (i = 0; i < pIBR->SizeOfBlock - 8; i += 2)  
    {  
      //pTypeOffset\[i\] & 0Xf000 == 0x3000 结果不对，郁闷。  
      if (pTypeOffset\[i\] >>12 == 3)  
      {  
        DWORD nTmp =((DWORD) pTypeOffset\[i\]) & 0xfff;  
        DWORD \*pReloc = (DWORD\*)(nTmp +(DWORD) pImageBase + pIBR->VirtualAddress);  
         \*pReloc = \*pReloc + dwDllRVA;          
      }         
    }  
    pIBR = (IMAGE\_BASE\_RELOCATION\*)(pIBR->SizeOfBlock + (DWORD)pIBR);  
  }  
  
  //填充 IAT 表  
  pIID = (IMAGE\_IMPORT\_DESCRIPTOR\*)((pOPHead->DataDirectory\[1\].VirtualAddress)  
    + (DWORD)pImageBase);  
  while (pIID->Name != 0)  
  {  
    char \*pDllName;  
    char \*pFacName;  
      
    HMODULE hDll;  
    IMAGE\_THUNK\_DATA32 \*pIDT,\*pIAT;  
    PIMAGE\_IMPORT\_BY\_NAME \*pImByName;  
    pDllName = (char\*)((DWORD)(pIID->Name) + (DWORD)pImageBase);  
    hDll = LoadLibrary((LPCSTR)pDllName);  
    pIDT = (IMAGE\_THUNK\_DATA32\*)(pIID->OriginalFirstThunk +   
      (DWORD)pImageBase);  
    pIAT = (IMAGE\_THUNK\_DATA32\*)(pIID->FirstThunk + (DWORD)pImageBase);  
      
    while (\*(DWORD\*)pIDT != 0)  
    {  
      pImByName = (PIMAGE\_IMPORT\_BY\_NAME\*)((DWORD)(pIDT->u1.AddressOfData)  
        + (DWORD)pImageBase);  
      pFacName =(char\*)pImByName+2;  
      pIAT->u1.Function = (PDWORD)GetProcAddress(hDll,pFacName);  
      pIDT++ ;  
    }  
    pIID++;  
  }  
//调用 DLLMAIN  
  \_asm  
  {  
    pushad;  
    push 0;  
    push DLL\_PROCESS\_ATTACH;  
    push pImageBase;  
    call OEP;  
    add esp,0x4;     //DLLMAIN 未平衡堆栈。  
    popad  
  }  
    
  return (HMODULE)pImageBase;  
    
  
}  
  
用此函数加载的 DLL。有部分 API 会发生问题。比如：GetProcAddress,GetModulHandle。另外，本函数也没有考虑 TLS 的处理。等以后有时间在慢慢修补吧。