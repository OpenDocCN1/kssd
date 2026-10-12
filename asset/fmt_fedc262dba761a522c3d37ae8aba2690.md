
```cpp
        MOV ECX, [dwStackSize]  
        MOV EDX, [NextIpAddr]  
        MOV EAX, [Retvalue]  
        MOV ESP, EBP  
        POP EBP  
        ADD ESP, ECX   //恢复堆栈  
        PUSH EDX  
        RET  
    }  
  }  
  lpMapFileToNewProcessId = pfnMapViewOfFile(hMapFileToNewProcessId, FILE_MAP_ALL_ACCESS, 0, 0, 0);  
  if(lpMapFileToNewProcessId == NULL)  
  {  
    pfnCloseHandle(hMapFileToNewProcessId);  
    __asm  
    {  
      POP EDI      
      POP ESI      
      POP EBX  
      MOV ECX, [dwStackSize]  
      MOV EDX, [NextIpAddr]  
      MOV EAX, [Retvalue]  
      MOV ESP, EBP  
      POP EBP  
      ADD ESP, ECX   //恢复堆栈  
      PUSH EDX  
      RET  
    }  
  }  
  hMapFileOperateMutex = pfnOpenMutex(MUTEX_MODIFY_STATE, FALSE, szMapFileOperateMutex);  
  if(hMapFileOperateMutex == NULL)  
  {  
    pfnUnMapViewOfFile(lpMapFileToNewProcessId);  
    pfnCloseHandle(hMapFileToNewProcessId);  
    __asm  
    {  
      POP EDI      
      POP ESI      
      POP EBX  
      MOV ECX, [dwStackSize]  
      MOV EDX, [NextIpAddr]  
      MOV EAX, [Retvalue]  
      MOV ESP, EBP  
      POP EBP  
      ADD ESP, ECX   //恢复堆栈  
      PUSH EDX  
      RET  
    }  
  }  
  DWORD dwPid = pfnGetProcessId(*ProcessHandle);  
  if(dwPid == 0)  
  {  
    pfnUnMapViewOfFile(lpMapFileToNewProcessId);  
    pfnCloseHandle(hMapFileToNewProcessId);  
    pfnCloseHandle(hMapFileOperateMutex);  
    __asm  
    {  
      POP EDI      
      POP ESI      
      POP EBX  
      MOV ECX, [dwStackSize]  
      MOV EDX, [NextIpAddr]  
      MOV EAX, [Retvalue]  
      MOV ESP, EBP  
      POP EBP  
      ADD ESP, ECX   //恢复堆栈  
      PUSH EDX  
      RET  
    }  
  }  
  pfnWaitForSingleObject(hMapFileOperateMutex, INFINITE);  
  DWORD * temp2 = (DWORD*)lpMapFileToNewProcessId;  
  *temp2 = dwPid;  
  pfnUnMapViewOfFile(lpMapFileToNewProcessId);  
  pfnCloseHandle(hMapFileToNewProcessId);  
  hMapFileModifiedEvent = pfnOpenEvent(EVENT_MODIFY_STATE, NULL, szMapFileModifiedEvent);  
  if(hMapFileModifiedEvent == NULL)  
  {  
    pfnReleaseMutex(hMapFileOperateMutex);  
    pfnCloseHandle(hMapFileOperateMutex);  
    __asm  
    {  
      POP EDI      
      POP ESI      
      POP EBX  
      MOV ECX, [dwStackSize]  
      MOV EDX, [NextIpAddr]  
      MOV EAX, [Retvalue]  
      MOV ESP, EBP  
      POP EBP  
      ADD ESP, ECX   //恢复堆栈  
      PUSH EDX  
      RET  
    }  
  }  
  pfnSetEvent(hMapFileModifiedEvent);  
  hHookNewProcessDoneEvent = pfnOpenEvent(EVENT_MODIFY_STATE, NULL, szHookNewProcessDoneEvent);  
   if(hHookNewProcessDoneEvent == NULL)  
  {  
    pfnReleaseMutex(hMapFileOperateMutex);  
    pfnCloseHandle(hMapFileOperateMutex);  
    pfnCloseHandle(hMapFileModifiedEvent);  
    __asm  
    {  
      POP EDI      
      POP ESI      
      POP EBX  
      MOV ECX, [dwStackSize]  
      MOV EDX, [NextIpAddr]  
      MOV EAX, [Retvalue]  
      MOV ESP, EBP  
      POP EBP  
      ADD ESP, ECX   //恢复堆栈  
      PUSH EDX  
      RET  
    }  
  }  
  pfnWaitForSingleObject(hHookNewProcessDoneEvent, INFINITE);  
  pfnCloseHandle(hMapFileModifiedEvent);  
  pfnCloseHandle(hHookNewProcessDoneEvent);  
  pfnReleaseMutex(hMapFileOperateMutex);  
  pfnCloseHandle(hMapFileOperateMutex);  
  }  
  else  
  {  
    pfnCloseHandle(hHookMainAliveEvent);  
  }  
  //这里对拦截函数堆栈的恢复  
  __asm  
  {  
    POP EDI      
    POP ESI      
    POP EBX  
    MOV ECX, [dwStackSize]  
    MOV EDX, [NextIpAddr]  
    MOV EAX, [Retvalue]  
    MOV ESP, EBP  
    POP EBP  
    ADD ESP, ECX   //恢复堆栈  
    PUSH EDX  
    RET  
  }      
}  

//////////////////////////////////////////////////////////////////////////  
//Hook ZwSetValueKey  
//Returns STATUS_SUCCESS or an error status, such as STATUS_ACCESS_DENIED,  
//STATUS_INVALID_HANDLE, STATUS_KEY_DELETED, or STATUS_NO_LOG_SPACE.  
//////////////////////////////////////////////////////////////////////////  

void HookZwSetValueKey(LPVOID lParam)  
{  
    RemoteParam* Rpm = (RemoteParam*)lParam;  
    PFN_ZWSETVALUEKEY pfnZwSetValueKey = (PFN_ZWSETVALUEKEY)Rpm->lpFunAddr;  
    PFN_WRITEPROCESSMEMORY pfnWriteProcessMemory = (PFN_WRITEPROCESSMEMORY)Rpm->lpWriteProcessMemory;  
    PFN_MESSAGEBOX pfnMessageBox = (PFN_MESSAGEBOX)Rpm->lpMessageBox;  
    PFN_ZWQUERYOBJECT pfnZwQueryObject = (PFN_ZWQUERYOBJECT)Rpm->lpZwQueryObject;  
    PFN_WSPRINTF pfnwsprintf = (PFN_WSPRINTF)Rpm->lpwsprintf;  
    PFN_GETPROCESSHEAP pfnGetProcessHeap = (PFN_GETPROCESSHEAP)Rpm->lpGetProcessHeap;  
    PFN_HEAPALLOC pfnHeapAlloc = (PFN_HEAPALLOC)Rpm->lpHeapAlloc;  
    PFN_HEAPREALLOC pfnHeapReAlloc = (PFN_HEAPREALLOC)Rpm->lpHeapReAlloc;  
    PFN_HEAPFREE pfnHeapFree = (PFN_HEAPFREE)Rpm->lpHeapFree;  
    PFN_CREATEEVENT pfnCreateEvent = (PFN_CREATEEVENT)Rpm->lpCreateEvent;  
    PFN_GETLASTERROR pfnGetLastError = (PFN_GETLASTERROR)Rpm->lpGetLastError;  
    PFN_CLOSEHANDLE pfnCloseHandle = (PFN_CLOSEHANDLE)Rpm->lpCloseHandle;  
    HANDLE KeyHandle = NULL;  
    PUNICODE_STRING ValueName = NULL;  
    ULONG TitleIndex = 0;  
    ULONG type1 = 0;  
    PVOID Data = NULL;  
    ULONG DataSize = 0;  
    NTSTATUS Retvalue = STATUS_SUCCESS; //定义要拦截的 api 的默认返回植  
    DWORD NextIpAddr = 0;  
    DWORD dwParamaAddr = 0;  
    DWORD dwStackSize = 12 + Rpm->dwParamSize;  
    __asm  
    {  
        MOV EAX, [EBP + 12]  
        MOV [NextIpAddr], EAX  
        MOV EAX, [EBP + 16]  
        MOV [KeyHandle], EAX  
        MOV EAX, [EBP + 20]  
        MOV [ValueName], EAX  
        MOV EAX, [EBP + 24]  
        MOV [TitleIndex], EAX  
        MOV EAX, [EBP + 28]  
        MOV [type1], EAX  
        MOV EAX, [EBP + 32]  
        MOV [Data], EAX  
        MOV EAX, [EBP + 36]  
        MOV [DataSize], EAX            
    }  
    HANDLE hHeap = pfnGetProcessHeap();  
    DWORD retSize = 0;  
    POBJECT_NAME_INFORMATION pName = (POBJECT_NAME_INFORMATION)pfnHeapAlloc(hHeap, HEAP_ZERO_MEMORY, BLOCKSIZE);     
    NTSTATUS ns = pfnZwQueryObject(KeyHandle, ObjectNameInformation, (PVOID)pName, BLOCKSIZE, &retSize);  
    DWORD i = 1;  
    while(ns == STATUS_INFO_LEN_MISMATCH)  
    {  
        pName = (POBJECT_NAME_INFORMATION)pfnHeapReAlloc(hHeap, HEAP_ZERO_MEMORY, (LPVOID)pName, BLOCKSIZE * i);  
        ns = pfnZwQueryObject(KeyHandle, ObjectNameInformation, (PVOID)pName, BLOCKSIZE, NULL);  
        i++;  
    }  
    char szHookMainAliveEvent[20] = {'H', 'o', 'o', 'k', 'M', 'a', 'i', 'n', 'A',  
                                     'l', 'i', 'v', 'e', 'E', 'v', 'e', 'n', 't', '\0'};    
    wchar_t wObjectPath[260] = {'\0'};  
```
