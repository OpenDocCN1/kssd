
# 一个 PsSetLoadImageNotifyRoutine 回调内核注入 DLL，支持 xp ~ win7 源码

> 原文：[`www.pediy.com/kssd/pediy12/139986.html`](https://www.pediy.com/kssd/pediy12/139986.html)

一个`PsSetLoadImageNotifyRoutine`回调内核注入`DLL`，支持`xp`~`win7`源码

最近看`sudimi`大牛写的《一个有趣的内核注`DLL`的`sys`》文章，确实很有趣，可惜没开源，俺也写个，把完整代码也贴上来了，欢迎拍砖。`![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")`

安装驱动，打开`dbgview.exe`，注入`mydll.dll`！！

```c
VOID Start (  
   IN PUNICODE_STRING  FullImageName,  
   IN HANDLE  ProcessId, // where image is mapped  
   IN PIMAGE_INFO  ImageInfo  
   )  
{  
  NTSTATUS ntStatus;  
  PIMAGE_IMPORT_DESCRIPTOR pImportNew;  
  HANDLE hProcessHandle;  
  int nImportDllCount = 0;  
  int size;  
  // PID 改变时，可以说明已经注入 DLL 了。  
  if(g_ulPid != (ULONG)ProcessId && g_ulPid != 0 && g_psaveDes != NULL)  
  {  
    KAPC_STATE apcState;  
    BOOLEAN bAttached = FALSE;  
    //_asm int 3;  
    if(g_psaveDes!= NULL)  
    {  
      if(PsGetCurrentProcess() != g_eprocess) {  
        KeStackAttachProcess((PRKPROCESS)g_eprocess, &apcState);  
        bAttached = TRUE;  
      }  
      //  
      __asm {  
        cli;  
        mov eax, cr0;  
        mov oldCr0, eax;  
        and eax, not 10000h;  
        mov cr0, eax  
      }  
      // 改导出表  
      pHeader->OptionalHeader.DataDirectory[1].Size -= sizeof(IMAGE_IMPORT_DESCRIPTOR);  
      pHeader->OptionalHeader.DataDirectory[1].VirtualAddress = (ULONG)g_psaveDes - ulBaseImage;  
      __asm {  
        mov eax, oldCr0;  
        mov cr0, eax;  
        sti;  
      }  
      g_psaveDes = NULL;  
      if(bAttached)  
        KeUnstackDetachProcess(&apcState);  
    }  
    return ;  
  }  
  // 注入进程没退出。  
  if(g_eprocess != NULL)  
    return;  
   if(_stricmp(PsGetProcessImageFileName(PsGetCurrentProcess()), "dbgview.exe") == 0)  
  {  
    //_asm int 3;  
    g_eprocess = PsGetCurrentProcess();  
    g_ulPid = (ULONG )ProcessId;  
     ulBaseImage = (ULONG)ImageInfo->ImageBase;// 进程基地址  
    pDos =(PIMAGE_DOS_HEADER) ulBaseImage;  
    pHeader = (PIMAGE_NT_HEADERS)(ulBaseImage+(ULONG)pDos->e_lfanew);  
    pImportDesc  = (PIMAGE_IMPORT_DESCRIPTOR)((ULONG)pHeader->OptionalHeader.DataDirectory[1].VirtualAddress + ulBaseImage);  
    // 导入 DLL 个数     nImportDllCount = pHeader->OptionalHeader.DataDirectory[1].Size / sizeof(IMAGE_IMPORT_DESCRIPTOR);  
    // 把原始值保存。  
    g_psaveDes = pImportDesc;  
    ntStatus = ObOpenObjectByPointer(g_eprocess, OBJ_KERNEL_HANDLE, NULL, 0x008, //PROCESS_VM_OPERATION  
      NULL, KernelMode, &hProcessHandle);  
    if(!NT_SUCCESS(ntStatus))   
      return ;  
    //                   加上一个自己的结构。  
    size = sizeof(IMAGE_IMPORT_DESCRIPTOR) * (nImportDllCount + 1);  
    //  分配导入表  
    ntStatus = ZwAllocateVirtualMemory(hProcessHandle, &lpBuffer, 0, &size,  
      MEM_COMMIT, PAGE_EXECUTE_READWRITE);  
    if(!NT_SUCCESS(ntStatus)) {  
      ZwClose(hProcessHandle);  
      return ;  
    }  
    RtlZeroMemory(lpBuffer,sizeof(IMAGE_IMPORT_DESCRIPTOR) * (nImportDllCount + 1));  
    size = 20;  
    // 分配当前进程空间。  
    ntStatus = ZwAllocateVirtualMemory(hProcessHandle, &lpDllName, 0, &size,  
      MEM_COMMIT, PAGE_EXECUTE_READWRITE);  
    if(!NT_SUCCESS(ntStatus)) {  
      ZwClose(hProcessHandle);  
      return ;  
    }  
    RtlZeroMemory(lpDllName,20);  
    /* 分配导出函数的进程地址空间。  
      注意这里是分配高位地址的。如分配低位地址，得到 PIMAGE_IMPORT_BY_NAME 结构的地址会以，如 ff 等开头的负数地址，  
      这样，系统就会默认按序号查找 API 地址，会弹出找不到序号的框框。  
    */  
    size = 20;  
    ntStatus = ZwAllocateVirtualMemory(hProcessHandle, &lpExportApi, 0, &size,  
      MEM_COMMIT|MEM_TOP_DOWN, PAGE_EXECUTE_READWRITE);  
    if(!NT_SUCCESS(ntStatus)) {  
      ZwClose(hProcessHandle);  
      return ;  
    }  
    RtlZeroMemory(lpExportApi,20);  
    // 分配当前进程空间。  
    size = 20;  
    ntStatus = ZwAllocateVirtualMemory(hProcessHandle, &lpTemp, 0, &size,  
      MEM_COMMIT, PAGE_EXECUTE_READWRITE);  
    if(!NT_SUCCESS(ntStatus)) {  
      ZwClose(hProcessHandle);  
      return ;  
    }  
    RtlZeroMemory(lpTemp,20);  
    // 分配当前进程空间。  
    size = 20;  
    ntStatus = ZwAllocateVirtualMemory(hProcessHandle, &lpTemp2, 0, &size,  
      MEM_COMMIT, PAGE_EXECUTE_READWRITE);  
    if(!NT_SUCCESS(ntStatus)) {  
      ZwClose(hProcessHandle);  
      return ;  
    }  
    RtlZeroMemory(lpTemp2,20);  
    pImportNew = lpBuffer;  
    // 把原来数据保存好。  
    RtlCopyMemory(pImportNew+1, pImportDesc, sizeof(IMAGE_IMPORT_DESCRIPTOR) * nImportDllCount );  
    // 构造自己的 DLL    IMAGE_IMPORT_DESCRIPTOR 结构  
    {  
      IMAGE_IMPORT_DESCRIPTOR Add_ImportDesc;  
      PULONG ulAddress;  
      ULONG oldCr0;  
      ULONG Func;  
      PIMAGE_IMPORT_BY_NAME ptmp;  
      IMAGE_THUNK_DATA  *pThunkData=(PIMAGE_THUNK_DATA)lpTemp2;  
      //_asm int 3;  
      //  
      // ThunkData 结构的地址  
      //  
      ptmp  =  (PIMAGE_IMPORT_BY_NAME)lpExportApi;  
      ptmp->Hint=0;  
      // 至少要一个导出 API  
      RtlCopyMemory(ptmp->Name,"ExportedFunction",18);  
      *(PULONG)lpTemp =(DWORD)ptmp-ulBaseImage;  
      pThunkData->u1.AddressOfData  =(DWORD)ptmp-ulBaseImage;  
      //pThunkData[1].u1.AddressOfData  =  0;//导出函数截断  
      Add_ImportDesc.Characteristics = (DWORD)pThunkData-ulBaseImage;  
      Add_ImportDesc.TimeDateStamp = 0;  
      Add_ImportDesc.ForwarderChain = 0;  
      //  
      // DLL 名字的 RVA  
      //     RtlCopyMemory(lpDllName,"mydll.dll",20);  
      Add_ImportDesc.Name = (DWORD)lpDllName-ulBaseImage;  
      Add_ImportDesc.FirstThunk = Add_ImportDesc.Characteristics;  
      RtlCopyMemory(pImportNew, &Add_ImportDesc, sizeof(IMAGE_IMPORT_DESCRIPTOR));  
      __asm {  
        cli;  
        mov eax, cr0;  
        mov oldCr0, eax;  
        and eax, not 10000h;  
        mov cr0, eax  
      }  
      // 改导出表  
      pHeader->OptionalHeader.DataDirectory[1].Size += sizeof(IMAGE_IMPORT_DESCRIPTOR);  
      pHeader->OptionalHeader.DataDirectory[1].VirtualAddress = (ULONG)pImportNew - ulBaseImage;  
      __asm {  
        mov eax, oldCr0;  
        mov cr0, eax;  
        sti;  
      }  
    }  
    ZwClose(hProcessHandle);  
    hProcessHandle = NULL;  
  }  
}
```

上传的附件

| `![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg)` | `kp.rar` |
| :--- | :--- |
