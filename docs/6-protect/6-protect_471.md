

# Hide your DebugPort in ring0

> 原文：[`www.pediy.com/kssd/pediy10/80971.html`](https://www.pediy.com/kssd/pediy10/80971.html)

一个程序被 ring3 调试器调试时，有很多的调试特征可以检测，本论坛也有专门的帖子详细论述，但有个非常根本的标志 ring3 也是可以检测的比较少人提及，那就是\_EPROCESS.DebugPort。DebugPort 对于 ring3 调试器来说非常重要，没有它正常的 ring3 调试是无法进行的。当然要检测这个标志的前提是程序能够读取 ring0 内存，在 XP 以上的系统有个非常简单的方法就是使用 ZwSystemDebugControl 的 SysDbgReadVirtualMemory 方法，我们也可以 map physicalmemory 来操作。检测 DebugPort 之前首先要得到进程的 eprocess 地址，这可以通过 ZwQuerySystemInformation 的 SystemHandleInformation 方法得到，也可以直接搜索 ring0 内存的 eprocess 结构。  
  
   对于 ring3 直接检测 DebugPort，我们可以通过禁止该进程访问 ring0 内存来对付，但是目标一旦使用驱动来检测，那么就非常麻烦了。下面介绍一种隐藏\_EPROCESS.DebugPort 的方法，这种方法的基本思路是，将一个正常被调试进程的 DebugPort 置零后，修正所有受影响的函数，使我们的调试器能够正常进行。这些函数如下：  
    PspCreateProcess、MmCreatePeb 进程创建，设置 DebugPort  
    DbgkCreateThread 发送线程或者进程创建的调试信息  
    KiDispatchException、DbgkForwardException 和 DbgkpQueueMessage 发送异常调试信息  
    PspExitThread、DbgkExitThread 和 DbgkExitProcess 发送线程退出、进程退出的调试信息  
    DbgkMapViewOfSection 和 DbgkUnMapViewOfSection 发送映像装载卸载调试信息  
    DbgkpSetProcessDebugObject 和 DbgkpMarkProcessPeb 当调试器附加进程时设置 DebugPort     
  
     这类函数非常多的，如果都 HOOK 处理的话，那太恐怖了，这里使用一个非常简单的办法：偷龙转凤。我们看系统访问 DebugPort 的代码都是这样的（XP）  
         8b89bc000000    mov     ecx,dword ptr \[ecx+0BCh\] //0BCh 就是 DebugPort 的偏移  
  
     我们可以把 DebugPort 转移到\_EPROCESS 的另外一个地方，比如我使用+0x070 CreateTime，它是纪录进程创建时间的，进程创建之后，在进程退出前系统不会对它进行任何修改，而且我们修改后对系统或进程没有任何影响。这样我们可以把上面的代码改成这样  
         8b8970000000    mov     ecx,dword ptr \[ecx+070h\] //指向 CreateTime，实际的 DebugPort 已经被移到这里  
     只需要修改一个字节，非常简单。  
  
     当然这种方法最麻烦的地方就是定位引用到 DebugPort 的函数（本人仅仅针对不同的 XP 系统制作特征码都累到吐血），这些函数都是不导出的，如果是特定系统，最简单的方法就是 WinDbg->uf \*\*\* 直接找地址硬编码，只需要几分钟时间。  
    

代码:

```
BOOLEAN InitHackAddress()
{
  \_SEH\_TRY
  {
    g\_KernelBase = GetKernelBaseAndSize( &g\_KernelSize );      
    g\_HackPspCreateProcess = SearchHackPspCreateProcess( &g\_NopPspCreateProcess.Address );
    g\_HackKiDispatchException = SearchKiDispatchException( g\_KernelBase,g\_KernelSize );  
    g\_HackDbgkpQueueMessage = SearchDbgkpQueueMessage( g\_KernelBase,g\_KernelSize );
    g\_HackDbgkCreateThread = SearchDbgkCreateThread( g\_KernelBase,g\_KernelSize );    
    SearchDbgkNotifyRoutine( g\_KernelBase,g\_KernelSize );
    g\_HackPspExitThread = SearchPspExitThread();  
    g\_HackMmCreatePeb = SearchMmCreatePeb( g\_HackPspCreateProcess );
    SearchDbgkpSetProcessDebugObject( g\_KernelBase,g\_KernelSize );
    if( g\_HackDbgkpSetProcessDebugObject\[3\] )
      g\_HackDbgkpMarkProcessPeb = SearchDbgkpMarkProcessPeb( g\_HackDbgkpSetProcessDebugObject\[3\] ) ;
    
    if( g\_NopPspCreateProcess.Address != 0 ){
      RtlFillMemory( g\_NopPspCreateProcess.NopCode,sizeof(g\_NopPspCreateProcess.NopCode),0x90 ); 
      g\_NopPspCreateProcess.Size = 9;
      RtlCopyMemory( g\_NopPspCreateProcess.OrigCode,(PVOID)g\_NopPspCreateProcess.Address,g\_NopPspCreateProcess.Size );
    }

    if( g\_NopDbgkForwardException.Address != 0 ){      
      RtlFillMemory( g\_NopDbgkForwardException.NopCode,sizeof(g\_NopDbgkForwardException.NopCode),0x90 );
      RtlCopyMemory( g\_NopDbgkForwardException.OrigCode,(PVOID)g\_NopDbgkForwardException.Address,g\_NopDbgkForwardException.Size );
    }

    if( g\_NopDbgkExitThread.Address != 0 ){      
      RtlFillMemory( g\_NopDbgkExitThread.NopCode,sizeof(g\_NopDbgkExitThread.NopCode),0x90 );
      RtlCopyMemory( g\_NopDbgkExitThread.OrigCode,(PVOID)g\_NopDbgkExitThread.Address,g\_NopDbgkExitThread.Size );
    }

    if( g\_NopDbgkExitProcess.Address != 0 ){
      RtlFillMemory( g\_NopDbgkExitProcess.NopCode,sizeof(g\_NopDbgkExitProcess.NopCode),0x90 );
      RtlCopyMemory( g\_NopDbgkExitProcess.OrigCode,(PVOID)g\_NopDbgkExitProcess.Address,g\_NopDbgkExitProcess.Size );
    }

    if( g\_NopDbgkMapViewOfSection.Address != 0){
      RtlFillMemory( g\_NopDbgkMapViewOfSection.NopCode,sizeof(g\_NopDbgkMapViewOfSection.NopCode),0x90 );
      RtlCopyMemory( g\_NopDbgkMapViewOfSection.OrigCode,(PVOID)g\_NopDbgkMapViewOfSection.Address,g\_NopDbgkMapViewOfSection.Size );
    }

    if( g\_NopDbgkUnMapViewOfSection.Address != 0 ){
      RtlFillMemory( g\_NopDbgkUnMapViewOfSection.NopCode,sizeof(g\_NopDbgkUnMapViewOfSection.NopCode),0x90 );
      RtlCopyMemory( g\_NopDbgkUnMapViewOfSection.OrigCode,(PVOID)g\_NopDbgkUnMapViewOfSection.Address,g\_NopDbgkUnMapViewOfSection.Size );
    }     
    
    
  }
  \_SEH\_HANDLER
  {    
    DbgPrint( "InitHackAddress Exception!\\n" );
  }

  return ( g\_HackPspCreateProcess != 0 &&
         g\_HackKiDispatchException != 0 &&
       g\_HackDbgkForwardException != 0 &&
       g\_HackDbgkpQueueMessage != 0 && 
       g\_NopPspCreateProcess.Address != 0 &&
       g\_NopDbgkForwardException.Address != 0 &&
       g\_HackDbgkCreateThread != 0 &&
       g\_HackDbgkExitThread != 0 &&
       g\_NopDbgkExitThread.Address != 0 &&
       g\_HackDbgkExitProcess != 0 &&
       g\_NopDbgkExitProcess.Address != 0 &&
       g\_HackDbgkMapViewOfSection != 0 &&
       g\_NopDbgkMapViewOfSection.Address != 0 &&
       g\_HackDbgkUnMapViewOfSection != 0 &&
       g\_NopDbgkUnMapViewOfSection.Address != 0 &&
       g\_HackPspExitThread != 0 &&
       g\_HackMmCreatePeb != 0 &&
       g\_HackDbgkpSetProcessDebugObject\[0\] != 0 &&
       g\_HackDbgkpSetProcessDebugObject\[1\] != 0 &&
       g\_HackDbgkpSetProcessDebugObject\[2\] != 0 &&
       g\_HackDbgkpSetProcessDebugObject\[3\] != 0 &&
       g\_HackDbgkpMarkProcessPeb != 0 );
}

```

代码:

```
//修改已经运行进程的 DebugPort 位置
BOOLEAN ChangeProcessDebugPort( BOOLEAN Hide )
{
  ULONG eProcess = (ULONG)PsInitialSystemProcess;
  PLIST\_ENTRY pListHead,pListWalk;
  ULONG DebugObject;

  if( !g\_bIsAddressStartup ){
    return FALSE;
  }

  pListHead = (PLIST\_ENTRY)( eProcess + ACTIVE\_LINKS\_OFFSET );
  pListWalk = pListHead;

  \_SEH\_TRY
  {    
    do{
      if( pListWalk == NULL || eProcess == 0 )
        break;

      eProcess = ( (ULONG)pListWalk - ACTIVE\_LINKS\_OFFSET );    

      if( Hide ){

        DebugObject = \*(ULONG\*)( eProcess + DEBUG\_PORT\_OFFSET );
        \*(ULONG\*)( eProcess + CREATE\_TIME\_OFFSET ) = DebugObject;
        \*(ULONG\*)( eProcess + DEBUG\_PORT\_OFFSET ) = 0;

      }else{

        DebugObject = \*(ULONG\*)( eProcess + CREATE\_TIME\_OFFSET );
        \*(ULONG\*)( eProcess + DEBUG\_PORT\_OFFSET ) = DebugObject;
      }

      pListWalk = pListWalk->Flink;

    }while( pListWalk != pListHead );

  }
  \_SEH\_HANDLER
  {
    DbgPrint( "ChangeProcessDebugPort exception!\\n" );
  }
  
  return TRUE;
}

```

代码:

```
BOOLEAN ModifyDebugFunction()
{
  if( !g\_bIsAddressStartup ){
    return FALSE;
  }

  \_\_asm{
    cli
    mov  eax,cr0
    and  eax,not 10000h
    mov  cr0,eax
  }

  \*(ULONG\*)g\_HackPspCreateProcess = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackKiDispatchException = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkForwardException = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpQueueMessage = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkCreateThread = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkExitThread = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkExitProcess = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkMapViewOfSection = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkUnMapViewOfSection = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackPspExitThread = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpMarkProcessPeb = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackMmCreatePeb = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[0\] = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[1\] = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[2\] = CREATE\_TIME\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[3\] = CREATE\_TIME\_OFFSET;
    

  RtlCopyMemory( (PVOID)g\_NopPspCreateProcess.Address,g\_NopPspCreateProcess.NopCode,g\_NopPspCreateProcess.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkForwardException.Address,g\_NopDbgkForwardException.NopCode,g\_NopDbgkForwardException.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkExitThread.Address,g\_NopDbgkExitThread.NopCode,g\_NopDbgkExitThread.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkExitProcess.Address,g\_NopDbgkExitProcess.NopCode,g\_NopDbgkExitProcess.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkMapViewOfSection.Address,g\_NopDbgkMapViewOfSection.NopCode,g\_NopDbgkMapViewOfSection.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkUnMapViewOfSection.Address,g\_NopDbgkUnMapViewOfSection.NopCode,g\_NopDbgkUnMapViewOfSection.Size );

  \_\_asm{
    mov  eax,cr0
    or   eax,10000h
    mov  cr0,eax
    sti
  }

  return TRUE;

}

BOOLEAN WriteBackDebugFunction()
{
  if( !g\_bIsAddressStartup ){
    return FALSE;
  }

  \_\_asm{
    cli
    mov  eax,cr0
    and  eax,not 10000h
    mov  cr0,eax
  }

  \*(ULONG\*)g\_HackPspCreateProcess = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackKiDispatchException = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkForwardException = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpQueueMessage = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkCreateThread = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkExitThread = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkExitProcess = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkMapViewOfSection = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkUnMapViewOfSection = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackPspExitThread = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpMarkProcessPeb = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackMmCreatePeb = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[0\] = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[1\] = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[2\] = DEBUG\_PORT\_OFFSET;
  \*(ULONG\*)g\_HackDbgkpSetProcessDebugObject\[3\] = DEBUG\_PORT\_OFFSET;

  RtlCopyMemory( (PVOID)g\_NopPspCreateProcess.Address,g\_NopPspCreateProcess.OrigCode,g\_NopPspCreateProcess.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkForwardException.Address,g\_NopDbgkForwardException.OrigCode,g\_NopDbgkForwardException.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkExitThread.Address,g\_NopDbgkExitThread.OrigCode,g\_NopDbgkExitThread.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkExitProcess.Address,g\_NopDbgkExitProcess.OrigCode,g\_NopDbgkExitProcess.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkMapViewOfSection.Address,g\_NopDbgkMapViewOfSection.OrigCode,g\_NopDbgkMapViewOfSection.Size );
  RtlCopyMemory( (PVOID)g\_NopDbgkUnMapViewOfSection.Address,g\_NopDbgkUnMapViewOfSection.OrigCode,g\_NopDbgkUnMapViewOfSection.Size );

  \_\_asm{
    mov  eax,cr0
    or   eax,10000h
    mov  cr0,eax
    sti
  }

  return TRUE;
}

```

     我想再嗦一下，上面代码大家看到很多函数有个 NOPCode，这个实际上是对付线程 PS\_CROSS\_THREAD\_FLAGS\_HIDEFROMDBG 的，NOP 掉相关地方后，就算线程被设置为 ThreadHideFromDebugger 也无法阻挡调试器接收调试信息。  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
祝大家牛年技术越来越牛，愿我等小菜在牛年可以望得见牛人项背！