
# Hide your DebugPort in ring0

> 原文：[`www.pediy.com/kssd/pediy10/80971.html`](https://www.pediy.com/kssd/pediy10/80971.html)

一个程序被 ring3 调试器调试时，有很多的调试特征可以检测，本论坛也有专门的帖子详细论述，但有个非常根本的标志 ring3 也是可以检测的比较少人提及，那就是`_EPROCESS.DebugPort`。`DebugPort` 对于 ring3 调试器来说非常重要，没有它正常的 ring3 调试是无法进行的。当然要检测这个标志的前提是程序能够读取 ring0 内存，在 XP 以上的系统有个非常简单的方法就是使用 `ZwSystemDebugControl` 的 `SysDbgReadVirtualMemory` 方法，我们也可以 `map physicalmemory` 来操作。检测 `DebugPort` 之前首先要得到进程的 `eprocess` 地址，这可以通过 `ZwQuerySystemInformation` 的 `SystemHandleInformation` 方法得到，也可以直接搜索 ring0 内存的 `eprocess` 结构。

对于 ring3 直接检测 `DebugPort`，我们可以通过禁止该进程访问 ring0 内存来对付，但是目标一旦使用驱动来检测，那么就非常麻烦了。下面介绍一种隐藏`_EPROCESS.DebugPort` 的方法，这种方法的基本思路是，将一个正常被调试进程的 `DebugPort` 置零后，修正所有受影响的函数，使我们的调试器能够正常进行。这些函数如下：

*   `PspCreateProcess`、`MmCreatePeb` 进程创建，设置 `DebugPort`
*   `DbgkCreateThread` 发送线程或者进程创建的调试信息
*   `KiDispatchException`、`DbgkForwardException` 和 `DbgkpQueueMessage` 发送异常调试信息
*   `PspExitThread`、`DbgkExitThread` 和 `DbgkExitProcess` 发送线程退出、进程退出的调试信息
*   `DbgkMapViewOfSection` 和 `DbgkUnMapViewOfSection` 发送映像装载卸载调试信息
*   `DbgkpSetProcessDebugObject` 和 `DbgkpMarkProcessPeb` 当调试器附加进程时设置 `DebugPort`

这类函数非常多的，如果都 `HOOK` 处理的话，那太恐怖了，这里使用一个非常简单的办法：偷龙转凤。我们看系统访问 `DebugPort` 的代码都是这样的（XP）

```
8b89bc000000    mov     ecx,dword ptr [ecx+0BCh] //0BCh 就是 DebugPort 的偏移
```

我们可以把 `DebugPort` 转移到`_EPROCESS` 的另外一个地方，比如我使用`+0x070 CreateTime`，它是纪录进程创建时间的，进程创建之后，在进程退出前系统不会对它进行任何修改，而且我们修改后对系统或进程没有任何影响。这样我们可以把上面的代码改成这样

```
8b8970000000    mov     ecx,dword ptr [ecx+070h] //指向 CreateTime，实际的 DebugPort 已经被移到这里
```

只需要修改一个字节，非常简单。

当然这种方法最麻烦的地方就是定位引用到 `DebugPort` 的函数（本人仅仅针对不同的 XP 系统制作特征码都累到吐血），这些函数都是不导出的，如果是特定系统，最简单的方法就是 `WinDbg->uf ***` 直接找地址硬编码，只需要几分钟时间。

代码:

```
BOOLEAN InitHackAddress()
{
  _SEH_TRY
  {
    g_KernelBase = GetKernelBaseAndSize( &g_KernelSize );       
    g_HackPspCreateProcess = SearchHackPspCreateProcess( &g_NopPspCreateProcess.Address );
    g_HackKiDispatchException = SearchKiDispatchException( g_KernelBase,g_KernelSize );   
    g_HackDbgkpQueueMessage = SearchDbgkpQueueMessage( g_KernelBase,g_KernelSize );
    g_HackDbgkCreateThread = SearchDbgkCreateThread( g_KernelBase,g_KernelSize );     
    SearchDbgkNotifyRoutine( g_KernelBase,g_KernelSize );
    g_HackPspExitThread = SearchPspExitThread();   
    g_HackMmCreatePeb = SearchMmCreatePeb( g_HackPspCreateProcess );
    SearchDbgkpSetProcessDebugObject( g_KernelBase,g_KernelSize );
    if( g_HackDbgkpSetProcessDebugObject[3] )
      g_HackDbgkpMarkProcessPeb = SearchDbgkpMarkProcessPeb( g_HackDbgkpSetProcessDebugObject[3] ) ;
     
    if( g_NopPspCreateProcess.Address != 0 ){
      RtlFillMemory( g_NopPspCreateProcess.NopCode,sizeof(g_NopPspCreateProcess.NopCode),0x90 ); 
      g_NopPspCreateProcess.Size = 9;
      RtlCopyMemory( g_NopPspCreateProcess.OrigCode,(PVOID)g_NopPspCreateProcess.Address,g_NopPspCreateProcess.Size );
    }

    if( g_NopDbgkForwardException.Address != 0 ){       
      RtlFillMemory( g_NopDbgkForwardException.NopCode,sizeof(g_NopDbgkForwardException.NopCode),0x90 );
      RtlCopyMemory( g_NopDbgkForwardException.OrigCode,(PVOID)g_NopDbgkForwardException.Address,g_NopDbgkForwardException.Size );
    }

    if( g_NopDbgkExitThread.Address != 0 ){       
      RtlFillMemory( g_NopDbgkExitThread.NopCode,sizeof(g_NopDbgkExitThread.NopCode),0x90 );
      RtlCopyMemory( g_NopDbgkExitThread.OrigCode,(PVOID)g_NopDbgkExitThread.Address,g_NopDbgkExitThread.Size );
    }

    if( g_NopDbgkExitProcess.Address != 0 ){
      RtlFillMemory( g_NopDbgkExitProcess.NopCode,sizeof(g_NopDbgkExitProcess.NopCode),0x90 );
      RtlCopyMemory( g_NopDbgkExitProcess.OrigCode,(PVOID)g_NopDbgkExitProcess.Address,g_NopDbgkExitProcess.Size );
    }

    if( g_NopDbgkMapViewOfSection.Address != 0){
      RtlFillMemory( g_NopDbgkMapViewOfSection.NopCode,sizeof(g_NopDbgkMapViewOfSection.NopCode),0x90 );
      RtlCopyMemory( g_NopDbgkMapViewOfSection.OrigCode,(PVOID)g_NopDbgkMapViewOfSection.Address,g_NopDbgkMapViewOfSection.Size );
    }

    if( g_NopDbgkUnMapViewOfSection.Address != 0 ){
      RtlFillMemory( g_NopDbgkUnMapViewOfSection.NopCode,sizeof(g_NopDbgkUnMapViewOfSection.NopCode),0x90 );
      RtlCopyMemory( g_NopDbgkUnMapViewOfSection.OrigCode,(PVOID)g_NopDbgkUnMapViewOfSection.Address,g_NopDbgkUnMapViewOfSection.Size );
    }     
     
     
  }
  _SEH_HANDLER
  {     
    DbgPrint( "InitHackAddress Exception!\n" );
  }

  return ( g_HackPspCreateProcess != 0 &&
           g_HackKiDispatchException != 0 &&
         g_HackDbgkForwardException != 0 &&
         g_HackDbgkpQueueMessage != 0 && 
         g_NopPspCreateProcess.Address != 0 &&
         g_NopDbgkForwardException.Address != 0 &&
         g_HackDbgkCreateThread != 0 &&
         g_HackDbgkExitThread != 0 &&
         g_NopDbgkExitThread.Address != 0 &&
         g_HackDbgkExitProcess != 0 &&
         g_NopDbgkExitProcess.Address != 0 &&
         g_HackDbgkMapViewOfSection != 0 &&
         g_NopDbgkMapViewOfSection.Address != 0 &&
         g_HackDbgkUnMapViewOfSection != 0 &&
         g_NopDbgkUnMapViewOfSection.Address != 0 &&
         g_HackPspExitThread != 0 &&
         g_HackMmCreatePeb != 0 &&
         g_HackDbgkpSetProcessDebugObject[0] != 0 &&
         g_HackDbgkpSetProcessDebugObject[1] != 0 &&
         g_HackDbgkpSetProcessDebugObject[2] != 0 &&
         g_HackDbgkpSetProcessDebugObject[3] != 0 &&
         g_HackDbgkpMarkProcessPeb != 0 );
}

```

代码:

```
//修改已经运行进程的 DebugPort 位置
BOOLEAN ChangeProcessDebugPort( BOOLEAN Hide )
{
  ULONG eProcess = (ULONG)PsInitialSystemProcess;
  PLIST_ENTRY pListHead,pListWalk;
  ULONG DebugObject;

  if( !g_bIsAddressStartup ){
    return FALSE;
  }

  pListHead = (PLIST_ENTRY)( eProcess + ACTIVE_LINKS_OFFSET );
  pListWalk = pListHead;

  _SEH_TRY
  {     
    do{
      if( pListWalk == NULL || eProcess == 0 )
        break;

      eProcess = ( (ULONG)pListWalk - ACTIVE_LINKS_OFFSET );     

      if( Hide ){

        DebugObject = *(ULONG*)( eProcess + DEBUG_PORT_OFFSET );
        *(ULONG*)( eProcess + CREATE_TIME_OFFSET ) = DebugObject;
        *(ULONG*)( eProcess + DEBUG_PORT_OFFSET ) = 0;

      }else{

        DebugObject = *(ULONG*)( eProcess + CREATE_TIME_OFFSET );
        *(ULONG*)( eProcess + DEBUG_PORT_OFFSET ) = DebugObject;
      }

      pListWalk = pListWalk->Flink;

    }while( pListWalk != pListHead );

  }
  _SEH_HANDLER
  {
    DbgPrint( "ChangeProcessDebugPort exception!\n" );
  }
   
  return TRUE;
}

```

代码:
