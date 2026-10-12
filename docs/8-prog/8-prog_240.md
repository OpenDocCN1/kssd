
# HOOK SwapContext 枚举隐藏进程(学习笔记 4)

> 原文：[`www.pediy.com/kssd/pediy10/78464.html`](https://www.pediy.com/kssd/pediy10/78464.html)

以前想检测一个被隐藏的进程 后来用暴力枚举的方法解决了 但是 `HOOK` `SwapContext` 没有看到有完整的代码 所以搜集了点网上有用的模块 自己整合实现了下 确定支持 `XP3`, `XP2` 没测试 应该也能支持

附完整工程代码

难点：是得到 `SwapContext` 地址 还有一些细节问题~

原理：`KiSwapContext` 调用 `SwapContext` 时候 `ESI` 存放的是要换进线程的上下文 `EDI` 存放的是要换出线程的上下文 然后截获 `ESI` 即换进的 `ETHREAD` 得到相应的 `EPROCESS` 然后搜集

缺点：对系统性能影响非常大,因为系统线程调度很频繁, `wrk` 的 `KiSwapContext` 用汇编写的就能体现出来对效率的要求高~.

另外有个有趣的问题

`KiSwapContext` 是 `fastcall` 调用方式 就是通过 `ECX` `EDX` 传 多出的按 `STDCALL` 传

`WRK` 是这样注释的

```
; BOOLEAN
; KiSwapContext (
;   IN PKTHREAD OldThread
;   IN PKTHREAD NewThread
;   )
```

但是看下 `WRK` 中的代码

```
mov     edi, ecx                 ; set old thread address
mov     esi, edx                 ; set next thread address
movzx   ecx, byte ptr [edi].ThWaitirql ; set APC interrupt bypass disable
call    SwapContext              ; swap context
```

发现的确是两个参数

用 `WINDBG` 看

```
80545975 8bf1            mov     esi,ecx
80545977 8bbb24010000    mov     edi,dword ptr [ebx+124h]
8054597d 89b324010000    mov     dword ptr [ebx+124h],esi
80545983 8a4f58          mov     cl,byte ptr [edi+58h]
80545986 e8f5000000      call    nt!SwapContext (80545a80)
```

发现 `KiSwapContext` 是一个参数， `esi` 是要换进的线程上下文 是通过 `ECX` (第一个参数得到)  `edi` 则不是

所以结论 `KiSwapContext` 的参数现在是一个了，也许我 `WRK` 没更新 老版本。。

`WRK` `KiSwapContext` 代码

```
cPublicFastCall KiSwapContext, 2
.fpo (0, 0, 0, 4, 1, 0)

;
; N.B. The following registers MUST be saved such that ebp is saved last.
;      This is done so the debugger can find the saved ebp for a thread
;      that is not currently in the running state.
;

    sub     esp, 4*4
    mov     [esp+12], ebx         ; save registers
    mov     [esp+8], esi          ;
    mov     [esp+4], edi          ;
    mov     [esp+0], ebp          ;

    mov     ebx, PCR[PcSelfPcr]   ; set address of PCR
    mov     edi, ecx             ; set old thread address
    mov     esi, edx             ; set next thread address
    movzx   ecx, byte ptr [edi].ThWaitirql ; set APC interrupt bypass disable
    call    SwapContext          ; swap context

    mov     ebp, [esp+0]        ; restore registers
    mov     edi, [esp+4]        ;
    mov     esi, [esp+8]        ;
    mov     ebx, [esp+12]       ;
    add     esp, 4*4            ;
    fstRET  KiSwapContext       ;
fstENDP KiSwapContext
```

`windbg` 得到的 `KiSwapContext` 代码

```
lkd> uf KiSwapContext
nt!KiSwapContext:
8054595c 83ec10          sub     esp,10h
8054595f 895c240c        mov     dword ptr [esp+0Ch],ebx
80545963 89742408        mov     dword ptr [esp+8],esi
80545967 897c2404        mov     dword ptr [esp+4],edi
8054596b 892c24          mov     dword ptr [esp],ebp
8054596e 648b1d1c000000  mov     ebx,dword ptr fs:[1Ch]
80545975 8bf1            mov     esi,ecx
80545977 8bbb24010000    mov     edi,dword ptr [ebx+124h]
8054597d 89b324010000    mov     dword ptr [ebx+124h],esi
80545983 8a4f58          mov     cl,byte ptr [edi+58h]
80545986 e8f5000000      call    nt!SwapContext (80545a80)
8054598b 8b2c24          mov     ebp,dword ptr [esp]
8054598e 8b7c2404        mov     edi,dword ptr [esp+4]
80545992 8b742408        mov     esi,dword ptr [esp+8]
80545996 8b5c240c        mov     ebx,dword ptr [esp+0Ch]
8054599a 83c410          add     esp,10h
8054599d c3              ret
```

驱动运行后的效果是：

搜集到的进程是

```
0x81FE7768   svchost.exe
0x81EC6478   alg.exe
0x81F5E990   svchost.exe
0x81FE83E0   DrvLoader.exe
0x81CA7020   taskmgr.exe
0x81595D70   VMwareService.e
0x815C6608   VMwareTray.exe
0x815D9C08   svchost.exe
0x8159C528   ctfmon.exe
0x8158EB60   VisualTaskTips.
0x815E5DA0   winlogon.exe
0x81E96418   DrvLoader.exe
0x815F8BD8   lsass.exe
0x81FE5020   MSDEV.EXE
0x81EC0B28   mdm.exe
0x81EE0500   explorer.exe
0x81F13020   MSDEV.EXE
0x815523A8   DBGVIEW.EXE
0x81F74020   csrss.exe
0x821507C0   System
0x82009830   VMwareUser.exe
0x81F66DA0   services.exe
0x81F07B88   svchost.exe
```

搜集到的进程肯定是不全的 因为有的进程在休息 没有线程调度发生 也就没有搜集到= =

下面是主要代码

```c
DWORD gThreadsProcessOffset =0x220;    // ETHREAD 在 EPROCESS 偏移

/*
+0x218 TopLevelIrp      : Uint4B
+0x21c DeviceToVerify   : Ptr32 _DEVICE_OBJECT
+0x220 ThreadsProcess   : Ptr32 _EPROCESS
*/

ULONG ProcessNameOffset  = 0x174;      // 进程对应的文件名 在 EPROCESS 偏移

/*
+0x170 Session          : Ptr32 Void
+0x174 ImageFileName    : [16] UChar
+0x184 JobLinks         : _LIST_ENTRY
*/

PProcessList wLastItem  = NULL;

int BeTerminate = 0;                  //1 表示线程必须要停止  3 表示线程不是 PENDING 状态  0 表示线程可以正常运行

void _stdcall CollectProcess(PEPROCESS pEPROCESS)  // 搜集 EPROCESS
{
  if (!IsAdded(wLastItem, pEPROCESS)) AddItem(&wLastItem, pEPROCESS);
  return;
}

void __stdcall ThreadCollect(PUCHAR pEthread)   //根据 ETHREAD 得到 EPROCESS 并调用 CollectProcess 来搜集
{
  PEPROCESS pEprocess = *(PEPROCESS *)(pEthread + gThreadsProcessOffset);
  if (pEprocess) CollectProcess(pEprocess);
  return;
}

DWORD outPEthread = 0;

void __stdcall ProcessData(DWORD pInEthread, DWORD pOutEthread)
{
  DWORD pid, eprocess;
  char * pname;
  if (MmIsAddressValid(PVOID(pInEthread+0x220)) )    // 这里以及下面要判断是不是一个真正的 Ethread 结构体 有时好像调用 SwapContext 传进来的不是 Ethread 结构体 然后就蓝屏 具体没有深究 加个判断就不蓝了~
  {
    eprocess = *(DWORD*)(pInEthread+0x220);
    if (MmIsAddressValid(PVOID(eprocess) )
    {
      ThreadCollect((PUCHAR)pInEthread);
    }
  }
}

PBYTE GoBackAddr  = NULL;
PBYTE ChangAddr   = NULL;
DWORD CallContextOffset = 0;

__declspec(naked) VOID HookSwap()
{

  _asm
  {
    pushad
      pushfd
      cli
  }

  _asm
  {
       // EDI 是换出的线程上下文
         push edi
      //ESI 是换入的线程上下文
      push esi
      call ProcessData   //搜集进程
  }

  _asm
  {

    sti
      popfd
      popad
  }

  _asm jmp DWORD PTR[GoBackAddr]
}

/*
得到 SwapContext 地址的原理是
用 PsLookupThreadByThreadId 得到 Idle System 的 KTHREAD
res=(PCHAR)(Thread->Tcb.KernelStack);
SwapAddr=*(DWORD *)(res+0x08);
*/

PCHAR GetSwapAddr()
{
  PCHAR res = 0;
  NTSTATUS  Status;
  PETHREAD Thread;
  if (*NtBuildNumber <= 2195)
    Status = PsLookupThreadByThreadId((PVOID)4, &(PETHREAD)Thread);
  else
    Status = PsLookupThreadByThreadId((PVOID)8, &(PETHREAD)Thread);
  if (NT_SUCCESS(Status))
  {
    if (MmIsAddressValid(Thread))
    {
      res = (PCHAR)(Thread->Tcb.KernelStack);
    }
    if (MmIsAddressValid(res+8))
    {
      _asm
      {
        mov eax,res
          add eax,8
          mov eax,[eax]
          mov res,eax
      }
    }
    else
    {
      res = 0;
      return NULL;
    }
  }

  _asm
  {
    mov eax,res
      sub eax,5
      mov ChangAddr,eax
      mov edx,[eax+1]
      mov CallContextOffset,edx
      add eax,edx
      add eax,5
      mov GoBackAddr,eax
      mov res,eax
  }

  return res;
}

BOOL  HookSwapFunction(BOOL flag)
{

  if (flag == TRUE)
  {

    KIRQL OldIrql=0;
    DWORD NewOffset;//HookSwap-ChangAddr-5;
    _asm
    {
      mov eax,HookSwap
        mov edx,ChangAddr
        sub eax,edx
        sub eax,5
        mov NewOffset,eax
    }

    PAGED_CODE()
      ASSERT(KeGetCurrentIrql()<=DISPATCH_LEVEL);
    KeRaiseIrql(2,&OldIrql);//HIGH_LEVEL
    __asm
    {
      CLI
        MOV   EAX, CR0
        AND   EAX, NOT 10000H  //disable WP bit
        MOV   CR0, EAX
```



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
