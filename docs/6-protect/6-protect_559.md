

# 软件保护壳技术专题 - 反调试器技术

> 原文：[`www.pediy.com/kssd/pediy10/71113.html`](https://www.pediy.com/kssd/pediy10/71113.html)

反调试是软件保护壳的最基本的功能之一。  
反调试方法也是多种多样。通过调用标准的 API 接口，计算指令时间差。查看当调试器加载后的  
内存的一些标志，还有就是判断当前运行环境是否合乎逻辑等方法。这里收集了一些反调试的方法，其中的命名规则使用了壳狼的反调试程序的方式，希望不要和我收取版权的费用。^\_^,其中借鉴了不少壳狼的函数。增加了一些，也删除了一些。大部分的参考资料来自<<脱壳的艺术>>,<<Anti-UnPacker Tricks>>与<<加密与解密第三版>>。  
这里要说明的一点是。每个函数编写都是自己建立堆栈了，看的不习惯的多看下就习惯了 呵呵。  
原因也很简单，MASM 不允许在函数内定义函数了。  
这些函数还有一个要讲的是。因为最后这些函数要在以后的章节中用到  
为了能允许用户自定义反调试的功能。免去重定位的麻烦，所以  
利用栈传递了 API 集合地址和外部函数集合的地址。  
朋友们还是先看代码了。。。  
  
  
利用 IsDebuggerPresent 确定是否存在,IsDebuggerPresent 是 WIN 提供的一个标准调试 API  
用于确定是否存在调试器。这个方法很简单 TRUE 为存在,FASLE 则为不存在。  

代码:

```
FD\_IsDebuggerPresent:
FD\_IsDebugger\_Arg\_Win32Api      equ 04h
    mov eax, dword ptr \[esp+FD\_IsDebugger\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    call dword ptr \[eax\].xIsDebuggerPresent
Exit\_FD\_IsDebuggerPresent:
    assume eax : nothing
    retn 04h
End\_FD\_IsDebuggerPresent:

```

我们更进一步的剖析 IsDebuggerPresent 函数，逆向它后即可得知。  
这个函数读取当前进程的 PEB 中的 BeingDebugger 标志。  
如果确定这个标志呢？  
当进程加载的时候 FS 寄存器总是被设置成当前线程的 TEB。这下就都 OK  
而在 TEB 的 30h 偏移处就是 ProcessEnvironmentBlock 了。  
而 PEB 的偏移 02h 出就是 BeingDebugged 了。当 BeingDebugger 为 0 则没有调试器  
不为 0 时则存在调试器。  
代码:

```
FD\_PEB\_BeingDebuggedFlag:
    assume fs : nothing
    mov eax, fs:\[30h\]   ; eax = TEB.ProcessEnvironmentBlock
    inc eax
    inc eax
    mov eax, dword ptr \[eax\]
    and eax, 000000FFh  ; al = PEB.BeingDebugged
    test eax, eax
    jnz FD\_PEB\_BeingDebuggedFlag\_Found
Exit\_PEB\_BeingDebuggedFlag:
    retn 0    
FD\_PEB\_BeingDebuggedFlag\_Found:
    mov eax, 1
    jmp  Exit\_PEB\_BeingDebuggedFlag
End\_FD\_PEB\_BeingDebuggedFlag:

```

在当 BeingDebugged 被设置为 TRUE 时，存在与 PEB 中的  
NtGlobalFlag 也会做出相应的改变。查看 WIN2K 代码的 LdrpInitialize  
会发现以下代码  
if (Peb->BeingDebugged)  
Peb->NtGlobalFlag |= \*\*\*\_HEAP\_ENABLE\_FREE\_CHECK |  
                     \*\*\*\_HEAP\_ENABLE\_TAIL\_CHECK |  
                     \*\*\*\_HEAP\_VALIDATE\_PARAMETERS;  
这个组合值为 70h。所以我们又得到一个反调试的函数  
代码:

```
FD\_PEB\_NtGlobalFlags:
    assume fs : nothing
    mov eax, fs:\[30h\]
    mov eax, dword ptr \[eax+68h\]
    and eax, 070h
    test eax, eax
    jnz FD\_PEB\_NtGlobalFlags\_Found
Exit\_FD\_PEB\_NtGlobalFlags:
    retn 0
FD\_PEB\_NtGlobalFlags\_Found:
    mov eax, 1
    jmp  Exit\_FD\_PEB\_NtGlobalFlags
End\_FD\_PEB\_NtGlobalFlags:

```

BeingDebugger 标志还会影响 ProcessHeap.Flags  
如果这个标志不为 0 则存在调试器。  
代码:

```
FD\_Heap\_ForceFlags:
    assume fs : nothing
    mov eax, fs:\[30h\]
    mov eax, dword ptr \[eax+18h\]    ; PEB.ProcessHeap
    mov eax, dword ptr \[eax+10h\]    ; PEB.ProcessHeap.Flags
    test eax, eax
    jnz Found\_FD\_Heap\_ForceFlags
Exit\_FD\_Heap\_ForceFlag:
    retn 0
Found\_FD\_Heap\_ForceFlags:
    mov eax, 1
    jmp Exit\_FD\_Heap\_ForceFlag           
End\_FD\_Heap\_ForceFlags:

```

在 BeingDebugger 下 ProcessHeap.ForceFlags 也受到影响  
如果不为 2 则存在调试器。细节部分可以查看 WIN2K 的代码。  
由于版本影响这里就不给出其中的代码了。  
代码:

```
FD\_Heap\_HeapFlags:
    assume fs : nothing
    mov eax, fs:\[30h\]
    mov eax, dword ptr \[eax+18h\]    ; PEB.ProcessHeap
    mov eax, dword ptr \[eax+0ch\]    ; PEB.ProcessHeap.ForceFlags
    cmp eax, 2
    jnz Found\_FD\_Heap\_HeapFlags
Exit\_FD\_Heap\_HeapFlags:
    retn 0  
Found\_FD\_Heap\_HeapFlags:
    mov eax, 1
    jmp Exit\_FD\_Heap\_HeapFlags  
End\_FD\_Heap\_HeapFlags:

```

通过另一个 API。CheckRemoteDebuggerPresent，这个 API 可以检测任何进程是否被调试器  
加载。这里通过 GetCurrentProcess 取得自身进程的句柄。进行鉴别  
代码:

```
FD\_CheckRemoteDebuggerPresent:
FD\_CheckRemoteDebuggerPresent\_Arg\_Win32Api    equ 04h
    mov eax, dword ptr \[esp+FD\_CheckRemoteDebuggerPresent\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    push esp
    push esp
    call dword ptr \[eax\].xGetCurrentProcess
    push eax
    call dword ptr \[eax\].xCheckRemoteDebuggerPresent
    pop esp
    assume eax : nothing
    retn 04h
End\_FD\_CheckRemoteDebuggerPresent:

```

如果逆向了 CheckRemoteDebuggerPresent 函数就可以明白，其实最终它是调用  
NtQueryInformationProcess，查询进程的 ProcessDebugPort。此值是用来维持系统  
与调试器通讯的，在程序被调试器加载时系统会设置这个值为非 0。  
代码:

```
FD\_NtQueryInfoProc\_DbgPort:
FD\_NtQueryInfoProc\_DbgPort\_Arg\_Win32Api    equ 08h
FD\_NtQueryInfoProc\_DbgPort\_StackSize    equ sizeof PROCESS\_DEBUG\_PORT\_INFO
FD\_NtQueryInfoProc\_DbgPort\_ProcessInfo  equ -(FD\_NtQueryInfoProc\_DbgPort\_StackSize)
    push ebp
    mov ebp, esp
    sub esp, FD\_NtQueryInfoProc\_DbgPort\_StackSize
    
    push ebx
    
    mov ebx, dword ptr \[ebp+FD\_NtQueryInfoProc\_DbgPort\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    push NULL
    push sizeof PROCESS\_DEBUG\_PORT\_INFO
    lea eax, \[ebp+FD\_NtQueryInfoProc\_DbgPort\_ProcessInfo\]
    push eax
    push ProcessDebugPort
    call dword ptr \[ebx\].xGetCurrentProcess
    push eax
    call dword ptr \[ebx\].xZwQueryInformationProcess
    test eax, eax
    jnz FD\_NtQueryInfoProc\_DbgPort\_Tmp1
    lea eax, \[ebp+FD\_NtQueryInfoProc\_DbgPort\_ProcessInfo\]
    assume eax : ptr PROCESS\_DEBUG\_PORT\_INFO
    mov eax, dword ptr \[eax\].DebugPort
    test eax, eax
    jnz Found\_FD\_NtQueryInfoProc\_DbgPort
FD\_NtQueryInfoProc\_DbgPort\_Tmp1:
    xor eax, eax
Exit\_FD\_NtQueryInfoProc\_DbgPort:
    assume eax : nothing
    assume ebx : nothing
    
    pop ebx
    
    mov esp, ebp
    pop ebp
    retn 04h
Found\_FD\_NtQueryInfoProc\_DbgPort:
    mov eax, 1
    jmp Exit\_FD\_NtQueryInfoProc\_DbgPort         
End\_FD\_NtQueryInfoProc\_DbgPort:

```

当 Windows 系统创建一个调试会话开始，一个调试对象也被创建并产生一个  
调试句柄。我们可以查询这个句柄的值来确定是否存在调试器。  
代码:

```
FD\_NtQueryInfoProc\_DbgObjHandle:
FD\_NtQueryInfoProc\_DbgObjHandle\_Arg\_Win32Api    equ 08h
FD\_NtQueryInfoProc\_DbgObjHandle\_StackSize        equ sizeof PROCESS\_DEBUG\_OBJECTHANDLE\_INFO
FD\_NtQueryInfoProc\_DbgObjHandle\_ProcessInfo     equ -(FD\_NtQueryInfoProc\_DbgObjHandle\_StackSize)
    push ebp
    mov ebp, esp
    sub esp, FD\_NtQueryInfoProc\_DbgObjHandle\_StackSize
    
    push ebx
    mov ebx, dword ptr \[ebp+FD\_NtQueryInfoProc\_DbgObjHandle\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    push NULL
    push sizeof PROCESS\_DEBUG\_OBJECTHANDLE\_INFO
    lea eax, \[ebp+FD\_NtQueryInfoProc\_DbgObjHandle\_ProcessInfo\]
    push eax    
    push SystemNotImplemented8
    call dword ptr \[ebx\].xGetCurrentProcess
    push eax
    call dword ptr \[ebx\].xZwQueryInformationProcess
    test eax, eax
    jnz FD\_NtQueryInfoProc\_DbgObjHandle\_Tmp1
    lea eax, \[ebp+FD\_NtQueryInfoProc\_DbgObjHandle\_ProcessInfo\]
    assume eax : ptr PROCESS\_DEBUG\_OBJECTHANDLE\_INFO
    mov eax, dword ptr \[eax\].ObjectHandle
    test eax, eax
    jnz Found\_FD\_NtQueryInfoProc\_DbgObjHandle
FD\_NtQueryInfoProc\_DbgObjHandle\_Tmp1:
    xor eax, eax
Exit\_FD\_NtQueryInfoProc\_DbgObjHandle:

    assume eax : nothing
    assume ebx : nothing
    
    pop ebx
    
    mov esp, ebp
    pop ebp
    retn 04h
Found\_FD\_NtQueryInfoProc\_DbgObjHandle:
    mov eax, 1
    jmp Exit\_FD\_NtQueryInfoProc\_DbgObjHandle
End\_FD\_NtQueryInfoProc\_DbgObjHandle:

```

也可以通过使用 ZwQueryInformationProcess 函数,利用 SystemNotImplemented9(1fh)  
返回的 PROCESS\_DEBUG\_FLAGS\_INFO 结构，如果 DebugFlags 返回 0 则检测到调试器。返回非 0  
则没发现调试器。  
代码:

```
FD\_NtQueryInfoProc\_DbgFlags:
FD\_NtQueryInfoProc\_DbgFlags\_Arg\_Win32Api    equ 08h
FD\_NtQueryInfoProc\_DbgFlags\_StackSize    equ sizeof PROCESS\_DEBUG\_FLAGS\_INFO
FD\_NtQueryInfoProc\_DbgFlags\_ProcessInfo  equ -(FD\_NtQueryInfoProc\_DbgFlags\_StackSize)
    push ebp
    mov ebp, esp
    sub esp, FD\_NtQueryInfoProc\_DbgFlags\_StackSize
    
    push ebx
    
    mov ebx, dword ptr \[ebp+FD\_NtQueryInfoProc\_DbgFlags\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    push NULL
    push sizeof PROCESS\_DEBUG\_FLAGS\_INFO
    lea eax, \[ebp+FD\_NtQueryInfoProc\_DbgFlags\_ProcessInfo\]
    push eax    
    push SystemNotImplemented9
    call dword ptr \[ebx\].xGetCurrentProcess
    push eax
    call dword ptr \[ebx\].xZwQueryInformationProcess
    test eax, eax
    jnz FD\_NtQueryInfoProc\_DbgFlags\_Tmp1
    lea eax, \[ebp+FD\_NtQueryInfoProc\_DbgFlags\_ProcessInfo\]
    assume eax : ptr PROCESS\_DEBUG\_FLAGS\_INFO
    mov eax, dword ptr \[eax\].DebugFlags
    test eax, eax
    jz Found\_FD\_NtQueryInfoProc\_DbgFlags
FD\_NtQueryInfoProc\_DbgFlags\_Tmp1: 
    xor eax, eax
Exit\_FD\_NtQueryInfoProc\_DbgFlags:
    assume eax : nothing
    assume ebx : nothing
    
    pop ebx
    
    mov esp, ebp
    pop ebp
    retn 04h
Found\_FD\_NtQueryInfoProc\_DbgFlags:
    mov eax, 1
    jmp Exit\_FD\_NtQueryInfoProc\_DbgFlags
End\_FD\_NtQueryInfoProc\_DbgFlags:

```

如果 Windows 以调试方式启动，并与系统调试器建立通讯。  
通过 ZwQuerySystemInformation 对 SystemKernelDebuggerInformation 进行查询  
系统中是否存在系统调试器。  
代码:

```
FD\_NtQueryInfoProc\_SysKrlDbgInfo:
FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Arg\_Win32Api    equ 08h
FD\_NtQueryInfoProc\_SysKrlDbgInfo\_StackSize      equ sizeof PROCESS\_DEBUG\_FLAGS\_INFO
FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Info           equ -(sizeof PROCESS\_DEBUG\_FLAGS\_INFO)
    push ebp
    mov ebp, esp
    sub esp, FD\_NtQueryInfoProc\_SysKrlDbgInfo\_StackSize
    
    push ebx
    
    mov ebx, dword ptr \[ebp+FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    push NULL
    push sizeof PROCESS\_DEBUG\_FLAGS\_INFO
    lea eax, \[ebp+FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Info\]
    push eax    
    push SystemKernelDebuggerInformation
    call dword ptr \[ebx\].xGetCurrentProcess
    push eax
    call dword ptr \[ebx\].xZwQuerySystemInformation
    test eax, eax
    jnz FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Tmp1
    lea eax, \[ebp+FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Info\]
    assume eax : ptr PROCESS\_DEBUG\_FLAGS\_INFO
    mov eax, dword ptr \[eax\].DebugFlags
    test eax, eax
    jz Found\_FD\_NtQueryInfoProc\_SysKrlDbgInfo
FD\_NtQueryInfoProc\_SysKrlDbgInfo\_Tmp1:    
    xor eax, eax
Exit\_FD\_NtQueryInfoProc\_SysKrlDbgInfo:
    assume eax : nothing
    assume ebx : nothing
    
    pop ebx
    
    mov esp, ebp
    pop ebp
    retn 04h
Found\_FD\_NtQueryInfoProc\_SysKrlDbgInfo:
    mov eax, 1
    jmp Exit\_FD\_NtQueryInfoProc\_SysKrlDbgInfo
End\_FD\_NtQueryInfoProc\_SysKrlDbgInfo:

```

当调试会话被创建,这个标志会影响堆的创建。初始化中的堆内存填充了  
很多类似 0ABABABABh,0BAADF00Dh,0FEEEFEEEh 这三个值。可以通过检测内存看  
是否存在过多的这样值。判断调试器的存在，而正常启动的程序则不会被填充。  
代码:

```
FD\_Heap\_Magic:
FD\_Heap\_Magic\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr \[esp+FD\_Heap\_Magic\_Arg\_Win32Api\]
    
    push ebx
    push ecx
    push edx
    push esi
    push edi
    
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    
    push 100h
    push NULL
    call dword ptr \[ebx\].xGetProcessHeap
    mov edi, eax    ; HeapHandle
    push eax
    call dword ptr \[ebx\].xHeapAlloc
    mov esi, eax    ; HeapMem
    xor ecx, ecx
    mov edx, 100h
    cld
    FD\_Heap\_Magic\_Loop:
    lodsd
    cmp eax, 0ABABABABh
    jnz FD\_Heap\_Magic\_Tmp1
    inc ecx
    FD\_Heap\_Magic\_Tmp1:
    cmp eax, 0BAADF00Dh
    jnz FD\_Heap\_Magic\_Tmp2
    inc ecx        
    FD\_Heap\_Magic\_Tmp2:
    cmp eax, 0FEEEFEEEh
    jnz FD\_Heap\_Magic\_Tmp3
    inc ecx
    FD\_Heap\_Magic\_Tmp3:
    sub edx, 04h
    jnz FD\_Heap\_Magic\_Loop
    push ecx    
    ;; free heap
    push esi
    push HEAP\_NO\_SERIALIZE
    push edi
    call dword ptr \[ebx\].xHeapFree
    pop ecx
    ;; judge count
    cmp ecx, 10h
    jae Found\_FD\_Heap\_Magic
    xor eax, eax
Exit\_FD\_Heap\_Magic:    
    pop edi    
    pop esi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing
    retn 04h
Found\_FD\_Heap\_Magic:
    mov eax, 1
    jmp Exit\_FD\_Heap\_Magic
End\_FD\_Heap\_Magic:

```

一般程序是没有被设置 SeDebugPrivilege，如果一个当前进程被设置 SeDebugPrivilege 后  
它就拥有了完全控制 CSRSS.EXE 的权限。通过进程表快照取得 CSRSS.EXE 进程的 PID，之后  
通过 OpenProcess 以 PROCESS\_ALL\_ACCESS 打开。开是否能打开此进程。  
代码:

```
FD\_SeDebugPrivilege:
FD\_SeDebugPrivilege\_Arg\_Win32Api    equ 08h
FD\_SeDebugPrivilege\_StackSize       equ 10h + sizeof PROCESSENTRY32
FD\_SeDebugPrivilege\_hProcessSnap    equ -04h
FD\_SeDebugPrivilege\_PID\_csrss       equ -08h
FD\_SeDebugPrivilege\_FingFlag        equ -0ch
FD\_SeDebugPrivilege\_pe32            equ -(10h+sizeof PROCESSENTRY32)
    push ebp
    mov ebp, esp
    sub esp, FD\_SeDebugPrivilege\_StackSize
    
    push ebx
    push ecx
    push edi
    
    ;; clear stack
    lea edi, \[ebp-FD\_SeDebugPrivilege\_StackSize\]
    mov ecx, FD\_SeDebugPrivilege\_StackSize
    xor eax, eax
    cld
    rep stosb
    
    mov ebx, dword ptr \[ebp+FD\_SeDebugPrivilege\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    lea edi, \[ebp+FD\_SeDebugPrivilege\_pe32\]
    assume edi : ptr PROCESSENTRY32
                     
    push 0
    push TH32CS\_SNAPPROCESS
    call dword ptr \[ebx\].xCreateToolhelp32Snapshot
    cmp eax, INVALID\_HANDLE\_VALUE
    jz NotFound\_FD\_SeDebugPrivilege
    mov dword ptr \[ebp+FD\_SeDebugPrivilege\_hProcessSnap\], eax
    push sizeof PROCESSENTRY32
    pop dword ptr \[edi\].dwSize
    
    push edi
    push dword ptr \[ebp+FD\_SeDebugPrivilege\_hProcessSnap\]
    call dword ptr \[ebx\].xProcess32First
    test eax, eax
    jnz FD\_SeDebugPrivilege\_Loop
    push dword ptr \[ebp+FD\_SeDebugPrivilege\_hProcessSnap\]
    call dword ptr \[ebx\].xCloseHandle
    jmp NotFound\_FD\_SeDebugPrivilege
    
    FD\_SeDebugPrivilege\_Loop:
    call FD\_SeDebugPrivilege\_Str
        db 'CSRSS.EXE',0
    FD\_SeDebugPrivilege\_Str:
    lea eax, \[edi\].szExeFile
    push eax    
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jnz FD\_SeDebugPrivilege\_Tmp2
    push dword ptr \[edi\].th32ProcessID
    pop dword ptr \[ebp+FD\_SeDebugPrivilege\_PID\_csrss\]
    push TRUE
    pop dword ptr \[ebp+FD\_SeDebugPrivilege\_FingFlag\]
    FD\_SeDebugPrivilege\_Tmp2:
    mov eax, dword ptr \[ebp+FD\_SeDebugPrivilege\_FingFlag\]
    test eax, eax
    jnz FD\_SeDebugPrivilege\_Tmp3
    push edi
    push dword ptr \[ebp+FD\_SeDebugPrivilege\_hProcessSnap\]
    call dword ptr \[ebx\].xProcess32Next
    test eax, eax
    jnz FD\_SeDebugPrivilege\_Loop
    
    FD\_SeDebugPrivilege\_Tmp3:
    mov eax, dword ptr \[ebp+FD\_SeDebugPrivilege\_FingFlag\]
    test eax, eax
    jz FD\_SeDebugPrivilege\_Tmp4
    push dword ptr \[ebp+FD\_SeDebugPrivilege\_PID\_csrss\]
    push FALSE
    push PROCESS\_QUERY\_INFORMATION
    call dword ptr \[ebx\].xOpenProcess
    test eax, eax
    jz FD\_SeDebugPrivilege\_Tmp4
    push dword ptr \[ebp+FD\_SeDebugPrivilege\_hProcessSnap\]
    call dword ptr \[ebx\].xCloseHandle
    jmp Found\_FD\_SeDebugPrivilege
    FD\_SeDebugPrivilege\_Tmp4:
    push dword ptr \[ebp+FD\_SeDebugPrivilege\_hProcessSnap\]
    call dword ptr \[ebx\].xCloseHandle
    jmp NotFound\_FD\_SeDebugPrivilege
Exit\_FD\_SeDebugPrivilege:
    pop edi
    pop ecx
    pop ebx
    assume ebx : nothing
    assume edi : nothing
    mov esp, ebp
    pop ebp
    retn 04h
NotFound\_FD\_SeDebugPrivilege:
    xor eax, eax
    jmp Exit\_FD\_SeDebugPrivilege
Found\_FD\_SeDebugPrivilege:
    mov eax, 1
    jmp Exit\_FD\_SeDebugPrivilege
End\_FD\_SeDebugPrivilege:

```

通过逻辑的判断也可以找到调试器所在，一般来讲程序都是有 explorer.exe 进程启动的(通过双击)  
如果我们的进程的父进程不是 explorer.exe 则发现调试器。如果有调试的名称冒名是 explorer.exe  
那么我们判断父进程 ID 后进一步判断 explorer.exe 进程的路径是否存在于 Windows 目录下。如果不是  
则发现调试器。此类方法也可以也被病毒用作穿透仿真机。  
代码:

```
FD\_Parent\_Process:
FD\_Parent\_Process\_Arg\_Win32Api      equ 08h
FD\_Parent\_Process\_StackSize         equ MAX\_PATH + sizeof PROCESSENTRY32 + sizeof MODULEENTRY32 + 20h
FD\_Parent\_Process\_hParnet           equ -04h
FD\_Parent\_Process\_PIDExplorer       equ -08h
FD\_Parent\_Process\_PIDParent         equ -0ch
FD\_Parent\_Process\_PIDChild          equ -10h
FD\_Parent\_Process\_hSnapshot         equ -14h
FD\_Parent\_Process\_pe32              equ -(20h + PROCESSENTRY32)
FD\_Parent\_Process\_me32              equ -(20h + PROCESSENTRY32 + MODULEENTRY32)
FD\_Parent\_Process\_lpszSystemInfo    equ -(20h + PROCESSENTRY32 + MODULEENTRY32 + MAX\_PATH)
    push ebp
    mov ebp, esp
    sub esp, FD\_Parent\_Process\_StackSize
    
    push ebx
    push ecx
    push edi
    push esi
    
    ;; clear the stack
    lea edi, \[ebp-FD\_Parent\_Process\_StackSize\]
    xor eax, eax
    mov ecx, FD\_Parent\_Process\_StackSize
    cld
    rep stosb
    
    mov ebx, dword ptr \[ebp+FD\_Parent\_Process\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    assume eax : ptr PROCESSENTRY32
    push sizeof PROCESSENTRY32
    pop dword ptr \[eax\].dwSize
    
    call dword ptr \[ebx\].xGetCurrentProcessId
    mov dword ptr \[ebp+FD\_Parent\_Process\_PIDChild\], eax
    
    push 0
    push TH32CS\_SNAPPROCESS
    call dword ptr \[ebx\].xCreateToolhelp32Snapshot
    mov dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\], eax
    
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    push eax
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xProcess32First
    test eax, eax
    jz FD\_Parent\_Process\_Tmp1
    FD\_Parent\_Process\_Loop1:
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    push eax
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xProcess32Next
    test eax, eax
    jz FD\_Parent\_Process\_Tmp2
    call FD\_Parent\_Process\_Str1
        db "EXPLORER.EXE",0
    FD\_Parent\_Process\_Str1:
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    lea eax, \[eax\].szExeFile    
    push eax
    call dword ptr \[ebx\].xlstrcmpiA
    jnz FD\_Parent\_Process\_Tmp3
    mov eax, dword ptr \[ebp+FD\_Parent\_Process\_PIDExplorer\]
    test eax, eax
    jnz FD\_Parent\_Process\_Tmp3
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    assume eax : ptr PROCESSENTRY32
    push dword ptr \[eax\].th32ProcessID
    pop dword ptr \[ebp+FD\_Parent\_Process\_PIDExplorer\]
    FD\_Parent\_Process\_Tmp3:
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    mov eax, dword ptr \[eax\].th32ProcessID
    sub eax, dword ptr \[ebp+FD\_Parent\_Process\_PIDChild\]
    jnz FD\_Parent\_Process\_Tmp4
    lea eax, \[ebp+FD\_Parent\_Process\_pe32\]
    push dword ptr \[eax\].th32ParentProcessID
    pop dword ptr \[ebp+FD\_Parent\_Process\_PIDParent\]
    FD\_Parent\_Process\_Tmp4:
    jmp FD\_Parent\_Process\_Loop1
    FD\_Parent\_Process\_Tmp1:
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xCloseHandle
    jmp NotFound\_FD\_Parent\_Process
    FD\_Parent\_Process\_Tmp2:
    mov eax, dword ptr \[ebp+FD\_Parent\_Process\_PIDExplorer\]
    sub eax, dword ptr \[ebp+FD\_Parent\_Process\_PIDParent\]
    jz FD\_Parent\_Process\_Tmp5
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xCloseHandle
    jmp Found\_FD\_Parent\_Process
    FD\_Parent\_Process\_Tmp5:
    lea eax, \[ebp+FD\_Parent\_Process\_me32\]
    assume eax : ptr MODULEENTRY32
    push sizeof MODULEENTRY32
    pop dword ptr \[eax\].dwSize  
    push dword ptr \[ebp+FD\_Parent\_Process\_PIDExplorer\]
    push TH32CS\_SNAPMODULE
    call dword ptr \[ebx\].xCreateToolhelp32Snapshot
    mov dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\], eax
    lea eax, \[ebp+FD\_Parent\_Process\_me32\]
    push eax
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xModule32First
    test eax, eax
    jz FD\_Parent\_Process\_Tmp6
    FD\_Parent\_Process\_Loop2:
    lea eax, \[ebp+FD\_Parent\_Process\_me32\]
    mov eax, dword ptr \[eax\].th32ProcessID
    sub eax, dword ptr \[ebp+FD\_Parent\_Process\_PIDExplorer\]
    jnz FD\_Parent\_Process\_Tmp7
    push MAX\_PATH
    lea eax, \[ebp+FD\_Parent\_Process\_lpszSystemInfo\]
    push eax
    call dword ptr \[ebx\].xGetWindowsDirectoryA
    call FD\_Parent\_Process\_Str2
        db '\\',0
    FD\_Parent\_Process\_Str2:
    lea eax, \[ebp+FD\_Parent\_Process\_lpszSystemInfo\]
    push eax
    call dword ptr \[ebx\].xlstrcatA
    call FD\_Parent\_Process\_Str3
        db "EXPLORER.EXE",0
    FD\_Parent\_Process\_Str3:
    lea eax, \[ebp+FD\_Parent\_Process\_lpszSystemInfo\]
    push eax
    call dword ptr \[ebx\].xlstrcatA
    lea eax, \[ebp+FD\_Parent\_Process\_lpszSystemInfo\]
    push eax
    lea eax, \[ebp+FD\_Parent\_Process\_me32\]
    lea eax, \[eax\].szExePath
    push eax
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz FD\_Parent\_Process\_Tmp6
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xCloseHandle
    jmp Found\_FD\_Parent\_Process
    FD\_Parent\_Process\_Tmp7:
    lea eax, \[ebp+FD\_Parent\_Process\_me32\]
    push eax
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xModule32Next
    test eax, eax
    jnz FD\_Parent\_Process\_Loop2
    FD\_Parent\_Process\_Tmp6:
    push dword ptr \[ebp+FD\_Parent\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xCloseHandle
    jmp NotFound\_FD\_Parent\_Process
Exit\_FD\_Parent\_Process:
    pop esi
    pop edi
    pop ecx
    pop ebx
    assume eax : nothing
    assume ebx : nothing           
    mov esp, ebp
    pop ebp
    retn 04h
NotFound\_FD\_Parent\_Process:
    xor eax, eax
    jmp Exit\_FD\_Parent\_Process
Found\_FD\_Parent\_Process:
    mov eax, 1
    jmp Exit\_FD\_Parent\_Process    
End\_FD\_Parent\_Process:

```

当调试会话创建，将产生一个调试对象，我们通过 ntdll 中的  
NtQueryObject 函数参看调试对象的个数是否不为零，来确定调试器的存在  
以 ObjectAllTypeInformation 使用 NtQueryObject 查询后会返回一个  
OBJECT\_ALL\_INFORMATION 的结构，其中 NumberOfObjectsTypes 成员为所有的  
对象类型在 ObjectTypeInformation 数组中的计数  
此对象如下  
typedef struct \_OBJECT\_ALL\_INFORMATION  
ULONG    NumberOfObjectsTypes;  
OBJECT\_TYPE\_INFORMATION ObjectTypeInformation\[1\];  
}  
typedef strcut \_OBJECT\_TYPE\_INFORMATION {  
\[00\]UNICODE\_STRING TypeName;  
\[08\]ULONG TotalNumberofHandles;  
\[0c\]ULONG TotalNumberofObjects;  
...  
}  
循环遍历 ObjectTypeInformation 对比类型的名字,如有类型名为 DebugObject 则  
检测 TotalNumberofHandles 与 TotalNumberofObjects 如果不为 0 则存在调试器。  
代码:

```
FD\_DebugObject\_NtQueryObject:
FD\_DebugObject\_NtQueryObject\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr \[esp+FD\_DebugObject\_NtQueryObject\_Arg\_Win32Api\]
        
    push ebx
    push ecx
    push edx
    push edi
    push esi
    
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    push edx    ; alloc the stack
    
    push esp    ; ReturnLength
    push 0
    push 0
    push ObjectAllTypeInformation
    push 0
    call dword ptr \[ebx\].xNtQueryObject
    pop ecx
    ;; make a tmp stack
    push ebp
    mov ebp, esp
    sub esp, ecx
    mov esi, esp
    ;; ObjectInformationLength
    push 0
    push ecx
    push esi
    push ObjectAllTypeInformation
    push 0
    call dword ptr \[ebx\].xNtQueryObject
    cld
    ;; NumberOfObjectsTypes
    lodsd
    xchg ecx, eax ; ecx = NumberOfObjectsTypes
    FD\_DebugObject\_NtQueryObject\_Loop:    
    ;; load string lengths
    lodsd
    movzx edx, ax
    ;; pointer to TypeName
    lodsd
    xchg esi, eax
    ;; sizeof(L"DebugObject")
    ;; avoids superstrings
    ;; like "DebugObjective"
    cmp edx, 16h
    jnz FD\_DebugObject\_NtQueryObject\_Tmp2
    xchg ecx, edx
    FD\_DebugObject\_NtQueryObject\_Tmp1:
    call FD\_DebugObject\_NtQueryObject\_UnicodeStr1
        dw 'D','e','b','u','g'
        dw 'O','b','j','e','c','t'
    FD\_DebugObject\_NtQueryObject\_UnicodeStr1:
    pop edi
    repe cmpsb
    xchg ecx, edx
    jnz FD\_DebugObject\_NtQueryObject\_Tmp2
    ;; TotalNumberOfObjects
    cmp dword ptr \[eax\], edx
    jnz Found\_FD\_DebugObject\_NtQueryObject
    ;; point to trailing  null
    FD\_DebugObject\_NtQueryObject\_Tmp2:
    add esi, edx
    ;; round down to dword
    and esi, -4
    ;; skip trailing null
    ;; and any alignment bytes
    lodsd
    loop FD\_DebugObject\_NtQueryObject\_Loop
    xor eax, eax
Exit\_FD\_DebugObject\_NtQueryObject:
    ;; clear the tmp stack
    mov esp, ebp
    pop ebp
    
    pop esi
    pop edi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing    
    retn 04h
Found\_FD\_DebugObject\_NtQueryObject:    
    mov eax, 1
    jmp Exit\_FD\_DebugObject\_NtQueryObject
End\_FD\_DebugObject\_NtQueryObject:

```

此类方法，利用 FindWindow 函数通过寻找是否存在一些常见调试软件的 Title。这里收集了一些  
如果有其他的 Title,请朋友们告知。。。  
代码:

```
FD\_Find\_Debugger\_Window:
FD\_Find\_Debugger\_Window\_Arg\_WinApi32   equ 08h
    push ebp
    mov ebp, esp
    
    push ebx
    
    mov ebx, dword ptr \[ebp+FD\_Find\_Debugger\_Window\_Arg\_WinApi32\]
    assume ebx : ptr WIN32APIBASE
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str1
        db "1212121",0
    FD\_Find\_Debugger\_Window\_Str1:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str2
        db "icu\_dbg",0
    FD\_Find\_Debugger\_Window\_Str2:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window    
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str3
        db "pe--diy",0
    FD\_Find\_Debugger\_Window\_Str3:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window     
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str5
        db "ollydbg",0
    FD\_Find\_Debugger\_Window\_Str5:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window 

    push NULL
    call FD\_Find\_Debugger\_Window\_Str6
        db "odbydyk",0
    FD\_Find\_Debugger\_Window\_Str6:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_Str7
        db "WinDbgFrameClass",0
    FD\_Find\_Debugger\_Window\_Str7:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_Str8
        db "TDeDeMainForm",0
    FD\_Find\_Debugger\_Window\_Str8:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_Str9
        db "TIdaWindow",0
    FD\_Find\_Debugger\_Window\_Str9:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_StrA
        db "TESTDBG",0
    FD\_Find\_Debugger\_Window\_StrA:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_StrB
        db "kk1",0
    FD\_Find\_Debugger\_Window\_StrB:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window
    
    push NULL
    call FD\_Find\_Debugger\_Window\_StrC
        db "Eew75",0
    FD\_Find\_Debugger\_Window\_StrC:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window  
    
    push NULL
    call FD\_Find\_Debugger\_Window\_StrD
        db "Shadow",0
    FD\_Find\_Debugger\_Window\_StrD:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window       
   
    push NULL
    call FD\_Find\_Debugger\_Window\_StrE
        db "PEiD v0.94",0
    FD\_Find\_Debugger\_Window\_StrE:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window    
   
    push NULL
    call FD\_Find\_Debugger\_Window\_StrF
        db "Registry Monitor - Sysinternals: www.sysinternals.com",0
    FD\_Find\_Debugger\_Window\_StrF:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window    
 
    push NULL
    call FD\_Find\_Debugger\_Window\_Str10
        db "File Monitor - Sysinternals: www.sysinternals.com",0
    FD\_Find\_Debugger\_Window\_Str10:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window    

    push NULL
    call FD\_Find\_Debugger\_Window\_Str11
        db "Import REConstructor v1.6 FINAL (C) 2001-2003 MackT/uCF",0
    FD\_Find\_Debugger\_Window\_Str11:
    call dword ptr \[ebx\].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window
    jmp NotFound\_Found\_FD\_Find\_Debugger\_Window
Exit\_FD\_Find\_Debugger\_Window:
    pop ebx
    assume ebx : nothing
    mov esp, ebp
    pop ebp
    retn 04h
NotFound\_Found\_FD\_Find\_Debugger\_Window:
    xor eax, eax
    jmp Exit\_FD\_Find\_Debugger\_Window
Found\_FD\_Find\_Debugger\_Window:
    mov eax, 1
    jmp Exit\_FD\_Find\_Debugger\_Window    
End\_FD\_Find\_Debugger\_Window:

```

这种方面的核心思想是比对在进程表中是否是出现了调试器进程名，如果出现则退出。  
不过这种方法很容易躲过，而且也可能造成误判。  
代码:

```
FD\_Find\_Debugger\_Process:
FD\_Find\_Debugger\_Process\_Arg\_Win32Api   equ 08h
FD\_Find\_Debugger\_Process\_StackSize      equ 10h + sizeof PROCESSENTRY32
FD\_Find\_Debugger\_Process\_hSnapshot      equ -04h
FD\_Find\_Debugger\_Process\_hParnet        equ -08h
FD\_Find\_Debugger\_Process\_pe32           equ -(10+sizeof PROCESSENTRY32)

    push ebp
    mov ebp, esp
    sub esp, FD\_Find\_Debugger\_Process\_StackSize
    
    push edi
    push esi
    push edx
    push ecx
    push ebx
    
    ;; clear the stack
      lea edi, \[ebp-FD\_Find\_Debugger\_Process\_StackSize\]
      mov ecx, FD\_Find\_Debugger\_Process\_StackSize
      xor eax, eax
      cld
      rep stosb
    
    mov ebx, dword ptr \[ebp+FD\_Find\_Debugger\_Process\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    lea esi, \[ebp+FD\_Find\_Debugger\_Process\_pe32\]
    assume esi : ptr PROCESSENTRY32
    push sizeof PROCESSENTRY32
    pop dword ptr \[esi\].dwSize
    
    push 0
    push TH32CS\_SNAPPROCESS
    call dword ptr \[ebx\].xCreateToolhelp32Snapshot
    mov dword ptr \[ebp+FD\_Find\_Debugger\_Process\_hSnapshot\], eax
    lea eax, \[ebp+FD\_Find\_Debugger\_Process\_pe32\]
    push eax
    push dword ptr \[ebp+FD\_Find\_Debugger\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xProcess32First
    test eax, eax
    jz NotFound\_FD\_Find\_Debugger\_Process
    FD\_Find\_Debugger\_Process\_Loop:
    lea eax, \[esi\].szExeFile
    mov edi, eax
    call FD\_Find\_Debugger\_Process\_Str1
        db "OLLYICE.EXE",0
    FD\_Find\_Debugger\_Process\_Str1:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process
    
    call FD\_Find\_Debugger\_Process\_Str2
        db "IDAG.EXE",0
    FD\_Find\_Debugger\_Process\_Str2:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process 
    
    call FD\_Find\_Debugger\_Process\_Str3
        db "OLLYDBG.EXE",0
    FD\_Find\_Debugger\_Process\_Str3:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process    
    
    call FD\_Find\_Debugger\_Process\_Str4
        db "PEID.EXE",0
    FD\_Find\_Debugger\_Process\_Str4:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process    
    
    call FD\_Find\_Debugger\_Process\_Str5
        db "SOFTICE.EXE",0
    FD\_Find\_Debugger\_Process\_Str5:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process

    call FD\_Find\_Debugger\_Process\_Str6
        db "LORDPE.EXE",0
    FD\_Find\_Debugger\_Process\_Str6:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process

    call FD\_Find\_Debugger\_Process\_Str7
        db "IMPORTREC.EXE",0
    FD\_Find\_Debugger\_Process\_Str7:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process
    
    call FD\_Find\_Debugger\_Process\_Str8
        db "W32DSM89.EXE",0
    FD\_Find\_Debugger\_Process\_Str8:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process    
    
    call FD\_Find\_Debugger\_Process\_Str9
        db "WINDBG.EXE",0
    FD\_Find\_Debugger\_Process\_Str9:
    push edi
    call dword ptr \[ebx\].xlstrcmpiA
    test eax, eax
    jz Found\_FD\_Find\_Debugger\_Process
    lea eax, \[ebp+FD\_Find\_Debugger\_Process\_pe32\]
    push eax
    push dword ptr \[ebp+FD\_Find\_Debugger\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xProcess32Next
    test eax, eax
    jnz FD\_Find\_Debugger\_Process\_Loop
    jmp NotFound\_FD\_Find\_Debugger\_Process
Exit\_FD\_Find\_Debugger\_Process:
    ;; close the Shotsnap handle
    push dword ptr \[ebp+FD\_Find\_Debugger\_Process\_hSnapshot\]
    call dword ptr \[ebx\].xCloseHandle
    
    pop edi
    pop esi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing
    assume esi : nothing
    mov esp, ebp
    pop ebp 
    retn 04h
NotFound\_FD\_Find\_Debugger\_Process:
    xor eax, eax
    jmp Exit\_FD\_Find\_Debugger\_Process
Found\_FD\_Find\_Debugger\_Process:
    mov eax, 1
    jmp Exit\_FD\_Find\_Debugger\_Process        
End\_FD\_Find\_Debugger\_Process:

```

这种方法通过打开一些调试软件加载到系统中驱动的句柄，从而判断是否有  
调试软件的存在。  
代码:

```
FD\_Find\_Device\_Driver:
FD\_Find\_Device\_Driver\_Arg\_Win32Api      equ 08h

    push ebp
    mov ebp, esp
    
    push ebx
    push ecx
    push edx
    push esi
    push edi
    
    mov ebx, dword ptr \[ebp+FD\_Find\_Device\_Driver\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    ;; check softice on unknow system
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str1
        db "\\\\.\\SIWVID",0
    FD\_Find\_Device\_Driver\_Str1:
    call dword ptr \[ebx\].xCreateFileA
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver
    
    ;; check softice 4.05 on win2k
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str2
        db "\\\\.\\NTICE",0
    FD\_Find\_Device\_Driver\_Str2:
    call dword ptr \[ebx\].xCreateFileA
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver
    
    ;; check softice on win9x
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str3
        db "\\\\.\\SICE",0
    FD\_Find\_Device\_Driver\_Str3:
    call dword ptr \[ebx\].xCreateFileA
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver    
    
    ;; check softice on win9x
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str4
        db "\\\\.\\SIWDEBUG",0
    FD\_Find\_Device\_Driver\_Str4:
    call dword ptr \[ebx\].xCreateFileA
    push eax
    call dword ptr \[ebx\].xGetLastError
    test al, 032h
    pop eax
    jz Found\_FD\_Find\_Device\_Driver
    
    ;; check regmon on win9x
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str5
        db "\\\\.\\REGVXD",0
    FD\_Find\_Device\_Driver\_Str5:
    call dword ptr \[ebx\].xCreateFileA    
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver
    
    ;; check RegMON
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str6
        db "\\\\.\\FILEM",0
    FD\_Find\_Device\_Driver\_Str6:
    call dword ptr \[ebx\].xCreateFileA      
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver
    
    ;; check TRW
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str7
        db "\\\\.\\TRW",0
    FD\_Find\_Device\_Driver\_Str7:
    call dword ptr \[ebx\].xCreateFileA      
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver       
    
    ;; check softice extender
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ + FILE\_SHARE\_WRITE
    push GENERIC\_READ + GENERIC\_WRITE
    call FD\_Find\_Device\_Driver\_Str8
        db "\\\\.\\ICEEXT",0
    FD\_Find\_Device\_Driver\_Str8:
    call dword ptr \[ebx\].xCreateFileA      
    cmp eax, INVALID\_HANDLE\_VALUE
    jnz Found\_FD\_Find\_Device\_Driver 
    jmp NotFound\_FD\_Find\_Device\_Driver
    
Exit\_FD\_Find\_Device\_Driver:    
    pop edi
    pop esi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing    
    mov esp, ebp
    pop ebp
    retn 04h
NotFound\_FD\_Find\_Device\_Driver:
    xor eax, eax
    jmp Exit\_FD\_Find\_Device\_Driver
Found\_FD\_Find\_Device\_Driver:
    push eax
    assume ebx : ptr WIN32APIBASE
    call dword ptr \[ebx\].xCloseHandle
    assume ebx : nothing
    mov eax, 1
    jmp Exit\_FD\_Find\_Device\_Driver
End\_FD\_Find\_Device\_Driver:

```

利用 int3 中断引起异常。如果，正常运行则触发异常。  
代码:

```
FD\_Exception\_Int3:
    call Get\_FD\_Exception\_Int3\_Eip
    Get\_FD\_Exception\_Int3\_Eip:
    pop eax
    add eax, offset FD\_Exception\_Int3\_Exception - offset Get\_FD\_Exception\_Int3\_Eip
    ;; setup exception
    assume fs : nothing
    push eax
    push dword ptr fs : \[0\]
    mov dword ptr fs : \[0\], esp
    ;; reset eax
    xor eax, eax
    int 03h
    ;; unsetup exception
    pop dword ptr fs : \[0\]
    add esp, 04h
    
    ;; check the flag
    test eax, eax
    jz Found\_FD\_Exception\_Int3
    jmp NotFound\_FD\_Exception\_Int3
FD\_Exception\_Int3\_Exception:
    mov eax, dword ptr \[esp+0ch\]
    ;; eax = ContextRecord
    assume eax : ptr CONTEXT
    mov dword ptr \[eax\].regEax, 0FFFFFFFFh
    inc dword ptr \[eax\].regEip
    xor eax, eax
    assume eax : nothing
    retn
Exit\_FD\_Exception\_Int3:
    retn 0h
NotFound\_FD\_Exception\_Int3:
    xor eax, eax
    jmp Exit\_FD\_Exception\_Int3
Found\_FD\_Exception\_Int3:
    mov eax, 1
    jmp Exit\_FD\_Exception\_Int3        
End\_FD\_Exception\_Int3:

```

通过对 EFLAGS 中的 TF 的检测来确定是否存在调试器。当 TF=1 的时会触发一个  
单步执行异常，可以通过安装异常处理针来捕获。  
代码:

```
FD\_Exception\_Popf:
    push ebx

    call Get\_FD\_Exception\_Popf\_Eip
    Get\_FD\_Exception\_Popf\_Eip:
    pop eax
    add eax, offset FD\_Exception\_Popf\_Exception - offset Get\_FD\_Exception\_Popf\_Eip
    assume fs : nothing    
    push eax
    push dword ptr fs:\[0\]
    mov dword ptr fs:\[0\], esp
    ;; reset eax
    xor eax, eax
    pushf
    mov dword ptr \[esp\], 0100h
    popf
    nop
FD\_Exception\_Popf\_NextEip:
    pop dword ptr fs:\[0\]
    add esp, 04h
    
    ;; check the flag
    test eax, eax
    jz Found\_FD\_Exception\_Popf
    jmp NotFound\_FD\_Exception\_Popf
    
FD\_Exception\_Popf\_Exception:
    mov eax, dword ptr \[esp+0ch\]
    assume eax : ptr CONTEXT
    mov dword ptr \[eax\].regEax, 0FFFFFFFFh
    call Get\_FD\_Exception\_Popf\_Exception\_Eip2
    Get\_FD\_Exception\_Popf\_Exception\_Eip2:
    pop ebx
    sub ebx, offset Get\_FD\_Exception\_Popf\_Exception\_Eip2 - offset FD\_Exception\_Popf\_NextEip
    mov dword ptr \[eax\].regEip, ebx
    xor eax, eax
    retn
    assume eax : nothing
Exit\_FD\_Exception\_Popf:
    pop ebx
    retn 0h
NotFound\_FD\_Exception\_Popf:
    xor eax, eax
    jmp Exit\_FD\_Exception\_Popf
Found\_FD\_Exception\_Popf:
    mov eax, 1
    jmp Exit\_FD\_Exception\_Popf
End\_FD\_Exception\_Popf:

```

触发 OutputDebugString 的错误。如果调试器存在则 GetLastError 为 0。  
代码:

```
FD\_OutputDebugString:
FD\_OutputDebugString\_Arg\_Win32Api       equ 04h
    
    mov eax, dword ptr \[esp+FD\_OutputDebugString\_Arg\_Win32Api\]
    push ebx
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    call FD\_OutputDebugString\_Str
        db 0
    FD\_OutputDebugString\_Str:
    call dword ptr \[ebx\].xOutputDebugStringA
    call dword ptr \[ebx\].xGetLastError 
    test eax, eax   
    jnz NotFound\_FD\_OutputDebugString
    inc eax
    pop ebx
    assume ebx : nothing
    retn 04h
NotFound\_FD\_OutputDebugString:
    pop ebx
    xor eax, eax
    retn 04h
    
End\_FD\_OutputDebugString:

```

Ollydbg 允许设置一个内存访问/写入断点。它通过页面保护来实现。  
页面保护提供了当应用程序的某块内存被访问时获得通知。  
页面保护通过 PAGE\_GUARD 页面保护修改符来设置。如果访问的内存  
是受保护的将会产生一个 STATUS\_GUARD\_PAGE\_VIOLATION(0x80000001)异常  
如果进程被 Ollydbg 调试并且受保护的页面被访问，将不会抛出异常。访问  
将会被当作内存断点来处理。  
代码:

```
FS\_OD\_Exception\_GuardPages:
FS\_OD\_Exception\_GuardPages\_Arg\_Win32Api     equ 04h

    mov eax, dword ptr \[esp+FS\_OD\_Exception\_GuardPages\_Arg\_Win32Api\]
    
    push edi    ; pAllocatedMem
    push ebx
    
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
     
    push PAGE\_READWRITE
    push MEM\_COMMIT
    push 01000h
    push NULL
    call dword ptr \[ebx\].xVirtualAlloc
    test eax, eax
    jz NotFound\_FS\_OD\_Exception\_GuardPages
    mov edi, eax
    ;; store a RENT on the allocated memory
    mov byte ptr \[edi\], 0c3h
    push edi        ; alloc stack
    
    push esp
    push PAGE\_EXECUTE\_READ + PAGE\_GUARD
    push 01000h
    push edi
    call dword ptr \[ebx\].xVirtualProtect
    ;; set eax to 0
    xor eax, eax
    ;; trigger a STATUS\_GUARD\_PAGE\_VIOLATION exception
    ;; setup expcetion
    call Get\_FS\_OD\_Exception\_GuardPages\_Eip
    Get\_FS\_OD\_Exception\_GuardPages\_Eip:
    pop eax
    add eax, offset FS\_OD\_Exception\_GuardPages\_Exception - offset Get\_FS\_OD\_Exception\_GuardPages\_Eip
    assume fs : nothing
    push eax
    push dword ptr fs:\[0\]
    mov dword ptr fs:\[0\], esp
    call edi
FS\_OD\_Exception\_GuardPages\_Continue:
    pop edi
    ;; unsetup exception
    pop dword ptr fs:\[0\]
    add esp, 04h
    test eax, eax
    jz Found\_FS\_OD\_Exception\_GuardPages
    jmp NotFound\_FS\_OD\_Exception\_GuardPages
FS\_OD\_Exception\_GuardPages\_Exception:
    mov eax, dword ptr \[esp+0ch\]
    assume eax : ptr CONTEXT
    mov dword ptr \[eax\].regEax, 0FFFFFFFFh
    call Get\_FS\_OD\_Exception\_GuardPages\_Exception\_Eip
    Get\_FS\_OD\_Exception\_GuardPages\_Exception\_Eip:
    pop ebx
    sub ebx, offset Get\_FS\_OD\_Exception\_GuardPages\_Exception\_Eip - offset FS\_OD\_Exception\_GuardPages\_Continue  
    mov dword ptr \[eax\].regEip, ebx
    assume eax : nothing
    xor eax, eax
    retn
    
Exit\_FS\_OD\_Exception\_GuardPages:
    pop ebx
    pop edi
    assume ebx : nothing
    retn 04h
    
NotFound\_FS\_OD\_Exception\_GuardPages:
    assume ebx : ptr WIN32APIBASE
    push MEM\_DECOMMIT
    push 01000h
    push edi
    call dword ptr \[ebx\].xVirtualFree
    assume ebx : nothing
    xor eax, eax
    jmp Exit\_FS\_OD\_Exception\_GuardPages
Found\_FS\_OD\_Exception\_GuardPages:
    assume ebx : ptr WIN32APIBASE
    push MEM\_DECOMMIT
    push 01000h
    push edi
    call dword ptr \[ebx\].xVirtualFree
    assume ebx : nothing
    mov eax, 1
    jmp Exit\_FS\_OD\_Exception\_GuardPages
End\_FS\_OD\_Exception\_GuardPages:

```

Softice 调试器会对 UnhandleExceptionFilter 进行修改,将第一个字节修改为 CCh(int 3)  
代码:

```
FS\_SI\_UnhandledExceptionFilter:
FS\_SI\_UnhandledExceptionFilter\_Arg\_Win32Api     equ 04h
    mov eax, dword ptr \[esp+FS\_SI\_UnhandledExceptionFilter\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    mov eax, dword ptr \[eax\].xUnhandledExceptionFilter
    mov al, byte ptr \[eax\]
    sub al, 0cch
    jz Found\_FS\_SI\_UnhandledExceptionFilter
    xor eax, eax
Exit\_FS\_SI\_UnhandledExceptionFilter:   
    assume eax : nothing
    retn 04h
Found\_FS\_SI\_UnhandledExceptionFilter:
    mov eax, 1
    jmp Exit\_FS\_SI\_UnhandledExceptionFilter    
End\_FS\_SI\_UnhandledExceptionFilter:

```

Ollydbg 的插件对几个函数的首部进行修改。通过对齐的检测来判断系统中是否存在调试器  
代码:

```
FS\_ODP\_Process32NextW:
FS\_ODP\_Process32NextW\_Arg\_Win32Api          equ 04h
    mov eax, dword ptr \[esp+FS\_ODP\_Process32NextW\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    mov eax, dword ptr \[eax\].xProcess32NextW
    mov ax, word ptr \[eax\]
    sub ax, 0FF88h
    jnz Found\_FS\_ODP\_Process32NextW
    xor eax, eax
Exit\_FS\_ODP\_Process32NextW:
    assume eax : nothing 
    retn 04h
Found\_FS\_ODP\_Process32NextW:
    mov eax, 1
    jmp Exit\_FS\_ODP\_Process32NextW
End\_FS\_ODP\_Process32NextW:


FS\_ODP\_OutputDebugStringA:
FS\_ODP\_OutputDebugStringA\_Arg\_Win32Api      equ 04h
    mov eax, dword ptr \[ebp+FS\_ODP\_OutputDebugStringA\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    mov eax, dword ptr \[eax\].xOutputDebugStringA
    mov ax, word ptr \[eax\]
    sub ax, 03468h
    jnz Found\_FS\_ODP\_OutputDebugStringA
    xor eax, eax
Exit\_FS\_ODP\_OutputDebugStringA:
    assume eax : nothing
    retn 04h
Found\_FS\_ODP\_OutputDebugStringA:
    mov eax, 1
    jmp Exit\_FS\_ODP\_OutputDebugStringA            
End\_FS\_ODP\_OutputDebugStringA:


FS\_ODP\_OpenProcess:
FS\_ODP\_OpenProcess\_Arg\_Win32Api     equ 04h
    mov eax, dword ptr \[ebp+FS\_ODP\_OpenProcess\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    mov eax, dword ptr \[eax\].xOpenProcess
    mov al, byte ptr \[eax+06h\]
    ;; Hide Debugger Plugin of OD is present
    sub al, 0eah
    jnz Found\_FS\_ODP\_OpenProcess
    xor eax, eax
Exit\_FS\_ODP\_OpenProcess:
    assume eax : nothing
    retn 04h  
Found\_FS\_ODP\_OpenProcess:
    mov eax, 1
    jmp Exit\_FS\_ODP\_OpenProcess      
End\_FS\_ODP\_OpenProcess:

FB\_HWBP\_Exception:
    call Get\_FB\_HWBP\_Exception\_Eip
    Get\_FB\_HWBP\_Exception\_Eip:
    pop eax
    add eax, offset FB\_HWBP\_Exception\_Exception - offset FB\_HWBP\_Exception
    assume fs : nothing
    push eax
    push fs:\[0\]
    mov dword ptr fs:\[0\], esp
    ;; reset eax
    xor eax, eax
    int 01h
    pop dword ptr fs:\[0\]
    add esp, 04h
    test eax, eax
    jnz Found\_FB\_HWBP\_Exception
    jmp NotFound\_FB\_HWBP\_Exception
FB\_HWBP\_Exception\_Exception:
    mov eax, dword ptr \[esp+0ch\]
    assume eax : ptr CONTEXT
    ;; check if debug Registers Context.Dr0-Dr3 is not zero
    cmp dword ptr \[eax\].iDr0, 0
    jnz FB\_HWBP\_Exception\_HardwareBp\_Found
    cmp dword ptr \[eax\].iDr1, 0
    jnz FB\_HWBP\_Exception\_HardwareBp\_Found  
    cmp dword ptr \[eax\].iDr2, 0
    jnz FB\_HWBP\_Exception\_HardwareBp\_Found
    cmp dword ptr \[eax\].iDr3, 0
    jnz FB\_HWBP\_Exception\_HardwareBp\_Found
    jmp FB\_HWBP\_Exception\_Exception\_Ret     
FB\_HWBP\_Exception\_HardwareBp\_Found:
    ;; set Context.Eax to signal breakpoint found
    mov dword ptr \[eax\].regEax, 0FFFFFFFFh
FB\_HWBP\_Exception\_Exception\_Ret:
    ;; set Context.Eip upon return
    add dword ptr \[eax\].regEip, 02h
    xor eax, eax
    retn
Exit\_FB\_HWBP\_Exception:
    retn 0h
NotFound\_FB\_HWBP\_Exception:
    xor eax, eax
    jmp Exit\_FB\_HWBP\_Exception
Found\_FB\_HWBP\_Exception:
    mov eax, 1
    jmp Exit\_FB\_HWBP\_Exception                
End\_FB\_HWBP\_Exception:

```

通过对代码节做 CRC 值的验证来判断是否存在内存补丁  
代码:

```
FB\_SWBP\_Memory\_CRC:
FB\_SWBP\_Memory\_CRC\_Arg\_Win32Api         equ 08h
FB\_SWBP\_Memory\_CRC\_Arg\_FuncAddr         equ 0ch
FB\_SWBP\_Memory\_CRC\_StackSize            equ 10h+MAX\_PATH
FB\_SWBP\_Memory\_CRC\_fileSize             equ -04h
FB\_SWBP\_Memory\_CRC\_NumberOfBytesRW      equ -08h
FB\_SWBP\_Memory\_CRC\_pBuffer              equ -0ch
FB\_SWBP\_Memory\_CRC\_szFileName           equ -(10h+MAX\_PATH)
    push ebp
    mov ebp, esp
    sub esp, FB\_SWBP\_Memory\_CRC\_StackSize
    
    push esi
    push edi
    push ecx
    push edx
    push ebx
    
    
    ;; clear the stack
    lea edi, \[ebp+FB\_SWBP\_Memory\_CRC\_szFileName\]
    mov ecx, FB\_SWBP\_Memory\_CRC\_StackSize
    xor eax, eax
    cld
    rep stosb
    
    
    mov ebx, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    push MAX\_PATH
    lea eax, \[ebp+FB\_SWBP\_Memory\_CRC\_szFileName\]
    push eax
    push NULL
    call dword ptr \[ebx\].xGetModuleFileNameA
    ;; open file
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ
    push GENERIC\_READ
    lea eax, \[ebp+FB\_SWBP\_Memory\_CRC\_szFileName\]
    push eax
    call dword ptr \[ebx\].xCreateFileA
    cmp eax, INVALID\_HANDLE\_VALUE
    jz Error\_FB\_SWBP\_Memory\_CRC
    mov edi, eax
    push NULL
    push eax
    call dword ptr \[ebx\].xGetFileSize
;    cmp eax, INVALID\_FILE\_SIZE
;    jz Error\_FB\_SWBP\_Memory\_CRC    
    mov dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_fileSize\], eax
    push PAGE\_READWRITE
    push MEM\_COMMIT
    push eax
    push NULL
    call dword ptr \[ebx\].xVirtualAlloc
    mov dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\], eax
    push NULL
    lea eax, \[ebp+FB\_SWBP\_Memory\_CRC\_NumberOfBytesRW\]
    push eax
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_fileSize\]
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    push edi
    call dword ptr \[ebx\].xReadFile
    push edi
    call dword ptr \[ebx\].xCloseHandle
        
    mov esi, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    add esi, dword ptr \[esi+3ch\]
    assume esi : ptr IMAGE\_NT\_HEADERS
    mov edx, dword ptr \[esi\].OptionalHeader.ImageBase
    mov eax, dword ptr \[esi\].OptionalHeader.AddressOfEntryPoint
    add eax, edx
    mov cx, word ptr \[esi\].FileHeader.NumberOfSections
    movzx ecx, cx
    mov edi, esi
    add edi, sizeof IMAGE\_NT\_HEADERS
    assume edi : ptr IMAGE\_SECTION\_HEADER
    ;; find the entry section
    ;; eax = AddressOfEntryPoint
    ;; ebx = Each Section Virtual Address
    ;; ecx = NumberOfSections    
    ;; edx = ImageBase
    ;; edi = Section Table Point
    FB\_SWBP\_Memory\_CRC\_Loop:
    mov ebx, dword ptr \[edi\].VirtualAddress
    add ebx, edx
    cmp ebx, eax
    jz FB\_SWBP\_Memory\_CRC\_Loop\_End
    add edi, sizeof IMAGE\_SECTION\_HEADER
    dec ecx
    jnz FB\_SWBP\_Memory\_CRC\_Loop
    jmp Error\_FB\_SWBP\_Memory\_CRC
    FB\_SWBP\_Memory\_CRC\_Loop\_End:
    mov edi, dword ptr \[edi\].Misc.VirtualSize
    mov esi, eax
    ;; alloc the memory
    push MEM\_DECOMMIT
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_fileSize\]
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    mov ebx, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    call dword ptr \[ebx\].xVirtualFree
    mov edx, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_Arg\_FuncAddr\]
    assume edx : ptr FUNCADDRTBL
    push edi
    push esi
    call dword ptr \[edx\].xCRC32
    call Get\_FB\_SWBP\_Memory\_CRC\_Eip
    Get\_FB\_SWBP\_Memory\_CRC\_Eip:
    pop ecx
    add ecx, offset FB\_SWBP\_Memory\_CRC\_Orig\_CRC - offset Get\_FB\_SWBP\_Memory\_CRC\_Eip
    cmp eax, ecx
    jz NotFound\_FB\_SWBP\_Memory\_CRC
    jmp Found\_FB\_SWBP\_Memory\_CRC
Exit\_FB\_SWBP\_Memory\_CRC:
    pop ebx
    pop edx
    pop ecx
    pop edi
    pop esi
    assume ebx : nothing
    assume edx : nothing
    assume edi : nothing
    assume esi : nothing
    mov esp, ebp
    pop ebp
    retn 08h
Error\_FB\_SWBP\_Memory\_CRC:
    mov eax, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    test eax, eax
    jz Error\_FB\_SWBP\_Memory\_CRC\_Continue
    push MEM\_DECOMMIT
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_fileSize\]
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    mov ebx, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE    
    call dword ptr \[ebx\].xVirtualFree
    assume ebx : nothing
    Error\_FB\_SWBP\_Memory\_CRC\_Continue:
    xor eax, eax
    jmp Exit\_FB\_SWBP\_Memory\_CRC
Found\_FB\_SWBP\_Memory\_CRC:
    push MEM\_DECOMMIT
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_fileSize\]
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    mov ebx, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE    
    call dword ptr \[ebx\].xVirtualFree
    assume ebx : nothing
    mov eax, 1
    jmp Exit\_FB\_SWBP\_Memory\_CRC
NotFound\_FB\_SWBP\_Memory\_CRC:
    push MEM\_DECOMMIT
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_fileSize\]
    push dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_pBuffer\]
    mov ebx, dword ptr \[ebp+FB\_SWBP\_Memory\_CRC\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE    
    call dword ptr \[ebx\].xVirtualFree
    assume ebx : nothing
    xor eax, eax
    jmp Exit\_FB\_SWBP\_Memory\_CRC        
FB\_SWBP\_Memory\_CRC\_Orig\_CRC:
    dd 0   
End\_FB\_SWBP\_Memory\_CRC:

```

通过检测标志位来检测是否存在单步运行标志  
代码:

```
FT\_PushSS\_PopSS:
    ;; release version
    push ebp
    mov ebp, esp
    push ss
    pop ss
    pushf
    pop eax
    and eax, 00000100h
    jnz Found\_FT\_PushSS\_PopSS
Exit\_FT\_PushSS\_PopSS:
    xor eax, eax
    mov esp, ebp
    pop ebp
    retn 0h    
Found\_FT\_PushSS\_PopSS:
    mov eax, 1
    jmp Exit\_FT\_PushSS\_PopSS   
End\_FT\_PushSS\_PopSS:

```

以下三个函数都是通过调用取单步运行时间差的原理来检测是否进行单步运行的  
分别使用了 rdtsc,GetTickCount,timeGetTime 三个函数来确定  
代码:

```
FT\_RDTSC:
    push ecx
      rdtsc
    xchg ecx, eax
    rdtsc
    sub eax, ecx
    cmp eax, 500h
    jnbe Found\_FT\_RDTSC
    xor eax, eax
Exit\_FT\_RDTSC:
    pop ecx
    retn 0h
Found\_FT\_RDTSC:
    mov eax, 1
    jmp Exit\_FT\_RDTSC
End\_FT\_RDTSC:

FT\_GetTickCount:
FT\_GetTickCount\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr \[esp+FT\_GetTickCount\_Arg\_Win32Api\]
    push ebx
    push ecx
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    call dword ptr \[ebx\].xGetTickCount
    xchg ecx, eax
    call dword ptr \[ebx\].xGetTickCount
    sub eax, ebx
    cmp eax, 1
    jnb Found\_FT\_GetTickCount
    xor eax, eax
Exit\_FT\_GetTickCount:
    pop ecx
    pop ebx
    assume ebx : nothing
    retn 04h    
Found\_FT\_GetTickCount:
    mov eax, 1
    jmp Exit\_FT\_GetTickCount    
End\_FT\_GetTickCount:

FT\_timeGetTime:
FT\_timeGetTime\_Arg\_Win32Api            equ 04h
    mov eax, dword ptr \[esp+FT\_timeGetTime\_Arg\_Win32Api\]
    push ebx
    push ecx
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    call dword ptr \[ebx\].xtimeGetTime
    xchg ecx, eax
    call dword ptr \[ebx\].xtimeGetTime
    sub eax, ebx
    cmp eax, 10h
    jnb Found\_FT\_timeGetTime
    xor eax, eax
Exit\_FT\_timeGetTime:
    pop ecx
    pop ebx
    assume ebx : nothing
    retn 04h    
Found\_FT\_timeGetTime:
    mov eax, 1
    jmp Exit\_FT\_timeGetTime
End\_FT\_timeGetTime:

```

手动检测位于 7FFE0000h 地址的 SharedUserData 数据结构的 TickCoutLow 与 TickCountMultipiler 字段  
来获取单步运行时间差。  
代码:

```
FT\_SharedUserData\_TickCount:
    push ecx
    push edx    
    mov edx, 07FFE0000h
    mov eax, dword ptr \[edx\]
    imul dword ptr \[edx+04h\]
    shrd eax, edx, 018h
    xchg eax, ecx
    mov edx, 07FFE0000h
    mov eax, dword ptr \[edx\]
    imul dword ptr \[edx+04h\]
    shrd eax, edx, 018h
    sub eax, ecx
       cmp eax, 500h
       jnbe Found\_FT\_SharedUserData\_TickCount
       xor eax, eax
Exit\_FT\_SharedUserData\_TickCount:
    pop edx
    pop ecx
    retn 0h
Found\_FT\_SharedUserData\_TickCount:
    mov eax, 1
    jmp Exit\_FT\_SharedUserData\_TickCount    
End\_FT\_SharedUserData\_TickCount:

```

中断 1 不能从 r3 下调用。如果从 r3 下调用,将产生一个 EXECEPTION\_ACCESS\_VIOLATION(0x0C0000005)的异常。  
如果 softice 存在，它 HOOK 了中断 1 并且使它的 DPL(descriptor privilege level)到环 3 下  
使得 softice 能单步处理用户态的程序。如果程序使用中断 1,softice 不检查它是否在单步运行或者是一软中断。  
softice 总是调用原始的中断 1 处理函数。并且使用 EXCEPTION\_SINGLE\_STEP(0x80000004)异常替换原本的 EXECEPTION\_ACCESS\_VIOLATION  
(0xC0000005)异常。  
代码:

```
FT\_INT1\_IceBreakpoint:
    push ebp
    mov ebp, esp
    call Get\_FT\_INT1\_IceBreakpoint\_Eip
    Get\_FT\_INT1\_IceBreakpoint\_Eip:
    pop eax
    add eax, offset FT\_INT1\_IceBreakpoint\_Exception - offset Get\_FT\_INT1\_IceBreakpoint\_Eip
    push eax
    assume fs : nothing
    push dword ptr fs:\[0\]
    mov dword ptr fs:\[0\], esp
    ;; reset eax
    xor eax, eax
    db 0CDh, 01h    ; int 1
    pop dword ptr fs:\[0\]
    add esp, 04h
    test eax, eax
    jz Found\_FT\_INT1\_IceBreakpoint
    jmp NotFound\_FT\_INT1\_IceBreakpoint
FT\_INT1\_IceBreakpoint\_Exception:
    ;mov eax, dword ptr \[esp+04h\]
    ;cmp dword ptr \[eax\], 80000004h
    mov eax, dword ptr \[esp+0ch\]
    assume eax : ptr CONTEXT 
    mov dword ptr \[eax\].regEax, 1
    inc dword ptr \[eax\].regEip
    xor eax, eax
    retn
Exit\_FT\_INT1\_IceBreakpoint:
    mov esp, ebp
    pop ebp
    retn 0h
NotFound\_FT\_INT1\_IceBreakpoint:
    xor eax, eax
    jmp Exit\_FT\_INT1\_IceBreakpoint
Found\_FT\_INT1\_IceBreakpoint:
    mov eax, 1
    jmp Exit\_FT\_INT1\_IceBreakpoint
End\_FT\_INT1\_IceBreakpoint:

```

以下为检测 Virtual PC 与 VMWARE 的两个例子,这类检测技术大多都是通过与真实环境中不同的  
异常，指令实现的。严格的描述可以在其规定的文档中找到。  
代码:

```
FV\_VPC\_Exception: 

    push ebx
    push ecx
   
    call Get\_FV\_VPC\_Exception\_Eip
    Get\_FV\_VPC\_Exception\_Eip:
    pop ecx
    add ecx, offset FV\_VPC\_Exception\_Exception - offset Get\_FV\_VPC\_Exception\_Eip
    assume fs : nothing        
    push ecx
    push dword ptr fs:\[0\]
    mov dword ptr fs:\[0\], esp
    
    ;; Flag
    xor ebx, ebx
    ;; VPC function number
    mov eax, 1
    
    ;; call VPC
    db 0Fh,3Fh,07h,0Bh
    
    pop dword ptr fs:\[0\]
    add esp, 04h    
    test ebx, ebx
    setz al
    
    pop ecx
    pop ebx    
    retn 0h
    
FV\_VPC\_Exception\_Exception:    
    mov ecx, dword ptr \[esp+0ch\]
    assume ecx : ptr CONTEXT
    ;; ebx = -1 : not running
    ;; ebx = 0 : running
    mov dword ptr \[ecx\].regEbx, -1
    ;; skip past the call to VPC
    add dword ptr \[ecx\].regEip, 04h
    xor eax, eax
    retn
End\_FV\_VPC\_Exception:

FV\_VMWare\_VMX:
    
    call IsInsideVMWare
    retn 0h
IsInsideVMWare:

    push ebx
    push ecx
    push edx

    assume fs : nothing
    call Get\_IsInsideVMWare\_Eip
    Get\_IsInsideVMWare\_Eip:
    pop eax
    add eax, offset IsInsideVMWare\_Exception - offset Get\_IsInsideVMWare\_Eip
    push eax
    push dword ptr fs:\[0\]
    mov dword ptr fs:\[0\], esp
    
    mov    eax, 'VMXh'
    mov    ebx, 0         ; any value but MAGIC VALUE
    mov    ecx, 10        ; get VMWare version
    mov    edx, 'VX'     ; port number
    in     eax, dx         ; read port
                           ; on return EAX returns the VERSION
    cmp    ebx, 'VMXh'     ; is it a reply from VMWare?
    setz   al             ; set return value
    movzx eax, al

    pop dword ptr fs:\[0\]
    add esp, 04h
    
    pop edx
    pop ecx
    pop ebx
    
    retn 0h
IsInsideVMWare\_Exception:
    mov eax, dword ptr \[esp+0ch\]
    assume eax : ptr CONTEXT
    inc dword ptr \[eax\].regEip
    retn       
End\_IsInsideVMWare:
    
End\_FV\_VMWare\_VMX:

```

通过计算文件的长度与原先的对比，来确定是否进行过修改。来发现补丁。  
代码:

```
FP\_Check\_FileSize:
FP\_Check\_FileSize\_Arg\_Win32Api      equ 08h
FP\_Check\_FileSize\_StackSize         equ MAX\_PATH
FP\_Check\_FileSize\_szPath            equ -MAX\_PATH
    push ebp
    mov ebp, esp
    sub esp, FP\_Check\_FileSize\_StackSize
    
    push ebx
    push ecx
    push edx
    mov ebx, dword ptr \[ebp+FP\_Check\_FileSize\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    push MAX\_PATH
    lea eax, \[ebp+FP\_Check\_FileSize\_szPath\]
    push eax
    push NULL
    call dword ptr \[ebx\].xGetModuleFileNameA
    test eax, eax
    jz NotFound\_FP\_Check\_FileSize
    
    ;; create file
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_ALWAYS
    push NULL
    push FILE\_SHARE\_READ
    push GENERIC\_READ
    lea eax, \[ebp+FP\_Check\_FileSize\_szPath\]
    push eax
    call dword ptr \[ebx\].xCreateFileA
    cmp eax, INVALID\_HANDLE\_VALUE
    jz NotFound\_FP\_Check\_FileSize
    mov ecx, eax
    
    push NULL
    push eax
    call dword ptr \[ebx\].xGetFileSize
    mov edx, eax
    
    push ecx
    call dword ptr \[ebx\].xCloseHandle
    
    call Get\_FP\_Check\_FileSize\_Eip
    Get\_FP\_Check\_FileSize\_Eip:
    pop eax
    add eax, offset FP\_Check\_FileSize\_Orig\_Size - offset Get\_FP\_Check\_FileSize\_Eip
    cmp edx, eax
    jnz Found\_FP\_Check\_FileSize
    jmp NotFound\_FP\_Check\_FileSize

Exit\_FP\_Check\_FileSize:
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing
    
    mov esp, ebp
    pop ebp
    retn 04h
NotFound\_FP\_Check\_FileSize:
    xor eax, eax
    jmp Exit\_FP\_Check\_FileSize
Found\_FP\_Check\_FileSize:
    mov eax, 1
    jmp Exit\_FP\_Check\_FileSize
FP\_Check\_FileSize\_Orig\_Size:
    dd 0   
End\_FP\_Check\_FileSize:

```

通过计算文件的 CRC 值来对比是否被 Cracker 进行过修改打过补丁。  
代码:

```
FP\_Check\_FileHashValue\_CRC:
FP\_Check\_FileHashValue\_CRC\_Arg\_Win32Api         equ 08h
FP\_Check\_FileHashValue\_CRC\_Arg\_FuncAddr            equ 0ch
FP\_Check\_FileHashValue\_CRC\_StackSize            equ 20h+MAX\_PATH
FP\_Check\_FileHashValue\_CRC\_fileSize             equ -04h
FP\_Check\_FileHashValue\_CRC\_NumberOfBytesRW      equ -08h
FP\_Check\_FileHashValue\_CRC\_CRCVALUE\_current     equ -0ch
FP\_Check\_FileHashValue\_CRC\_pBuffer              equ -10h
FP\_Check\_FileHashValue\_CRC\_hFile                equ -14h
FP\_Check\_FileHashValue\_CRC\_szFileName           equ -(20h+MAX\_PATH)
    
    push ebp
    mov ebp, esp
    sub esp, FP\_Check\_FileHashValue\_CRC\_StackSize
    
    push ebx
    push ecx
    push edx
    push esi
    push edi
    
    ;; clear the stack
    lea edi, \[ebp-FP\_Check\_FileHashValue\_CRC\_StackSize\]
    mov ecx, FP\_Check\_FileHashValue\_CRC\_StackSize
    xor eax, eax
    cld
    rep stosb
    
    mov ebx, dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    push MAX\_PATH
    lea eax, \[ebp+FP\_Check\_FileHashValue\_CRC\_szFileName\]
    push eax
    push NULL
    call dword ptr \[ebx\].xGetModuleFileNameA
    
    push NULL
    push FILE\_ATTRIBUTE\_NORMAL
    push OPEN\_EXISTING
    push NULL
    push FILE\_SHARE\_READ
    push GENERIC\_READ
    lea eax, \[ebp+FP\_Check\_FileHashValue\_CRC\_szFileName\]
    push eax
    call dword ptr \[ebx\].xCreateFileA
    cmp eax, INVALID\_HANDLE\_VALUE
    jz NotFound\_FP\_Check\_FileHashValue\_CRC
    mov dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_hFile\], eax
    
    push NULL
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_hFile\]
    call dword ptr \[ebx\].xGetFileSize
    ;cmp eax, INVALID\_FILE\_SIZE
    ;jz NotFound\_FP\_Check\_FileHashValue\_CRC
    mov dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_fileSize\], eax
    
    push PAGE\_READWRITE
    push MEM\_COMMIT
    push eax
    push NULL
    call dword ptr \[ebx\].xVirtualAlloc
    mov dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_pBuffer\], eax
    
    ;; read file
    push NULL
    lea eax, \[ebp+FP\_Check\_FileHashValue\_CRC\_NumberOfBytesRW\]
    push eax
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_fileSize\]
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_pBuffer\]
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_hFile\]
    call dword ptr \[ebx\].xReadFile
    
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_hFile\]
    call dword ptr \[ebx\].xCloseHandle
    
    ;; get CRC value
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_fileSize\]
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_pBuffer\]
    mov eax, dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_Arg\_FuncAddr\]
    assume eax : ptr FUNCADDRTBL
    call dword ptr \[eax\].xCRC32
    mov esi, eax
    
    ;; release the memory
    push MEM\_RELEASE
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_fileSize\]
    push dword ptr \[ebp+FP\_Check\_FileHashValue\_CRC\_pBuffer\]
    call dword ptr \[ebx\].xVirtualFree    
    
    call Get\_FP\_Check\_FileHashValue\_CRC\_Eip
    Get\_FP\_Check\_FileHashValue\_CRC\_Eip:
    pop edi
    add edi, offset FP\_Check\_FileHashValue\_Orig\_CRC - offset Get\_FP\_Check\_FileHashValue\_CRC\_Eip
    sub edi, esi
    jz NotFound\_FP\_Check\_FileHashValue\_CRC
    jmp Found\_FP\_Check\_FileHashValue\_CRC
    
Exit\_FP\_Check\_FileHashValue\_CRC:
    pop edi
    pop esi
    pop edx
    pop ecx
    pop ebx
    assume eax : nothing
    assume ebx : nothing
    mov esp, ebp
    pop ebp
    retn 08h
NotFound\_FP\_Check\_FileHashValue\_CRC:
    xor eax, eax
    jmp  Exit\_FP\_Check\_FileHashValue\_CRC
Found\_FP\_Check\_FileHashValue\_CRC:
    mov eax, 1
    jmp  Exit\_FP\_Check\_FileHashValue\_CRC
FP\_Check\_FileHashValue\_Orig\_CRC:
    dd 0   
End\_FP\_Check\_FileHashValue\_CRC:

```

Windows NT 系列平台支持一个会话多个桌面,将线程的桌面交换到自己的桌面。  
目前还没有有效的方法转换到旧的桌面。  
代码:

```
AD\_SwitchDesktop:
AD\_SwitchDesktop\_Arg\_Win32Api            equ 04h
    mov eax, dword ptr \[esp+AD\_SwitchDesktop\_Arg\_Win32Api\]
    
    push ebx
    push ecx
    push edx
    push edi
    push esi
    
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    
    call dword ptr \[ebx\].xGetCurrentThreadId
    push eax
    call dword ptr \[ebx\].xGetThreadDesktop
    mov ecx, eax
    
    push DESKTOP\_SWITCHDESKTOP
    push FALSE
    push 0
    call dword ptr \[ebx\].xOpenInputDesktop
    mov edx, eax
    
    push NULL
    push GENERIC\_ALL
    push 0
    push NULL
    push NULL
    call AD\_SwitchDesktop\_Str
        db "MyDesktop",0
    AD\_SwitchDesktop\_Str:
    call dword ptr \[ebx\].xCreateDesktopA
    mov esi, eax
    
    push esi
    call dword ptr \[ebx\].xSetThreadDesktop
    
    push esi
    call dword ptr \[ebx\].xSwitchDesktop
    
    push 5000
    call dword ptr \[ebx\].xSleep
    
    push edx
    call dword ptr \[ebx\].xSwitchDesktop
    
    push ecx
    call dword ptr \[ebx\].xSetThreadDesktop
    
    push esi
    call dword ptr \[ebx\].xCloseDesktop
    
    mov eax, 1
    
    pop esi
    pop edi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing
    retn 04h
End\_AD\_SwitchDesktop:

```

BlockInput 函数阻止了鼠标和键盘事件到达应用程序。  
代码:

```
AD\_BlockInput:
AD\_BlockInput\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr \[esp+AD\_BlockInput\_Arg\_Win32Api\]
    
    push ebx
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    
    push TRUE
    call dword ptr \[ebx\].xBlockInput
    test eax, eax
    jz Exit\_AD\_BlockInput
    push 5000
    call dword ptr \[ebx\].xSleep
    
    push FALSE
    call dword ptr \[ebx\].xBlockInput
    xor eax, eax
Exit\_AD\_BlockInput:
    pop ebx
    assume ebx : nothing
    retn 04h
End\_AD\_BlockInput:

```

利用 Nativa API 函数 ZwSetInformationThread 函数通过 ThreadHideFromDebugger 来达到隐藏调试器的目的  
代码:

```
AD\_ZwSetInfomationThread:
AD\_ZwSetInfomationThread\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr \[esp+AD\_ZwSetInfomationThread\_Arg\_Win32Api\]
    push ebx
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    
    push 0
    push NULL
    push ThreadHideFromDebugger
    call dword ptr \[ebx\].xGetCurrentThread
    push eax
    call dword ptr \[ebx\].xZwSetInformationThread
    cmp eax, STATUS\_SUCCESS
Exit\_AD\_ZwSetInfomationThread:
    pop ebx
    assume ebx : nothing
    retn 04h
End\_AD\_ZwSetInfomationThread:

```

通过系统索引号调用 NtSetInformationThread 使用 ThreadHideFromDebugger 来达到隐藏调试器的目的  
代码:

```
AD\_INT\_2e:
AD\_INT\_2e\_Arg\_Win32Api            equ 08h
AD\_INT\_2e\_StackSize                equ 10h + sizeof OSVERSIONINFOEX
AD\_INT\_2e\_bOsVersionInfoEx      equ -04h
AD\_INT\_2e\_osvi                  equ -(10h+sizeof OSVERSIONINFOEX)
    push ebp
    mov ebp, esp
    sub esp, AD\_INT\_2e\_StackSize
    
    push ebx
    push ecx
    push edx
    push edi
    push esi
    
    mov ebx, dword ptr \[ebp+AD\_INT\_2e\_Arg\_Win32Api\]
    assume ebx : ptr WIN32APIBASE
    
    lea edx, \[ebp+AD\_INT\_2e\_osvi\]
    assume edx : ptr OSVERSIONINFOEX
    
    ;; clear the stack
    lea edi, \[ebp-AD\_INT\_2e\_StackSize\]
    mov ecx, AD\_INT\_2e\_StackSize
    xor eax, eax
    cld
    rep stosb
    
    push sizeof OSVERSIONINFOEX
    pop dword ptr \[edx\].dwOSVersionInfoSize
    
    call dword ptr \[ebx\].xGetCurrentThread
    mov esi, eax
    
    lea edx, \[ebp+AD\_INT\_2e\_osvi\]
    push edx
    call dword ptr \[ebx\].xGetVersionExA
    test eax, eax
    jz Exit\_AD\_INT\_2e
    ;; Test for the Windows NT product family
    lea edx, \[ebp+AD\_INT\_2e\_osvi\]    
    cmp dword ptr \[edx\].dwPlatformId, VER\_PLATFORM\_WIN32\_NT
    jnz Exit\_AD\_INT\_2e
    push 0
    push 0
    push ThreadHideFromDebugger
    push esi
    mov eax, 0EEh        ; NtSetInformationThread
    mov edx, esp
    int 2eh
    add esp, 10h
    
    push 0
    push 0
    push ThreadHideFromDebugger
    push esi
    mov eax, 0E5h        ; NtSetInformationThread
    mov edx, esp
    int 2eh
    add esp, 10h
    ;jmp Found\_AD\_INT\_2e
    ;AD\_INT\_2e\_Tmp1:
    ;; Test for the Windows 95 product family
    ;cmp dword ptr \[edx\].dwPlatformId, VER\_PLATFORM\_WIN32\_WINDOWS
    ;jnz AD\_INT\_2e\_Tmp2
    ;AD\_INT\_2e\_Tmp2:
    ;; Microsoft Win32s
Exit\_AD\_INT\_2e:
    xor eax, eax
    pop esi
    pop edi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing
    assume edx : nothing
    mov esp, ebp
    pop ebp
    retn 04h
End\_AD\_INT\_2e:

```

Ollydbg 传递用户定义的数据给 msvcrt 的 vsprintf 函数,如果这些数据包含  
格式字符串标记.如果包换多个%s 标记，这样会指向一个无效的内存导致  
ollydbg 失效。  
代码:

```
AS\_OD\_OutputDebugString:
AS\_OD\_OutputDebugString\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr \[esp+AS\_OD\_OutputDebugString\_Arg\_Win32Api\]
    assume eax : ptr WIN32APIBASE
    call AS\_OD\_OutputDebugString\_Str
        db "%s%s",0
    AS\_OD\_OutputDebugString\_Str:
    call dword ptr \[eax\].xOutputDebugStringA
    xor eax, eax
    assume eax : nothing
    retn 04h
End\_AS\_OD\_OutputDebugString:

```

通过触发异常进入异常处理程序，将调试寄存器清 0，清除断点。  
Dr0，Dr1，Dr2，Dr3，Dr6 和 Dr7。Dr0-Dr3 用作一般的线性地址，Dr6 为调试状态寄存器  
Dr7 为调试控制寄存器。  
代码:

```
AB\_HWP\_CLR\_Exception:
    call Get\_AB\_HWP\_CLR\_Exception\_Eip
    Get\_AB\_HWP\_CLR\_Exception\_Eip:
    pop eax
    add eax, offset AB\_HWP\_CLR\_Exception\_Exception - offset Get\_AB\_HWP\_CLR\_Exception\_Eip
    push eax
    assume fs : nothing
    push dword ptr fs:\[0\]
    mov dword ptr fs:\[0\], esp
    xor eax, eax
    ;; throw an exception
    div eax
    ;; restore exception handler
    pop dword ptr fs:\[0\]
    add esp, 04h
    jmp Exit\_AB\_HWP\_CLR\_Exception
AB\_HWP\_CLR\_Exception\_Exception:
    mov eax, dword ptr \[esp+0ch\]
    assume eax : ptr CONTEXT
    ;; clr debug registers context Dr0-Dr3
    mov dword ptr \[eax\].iDr0, 0
    mov dword ptr \[eax\].iDr1, 0
    mov dword ptr \[eax\].iDr2, 0
    mov dword ptr \[eax\].iDr3, 0
    mov dword ptr \[eax\].iDr6, 0
    mov dword ptr \[eax\].iDr7, 0
    ;; set eip upon return
    add dword ptr \[eax\].regEip, 02h
    xor eax, eax
    retn
Exit\_AB\_HWP\_CLR\_Exception:
    xor eax, eax
    assume eax : nothing
    retn 0h
End\_AB\_HWP\_CLR\_Exception:

```

还有很多反调试的方法没有归结到上面，一开始想做的全些。但是后来发现，太多了。还是以后慢慢归纳吧。  
附件中的程序是一个可以添加反调试代码到其他程序的一个小玩意。  
使用命令为 AntiDebugger 1 2 3 4 file.exe 类似这样 ^\_^  
终于写完了，累吖。。。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || AntiDebugger\_bin.rar \[解压密码：pediy.com\]   
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || AntiDebugger\_src.rar  
 |

附件的程序有三个选项没有完成 验证文件尺寸，文件 CRC，内存 CRC 这三个功能的填充原先值的代码没有完成。由于这期拖的时间有点长了。 为了敢时间就没有写填充原先值的功能。（比较麻烦，添加到目标程序 ANTI 函数是动态的所以。以后添加上）如果各位大大们有兴趣可以利用提供的 CRC 函数补写完这个功能。CRC 函数的标号为 xCRC 可以到源文件中搜索下得到。![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
如果代码中出现什么 BUG 之流的东东 请各位尽情指出咯。。。