

# shadow ssdt 学习笔记(一)(二)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-350.htm`](https://www.pediy.com/kssd/pediy09/pediy09-350.htm)

> 1。取得 shadow ssdt 真实地址  
> 系统只提供了 KeServiceDescriptorTable 导出  
> KeServiceDescriptorTableShadow 是个未导出结构  
>   
> 定义  
> Copy code  
>   
>     typedef struct \_SYSTEM\_SERVICE\_TABLE  
>     {  
>           PNTPROC  ServiceTable;  // array of entry points  
>           PDWORD  CounterTable;  // array of usage counters  
>           DWORD  ServiceLimit;    // number of table entries  
>           PBYTE    ArgumentTable;  // array of byte counts  
>     }  
>     SYSTEM\_SERVICE\_TABLE,  
>     \*PSYSTEM\_SERVICE\_TABLE,  
>     \*\*PPSYSTEM\_SERVICE\_TABLE;  
>     //-----------------------------------------------------------------------------------------------------------  
>     typedef struct \_SERVICE\_DESCRIPTOR\_TABLE  
>     {  
>           SYSTEM\_SERVICE\_TABLE ntoskrnl;  // ntoskrnl.exe ( native api )  
>         SYSTEM\_SERVICE\_TABLE win32k;    // win32k.sys (gdi/user support)  
>           SYSTEM\_SERVICE\_TABLE Table3;    // not used  
>           SYSTEM\_SERVICE\_TABLE Table4;    // not used  
>     }  
>     SYSTEM\_DESCRIPTOR\_TABLE,  
>     \*PSYSTEM\_DESCRIPTOR\_TABLE,  
>     \*\*PPSYSTEM\_DESCRIPTOR\_TABLE;  
>   
>   
>   
> 其实 KeServiceDescriptorTableShadow 包含 4 个子结构，其中第一个就是 ntoskrnl.exe ( native api )，和 KeServiceDescriptorTable 指向一样 我们真正需要获得的是第二个 win32k.sys (gdi/user support)，第三个和第四个一般不使用  
>   
> 定位方法  
> 1。硬编码  
> Copy code  
>   
>     //for xp  
>     if(gKernelVersion==WINXP)  
>       KeServiceDescriptorTableShadow=KeServiceDescriptorTable-0x40;  
>     //for 2k  
>     if(gKernelVersion==WIN2K)  
>       KeServiceDescriptorTableShadow=KeServiceDescriptorTable+0xE0;  
>   
>   
>   
> 2。搜索 KeAddSystemServiceTable  
> 反汇编代码可以看出  
> Copy code  
>   
>     lkd> u KeAddSystemServiceTable l 40  
>     nt!KeAddSystemServiceTable:  
>     805ba589 8bff            mov    edi,edi  
>     805ba58b 55              push    ebp  
>     805ba58c 8bec            mov    ebp,esp  
>     805ba58e 837d1803        cmp    dword ptr \[ebp+18h\],3  
>     805ba592 774e            ja      nt!KeAddSystemServiceTable+0x6b (805ba5e2)  
>     805ba594 8b4518          mov    eax,dword ptr \[ebp+18h\]  
>     805ba597 c1e004          shl    eax,4  
>     805ba59a 83b880a6558000  cmp    dword ptr nt!KeServiceDescriptorTable (8055a680)\[eax\],0  
>     805ba5a1 753f            jne    nt!KeAddSystemServiceTable+0x6b (805ba5e2)  
>     805ba5a3 8d8840a65580    lea    ecx,nt!KeServiceDescriptorTableShadow (8055a640)\[eax\]  
>     805ba5a9 833900          cmp    dword ptr \[ecx\],0  
>     805ba5ac 7534            jne    nt!KeAddSystemServiceTable+0x6b (805ba5e2)  
>     805ba5ae 837d1801        cmp    dword ptr \[ebp+18h\],1  
>     805ba5b2 8b5508          mov    edx,dword ptr \[ebp+8\]  
>     805ba5b5 56              push    esi  
>     805ba5b6 8b7510          mov    esi,dword ptr \[ebp+10h\]  
>     805ba5b9 57              push    edi  
>     805ba5ba 8b7d14          mov    edi,dword ptr \[ebp+14h\]  
>     805ba5bd 8911            mov    dword ptr \[ecx\],edx  
>     805ba5bf 8b4d0c          mov    ecx,dword ptr \[ebp+0Ch\]  
>     805ba5c2 898844a65580    mov    dword ptr nt!KeServiceDescriptorTableShadow+0x4 (8055a644)\[eax\],ecx  
>     805ba5c8 89b048a65580    mov    dword ptr nt!KeServiceDescriptorTableShadow+0x8 (8055a648)\[eax\],esi  
>     805ba5ce 89b84ca65580    mov    dword ptr nt!KeServiceDescriptorTableShadow+0xc (8055a64c)\[eax\],edi  
>     805ba5d4 0f855a3e0300    jne    nt!KeAddSystemServiceTable+0x4d (805ee434)  
>     805ba5da 5f              pop    edi  
>     805ba5db b001            mov    al,1  
>     805ba5dd 5e              pop    esi  
>     805ba5de 5d              pop    ebp  
>     805ba5df c21400          ret    14h  
>     805ba5e2 32c0            xor    al,al  
>     805ba5e4 ebf8            jmp    nt!KeAddSystemServiceTable+0x6d (805ba5de)  
>     805ba5e6 90              nop  
>     805ba5e7 90              nop  
>     805ba5e8 90              nop  
>     805ba5e9 90              nop  
>     805ba5ea 90              nop  
>   
>   
>   
> 搜索办法很简单  
> 就是找到  
> 805ba5a3 8d8840a65580    lea    ecx,nt!KeServiceDescriptorTableShadow (8055a640)\[eax\]  
> 的 8d88 40a65580    lea    ecx, nt!KeServiceDescriptorTableShadow  
> 代码如下  
> Copy code  
>   
>     void GetKeServiceDescriptorTableShadow()  
>     {  
>         PUCHAR cPtr, pOpcode;  
>         ULONG Length;  
>        
>         for (cPtr = (PUCHAR)KeAddSystemServiceTable;  
>             cPtr < (PUCHAR)KeAddSystemServiceTable + PAGE\_SIZE;  
>             cPtr += Length)  
>         {  
>             if (!MmIsAddressValid(cPtr)) break;  
>   
>             Length = SizeOfCode(cPtr, &pOpcode);  
>   
>             if (!Length || (Length == 1 && \*pOpcode == 0xC3)) break;  
>            
>             if (\*(PUSHORT)pOpcode == 0x888D)  
>             {  
>                 KeServiceDescriptorTableShadow = \*(PVOID \*)(pOpcode + 2);  
>                 break;  
>             }  
>         }  
>     }  
>   
>   
>   
> 3。从 KTHREAD.ServiceTable 里面搜索  
> 如果是 GUI 线程 那么就是指向 ServiceDescriptorTableShadow  
> 其实个人更加倾向这个办法 稳定安全可靠，上面那个代码也许会受到以后系统代码变化影响 但是这个肯定不会  
>   
> Copy code  
>   
>     GetServiceDescriptorTableShadowAddress proc uses esi edi ebx  
>   
>     local dwThreadId:DWORD  
>   
>         xor ebx, ebx                ; = NULL. Assume ServiceDescriptorTableShadow will be not found  
>   
>         mov eax, KeServiceDescriptorTable  
>         mov esi, \[eax\]  
>   
>         ; Find KTHREAD.ServiceTable field  
>         ; For non-GUI threads this field == KeServiceDescriptorTable  
>         ; and it points to ServiceDescriptorTable  
>         ; For GUI threads  
>         ; ServiceDescriptorTableShadow  
>   
>         invoke KeGetCurrentThread  
>         mov edi, 200h-4  
>         .while edi  
>             .break .if dword ptr \[eax\]\[edi\] == esi  
>             dec edi  
>         .endw  
>   
>         .if edi != 0  
>             ; edi = offset to ServiceTable field in KTHREAD structure  
>             mov dwThreadId, 080h  
>             .while dwThreadId < 400h  
>                 push eax                    ; reserve DWORD on stack  
>                 invoke PsLookupThreadByThreadId, dwThreadId, esp  
>                 pop ecx                        ; -> ETHREAD/KTHREAD  
>                 .if eax == STATUS\_SUCCESS  
>                     push dword ptr \[ecx\]\[edi\]  
>                     fastcall ObfDereferenceObject, ecx  
>                     pop eax  
>                     .if eax != esi  
>                         mov edx, MmSystemRangeStart  
>                         mov edx, \[edx\]  
>                         mov edx, \[edx\]  
>                         .if eax > edx        ; some stupid error checking  
>                             mov ebx, eax  
>                             invoke DbgPrint, $CTA0("FindShadowTable: Found in thread with ID: %X\\n"), dwThreadId  
>                             .break  
>                         .endif  
>                     .endif  
>                 .endif  
>                 add dwThreadId, 4  
>             .endw  
>         .endif  
>   
>         mov eax, ebx  
>         ret  
>   
>     GetServiceDescriptorTableShadowAddress endp  
>   
>   
>   
> 4.mj0011 所说的搜索有效内存地址的办法  
> Copy code  
>   
>     /\*  
>     define structure for the system service table  
>     \*/  
>     struct SYS\_SERVICE\_TABLE {  
>     void \*\*ServiceTable;  
>     unsigned long CounterTable;  
>     unsigned long ServiceLimit;  
>     void \*\*ArgumentsTable;  
>     };  
>   
>     SYSTEM\_DESCRIPTOR\_TABLE KeServiceDescriptorTableShadow;  
>   
>     /\*  
>     Define KeServiceDescriptorTable based on the SST structure  
>     \*/  
>     extern struct SYS\_SERVICE\_TABLE \*KeServiceDescriptorTable;  
>   
>     /\*  
>     Declare function GetServiceDescriptorShadowTableAddress()  
>     \*/  
>     //struct SYS\_SERVICE\_TABLE \* GetServiceDescriptorShadowTableAddress ();  
>     /\*  
>     Declare the KeAddSystemServiceTable. This is just a  
>     handle to the call function, it will be used by the function  
>     above to obtain the correct address of the KeServiceDescriptorShadowTable  
>     \*/  
>     \_\_declspec(dllimport) KeAddSystemServiceTable (ULONG, ULONG, ULONG, ULONG, ULONG);  
>   
>     struct SYS\_SERVICE\_TABLE \* GetServiceDescriptorShadowTableAddress ()  
>     {  
>     // First, obtain a pointer to KeAddSystemServiceTable  
>     unsigned char \*check = (unsigned char\*)KeAddSystemServiceTable;  
>     int i;  
>       //Initialize an instance of System Service Table, will be used to  
>     //obtain an address from KeAddSystemServiceTable  
>     struct SYS\_SERVICE\_TABLE \*rc=0;  
>     // Make 100 attempts to match a valid address with that of KeServiceDescriptorTable  
>     for (i=0; i<=99; i++) {  
>       \_\_try {  
>       // try to obtain an address from  KeAddSystemServiceTable  
>       rc = \*(struct SYS\_SERVICE\_TABLE\*\*)check;  
>       // if this address is NOT valid OR it itself is the address of  
>       //KeServiceDescriptorTable OR its first entry is NOT equal  
>       //to the first entry of KeServiceDescriptorTable  
>       if (!MmIsAddressValid (rc) || (rc == KeServiceDescriptorTable)  
>         || (memcmp (rc, KeServiceDescriptorTable, sizeof (\*rc)) != 0)) {  
>         // Proceed with the next address  
>         check++;  
>         // don't forget to reset the old address  
>         rc = 0;  
>       }  
>       } \_\_except (EXCEPTION\_EXECUTE\_HANDLER) { rc = 0; }  
>       // when the loop is completed, check if it produced a valid address  
>       if (rc)  
>       // because if it didn't, we failed to find the address of KeServiceDescriptorTableShadow  
>       break;  
>     }  
>     // otherwise, there is a valid address! So return it!  
>     return rc;  
>     }  
>   
>   
>   
>   
>   
> 二。函数名定位  
> 这个似乎没有多少好办法，解析 pdb 可以是可以 但是很麻烦  
> 不过还好的是 同一个版本的系统 调用号一样  
> 所以只需要 3 套 2k xp 2k3 的调用号就可以完成 hook  
> pdb 办法  
> SymInitialize 初始化  
> SymSetSearchPath “srv\*\*symbols\*http://msdl.microsoft.com/”  
> SymLoadModule  
> SymGetSymFromName  
> 老 v 曾经发出 1 个获取 shadow 地址和函数名称的工具 使用他可以很方便的获取  
> 具体代码可以 F5 查看  
> 以上读取 pdb 的方法测试使用的可以的  
> 但是实际使用很麻烦  
>   
> 经过分析 我找到了 1 个比较好的定位办法  
> 我们以 SetWindowsHookExA 为例子  
> Copy code  
>   
>     .text:77D311D1 ; HHOOK \_\_stdcall SetWindowsHookExA(int idHook, HOOKPROC lpfn, HINSTANCE hmod, DWORD dwThreadId)  
>     .text:77D311D1                public \_SetWindowsHookExA@16  
>     .text:77D311D1 \_SetWindowsHookExA@16 proc near  
>     .text:77D311D1  
>     .text:77D311D1 idHook          = dword ptr  8  
>     .text:77D311D1 lpfn            = dword ptr  0Ch  
>     .text:77D311D1 hModule        = dword ptr  10h  
>     .text:77D311D1 dwThreadId      = dword ptr  14h  
>     .text:77D311D1  
>     .text:77D311D1                mov    edi, edi  
>     .text:77D311D3                push    ebp  
>     .text:77D311D4                mov    ebp, esp  
>     .text:77D311D6                push    2              ; int  
>     .text:77D311D8                push    \[ebp+dwThreadId\] ; int  
>     .text:77D311DB                push    \[ebp+hModule\]  ; hModule  
>     .text:77D311DE                push    \[ebp+lpfn\]      ; int  
>     .text:77D311E1                push    \[ebp+idHook\]    ; int  
>     .text:77D311E4                call    \_SetWindowsHookExAW@20 ; SetWindowsHookExAW(x,x,x,x,x)  
>     .text:77D311E9                pop    ebp  
>     .text:77D311EA                retn    10h  
>     .text:77D311EA \_SetWindowsHookExA@16 endp  
>   
>     .text:77D2DCFD ; int \_\_stdcall SetWindowsHookExAW(int, int, HMODULE hModule, int, int)  
>     .text:77D2DCFD \_SetWindowsHookExAW@20 proc near        ; CODE XREF: SetWindowsHookExW(x,x,x,x)+13p  
>     .text:77D2DCFD                                        ; SetWindowsHookExA(x,x,x,x)+13p  
>     .text:77D2DCFD  
>     .text:77D2DCFD Filename        = word ptr -20Ch  
>     .text:77D2DCFD var\_4          = dword ptr -4  
>     .text:77D2DCFD arg\_0          = dword ptr  8  
>     .text:77D2DCFD arg\_4          = dword ptr  0Ch  
>     .text:77D2DCFD hModule        = dword ptr  10h  
>     .text:77D2DCFD arg\_C          = dword ptr  14h  
>     .text:77D2DCFD arg\_10          = dword ptr  18h  
>     .text:77D2DCFD  
>     .text:77D2DCFD                mov    edi, edi  
>     .text:77D2DCFF                push    ebp  
>     .text:77D2DD00                mov    ebp, esp  
>     .text:77D2DD02                sub    esp, 20Ch  
>     .text:77D2DD08                mov    eax, \_\_\_security\_cookie  
>     .text:77D2DD0D                push    esi  
>     .text:77D2DD0E                mov    esi, \[ebp+hModule\]  
>     .text:77D2DD11                test    esi, esi  
>     .text:77D2DD13                push    edi  
>     .text:77D2DD14                mov    edi, \[ebp+arg\_4\]  
>     .text:77D2DD17                mov    \[ebp+var\_4\], eax  
>     .text:77D2DD1A                jz      short loc\_77D2DD33  
>     .text:77D2DD1C                push    104h            ; nSize  
>     .text:77D2DD21                lea    eax, \[ebp+Filename\]  
>     .text:77D2DD27                push    eax            ; lpFilename  
>     .text:77D2DD28                push    esi            ; hModule  
>     .text:77D2DD29                call    ds:\_\_imp\_\_GetModuleFileNameW@12 ; GetModuleFileNameW(x,x,x)  
>     .text:77D2DD2F                test    eax, eax  
>     .text:77D2DD31                jz      short loc\_77D2DD52  
>     .text:77D2DD33  
>     .text:77D2DD33 loc\_77D2DD33:                          ; CODE XREF: SetWindowsHookExAW(x,x,x,x,x)+1Dj  
>     .text:77D2DD33                push    \[ebp+arg\_10\]  
>     .text:77D2DD36                mov    eax, esi  
>     .text:77D2DD38                push    edi  
>     .text:77D2DD39                push    \[ebp+arg\_0\]  
>     .text:77D2DD3C                neg    eax  
>     .text:77D2DD3E                push    \[ebp+arg\_C\]  
>     .text:77D2DD41                sbb    eax, eax  
>     .text:77D2DD43                lea    ecx, \[ebp+Filename\]  
>     .text:77D2DD49                and    eax, ecx  
>     .text:77D2DD4B                push    eax  
>     .text:77D2DD4C                push    esi  
>     .text:77D2DD4D                call    \_\_SetWindowsHookEx@24 ; \_SetWindowsHookEx(x,x,x,x,x,x)  
>     .text:77D2DD52  
>     .text:77D2DD52 loc\_77D2DD52:                          ; CODE XREF: SetWindowsHookExAW(x,x,x,x,x)+34j  
>     .text:77D2DD52                mov    ecx, \[ebp+var\_4\]  
>     .text:77D2DD55                pop    edi  
>     .text:77D2DD56                pop    esi  
>     .text:77D2DD57                call    @\_\_security\_check\_cookie@4 ; \_\_security\_check\_cookie(x)  
>     .text:77D2DD5C                leave  
>     .text:77D2DD5D                retn    14h  
>     .text:77D2DD5D \_SetWindowsHookExAW@20 endp  
>     .text:77D2DD5D  
>   
>     .text:77D2DD65 ; \_\_stdcall \_SetWindowsHookEx(x, x, x, x, x, x)  
>     .text:77D2DD65 \_\_SetWindowsHookEx@24 proc near        ; CODE XREF: SetWindowsHookExAW(x,x,x,x,x)+50p  
>     .text:77D2DD65  
>     .text:77D2DD65 var\_10          = byte ptr -10h  
>     .text:77D2DD65 var\_8          = dword ptr -8  
>     .text:77D2DD65 var\_4          = dword ptr -4  
>     .text:77D2DD65 arg\_0          = dword ptr  8  
>     .text:77D2DD65 arg\_4          = dword ptr  0Ch  
>     .text:77D2DD65 arg\_8          = dword ptr  10h  
>     .text:77D2DD65 arg\_C          = dword ptr  14h  
>     .text:77D2DD65 arg\_10          = dword ptr  18h  
>     .text:77D2DD65 arg\_14          = dword ptr  1Ch  
>     .text:77D2DD65  
>     .text:77D2DD65                mov    edi, edi  
>     .text:77D2DD67                push    ebp  
>     .text:77D2DD68                mov    ebp, esp  
>     .text:77D2DD6A                sub    esp, 10h  
>     .text:77D2DD6D                push    \[ebp+arg\_4\]  
>     .text:77D2DD70                and    \[ebp+var\_4\], 0  
>     .text:77D2DD74                lea    eax, \[ebp+var\_10\]  
>     .text:77D2DD77                push    eax  
>     .text:77D2DD78                mov    \[ebp+var\_8\], eax  
>     .text:77D2DD7B                call    ds:\_\_imp\_\_RtlInitUnicodeString@8 ; RtlInitUnicodeString(x,x)  
>     .text:77D2DD81                push    \[ebp+arg\_14\]  
>     .text:77D2DD84                push    \[ebp+arg\_10\]  
>     .text:77D2DD87                push    \[ebp+arg\_C\]  
>     .text:77D2DD8A                push    \[ebp+arg\_8\]  
>     .text:77D2DD8D                push    \[ebp+var\_8\]  
>     .text:77D2DD90                push    \[ebp+arg\_0\]  
>     .text:77D2DD93                call    \_NtUserSetWindowsHookEx@24 ; NtUserSetWindowsHookEx(x,x,x,x,x,x)  
>     .text:77D2DD98                leave  
>     .text:77D2DD99                retn    18h  
>     .text:77D2DD99 \_\_SetWindowsHookEx@24 endp  
>     .text:77D2DD99  
>   
>     .text:77D2DDA1 ; \_\_stdcall NtUserSetWindowsHookEx(x, x, x, x, x, x)  
>     .text:77D2DDA1 \_NtUserSetWindowsHookEx@24 proc near    ; CODE XREF: \_SetWindowsHookEx(x,x,x,x,x,x)+2Ep  
>     .text:77D2DDA1                mov    eax, 1225h  
>     .text:77D2DDA6                mov    edx, 7FFE0300h  
>     .text:77D2DDAB                call    dword ptr \[edx\]  
>     .text:77D2DDAD                retn    18h  
>     .text:77D2DDAD \_NtUserSetWindowsHookEx@24 endp  
>   
>   
>   
> 我们看这里 .text:77D2DDA1                mov    eax, 1225h  
> 里面有和 ssdt 一样的调用号 但是有个问题 NtUserSetWindowsHookEx 在 user32 里面没有导出，我们需要多次搜索才行  
> Copy code  
>   
>     HHOOK STDCALL    NtUserSetWindowsHookEx (HINSTANCE Mod, PUNICODE\_STRING UnsafeModuleName, DWORD ThreadId, int HookId, HOOKPROC HookProc, BOOL Ansi)  
>   
> 这个就是原型 不过搜索起来确实是比较麻烦的，不知道还有无其他好办法了 

> 三。如何 hook  
> 似乎这个问题并不大，shadow ssdt 和 ssdt 本质上都是 1 个地址表，最为简单的方法是把你的函数替换地址表的对应项，具体 hook 代码甚至可以完全照抄 ssdt 的，这里只说 1 下几个偶遇到的小问题  
> 1。win32k.sys 不是常在内存的，如果不是 GUI 线程，shadow ssdt 地址无效  
> 解决办法：  
> 1。在 driverdispatch 中 hokk  
> driverdispatch 是位于执行 driveriocontrol 的线程上下文的  
> 我们使用 1 个 GUI 线程去 driveriocontrol  
>   
> 2。attachtoprocess  
> 通常 我们使用 cerss.exe  
> Copy code  
>   
>     HANDLE GetCsrPid()  
>     {  
>         HANDLE                        Process, hObject;  
>         HANDLE                        CsrId = (HANDLE)0;  
>         OBJECT\_ATTRIBUTES            obj;  
>         CLIENT\_ID                    cid;  
>         UCHAR                        Buff\[0x100\];  
>         POBJECT\_NAME\_INFORMATION      ObjName = (PVOID)&Buff;  
>         PSYSTEM\_HANDLE\_INFORMATION\_EX Handles;  
>         ULONG                        r;  
>   
>         Handles = GetInfoTable(SystemHandleInformation);  
>   
>         if (!Handles) return CsrId;  
>   
>         for (r = 0; r < Handles->NumberOfHandles; r++)  
>         {  
>             if (Handles->Information\[r\].ObjectTypeNumber == 21) //Port object  
>             {  
>                 InitializeObjectAttributes(&obj, NULL, OBJ\_KERNEL\_HANDLE, NULL, NULL);  
>                
>                 cid.UniqueProcess = (HANDLE)Handles->Information\[r\].ProcessId;  
>                 cid.UniqueThread  = 0;  
>   
>                 if (NT\_SUCCESS(NtOpenProcess(&Process, PROCESS\_DUP\_HANDLE, &obj, &cid)))  
>                 {  
>                     if (NT\_SUCCESS(ZwDuplicateObject(Process,  
>                                                     (HANDLE)Handles->Information\[r\].Handle,  
>                                                     NtCurrentProcess(),  
>                                                     &hObject,  
>                                                       0, 0, DUPLICATE\_SAME\_ACCESS)))  
>                     {  
>                         if (NT\_SUCCESS(ZwQueryObject(hObject,  
>                                                     ObjectNameInformation,  
>                                                         ObjName,  
>                                                     0x100, NULL)))  
>                         {  
>                             if (ObjName->Name.Buffer &&  
>                                 !wcsncmp(L"\\\\Windows\\\\ApiPort", ObjName->Name.Buffer, 20))  
>                             {  
>                                 CsrId = (HANDLE)Handles->Information\[r\].ProcessId;  
>                             }  
>                         }  
>   
>                         ZwClose(hObject);  
>                     }  
>   
>                     ZwClose(Process);  
>                 }  
>             }  
>         }  
>   
>         ExFreePool(Handles);  
>   
>         return CsrId;  
>     }  
>   
>   
>   
> 然后我们 KeAttachProcess  
> Copy code  
>   
>       ntStatus = PsLookupProcessByProcessId(GetCsrPid(), &EProcess);  
>   
>             if (!NT\_SUCCESS( ntStatus ))  
>       {  
>   
>       DbgPrint("PsLookupProcessByProcessId()\\n");  
>                   return ntStatus;  
>       }  
>   
>             KeAttachProcess(EProcess);  
>   
>   
>   
> 3.使用 MDL 映射一块不分页内存，设置成可以写入  
> 不分页内存  ExAllocatePool(NonPagedPool  
> 即为不会被切换到 pagefile 的内存，是常驻在物理内存的，比较希缺，使用完毕以后记住释放掉  
> 参考 regmon 的代码  
> Copy code  
>   
>     PVOID \*  
>     RegmonMapServiceTable(  
>       SERVICE\_HOOK\_DESCRIPTOR \*\*ServiceIsHooked  
>       )  
>     {  
>       \*ServiceIsHooked = (SERVICE\_HOOK\_DESCRIPTOR\*)  
>                   ExAllocatePool(NonPagedPool, KeServiceDescriptorTable->TableSize);  
>   
>       if( \*ServiceIsHooked )  
>       {  
>         RtlZeroMemory(\*ServiceIsHooked, KeServiceDescriptorTable->TableSize);  
>   
>         KeServiceTableMdl = MmCreateMdl(0, KeServiceDescriptorTable->ServiceTable, KeServiceDescriptorTable->TableSize << 2);  
>   
>         if( KeServiceTableMdl )  
>         {  
>             MmBuildMdlForNonPagedPool(KeServiceTableMdl);  
>   
>             KeServiceTableMdl->MdlFlags |= 1;  
>   
>             return (PVOID\*)MmMapLockedPages(KeServiceTableMdl, KernelMode);  
>         }  
>       }  
>   
>       return NULL;  
>     };  
>   
>   
>   
> 看起来 shadowssdt 何 hook 本身并没有什么难度  
> 只要正确定位了地址，hook 起来和 ssdt 完全一样  
> unhook 不在重复 完全抄 ssdt 的代码即可  
>   
> 四。checkhook  
> hook 用的人多了 难免出现重复 hook 的情况 为了实现和别人的东西和平共处，最好 hook 之前做个检测  
> hook 的方法是固定的，就那么 2 种 改地址表 或者 inline hook，我们分别来对待  
> 1。修改地址表的 hook  
>   这个对付起来几乎不用特别对待，只是你保存的 OldFunc 是其他的程序 hook 函数所在的地址，判断方法很简单，看看这个地址在不在 win32k 的模块里面，和平共处的办法是你的 myfunc 里面 call 别人的 hook 函数，当然 如果你不想要别人的 hook 了，直接搜索到原始地址 call 之也是可行的  
> 另外说 1 下 在别人的函数里面搜索原始地址 用 MmIsAddressValid＋ismodulewin32k  
> 可以判断（这个办法好像是在 mj 文章里面看到的 记不清了）  
> 2。inlinehook  
> 这个比较麻烦了 如果是修改前面 5 个字节还好，直接替换处理都可以不改变  
> 麻烦是函数中间地方被 hook，盲目修改可能直接 BSOD 了，这个需要 1 个反汇编引擎来帮助分析代码字节长度，海风大大的 hooklib 很不错 可以完成这个功能  
>   
> 五，简单说 1 下 restore  
> Copy code  
>   
>     pWin32k = GetModuleHandle("win32k.sys");  
>     SdtRVA = &KeServiceDescriptorTable->win32k.ServiceTable - pWin32k ;  
>     w32kCopy = LoadPeFile("win32k.sys");  
>     ProcessRelocs(w32kCopy, pWin32k);  
>     memcpy(&KeServiceDescriptorTable->win32k.ServiceTable, w32kCopy + SdtRVA, SdtSize);  
>   
> 不过 其实可以不用 ProcessRelocs 处理  
> 因为你可以使用 lordpe 查看 1 下 win32k.sys 的默认基址就是系统加载的基址  
> 可以不需要 ProcessRelocs，不过为了更加方便和通用，这么做也可以的  
>   
> 休息去咯－－－ 

> ssdt 在哪里？在 ntoskrnl.exe 中。内存中 ntoskrnl.exe 位置在 80000000 以上，直接读写不方便。  
>   
> 所以我以前做的时候，先读取硬盘上的 ntoskrnl.exe 文件，根据 export table 定位原始的 SSDT，再用物理内存与虚拟内存映射的办法转换，得到系统中 SSDT 的位置，对此进行处理（rootkit 检测，hook/unhook） 

> 方法三是有问题的，HOOK SSDT 还是没问题，HOOK SSDT Shadow 是有问题，不知道你尝试过没有。  
>   
> mjxxx 的方法可以参见卡巴的 FindSystemServiceDescriptionTableShadow 基本一致，  
> 卡巴的相对安全一些:)  
>   
>   
> 我 HOOK Shadow 使用的是修改 ServerPointer. 指向我自己的区域。然后转跳到原始的里面去 

> combojiang：  
> 你仔细看 1 下 是地址不同 操作一样 都是 lea     ecx   
>   
> DiKeN“  
> 感谢大牛对偶小文的关注 确实存在问题  
> 我想应该是 GUI 线程的问题  
>   
> 参考  
> 
> 引用:
> 
> | 
> 
> * * *
> 
> 我们所说的线程，实际上分为核心态和用户态两部分。Win32 下这两者基本上是 1 对 1 的关系，其他平台如 Solaris 或 2.4 以前的 Linux 则使用不同的映射模型。而 Win32 系统中核心态的线程，实际上也分为两类：非 GUI 和 GUI 线程。前者是建立核心线程的缺省类型，后者在线程第一次使用 Win32k.sys 系统服务时自动转换，或者使用 PsConvertToGuiThread 函数(ntos\\ps\\psquery.c:3247)显式转换。两者之间的区别主要在于使用的资源缺省大小不同，以及使用的系统服务描述表不同。这也是为什么系统服务描述表要分为 KeServiceDescriptorTable 和 KeServiceDescriptorTableShadow 的原因之一，后者包括前者没有的对 GDI 服务的入口函数地址，一般在 Win32k.sys 中实现。核心线程对象的 ETHREAD::KTHREAD:: ServiceTable 字段保存了此线程适用的系统调用服务表地址，此字段也被 PsConvertToGuiThread 函数用于判断线程类型。功能与 Windows XP/2003 提供的 IsGUIThread 函数类型。  
>      使用上我们可以创建一个线程，此线程不做任何实际工作，只是根据我们要取哪个系统服务描述表来决定是否调用 GDI 函数 
> 
> * * *
> 
>  |
> | --- |

 4dd466d0e1e0cc123afbe504da2462ca