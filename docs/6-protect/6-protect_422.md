

# Themida1950 一个不起眼的 anti-debug

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-558.htm`](https://www.pediy.com/kssd/pediy09/pediy09-558.htm)

> Themida1950 一个不起眼的 anti-debug  
>   
>   
> 前段时间在抄袭西裤哥的代码,替换进程的 KUSER\_SHARED\_DATA,发现  
> 新版 Themida 加壳的程序不能跑了,试了低版本没问题,于是看了一下.  
>   
> 这几句代码 VM 了,在 VM 出口很容易发现:  
>   
> 先调用了 timeGetTime:  
>   
> 76B11AD8    833D 14F0B276>cmp dword ptr ds:\[76B2F014\],0  
> 76B11ADF    0F85 ADCA0000 jnz 76B1E592                       ; jmp to kernel32.GetTickCount  
> 76B11AE5    E8 DAFFFFFF   call 76B11AC4  
> 76B11AEA    2B05 18F0B276 sub eax,dword ptr ds:\[76B2F018\]  
> 76B11AF0    6A 00         push 0  
> 76B11AF2    1B15 1CF0B276 sbb edx,dword ptr ds:\[76B2F01C\]  
> 76B11AF8    68 10270000   push 2710  
> 76B11AFD    52            push edx  
> 76B11AFE    50            push eax                                       
> 76B11AFF    E8 07000000   call 76B11B0B  
> 76B11B04    0305 20F0B276 add eax,dword ptr ds:\[76B2F020\]              
> 76B11B0A    C3            retn  
>   
> 结果为 00C38E41,单位是 ms.  
>   
> 其中 call 的 76B11AC4 为:  
>   
> .text:76B11AC4 sub\_76B11AC4    proc near                 
> .text:76B11AC4                                           
> .text:76B11AC4                 mov     edx, ds:7FFE000Ch  
> .text:76B11ACA                 mov     eax, ds:7FFE0008h  
> .text:76B11ACF                 cmp     edx, ds:7FFE0010h  
> .text:76B11AD5                 jnz     short sub\_76B11AC4  
> .text:76B11AD7                 retn  
> .text:76B11AD7 sub\_76B11AC4    endp  
>   
>   
> 用的是 KUSER\_SHARED\_DATA 的 InterruptTime.  
>   
>   
> lkd> dt \_KUSER\_SHARED\_DATA -r  
>    +0x000 TickCountLow     : Uint4B  
>    +0x004 TickCountMultiplier : Uint4B  
>    +0x008 InterruptTime    : \_KSYSTEM\_TIME  
>       +0x000 LowPart          : Uint4B  
>       +0x004 High1Time        : Int4B  
>       +0x008 High2Time        : Int4B  
>    +0x014 SystemTime       : \_KSYSTEM\_TIME  
>       +0x000 LowPart          : Uint4B  
>       +0x004 High1Time        : Int4B  
>       +0x008 High2Time        : Int4B  
>   
>   
> 然后 Sleep:  
>   
>   
> 0012FF9C   00F138AB      /CALL to Sleep  
> 0012FFA0   00000032      \\Timeout = 50. ms  
> 0012FFA4   00000000  
>   
> 参数为 50ms.  
>   
>   
> 然后再次调用 timeGetTime,我的返回是 00C38E41,结果相同,  
> 试试加上 60ms,又跑起来了,有点无聊. 大概是用来对付月光宝  
> 盒之类的工具的,平时用不着.  
>   
>   
> Sleep 调用 SleepEx:  
>   
>   
> ......  
> .text:7C8023CE                 push    \[ebp+dwMilliseconds\]  
> .text:7C8023D1                 lea     eax, \[ebp+var\_28\]  
> .text:7C8023D4                 push    eax  
> .text:7C8023D5                 call    BaseFormatTimeOut(x,x)  
> .text:7C8023DA                 mov     \[ebp+var\_1C\], eax  
> .text:7C8023DD                 cmp     eax, esi  
> .text:7C8023DF                 jz      short loc\_7C802412  
> .text:7C8023E1  
> .text:7C8023E1 loc\_7C8023E1:                           ; CODE XREF: SleepEx(x,x)+86j  
> .text:7C8023E1                                         ; SleepEx(x,x)+8Fj  
> .text:7C8023E1                 push    \[ebp+var\_1C\]  
> .text:7C8023E4                 push    \[ebp+bAlertable\]  
> .text:7C8023E7                 call    ds:NtDelayExecution(x,x)  
> .text:7C8023ED                 mov     \[ebp+var\_20\], eax  
> .text:7C8023F0                 cmp     \[ebp+bAlertable\], esi  
> .text:7C8023F3                 jnz     short loc\_7C802424  
> ......  
>   
> 毫秒参数转换为 LARGE\_INTEGER,调用 NtDelayExecution.  
>   
> 改改 NtDelayExecution,把 Sleep 传递的时间加上去就行了,详细的  
> 参考西裤的替换 KUSER\_SHARD\_DATA PTE. 

 4dd466d0e1e0cc123afbe504da2462ca