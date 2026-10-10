

# 突破 \*\*\* 反外挂系统

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-563.htm`](https://www.pediy.com/kssd/pediy09/pediy09-563.htm)

> 在给 MM 朋友做了个诛仙外挂(脚本写的![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") )后,再应她要求,小试一下劲舞团!  
>   
> 2G 的劲舞一个通宵下载下来后安装,先注册一个 ID 吧,进游戏,CPU 好卡!!!!!  
> 打开 CE 准备搜索内存了,选中游戏进程,我搜!! 咔--------居然说无法打开进程!!!  
> 预料之中,看了 NP 的介绍,觉得问题不大,但仔细一看,好像不是 NP,  SSDT 有以下几个地方被 HOOK 了:  
> NtOpenProcess              ;防止你打开游戏进程  
> NtReadVirtualMemory     ;防止你读游戏内存  
> NtWriteVirtualMemory    ;防止你写游戏内存  
> NtSuspendThread          ;游戏有一个线程不停的反外挂检测,防止被你停掉了!!!  
> NtTerminateThread        ;防止进程被意外终止后驱动不能正常退出,SSDT 不能恢复!!  
>   
> 写驱动注入 NtOpenProcess 和 NtReadVirtualMemory,  
> OK,可以打开游戏了,继续读内存试试,  
> 怪事来了,返回 EAX=1,好像成功了!但读出的东西全是 0!怀疑根本没读成功!!!  
>   
> 这还不是最怪的,用 VirtualQueryEx 查询游戏内存情况,这时反外挂跳出来了!游戏终止,  
> VirtualQueryEx===>NtQueryVirtualMemory 都没有被 HOOK 啊!!!是怎么被发现的??  
>   
> 还有用冰刃查看进程,也会被游戏侦查到,然后提示反外挂出现问题后退出,  
> 冰刃查看进程用了比较底层的东西了,但仍会被检查到!!!  
> SSDT 就只有以上的地方被 HOOK,  NtQuerySystemInformation 没有遭毒手!  
>   
> 在冰刃的进线程监控中得到的游戏 PID 和任务管理器里的一样,我 openprocess 和 ReadVirtualMemory 完全绕过了 SSDT,但就是读不出东西!然后一调用 VirtualQueryEx 就会被游戏发现!  
> openprocess 得到的句柄绝对是正确的!因为我可以在游戏中创建远线程!! 

> 嗯，inline hook 哪里可能都会躲。  
> 不然就自己实现一下 ReadVirtualMemory   
>   
> \==================================  
> NTSTATUS  
> NtReadVirtualMemory (  
>      IN HANDLE ProcessHandle,  
>      IN PVOID BaseAddress,  
>      OUT PVOID Buffer,  
>      IN ULONG BufferSize,  
>      OUT PULONG NumberOfBytesRead OPTIONAL  
>      )  
> {  
>     ULONG BytesCopied;  
>     KPROCESSOR\_MODE PreviousMode;  
>     PEPROCESS Process;  
>     NTSTATUS Status;  
>   
>     PAGED\_CODE();  
>   
>     PreviousMode = KeGetPreviousMode();  
>     if (PreviousMode != KernelMode) {  
>   
>         Status = MiValidateUserTransfer(BaseAddress, Buffer, BufferSize);  
>         if (Status != STATUS\_SUCCESS) {  
>             return Status;  
>         }  
>   
>         if (ARGUMENT\_PRESENT(NumberOfBytesRead)) {  
>             try {  
>                 ProbeForWriteUlong(NumberOfBytesRead);  
>   
>             } except(EXCEPTION\_EXECUTE\_HANDLER) {  
>                 return GetExceptionCode();  
>             }  
>         }  
>     }  
>   
>     BytesCopied = 0;  
>     Status = STATUS\_SUCCESS;  
>     if (BufferSize != 0) {  
>   
>         Status = ObReferenceObjectByHandle(ProcessHandle,  
>                                            PROCESS\_VM\_READ,  
>                                            PsProcessType,  
>                                            PreviousMode,  
>                                            (PVOID \*)&Process,  
>                                            NULL);  
>   
>         if (Status == STATUS\_SUCCESS) {  
>   
>             Status = MmCopyVirtualMemory (Process,  
>                                           BaseAddress,  
>                                           PsGetCurrentProcess(),  
>                                           Buffer,  
>                                           BufferSize,  
>                                           PreviousMode,  
>                                           &BytesCopied);  
>   
>             ObDereferenceObject(Process);  
>         }  
>     }  
>   
>     if (ARGUMENT\_PRESENT(NumberOfBytesRead)) {  
>         try {  
>             \*NumberOfBytesRead = BytesCopied;  
>   
>         } except(EXCEPTION\_EXECUTE\_HANDLER) {  
>             NOTHING;  
>         }  
>     }  
>   
>     return Status;  
> } 

> HS 的反外挂一共核心态拦截了 14 个 NTAPI  
> 其中 9 个是过滤 5 个是监控 而且 3.2 版的游戏还检测了钩子链  与钩子链所插入的 DLL 的 5 个监控 API 的调用规则 

 4dd466d0e1e0cc123afbe504da2462ca