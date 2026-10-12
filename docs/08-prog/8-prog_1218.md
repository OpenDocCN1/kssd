
# WriteProcessMemory 之后一定要 FlushInstructionCache 么？

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-284.htm`](https://www.pediy.com/kssd/pediy09/pediy09-284.htm)

> 如题，很多地方都这么说，包括 `MSDN`。

> 但是我用 `WriteProcessMemory` 修改代码之后没有 `Flush`，然后立即跳转回去执行。按说刚刚执行过的指令应该在 cache 里才对，那样修改内存应该是无效的。但是运行结果显示执行的是修改过的代码。

> 莫非 `WriteProcessMemory` 自动 flush 了？

> 不用这个如果你正执行 `rep movsb edi=eip` 的话可能出现一些问题。

> 你 `WriteProcessMemory` 是写另一个进程的内存吧? 这样在你再返回去执行你的代码的时候,CPU 上已经执行过很多代码了,这些代码是其它进程的,这些进程很可能用了 cache 的相同地方,这样就相当你那个进程刷 cache 了,另外进程切换的时候也要刷 cache(至于会不会刷你的那个 cache,记不清了).....

> 但是二楼说的情况就不同了,他说的那种情况是在同一个进程中,自己改写自己,如果操作系统在这期间,没有进行任务切换,也没有被中断(也就是说你的代码在这个时候,没有被打断过),cache 中的数据**可能就是脏的......**

> 好像不一定需要，你可以查看一下 2000 和 XP 的 `WriteProcessMemory` 的汇编代码，我记得函数自己已经调用过 `FlushInstructionCache` 了。Win95 的具体实行我没看过

> 请问是怎么个自己改自己法？总不能自己一边执行 `WriteProcessMemory` 系统调用一边执行 `rep movsb` 吧？

> 如果是同一个进程先 `Write` 之后再 `rep mov` 的话，我觉得反倒更没问题。因为。之所以说“有可能出问题”是因为，改代码时只能通过 data cache 作用于内存，而执行代码时从 instruction cache 读，因此可能出现不一致。但是如果是 `mov di=ip` 的话，那等于现在是从 data cache 或者内存读 ip 指向的数据，而 data cache 和内存总是一致的，所以反倒不会出问题才对。

> 同理，如果是两个进程，被写的进程正在执行 `rep mov`，那这里面也没有 instruction cache 的事了，自然也不会出现 data cache 和内存的不一致现象。

> 不过我觉得 linxer 对于两个进程的情况分析的在理，可不可以这么认为：

> 只有在同一个进程先 `Write` 然后立即 `jmp short` 回去执行刚刚改过的代码，才会因为 instruction cache 和 data cache 的不一致性而出错。

> 另外，对于 justlovemm 说的问题，我刚刚写了个程序检查了一下，代码如下：

```c
#include <windows.h>   
int _tmain(int argc, _TCHAR* argv[])   
{    
  SIZE_T n;    
  HANDLE hProc = GetCurrentProcess();    
  BYTE code[3]={0xCC, 0xCC , 0xCC};    
  BYTE buf[3]={0};       
  WriteProcessMemory(hProc, (LPVOID)0x00411C14, code, 3, &n);    
  printf("wroten.\n");    
  FlushInstructionCache(hProc, (LPVOID)0x00411C14, 3);       
  return 0;   
}
```

> 我在程序运行时通过 `OD` 在 `ZwFlushInstructionCache` 的入口处（`7C92D9DF`）下了断点。运行结果是，在显示`"wroten."`的前后各有一次停在 `ZwFlushInstructionCache` 入口处，并且程序产生了 `INT3` 中断。这说明：

> 1. `WriteProcessMemory` 确实调用了 `ZwflushInstructionCache`，至少在自己改自己时是这样
> 2. `INT3` 是由修改的 `CC` 引起的，这说明修改成功，“同一进程自己改自己后立即执行”，这种情况也没有出现不一致情况，执行的是新代码

> 最后，又回到最开始的问题了：既然 `WriteProcessMemory` 自动调用 `FlushInstrctionCache` 了，那还有必要再显式调用一遍么？

> 引用:
>
> *最初由 `铂钧`发布* (帖子 363433) 另外，对于 justlovemm 说的问题，我刚刚写了个程序检查了一下，代码如下：

```c
#include <windows.h>  

int _tmain(int argc, _TCHAR* argv[])  

{  

  SIZE_T n;  

  HANDLE hProc = GetCurrentPr...
```

```c
BOOL  
NTAPI  
WriteProcessMemory(IN HANDLE hProcess,  
                  IN LPVOID lpBaseAddress,  
                  IN LPCVOID lpBuffer,  
                  IN SIZE_T nSize,  
                  OUT SIZE_T *lpNumberOfBytesWritten)  
{  
    NTSTATUS Status;  
    ULONG OldValue;  
    SIZE_T RegionSize;  
    PVOID Base;  
    BOOLEAN UnProtect;  
   
    /* Set parameters for protect call */  
    RegionSize = nSize;  
    Base = lpBaseAddress;  
   
    /* Check the current status */  
    Status = NtProtectVirtualMemory(hProcess,  
                                   &Base,  
                                   &RegionSize,  
                                   PAGE_EXECUTE_READWRITE,  
                                   &OldValue);  
    if (NT_SUCCESS(Status))  
    {  
        /* Check if we are unprotecting */  
        UnProtect = OldValue & (PAGE_READWRITE |  
                               PAGE_WRITECOPY |  
                               PAGE_EXECUTE_READWRITE |  
                               PAGE_EXECUTE_WRITECOPY) ? FALSE : TRUE;  
        if (UnProtect)  
        {  
            /* Set the new protection */  
            Status = NtProtectVirtualMemory(hProcess,  
                                           &Base,  
                                           &RegionSize,  
                                           OldValue,  
                                           &OldValue);  
   
            /* Write the memory */  
            Status = NtWriteVirtualMemory(hProcess,  
                                          lpBaseAddress,  
                                          (LPVOID)lpBuffer,  
                                          nSize,  
                                          lpNumberOfBytesWritten);  
            if (!NT_SUCCESS(Status))  
            {  
                /* We failed */  
                SetLastErrorByStatus(Status);  
                return FALSE;  
            }  
   
            /* Flush the ITLB */  
            NtFlushInstructionCache(hProcess, lpBaseAddress, nSize);  
            return TRUE;  
        }  
        else  
        {  
            /* Check if we were read only */  
            if ((OldValue & PAGE_NOACCESS) || (OldValue & PAGE_READONLY))  
            {  
                /* Restore protection and fail */  
                NtProtectVirtualMemory(hProcess,  
                                       &Base,  
                                       &RegionSize,  
                                       OldValue,  
                                       &OldValue);  
                SetLastErrorByStatus(STATUS_ACCESS_VIOLATION);  
                return FALSE;  
            }  
   
            /* Otherwise, do the write */  
            Status = NtWriteVirtualMemory(hProcess,  
                                          lpBaseAddress,  
                                          (LPVOID)lpBuffer,  
                                          nSize,  
                                          lpNumberOfBytesWritten);  
   
            /* And restore the protection */  
            NtProtectVirtualMemory(hProcess,  
                                   &Base,  
                                   &RegionSize,  
                                   OldValue,  
                                   &OldValue);  
            if (!NT_SUCCESS(Status))  
            {  
                /* We failed */  
                SetLastErrorByStatus(STATUS_ACCESS_VIOLATION);  
                return FALSE;  
            }  
   
            /* Flush the ITLB */  
            NtFlushInstructionCache(hProcess, lpBaseAddress, nSize);  
            return TRUE;  
        }  
    }  
    else  
    {  
        /* We failed */  
        SetLastErrorByStatus(Status);  
        return FALSE;  
    }  
}  
```

> 代码比较说明问题。已经 `NtFlushInstructionCache` 当然不用再去了

> 如果是 `memcpy` 的 则需要。 尤其在多核中 不 `Flush` 出错的几率不会小。俺实践过一下。

> 我没有找到哪里说 `WriteProcessMemory` 以后还需要 `Flush` 的例子。至少 `MSDN` 的 `WriteProcessMemory` 函数标准文档里面貌似并没有提到（`MSDN for VS2005 extended`）

> `MSDN` 中 `FlushInstructionCache` 的说明：

> `"Applications should call FlushInstructionCache if they generate or modify code in memory. The CPU cannot detect the change, and may execute the old code it cached."`

> 是我粗心了，里面只是说“generate or modify”，并没针对 `WriteProcessMemory` 这个函数。

> 另外，关于拜月叔叔和 linxer 说的 `rep movsb`，我也搞明白了。你们的意思是说，没有 `WriteProcessMemory` 的事，就是 `rep movsb edi=eip` 之后继续执行。由于 `EIP` 一直没动，因此执行的“本应该”是刚刚 `mov` 过来的新指令，但由于 `Instruction Cache` 的问题实际执行的会是旧指令。

> 呵呵，这下都清楚了。有机会再试下 `rep movsb edi=eip` 在多核下的情况。多谢各位了！

`4dd466d0e1e0cc123afbe504da2462ca`
