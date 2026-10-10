

# 寻找原始表，恢复 ssdt 表

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-635.htm`](https://www.pediy.com/kssd/pediy09/pediy09-635.htm)

> // 只贴 ring3 部分，这也是最主要的代码，驱动只是负责把得到的原始值写到 ssdt，其他啥也不做  
> // 简单说一下，找 KiServiceTable 表过程， 是直接读取原始内核文件，寻找输出符号，然后结合重定位信息，谁向 KeServiceDescriptorTable 赋值，从而确定 KiServiceTable 的位置。  
> // 这是第一写驱动相关的代码，写的很挫，高手就不用看了。![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心") ![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心")   
>   
> #include <tchar.h>  
> #include <Windows.h>  
> #include <winioctl.h>  
> #include "RSSDT.h"  
> /\*  
> // RSSDT.h  
>   
> typedef struct \_tagSST\_ENTRY  
> {  
>   ULONG cbSize;  
>   ULONG Index;  
>   ULONG OrgValue;    // 类型 RVA, 传给驱动的时做 Set Value  
>   ULONG NowValue; // 类型 RVA, 传给驱动做 get value  
> } SST\_ENTRY, \*PSST\_ENTRY;  
> #define IOCTL\_RSSDT\_ENTRY\_SET  CTL\_CODE(FILE\_DEVICE\_UNKNOWN, 0x801, METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
> \*/  
>   
> ULONG Rav2Raw(PVOID pFileData, ULONG rav)  
> {  
>   PIMAGE\_DOS\_HEADER pDosH;  
>   PIMAGE\_NT\_HEADERS pNtH;  
>   PIMAGE\_SECTION\_HEADER pSectH;  
>   ULONG Index;  
>   
>   pDosH = (PIMAGE\_DOS\_HEADER)pFileData;  
>   pNtH = (PIMAGE\_NT\_HEADERS)((ULONG)pFileData + pDosH->e\_lfanew);  
>   pSectH = IMAGE\_FIRST\_SECTION(pNtH);  
>   
>   if ( rav >= 0 && rav < pSectH->VirtualAddress )  
>     return rav;  
>   
>   for ( Index = 0; Index < pNtH->FileHeader.NumberOfSections; Index++ )  
>   {  
>     if ( rav >= pSectH->VirtualAddress && rav < pSectH->VirtualAddress + pSectH->Misc.VirtualSize )  
>       return rav - pSectH->VirtualAddress + pSectH->PointerToRawData;  
>     pSectH++;  
>   }  
>   
>   return (ULONG)-1;  
> }  
> PVOID NTAPI   
> GetImageDirEntry(  
>   IN ULONG ImageBase,   
>   IN ULONG DirIndex,  
>   OUT OPTIONAL PIMAGE\_NT\_HEADERS\* ppNtH,  
>   OUT OPTIONAL PIMAGE\_DATA\_DIRECTORY\* ppDataDir  
>   )  
> {  
>   PIMAGE\_DOS\_HEADER pDosH;  
>   PIMAGE\_NT\_HEADERS pNtH;  
>   PIMAGE\_DATA\_DIRECTORY pDataDir;  
>   
>   pDosH = (PIMAGE\_DOS\_HEADER)ImageBase;  
>   if ( pDosH->e\_magic != IMAGE\_DOS\_SIGNATURE )  
>   {  
>     return NULL;  
>   }  
>     
>   pNtH = (PIMAGE\_NT\_HEADERS)(ImageBase + pDosH->e\_lfanew);  
>   if ( pNtH->Signature != IMAGE\_NT\_SIGNATURE )  
>   {  
>     return NULL;  
>   }  
>   
>   if ( ppNtH != NULL )  
>   {  
>     \*ppNtH = pNtH;  
>   }  
>     
>   pDataDir = &pNtH->OptionalHeader.DataDirectory\[DirIndex\];  
>   
>   if ( pDataDir->VirtualAddress == 0 )  
>   {  
>     return NULL;  
>   }  
>   
>   if ( ppDataDir != NULL )  
>   {  
>     \*ppDataDir = pDataDir;  
>   }  
>     
>   return (PVOID)(ImageBase + Rav2Raw((PVOID)ImageBase, pDataDir->VirtualAddress));  
> }  
>   
> typedef BOOLEAN (NTAPI\* LOOPUPXREF\_CALLBACK)(PULONG RefAddr, PVOID Param);  
>   
> ULONG NTAPI // 返回找到的引用个数  
> pLookupImageXRef(  
>   IN ULONG ImageBase, // 参考地址  
>   IN PIMAGE\_DATA\_DIRECTORY pBrDir,  
>   IN PIMAGE\_BASE\_RELOCATION pBr,  
>   IN ULONG SymAddr,  
>   IN LOOPUPXREF\_CALLBACK LookupXRefCallback, // 回调  
>   IN PVOID pParam  
>   )  
> {  
>   ULONG nRefCount = 0;  
>   
>   \_\_try  
>   {  
>     ULONG Size = 0;  
>     while ( pBr->SizeOfBlock != 0 && Size < pBrDir->Size )  
>     {  
>       PWORD pRelItem = (PWORD)(pBr + 1);  
>       PIMAGE\_BASE\_RELOCATION pNextBr = (PIMAGE\_BASE\_RELOCATION)((ULONG)pBr + pBr->SizeOfBlock);  
>   
>       while ( (ULONG)pRelItem < (ULONG)pNextBr )  
>       {  
>         ULONG RefBlock = Rav2Raw((PVOID)ImageBase, pBr->VirtualAddress);  
>   
>         if ( (pRelItem\[0\] >> 12) == 3 )  
>         {  
>           PULONG RefAddr = (PULONG)(ImageBase + RefBlock + (pRelItem\[0\] & 0x0FFF));  
>   
>           // 验证内存是否有效  
>           if ( !IsBadReadPtr(RefAddr, 1) && RefAddr\[0\] == SymAddr )  
>           {  
>             nRefCount++;  
>             if ( LookupXRefCallback(RefAddr, pParam)  )  
>               break;  
>           }  
>         }  
>           
>         pRelItem++;  
>       }  
>         
>       Size += pBr->SizeOfBlock;  
>       pBr = pNextBr;  
>     }  
>   }  
>   \_\_except ( EXCEPTION\_EXECUTE\_HANDLER )  
>   {  
>   //  DbgPrint("pLookupImageXRef: Found a Exception!\\n");  
>   }  
>     
>   return nRefCount;  
> }  
>   
> ULONG NTAPI  
> LookupImageXRef(  
>   IN ULONG ImageBase, // 有效地址  
>   IN ULONG SymAddr,  
>   IN LOOPUPXREF\_CALLBACK LookupXRefCallback, // 回调  
>   IN PVOID Param  
>   )  
> {  
>   if ( LookupXRefCallback != NULL )  
>   {  
>     PIMAGE\_DATA\_DIRECTORY pBrDir;  
>     PIMAGE\_BASE\_RELOCATION pBr;  
>       
>     pBr = GetImageDirEntry(ImageBase, IMAGE\_DIRECTORY\_ENTRY\_BASERELOC, NULL, &pBrDir);  
>     if ( pBr != NULL )  
>     {  
>       return pLookupImageXRef(ImageBase, pBrDir, pBr, SymAddr, LookupXRefCallback, Param);  
>     }  
>   }  
>   
>   return 0;  
> }  
>   
> typedef struct tag\_SYSTEM\_SERVICE\_TABLE {  
>   PULONG  ServiceTable;  // array of entry points  
>   PULONG  CounterTable;  // array of usage counters  
>   ULONG  ServiceLimit;    // number of table entries  
>   PCHAR  ArgumentTable;  // array of argument counts  
> } SYSTEM\_SERVICE\_TABLE, \*PSYSTEM\_SERVICE\_TABLE, \*\*PPSYSTEM\_SERVICE\_TABLE;  
>   
> BOOLEAN WINAPI LookXRefCallback(PULONG pRefAddr, PVOID pParam)  
> {  
>   USHORT OpCode = \*((PUSHORT)pRefAddr - 1);  
>   if ( OpCode == 0x05C7 ) // mov     ds:\_KeServiceDescriptorTable, offset \_KiServiceTable  
>   {  
>     if ( pParam != NULL )  
>     {  
>       \*(PULONG)pParam = pRefAddr\[1\]; // KiServiceTable  
>     }  
>   
>     return FALSE;  
>   }  
>   
>   return TRUE;  
> }  
>   
> BOOL GetSSTEntry(PCTSTR pszServiceName, PSST\_ENTRY pSSTEntry)  
> {  
>   HANDLE hFile, hFileMap;  
>   LPVOID pFileData;  
>   HMODULE hNtdll;  
>   FARPROC pfnService;  
>   BOOL bResult;  
>   TCHAR szNtoskrnlPath\[MAX\_PATH\];  
>     
>   hNtdll = GetModuleHandle(\_T("ntdll.dll"));  
>   pfnService = GetProcAddress(hNtdll, pszServiceName);  
>   if ( pfnService == NULL || \*(PBYTE)pfnService != 0xB8 )  
>     return FALSE;  
>   
>   // mov eax, ??  
>   pSSTEntry->Index = \*(PULONG)((PBYTE)pfnService + 1);  
>   
>   GetSystemDirectory(szNtoskrnlPath, MAX\_PATH);  
>   \_tcscat(szNtoskrnlPath, \_T("\\\\ntoskrnl.exe")); // or ntkrnlpa.exe  
>     
>   hFile = CreateFile(szNtoskrnlPath, GENERIC\_READ, FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,  
>     NULL, OPEN\_EXISTING, 0, NULL);  
>   if ( hFile == INVALID\_HANDLE\_VALUE )  
>     return FALSE;  
>   
>   hFileMap = CreateFileMapping(hFile, NULL, PAGE\_READONLY, 0, 0, NULL);  
>   CloseHandle(hFile);  
>   
>   if ( hFileMap == NULL )  
>     return FALSE;  
>     
>   pFileData = MapViewOfFile(hFileMap, FILE\_MAP\_READ, 0, 0, 0);  
>   CloseHandle(hFileMap);  
>     
>   if ( NULL == pFileData )  
>     return FALSE;  
>     
>   bResult = FALSE;  
>   \_\_try  
>   {  
>     PIMAGE\_DOS\_HEADER pDosH;  
>     PIMAGE\_NT\_HEADERS pNtH;  
>     PIMAGE\_EXPORT\_DIRECTORY pExpDir;  
>     PULONG pFunName, pFunAddr;  
>     PUSHORT pFunNameOrd;  
>     ULONG Index;  
>       
>     pDosH = (PIMAGE\_DOS\_HEADER)pFileData;  
>     pNtH = (PIMAGE\_NT\_HEADERS)((ULONG)pFileData + pDosH->e\_lfanew);  
>     pExpDir = (PIMAGE\_EXPORT\_DIRECTORY)((ULONG)pFileData + pNtH->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress);  
>   
>     pFunName = (PULONG)((ULONG)pFileData + Rav2Raw(pFileData, pExpDir->AddressOfNames));  
>     pFunNameOrd = (PUSHORT)((ULONG)pFileData + Rav2Raw(pFileData, pExpDir->AddressOfNameOrdinals));  
>     pFunAddr = (PULONG)((ULONG)pFileData + Rav2Raw(pFileData, pExpDir->AddressOfFunctions));  
>   
>     for ( Index = 0; Index < pExpDir->NumberOfNames; Index++ )      
>     {  
>       PCSTR pName = (PCSTR)((ULONG)pFileData + Rav2Raw(pFileData, pFunName\[Index\]));  
>       if ( strcmp(pName, "KeServiceDescriptorTable") == 0 )  
>       {  
>         ULONG \_KiServiceTable, \_KeServiceDescriptorTable; // 类型都是 va  
>           
>         \_KiServiceTable = 0;  
>         \_KeServiceDescriptorTable = pNtH->OptionalHeader.ImageBase + pFunAddr\[pFunNameOrd\[Index\]\];  
>     
>         LookupImageXRef((ULONG)pFileData, \_KeServiceDescriptorTable, LookXRefCallback, &\_KiServiceTable);  
>         if ( \_KiServiceTable != 0 )  
>         {  
>           PULONG ServiceTable;  
>           \_KiServiceTable -= pNtH->OptionalHeader.ImageBase;  
>   
>           ServiceTable = (PULONG)((ULONG)pFileData + Rav2Raw(pFileData, \_KiServiceTable));  
>           pSSTEntry->OrgValue = ServiceTable\[pSSTEntry->Index\] - pNtH->OptionalHeader.ImageBase;  
>         }  
>   
>         bResult = TRUE;  
>         break;  
>       }  
>     }  
>   }  
>   \_\_except ( EXCEPTION\_EXECUTE\_HANDLER )  
>   {  
>     bResult = FALSE;  
>   }  
>   
>   UnmapViewOfFile(pFileData);  
>   return bResult;  
>   
> }  
>   
> HANDLE GetServiceByName(IN PCTSTR pszServiceName, OUT OPTIONAL SC\_HANDLE\* pScManager)  
> {  
>   SC\_HANDLE hScManager, hScService;  
>     
>   hScService = NULL;  
>   hScManager = OpenSCManager(NULL, NULL, SC\_MANAGER\_ALL\_ACCESS);  
>     
>   if ( pScManager != NULL )  
>     \*pScManager = hScManager;  
>   
>   if ( hScManager != NULL )  
>   {  
>     hScService = OpenService(hScManager, pszServiceName, SERVICE\_ALL\_ACCESS);  
>     if ( pScManager == NULL )  
>       CloseServiceHandle(hScManager);  
>   }  
>   
>   return hScService;  
> }  
>   
> SC\_HANDLE InstallDriver(IN PCTSTR pszServiceName, IN PCTSTR pszDriverFile)  
> {  
>   SC\_HANDLE hScManager, hScService;  
>     
>   hScManager = NULL;  
>   hScService = GetServiceByName(pszServiceName, &hScManager);  
>   
>   if ( hScService == NULL && hScManager != NULL )  
>   {  
>     hScService = CreateService(hScManager, pszServiceName, pszServiceName, SERVICE\_ALL\_ACCESS,  
>       SERVICE\_KERNEL\_DRIVER, SERVICE\_DEMAND\_START, SERVICE\_ERROR\_NORMAL, pszDriverFile, NULL,  
>       NULL, NULL, NULL, NULL  
>       );  
> //    GetLastError();  
>   }  
>   
>   if ( hScManager != NULL )  
>     CloseServiceHandle(hScManager);  
>     
>   return hScService;  
> }  
>   
> BOOL UninstallDriver(SC\_HANDLE hScService)  
> {  
>   SERVICE\_STATUS Status;  
>     
>   if ( ControlService(hScService, SERVICE\_CONTROL\_STOP, &Status) )  
>   {  
>     DeleteService(hScService);  
>     CloseServiceHandle(hScService);  
>   
>     return TRUE;  
>   }  
> //  GetLastError();  
>   
>   return FALSE;  
> }  
>   
> BOOL ResetSSTEntry(PCSTR lpServiceName)  
> {  
>   SC\_HANDLE hScService;  
>   HANDLE hDevice;  
>   SST\_ENTRY SstEntry;  
>   ULONG cbReturn;  
>   TCHAR szDriverPath\[MAX\_PATH\];  
>   PTSTR pszDriverName;  
>   BOOL bResult;  
>   
>   SstEntry.cbSize = sizeof (SST\_ENTRY);  
>   if ( !GetSSTEntry(lpServiceName, &SstEntry) )  
>     return FALSE;  
>     
>   GetModuleFileName(NULL, szDriverPath, MAX\_PATH);  
>   pszDriverName = \_tcsrchr(szDriverPath, \_T('\\\\'));  
>   \_tcscpy(pszDriverName, \_T("\\\\RSSDT.sys"));  
>     
>   hScService = GetServiceByName(\_T("RSSDT"), NULL);  
>   if ( hScService == NULL )  
>   {  
>     hScService = InstallDriver(\_T("RSSDT"), szDriverPath);  
>     if ( hScService == NULL )  
>       return FALSE;  
>   }  
>   
>   bResult = FALSE;  
>   if ( StartService(hScService, 0, NULL) || GetLastError() == ERROR\_ALREADY\_EXISTS )  
>   {  
>     hDevice = CreateFile(\_T("\\\\\\\\.\\\\RSSDT"),   
>       GENERIC\_READ,   
>       FILE\_SHARE\_READ | FILE\_SHARE\_WRITE,  
>       NULL,   
>       OPEN\_EXISTING,   
>       0,   
>       NULL  
>       );  
>     GetLastError();  
>   
>     if ( hDevice != INVALID\_HANDLE\_VALUE )  
>     {  
>       bResult = DeviceIoControl(hDevice, IOCTL\_RSSDT\_ENTRY\_SET, &SstEntry, sizeof (SstEntry),   
>         &SstEntry, sizeof (SstEntry), &cbReturn, NULL);  
>   
>       CloseHandle(hDevice);  
>     }  
>   }  
>   
>   UninstallDriver(hScService);  
>   return bResult;  
> }  
>   
> #if 1  
> int \_\_cdecl main()  
> {  
>   ResetSSTEntry("NtCreateKey");  
>   ResetSSTEntry("NtCreatePagingFile");  
>   ResetSSTEntry("NtEnumerateValueKey");  
>   ResetSSTEntry("NtOpenKey");  
>   ResetSSTEntry("NtQueryKey");  
>   ResetSSTEntry("NtQueryValueKey");  
>   ResetSSTEntry("NtSetSystemPowerState");  
>   
>   return 0;  
> }  
>   
> #endif 

 4dd466d0e1e0cc123afbe504da2462ca