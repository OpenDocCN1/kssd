
# Win32 汇编 之 DLL 与进程

> 原文：[`www.pediy.com/kssd/pediy11/121736.html`](https://www.pediy.com/kssd/pediy11/121736.html)

## 文章信息

- **【文章标题】**: DLL 与线程
- **【文章作者】**: Gall
- **【作者主页】**: `http://hi.baidu.com/8ohack`
- **【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

### DLL 与进程

将 DLL 与进程一起总结,是因为二者关联紧密.在做深入介绍前,县介绍一下 DLL 与进程等的概念.

#### DLL

1. **什么是动态链接库,静态链接库又指什么?区别在哪里?**
2. **Win32 下 DLL 的编写**
   - **DLL 初始化等情况**
     - `DLL_PROCESS_ATTACH`
     - `DLL_PROCESS_DETACH`
     - `DLL_THREAD_ATTACH`
     - `DLL_THREAD_DETACH`
   - **DLL 导入表与导出表中函数信息**
     - **DLL 中函数的调用**
       - `pushParam1`
       - `pushParam2`
       - ...
       - `callFunctionName`
   - **DLL 编译时,需要注意的参数情况**
   - **DLL 编译完成产生的文件,各有什么作用**
3. **DLL 的调用**
   - **静态链接**
     - 开发程序的时,利用 `include` 与 `includelib` 使用
     - 程序编译后,链接 DLL 文件
   - **动态链接**
     - `LoadLibrary` 与 `GetProcAddress`
       - `invokeFunctionName,Param1,Param2......`
       - `[需要定义函数原形]`
       - `_FunctionNametypedefproto:DWORD,:DWORD`
       - `FunctionNametypedefptr_FunctionName`
       - `FunctionName` 的值为 `GetProcAddress` 传入的地址
     - `Rundll32` 等
4. **DLL 的保护**
   - 加壳等

DLL 的常见用途如下:

- 钩子
- 线程
- 资源库等

### 进程

1. 进程,堆,栈的含义?
2. 远程线程
3. 进程间的通讯
4. 异常处理等

有待深入学习 `[慢慢总结]`

## 经验总结

### Hook.asm [钩子案例]

```asm
;==============Hook.asm[钩子案例]=====================================
.386
.model flat,stdcall
option casemap:none
include    windows.inc
include    user32.inc
includelib user32.lib
include    kernel32.inc
includelib kernel32.lib

.data
hInstance        dd          ?

.data?
hWnd             dd          ?
hHook            dd          ?
dwMessage        dd          ?
szAscii          db          4 dup (?)

.code
DllEntry         proc        _hInstance,_dwReason,_dwReserved
                 push        _hInstance
                 pop         hInstance         ; 保存 DLL 的句柄
                 mov         eax,TRUE
                 ret
DllEntry         endp

HookProc         proc        _dwCode,_wParam,_lParam
                 local       @szKeyState[256]:byte
                 invoke      CallNextHookEx,hHook,_dwCode,_wParam,_lParam
                 invoke      GetKeyboardState,addr @szKeyState
                 invoke      GetKeyState,VK_SHIFT
                 mov         @szKeyState + VK_SHIFT,al         ;   不太清楚这句是什么意思
                 mov         ecx,_lParam
                 shr         ecx,16
                 invoke      ToAscii,_wParam,ecx,addr @szKeyState,addr szAscii,0
                 mov         byte ptr szAscii [eax],0
                 invoke      SendMessage,hWnd,dwMessage,dword ptr szAscii,NULL
                 xor         eax,eax
                 ret
HookProc         endp

InstallHook      proc        _hWnd,_dwMessage
                 push        _hWnd
                 pop         hWnd
                 push        _dwMessage
                 pop         dwMessage
                 invoke      SetWindowsHookEx,WH_KEYBOARD,addr HookProc,hInstance,NULL
                 mov         hHook,eax
                 ret
InstallHook      endp

UninstallHook    proc
                 invoke      UnhookWindowsHookEx,hHook
                 ret
UninstallHook    endp

end DllEntry
```

### DLL 注入技术 二案例

```asm
;============================================================================================
; DLL 注入技术 二案例
;========================================================================================================
;Dynamic Linker Library
;DLL 线程插入
;=================================================
.386
.model flat,stdcall
option casemap:none
include       windows.inc
include       user32.inc
includelib    user32.lib
include       kernel32.inc
includeLIB    kernel32.lib

.const
szCls          db           'Shell_TrayWnd',0
szSign         db           '\\',0
szDll          db           'server.dat',0
szKernel       db           'kernel32.dll',0
szLoadLibrary  db           'LoadLibraryA',0

.data?
cDir           db           MAX_PATH dup (?)
pDll           dd           ?
pDllSize       dd           ?
hWnd1          dd           ?
Pid1           dd           ?
hProcess1      dd           ?
vAddress1      dd           ?
wmAddress      dd           ?
hThread        dd           ?

.code
start:
               ;===================================================
               ; 获取当前路径
               invoke      GetCurrentDirectory,MAX_PATH,addr cDir
               ;===================================================
               ; [当前路径 + "\\" + Dll 文件名]
               invoke      lstrcat,addr cDir,addr szSign
               invoke      lstrcat,eax,addr szDll
               mov          pDll,eax
               ; pDll 完整路径[指针]
               ;===================================================
               invoke      lstrlen,pDll
               ; pDllSize 存放路径长度
               mov          pDllSize,eax
               ;===================================================
               invoke      MessageBoxA,NULL,pDll,pDll,NULL
               invoke      FindWindow,addr szCls,NULL
               mov          hWnd1,eax
               invoke      GetWindowThreadProcessId,hWnd1,addr Pid1
               invoke      OpenProcess,PROCESS_ALL_ACCESS,FALSE,Pid1
               mov          hProcess1,eax
               invoke      VirtualAllocEx,hProcess1,NULL,\
                                        pDllSize,\
                                        MEM_COMMIT,\
                                        PAGE_EXECUTE_READWRITE
               mov          vAddress1,eax
               invoke      WriteProcessMemory,hProcess1,\
                                        vAddress1,\
                                        pDll,\
                                        pDllSize,\
                                        wmAddress
               invoke      LoadLibrary,addr szKernel
               invoke      GetProcAddress,eax,addr szLoadLibrary
               invoke      CreateRemoteThread,hProcess1,NULL,NULL,\
                                        eax,\
                                        vAddress1,NULL,NULL
               mov          hThread,eax
               .if          hThread
                           invoke WaitForSingleObject,hThread,INFINITE
                           invoke CloseHandle,hThread
               .endif
                           invoke VirtualFreeEx,hProcess1,vAddress1,NULL,\
                                        MEM_RELEASE
                           invoke CloseHandle,hProcess1
                           invoke ExitProcess,NULL
end start
```



# DLL 自删除

```asm
;=============================================================================================
; DLL 自删除
;============================================================
;
; 汇编代码部分
;
.386
.model flat,stdcall
option casemap:none

;============================================================
include             user32.inc
includelib          user32.lib
include             kernel32.inc
includelib          kernel32.lib
include             windows.inc

;=============================================================
.data
;=============================================================
tit           db    '计算器',0
pid           dd    0
hProcess      dd    0
hThread       dd    0
pCodeRemote   dd    0

;=============================================================
.const
;=============================================================
szmsg                 db                  'MessageBoxA',0
userdll               db                  'User32.dll',0
szDel                 db                  'DeleteFileA',0
szloadlib             db                  'LoadLibraryA',0
kerdll                db                  'kernel32.dll',0

;=============================================================
.code
;=============================================================
codebegin:
dispdata            db                  "C:\1.dll",0
datalen = $-codebegin

Rproc proc msgbox ;MessageBoxA 的地址为参数
          CALL @F ;push esi
          @@:
          POP EBX
          SUB EBX,OFFSET @B
          LEA ECX,[EBX+dispdata]
          push ecx
          call msgbox
          ret ;重要
Rproc endp
codelen = $-codebegin ;代码长度 13 字节

;=============================================================
DllEntry    proc      hInstance,dwReason,dwReserved
                       mov eax,TRUE
                       ret
DllEntry    endp

deleteMe    proc
            invoke FindWindow,0,offset tit                          ;返回计算器窗口句柄
            invoke GetWindowThreadProcessId,eax,offset pid         ;计算机器程序的进程 PID 号
            invoke OpenProcess,PROCESS_ALL_ACCESS,FALSE,pid        ;打开进程,得到进程句柄
            mov hProcess,eax                                       ;保存进程句柄
            ;-==============-[第一部分]程序代码注入===============
            invoke VirtualAllocEx,hProcess,0, codelen,\
                        MEM_COMMIT, PAGE_EXECUTE_READWRITE
            mov pCodeRemote,eax
            invoke WriteProcessMemory,hProcess,\
                            pCodeRemote,\
                            offset codebegin,\
                            codelen,NULL
            mov esi,pCodeRemote
            add esi,datalen
            push esi
            invoke LoadLibrary,offset kerdll
            invoke GetProcAddress,eax,offset szDel
            pop esi
            invoke CreateRemoteThread,hProcess,0,0,esi,eax,0,0
            mov hThread,eax                                         ; 返回线程句柄
            .if hThread
             ;invoke WaitForSingleObject,hThread,INFINITE           ;等待线程结束
             invoke CloseHandle,hThread                             ;关闭线程句柄
            .endif
             invoke VirtualFreeEx,hProcess,pCodeRemote,\
                            MEM_COMMIT,MEM_RELEASE
             invoke CloseHandle,hProcess                             ;关闭进程句柄
             invoke ExitProcess,NULL

deleteMe  endp
end DllEntry
```

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2010 年 10 月 06 日 16:27:14
