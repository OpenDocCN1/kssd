
# Win32 汇编实现 DLL 的远程注入及卸载

> 原文：[`www.pediy.com/kssd/pediy10/60763.html`](https://www.pediy.com/kssd/pediy10/60763.html)

（声明：魏滔序原创，转贴请注明出处。）

所谓 DLL 远程注入，就是强迫 DLL 程序运行在其他进程中，这样做的目的无非有两种：第一是伪装自身，第二是控制宿主。前者常见于病毒或木马，后者则一般用于正规之场合，比如常见的输入法、外挂等等，有时 Hook(钩子)也用到该技术。由此可见，技术是把双刃剑，区别在于使用技术的人。

DLL 远程注入的方法一般有如下几种：

1. 修改注册表，系统启动时自动加载。当然，如果卸载就要关闭系统了，或者使用本文的卸载功能。
2. 使用 `SetWindowsHookEx` 进行注入，该参数最后一个参数值决定注入的全局与否。
3. BHO，这个没什么可说的，仅局限与浏览器。
4. 使用 `CreateRemoteThread` 进行注入，也就是本文的主要内容。

注入过程大致是：

1. 打开目标进程。
2. 在目标进程中申请一块内存，用来存放要注入的 dll 名称。
3. 将 dll 名称写入该内存。
4. 获得 `LoadLibraryA` 函数的地址，该函数在任何进程中的地址都相同。
5. 创建远程线程，也就是执行 `LoadLibraryA` 函数了。当然 `LoadLibraryA` 的参数就是上面保存的 dll 名称。
6. 等待线程返回。

到此为止，dll 已经被成功在远程进程中运行了，至于运行后该如何工作，那就发挥你无穷的想像力吧。卸载过程与注入过程雷同，只是区别于后面使用了 `FreeLibrary` 函数进行卸载。注意，`FreeLibrary` 函数只能传入模块的句柄，可使用 `GetModuleHandleA` 函数获得。

源码奉上：

```asm
;::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
; Win32 汇编实现 DLL 的远程注入及卸载   
; Programmed by 魏滔序                   
; WebSite: http://www.chenoe.com         
; Blog: http://blog.csdn.net/Modest      
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

    .486            ;   create   32   bit   code 
    .model    flat,    stdcall    ;   32   bit   memory   model 
    option    casemap    :none    ;   case   sensitive 
    include    windows.inc 
    include    kernel32.inc 
    includelib kernel32.lib 

    RemoteInjectModule    PROTO :DWORD,:DWORD
    RemoteUnloadModule    PROTO :DWORD,:DWORD 

.data 
    szKernel32     db 'Kernel32',0
    szGetModuleHandleA db 'GetModuleHandleA',0
    szLoadLibraryA     db 'LoadLibraryA',0
    szFreeLibrary      db 'FreeLibrary',0 

.code 

Start: 

RemoteInjectModule proc dwProcID,pszModule 
    LOCAL    hProcess, hThread, dwExitCode 
    LOCAL    pszBuffer,pdwAddress 
    Invoke    OpenProcess,PROCESS_QUERY_INFORMATION Or PROCESS_CREATE_THREAD Or PROCESS_VM_OPERATION Or PROCESS_VM_WRITE, 0, dwProcID 
    MOV    hProcess,EAX 
    .If     hProcess == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    VirtualAllocEx,hProcess, 0, MAX_PATH, MEM_COMMIT, PAGE_READWRITE 
    MOV    pszBuffer,EAX 
    .If     pszBuffer == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    WriteProcessMemory,hProcess, pszBuffer, pszModule, MAX_PATH, 0 
    .If     EAX == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    GetModuleHandle,addr szKernel32 
    Invoke    GetProcAddress,EAX, addr szLoadLibraryA 
    MOV    pdwAddress,EAX 
    .If     pdwAddress == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    CreateRemoteThread,hProcess, 0,0, pdwAddress, pszBuffer, 0, 0 
    MOV    hThread,EAX 
    .If     hThread == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    WaitForSingleObject,hThread, INFINITE 
    Invoke    GetExitCodeThread,hThread,addr dwExitCode 
    .If     dwExitCode != 0 
        Invoke    VirtualFreeEx, hProcess, pszBuffer, 0, MEM_RELEASE 
        Invoke    CloseHandle,hProcess 
        MOV    EAX,1 
    .else 
        MOV    EAX,0 
    .endif 
    RET 

ErrHandle: 
    .If     pszBuffer != 0 
        Invoke    VirtualFreeEx, hProcess, pszBuffer, 0, MEM_RELEASE 
    .Endif 
    .If     hThread != 0 
        Invoke    CloseHandle, hThread 
    .Endif 
    .If     hProcess != 0 
        Invoke    CloseHandle,hProcess 
    .Endif 
    MOV    EAX,0 
    RET 

RemoteInjectModule endp 

[HIDE]RemoteUnloadModule proc dwProcID,pszModule 
    LOCAL    hProcess, hThread, dwExitCode 
    LOCAL    pszBuffer,pdwAddress 
    Invoke    OpenProcess,PROCESS_QUERY_INFORMATION Or PROCESS_CREATE_THREAD Or PROCESS_VM_OPERATION Or PROCESS_VM_WRITE, 0, dwProcID 
    MOV    hProcess,EAX 
    .If     hProcess == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    VirtualAllocEx,hProcess, 0, MAX_PATH, MEM_COMMIT, PAGE_READWRITE 
    MOV    pszBuffer,EAX 
    .If     pszBuffer == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    WriteProcessMemory,hProcess, pszBuffer, pszModule, MAX_PATH, 0 
    .If     EAX==0 
        JMP ErrHandle 
    .Endif 
    Invoke    GetModuleHandle,addr szKernel32 
    Invoke    GetProcAddress,EAX, addr szGetModuleHandleA 
    MOV    pdwAddress,EAX 
    .If     pdwAddress == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    CreateRemoteThread,hProcess, 0, 0, pdwAddress, pszBuffer, 0,0 
    MOV    hThread,EAX 
    .If     hThread == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    WaitForSingleObject,hThread, INFINITE 
    Invoke    GetExitCodeThread, hThread, addr dwExitCode 
    Invoke    VirtualFreeEx, hProcess, pszBuffer, 0, MEM_RELEASE 
    Invoke    CloseHandle, hThread 
    Invoke    GetModuleHandle,addr szKernel32 
    Invoke    GetProcAddress,EAX, addr szFreeLibrary 
    MOV    pdwAddress,EAX 
    .If     pdwAddress == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    CreateRemoteThread,hProcess, 0, 0, pdwAddress, dwExitCode, 0,0 
    MOV    hThread,EAX 
    .If     hThread == 0 
        JMP ErrHandle 
    .Endif 
    Invoke    WaitForSingleObject,hThread, INFINITE 
    Invoke    GetExitCodeThread, hThread, addr dwExitCode 
    .If     dwExitCode != 0 
        Invoke    VirtualFreeEx, hProcess, pszBuffer, 0, MEM_RELEASE 
        Invoke    CloseHandle,hProcess 
        MOV    EAX,1 
    .else 
        MOV    EAX,0 
    .endif 
    RET 

ErrHandle: 
    .If     pszBuffer != 0 
        Invoke    VirtualFreeEx, hProcess, pszBuffer, 0, MEM_RELEASE 
    .Endif 
    .If     hThread != 0 
        Invoke    CloseHandle, hThread 
    .Endif 
    .If     hProcess != 0 
        Invoke    CloseHandle,hProcess 
    .Endif 
    MOV    EAX,0 
    RET 

RemoteUnloadModule endp[/HIDE] 

End   Start
```

`http://blog.csdn.net/Modest/archive/2008/01/20/2054975.aspx`
