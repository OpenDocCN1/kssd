
```
    }  
    _asm  
    {  
      mov eax,ChangAddr  
        push NewOffset  
        pop dword ptr[eax+1]  
    }  
    __asm  
    {  
      MOV   EAX, CR0  
        OR    EAX, 10000H  //enable WP bit  
        MOV   CR0, EAX  
        STI  
    }  
    KeLowerIrql(OldIrql);  
  }  
  //Bug Check 0xD1: DRIVER_IRQL_NOT_LESS_OR_EQUAL  
  else  
  {  
    KIRQL OldIrql=0;  
    KeRaiseIrql(2,&OldIrql);///HIGH_LEVEL  
    __asm  
    {  
      CLI  
        MOV   EAX, CR0  
        AND   EAX, NOT 10000H  //disable WP bit  
        MOV   CR0, EAX  
    }  
    _asm  
    {  
      mov eax,ChangAddr  
        push CallContextOffset  
        pop dword ptr[eax+1]  
    }  
    __asm  
    {  
      MOV   EAX, CR0  
        OR    EAX, 10000H  //enable WP bit  
        MOV   CR0, EAX  
        STI  
    }  
    KeLowerIrql(OldIrql);  
    //    DbgPrint("HookSwapFunctionFALSE");//jution  
  }  
}  
```

```
PEPROCESS processObject (PETHREAD ethread) {  
  return  (PEPROCESS)(ethread->Tcb.ApcState.Process);  
}  
```

```
void  klisterUnload(IN PDRIVER_OBJECT pDriverObject)  
{  
  BeTerminate = 1;  
  while(BeTerminate != 3)    // = 3 时说明创建的线程不是 pending 状态且马上会结束 这时候可以 UNLOAD   否则线程在 PENDING 状态 UNLOADE 会直接蓝  
  {  
  }  
  if (GoBackAddr)//PBYTE GoBackAddr = NULL;  
    HookSwapFunction(FALSE);   
}  
```

```
void showProcess()  
{  
  PProcessList temp;  
  DWORD count = 0;  
  PUCHAR pFileName;  
   temp = wLastItem;  
  while (temp)     //遍历链表  
  {   
      if (temp->pEPROCESS)  
    {  
        count++;  
        pFileName = (PUCHAR)((unsigned int)(temp->pEPROCESS) + 0x174);  
        DbgPrint("0x%08X   %s \n",(unsigned int)(temp->pEPROCESS), pFileName);  
    }  
    temp = PProcessList(temp->NextItem);  
  }  
  DbgPrint("共有%d 个进程", count);  
}  
```

```
void WorkThread(IN PVOID pContext)  
{  
  LARGE_INTEGER timeout;  
  while(true)  
  {  
    if (MmIsAddressValid(&BeTerminate) )    // 因为 BeTerminate 是在 UNLOAD 中设置的 可能驱动卸载后 这个变量不能访问 所以用 MmIsAddressValid 判断下   
    {  
      if(BeTerminate == 0)  
      {  
        //等待单位是 100ns         //-10 作用是转换成微秒  //2000000 微秒=2 秒  
        timeout = RtlConvertLongToLargeInteger(-10             *    2000000);    
        KeDelayExecutionThread(KernelMode, FALSE, &timeout);  
        DbgPrint("搜集到的进程是");         
        showProcess();  
      }  
      else  
      {  
          BeTerminate = 3;  
        PsTerminateSystemThread(STATUS_SUCCESS);  
        goto __end;  
      }  
    }  
    else  
    {  
        BeTerminate = 3;  
      PsTerminateSystemThread(STATUS_SUCCESS);  
      goto __end;   
    }  
  }  
__end:;  
}  
```

```
// 驱动程序加载时调用 DriverEntry 例程  
NTSTATUS DriverEntry(  
           IN PDRIVER_OBJECT  pDriverObject,  
           IN PUNICODE_STRING pRegistryPath  
           )  
{  
  NTSTATUS dwStAtus;  
  HANDLE hThread;  
  pDriverObject->DriverUnload=klisterUnload;  
  dwStAtus = PsCreateSystemThread(&hThread,  
    (ACCESS_MASK)0,  
    NULL,  
    (HANDLE)0,  
    NULL,  
    WorkThread,  
    NULL  
    );   
      GetSwapAddr();  
      if (GoBackAddr){  
      HookSwapFunction(TRUE);  
}   
  return STATUS_SUCCESS;  
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `HookSwapContext.rar`  
 |
