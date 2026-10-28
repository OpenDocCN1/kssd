
```
81570669 8bcb            mov     ecx,ebx

8157066b e83343ffff      call    nt!PerfInfoLogSysCallEntry (815649a3)

81570670 e985fcffff      jmp     nt!KiFastCallEntry+0x12a (815702fa)

81570675 50              push    eax

81570676 8bc8            mov     ecx,eax

81570678 e8ca42ffff      call    nt!PerfInfoLogSysCallExit (81564947)

8157067d 58              pop     eax

8157067e e9bffcffff      jmp     nt!KiFastCallEntry+0x172 (81570342)
```

从上面`KiFastCallEntry`分析可以得出结论 `ssdt Shadow` 表只有在GUI线程使用时才加入到内存中，当`KTHREAD`中的成员`Win32Thread`域由0变成非0，说明这个线程是一个GUI线程。如果是GUI线程`KTHREAD`中的成员`ServiceTable`域就指向`SSDT Shadow`表；反之，如果是非GUI线程`KTHREAD`中的成员`ServiceTable`域将执行`SSDT`表。

`_KPROCESS`结构中成员`ThreadListHead`是指向`_KTHREAD`结构中成员`ThreadListEntry`链表。通过它可以定位`_KTHREAD`结构了，就可判断这个线程是否是GUI线程了。

在`_EPROCESS`结构的第一个成员`Pcb`域指向`_KPROCESS`结构;  同样在`_ETHREAD`结构中第一个成员`Tcb`域指向`_KTHREAD`结构。所以得到这些结构地址后，结构体是可以互换的。

### 三、根据分析结果，代码实现过程：

```
typedef struct \_SSDT

{

        PVOID ServiceTableBase;         //System Service Dispatch Table 的基地址

        PVOID ServiceCounterTable;      //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由sysenter 更新。

        PVOID NumberOfServices;         //由 ServiceTableBase 描述的服务的数目。

        PVOID ParamTableBase;           //包含每个系统服务参数字节数表的基地址-系统服务参数表

}\* pSSDT,SSDT;

//切换到带有UI进程的内存空间。否则shadow表将不会出现在内存中。我们使用csrss作为目标进程。

HANDLE GetCsrPid()

{

        HANDLE Process,hObject;

        HANDLE CsrId = (HANDLE)0;

        OBJECT\_ATTRIBUTES obj;

        CLIENT\_ID cid;

        UCHAR Buff[0x100];

        POBJECT\_NAME\_INFORMATION ObjName = (PVOID)&Buff;

        PSYSTEM\_HANDLE\_INFORMATION\_EX Handles=NULL;

        ULONG i;

        ULONG nSize;

        NTSTATUS status;

        PEPROCESS pEproc;

        PETHREAD pEthr;

        PKTHREAD pKthr;

        PKPROCESS pKproc;

        //得到csrss.exe句柄。

        Handles = GetInfoTable(&nSize);

        //等于null，返回0

        if(Handles==(PSYSTEM\_HANDLE\_INFORMATION\_EX)0)

        {

                return CsrId;

        }

        for(i = 0; i < Handles->NumberOfHandles; i++)

        {

                if(Handles->Information[i].ObjectTypeNumber == 21)  //Port object  "\\Windows\\ApiPort"

                {

                        InitializeObjectAttributes(&obj, NULL, OBJ\_KERNEL\_HANDLE, NULL, NULL);

                        cid.UniqueProcess = (HANDLE)Handles->Information[i].ProcessID;

                        cid.UniqueThread  = (HANDLE)0;

                        status=PsLookupProcessByProcessId(Handles->Information[i].ProcessID,&pEproc);

                        if(!NT\_SUCCESS(status))

                        {

                                KdPrint(("PsLookupProcessByProcessId operation failed!\n"));

                                continue;

                        }

                        if(CheckProcessName(pEproc))

                        {

                                CsrId = (HANDLE)Handles->Information[i].ProcessID;

                                KdPrint(("Csrss.exe PID = %d\n", CsrId));              //如果上面得不到CsrssId，这里可以。

                                ObDereferenceObject(pEproc);

                                goto Done;

                        }

            ObDereferenceObject(pEproc);

                }

        }

Done:

    if(Handles!=NULL) ExFreePool(Handles);

        return CsrId;    //返回进程id

}

                //得到csrss.exe的PID。根据进程Pid得到进程的PEPROCESS结构。

                status = PsLookupProcessByProcessId((ULONG)GetCsrPid(), &crsEProc);

                if (!NT\_SUCCESS(status))

                {

                        DbgPrint("PsLookupProcessByProcessId() error\n");

                        return ;

                }

                else

                {

                        DbgPrint("csrss.exe PEPROCESS address 0x%8x\n",crsEProc);

                }

                ObDereferenceObject(crsEProc);

//根据csrss进程结构，遍历线程得到shadow地址。

ULONG  KeQueryRuntimeProcess (PKPROCESS Process)

{

        KLOCK\_QUEUE\_HANDLE LockHandle;

        PULONG SSDTShadowAddress=0;

        PLIST\_ENTRY NextEntry;

        PKTHREAD Thread;

        ULONG TotalTime;

        PKSPIN\_LOCK ProcessLock;

        PLIST\_ENTRY  Thread\_List\_Head;

        ASSERT(KeGetCurrentIrql() <= DISPATCH\_LEVEL);

        ProcessLock = (PKSPIN\_LOCK)((ULONG)Process+0x034);      // ProcessLock

//\_EPROCESS结构中的ThreadListHead  成员，指向进程保护的线程列表。

        Thread\_List\_Head=(PLIST\_ENTRY)((ULONG)Process+0x02c);

        NextEntry = Thread\_List\_Head->Flink;

        while (NextEntry != Thread\_List\_Head)

        {

        //循环线程列表取出每一个线程\_KTHREAD结构体。

                Thread = (PKTHREAD)((ULONG)NextEntry - 0x1d4);

        //查找每一个线程,看看这个成员是否为0。非0是gui线程表示这个线程是GUI线程。

                if(((ULONG)Thread+0x124)!=0)

                {

          //如果是GUI线程，那么\_KTHREAD的ServiceTable成员将指向SSDT SHADOW table地址。

                        SSDTShadowAddress = (PULONG)((ULONG)Thread +0x03c);

                        break;

                }

                NextEntry = NextEntry->Flink;

        }

        return \*SSDTShadowAddress;   //返回shadow地址。

}

//取得SSDT Shadow表地址

KeServiceDescriptorTableShadow = (pSSDTShadow)GetSSDTShadowTableAddress(&ModuleID);

        if(KeServiceDescriptorTableShadow==NULL)

                return NULL;

  //根据\_SSDT 结构体，算出目标函数地址，然后HOOK。

        pTempSSDT = & KeServiceDescriptorTableShadow->win32k;

        t\_addr=(ULONG)pTempSSDT->ServiceTableBase;

        Module\_Adr=(PULONG)(t\_addr+ModuleID\*4);

        return Module\_Adr;
```

这个Demo目前可以屏蔽 画图， 写字板，计算器窗口弹出

源码附件密码：huihai-it

学驱动安全的朋友关注 驱动培训

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`MFC_NewWindowsFilter.rar`](javascript:void(0);) （1.62MB，173次下载）

- [`SSDTShadowHookCourse.rar`](javascript:void(0);) （1.90MB，239次下载）
