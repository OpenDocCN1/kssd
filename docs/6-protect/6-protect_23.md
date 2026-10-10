

# 这是最近学习 PE 结构写的 GetProcAddress 汇编版【原创】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-767.htm`](https://www.pediy.com/kssd/pediy07/pediy7-767.htm)

; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; GetAPIAddress  
; ebx = hModule  
; esi = lpszProcName  
; eax = api address  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
GetAPIAddress    proc  
          pushad  
  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 检查是否是有效的 PE 文件格式        \*  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
          cmp    word ptr \[ebx\], 'ZM'  
          jne    \_\_@@1\_GetAPIAddress    ;是不是有效的 PE 文件  
  
          mov    eax, \[ebx + 3CH\]  
          add    eax, ebx        ;eax = PE Header  
          cmp    dword ptr \[eax\], 00004550H  ;PE\\0\\0  
          jne    \_\_@@1\_GetAPIAddress  
              
  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 搜索需要的 API 地址  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;typedef struct \_IMAGE\_EXPORT\_DIRECTORY  
;{  
;    DWORD   Characteristics;        // +0x00  
;    DWORD   TimeDateStamp;          // +0x04  
;    WORD    MajorVersion;           // +0x08  
;    WORD    MinorVersion;           // +0x0a  
;    DWORD   Name;                   // +0x0c Name of the DLL  
;    DWORD   Base;                   // +0x10 Starting ordinal number for exports  
;    DWORD   NumberOfFunctions;      // +0x14 Number of entries in the EAT  
;    DWORD   NumberOfNames;          // +0x18 Number of entries in the ENPT/EOT  
;    DWORD   AddressOfFunctions;     // +0x1c RVA from base of image  
;    DWORD   AddressOfNames;         // +0x20 RVA from base of image  
;    DWORD   AddressOfNameOrdinals;  // +0x24 RVA from base of image  
;                                    // +0x28  
;} IMAGE\_EXPORT\_DIRECTORY, \*PIMAGE\_EXPORT\_DIRECTORY;  
  
          mov    edx, \[eax + 04H + 14H + 60H\]  
          test  edx, edx  
          jz    \_\_@@1\_GetAPIAddress      ;没有输出表  
  
          add    edx, ebx          ;计算输出表位置  
          ;NumberOfFunctions  
          mov    ecx, \[edx + 18H\]  
          ;AddressOfNames  
          mov    eax, \[edx + 20H\]        
          add    eax, ebx  
          ;AddressOfFunctions  
          mov    ebp, \[edx + 1CH\]  
          add    ebp, ebx  
          ;AddressOfNameOrdinals  
          mov    edx, \[edx + 24H\]  
          add    edx, ebx  
  
          dec    ecx  
          cld  
            
          push  eax  
          xor    eax, eax  
    \_\_@@2\_GetAPIAddress:  
          inc    eax  
          cmp    byte ptr \[esi + eax\], 0  
          jne    \_\_@@2\_GetAPIAddress  
    ;      dec    eax  
          xchg  eax, \[esp\]  
            
          push  esi      ;\[esp\] = esi  
                    ;\[esp + 4\] = strlen(lpszProcName)  
    \_\_@@FindNextExportFunction\_GetAPIAddress:  
          mov    edi, \[eax + ecx \* 4\]  
          add    edi, ebx          ;edi = API NAME  
            
          push  ecx  
          mov    ecx, \[esp + 4 + 4\]  
          repz  cmpsb  
          jnz    \_\_@@3\_GetAPIAddress  
          test  ecx, ecx  
          jnz    \_\_@@3\_GetAPIAddress  
          ;找到匹配的字符串  
          pop    ecx  
          movzx  ecx, word ptr \[edx + ecx \* 2\]  
          mov    eax, \[ebp + ecx \* 4\]  
          add    eax, ebx          ;eax = API ADDRESS  
          mov    \[esp + 32 - 4 + 8\], eax        ;返回值  
          jmp    \_\_@@4\_GetAPIAddress  
    \_\_@@3\_GetAPIAddress:  
          pop    ecx  
          mov    esi, \[esp\]  
          dec    ecx  
          jns    \_\_@@FindNextExportFunction\_GetAPIAddress  
    \_\_@@4\_GetAPIAddress:  
          pop    ebx  
          pop    ebx  
    \_\_@@1\_GetAPIAddress:  
          popad  
          ret  
GetAPIAddress    endp 

; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 检查是否是有效的 PE 文件格式        \*  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
          cmp    word ptr \[ebx\], 'ZM'  <- 应该是'MZ'  
          jne    \_\_@@1\_GetAPIAddress   
感觉是 e\_magic  MZ 

> 引用:
> 
> * * *
> 
> *最初由 xzchina 发布*  
> **; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> ; 检查是否是有效的 PE 文件格式        \*  
> ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>           cmp    word ptr \[ebx\], 'ZM'  <- 应该是'MZ'  
>           jne    \_\_@@1\_GetAPIAddress   
> ........ **
> 
> * * *

  
  
在 masm 中必须这样写反过来，有些汇编器可以自动处理，不需要.  
  
这个过程已经测试成功. 

FARPROC WINAPI MyGetProcAddress(HMODULE hModule, LPCSTR lpProcName)  
{  
  PIMAGE\_DOS\_HEADER pDOSHeader;  
  PIMAGE\_NT\_HEADERS pNTHeader;  
  PIMAGE\_EXPORT\_DIRECTORY pExportDir;  
  LPCSTR \*pFunctionName; /\* 指向函数名数组 \*/  
  LPCSTR pszFunName; /\* 指向函数名 \*/  
  LPDWORD pFunction;  
  LPWORD pIndex;  
  DWORD n; /\* 以名字输出的函数的个数 \*/  
  FARPROC ret = NULL; /\* 返回值 \*/  
  
  \_\_try  
  {  
    if (NULL == hModule && NULL == lpProcName) \_\_leave; /\* 快速离开这里 \*/  
      
    pDOSHeader = (PIMAGE\_DOS\_HEADER)hModule;  
    if (IsBadReadPtr(pDOSHeader, sizeof(IMAGE\_DOS\_HEADER))) \_\_leave;  
    if (pDOSHeader->e\_magic != IMAGE\_DOS\_SIGNATURE) \_\_leave; /\* 是不是有效的 DOS 文件头标志 \*/  
      
    pNTHeader = (PIMAGE\_NT\_HEADERS)((DWORD)pDOSHeader + (DWORD)pDOSHeader->e\_lfanew);  
    if (pNTHeader->Signature != IMAGE\_NT\_SIGNATURE) \_\_leave; /\* 不是有效的 PE 文件头标志 \*/  
      
    pExportDir = (PIMAGE\_EXPORT\_DIRECTORY)((DWORD)pDOSHeader + \\  
      (DWORD)pNTHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress);  
    if ((DWORD)pExportDir == (DWORD)pDOSHeader) \_\_leave; /\* 没有输出表 \*/  
      
    pFunctionName = (LPCSTR \*)((DWORD)pExportDir->AddressOfNames + (DWORD)pDOSHeader);  
    pFunction = (LPDWORD)((DWORD)pExportDir->AddressOfFunctions + (DWORD)pDOSHeader);  
    pIndex = (LPWORD)((DWORD)pExportDir->AddressOfNameOrdinals + (DWORD)pDOSHeader);  
    n = pExportDir->NumberOfNames;  
      
    while (n--)  
    {  
      pszFunName = (LPCSTR)((DWORD)\*pFunctionName + (DWORD)pDOSHeader);  
      if (strcmp(pszFunName, lpProcName) == 0)  
      {  
        ret = (FARPROC)(pFunction\[\*pIndex\] + (DWORD)pDOSHeader);  
        break;  
      }  
      pFunctionName++;  
      pIndex++;  
    }  
  }  
  \_\_finally  
  {  
    /\* 什么也不需要做 \*/  
  }  
  
  return ret;  
} 

 4dd466d0e1e0cc123afbe504da2462ca