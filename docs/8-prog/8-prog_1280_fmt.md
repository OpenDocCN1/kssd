
# FileMonitor1.0 for ring3

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-616.htm`](https://www.pediy.com/kssd/pediy09/pediy09-616.htm)

> 纯汇编写的,主要拦截 `KERNEL32.DLL` 中的 `OpenFile`、`CreateFileA`、`CreateFileW`、`ReadFile`、`ReadFileEx`、`WriteFile`、`WriteFileEx`、`DeviceIoControl` 等函数，`HOOK` 到的数据未做过滤处理。

> 其中 `MYDLL` 利用了 skyer 的 `HOOKAPI` `LIB`,放出源码,主程序的过程是创建进程后挂起,注入 `MYDLL` 后恢复进程,拦截过程中的相关函数,汇编写的程序,反汇编后基本上可以理出代码,就不放源码了.

> 代码:



```
.data
OutHwndAdr  Proc
            mov  eax,offset lbHwnd
            ret
lbHwnd  DD ?
lbStr   DD 10h dup (?)
OutHwndAdr Endp
lpbuffer byte BUFLEN dup(0), 0

.data?
NewOpenFile             dd ?
NewCreateFileA          dd ?
NewCreateFileW          dd ?
NewReadFile             dd ?
NewReadFileEx           dd ?
NewWriteFile            dd ?
NewWriteFileEx          dd ?
NewDeviceIoControl      dd ?

.code
DllEntry proc hInstance:HINSTANCE, reason:DWORD, reserved1:DWORD
    .if reason==DLL_PROCESS_ATTACH
        mov  eax,TRUE
        call hookstart
    .elseif reason==DLL_PROCESS_DETACH
        call hookend
        invoke CloseHandle,hInstance
    .endif

    ret
DllEntry Endp

myOpenFile proc lpFileName:LPCSTR,lpReOpenBuff:DWORD, wStyle:UINT
LOCAL temp[200]:byte

    invoke GetModuleFileName,NULL, addr temp, sizeof temp
    invoke  lstrcpyA,addr temp,CTEXT("OpenFile-lpFileName:")
    invoke  lstrcatA,addr temp,lpFileName
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp

    push wStyle
    push lpReOpenBuff
    push lpFileName
    call NewOpenFile
    ret
myOpenFile endp

myCreateFileA proc lpFileName:LPCSTR, dwDesiredAccess:DWORD, dwShareMode: DWORD, lpSecurityAttributes:DWORD, dwCreationDisposition:DWORD, dwFlagsAndAttributes:DWORD, hTemplateFile:HANDLE
LOCAL temp[200]:byte

    invoke GetModuleFileName,NULL, addr temp, sizeof temp
    invoke  lstrcpyA,addr temp,CTEXT("CreateFileA-lpFileName:")
    invoke  lstrcatA,addr temp,lpFileName
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp
    push hTemplateFile
    push dwFlagsAndAttributes
    push dwCreationDisposition
    push lpSecurityAttributes
    push dwShareMode
    push dwDesiredAccess
    push lpFileName
    call NewCreateFileA
    ret
myCreateFileA endp

myCreateFileW proc lpFileName:LPCSTR, dwDesiredAccess:DWORD, dwShareMode: DWORD, lpSecurityAttributes:DWORD, dwCreationDisposition:DWORD, dwFlagsAndAttributes:DWORD, hTemplateFile:HANDLE
LOCAL temp[200]:byte
LOCAL fname[128]:byte
invoke GetModuleFileName,NULL, addr temp, sizeof temp

    invoke  lstrcpyA,addr temp,CTEXT("CreateFileW-lpFileName:");
    invoke WideCharToMultiByte,CP_ACP, 0, lpFileName, -1, addr fname, 128,NULL,NULL
    invoke  lstrcatA,addr temp,addr fname
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp
    push hTemplateFile
    push dwFlagsAndAttributes
    push dwCreationDisposition
    push lpSecurityAttributes
    push dwShareMode
    push dwDesiredAccess
    push lpFileName
     call NewCreateFileW
    ret
myCreateFileW endp

myReadFile proc hFile:HANDLE,lpBuffer:LPVOID,nNumberOfBytesToRead:DWORD,lpOverlapped:DWORD,lpCompletionRoutine:DWORD
LOCAL temp[200]:byte
LOCAL buf:dword

    invoke  lstrcpyA,addr temp,CTEXT("ReadFile-hFile:0x");
    invoke  wsprintfA,addr buf,CTEXT("%x"), hFile
    invoke  lstrcatA,addr temp,addr buf
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp
    push lpCompletionRoutine
    push lpOverlapped
    push nNumberOfBytesToRead
    push lpBuffer
    push hFile
    call NewReadFile
    ret
myReadFile endp

myReadFileEx proc hFile:HANDLE,lpBuffer:LPVOID,nNumberOfBytesToRead:DWORD,lpOverlapped:DWORD,lpCompletionRoutine:DWORD
LOCAL temp[200]:byte
LOCAL buf:dword

    invoke  lstrcpyA,addr temp,CTEXT("ReadFileEx-hFileEx:0x");
    invoke  wsprintfA,addr buf,CTEXT("%x"), hFile
    invoke  lstrcatA,addr temp,addr buf
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp
    push lpCompletionRoutine
    push lpOverlapped
    push nNumberOfBytesToRead
    push lpBuffer
    push hFile
    call NewReadFileEx
    ret
myReadFileEx endp

myWriteFile proc hFile:HANDLE,lpBuffer:LPCVOID, nNumberOfBytesToWrite:DWORD,lpNumberOfBytesWritten:LPDWORD,lpOverlapped:DWORD
LOCAL temp[200]:byte
LOCAL buf:dword

    invoke  lstrcpyA,addr temp,CTEXT("WriteFile-hFile:0x");
    invoke  wsprintfA,addr buf,CTEXT("%x"), hFile
    invoke  lstrcatA,addr temp,addr buf
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp
    push lpOverlapped
    push lpNumberOfBytesWritten
    push nNumberOfBytesToWrite
    push lpBuffer
    push hFile
    call NewWriteFile
    ret
myWriteFile endp

myWriteFileEx proc hFile:HANDLE,lpBuffer:LPCVOID, nNumberOfBytesToWrite:DWORD,lpNumberOfBytesWritten:LPDWORD,lpOverlapped:DWORD
LOCAL temp[200]:byte
LOCAL buf:dword

    invoke  lstrcpyA,addr temp,CTEXT("WriteFileEx-hFile:0x");
    invoke  wsprintfA,addr buf,CTEXT("%x"), hFile
    invoke  lstrcatA,addr temp,addr buf
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp
    push lpOverlapped
    push lpNumberOfBytesWritten
    push nNumberOfBytesToWrite
    push lpBuffer
    push hFile
    call NewWriteFile
    ret
myWriteFileEx endp

myDeviceIoControl proc hDevice:HANDLE,dwIoControlCode:DWORD,IpInBuffer:LPVOID,nInbufferSize:DWORD,IpOutBuffer:LPVOID,nOutBufferSize:DWORD,IpBytesReturned:LPDWORD,IpOverlapped:DWORD
LOCAL temp[200]:byte
LOCAL buf1:dword
LOCAL buf2:dword
LOCAL buf3:dword
LOCAL buf4:dword
LOCAL buf5:dword
LOCAL InbufferSize:dword
LOCAL IoControlCode:dword
LOCAL OutBuffer:dword
LOCAL OutBufferSize:dword
LOCAL BytesReturned:dword

    invoke  lstrcpyA,addr temp,CTEXT("hDevice:");
    invoke  lstrcatA,addr temp,hDevice
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr temp

    invoke  lstrcpyA,addr IoControlCode,CTEXT("dwIoControlCode:");
    invoke  wsprintfA,addr buf1,CTEXT("%x"), dwIoControlCode
    invoke  lstrcatA,addr IoControlCode,addr buf1
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr IoControlCode

    invoke  lstrcpyA,addr OutBuffer,CTEXT("nInbufferSize:");
    invoke  wsprintfA,addr buf5,CTEXT("%d"), nInbufferSize
    invoke  lstrcatA,addr InbufferSize,addr buf5
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr InbufferSize

    invoke  lstrcpyA,addr OutBuffer,CTEXT("IpOutBuffer:");
    invoke  wsprintfA,addr buf2,CTEXT("0x%x"), IpOutBuffer
    invoke  lstrcatA,addr OutBuffer,addr buf2
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr OutBuffer

    invoke  lstrcpyA,addr OutBufferSize,CTEXT("nOutBufferSize:");
    invoke  wsprintfA,addr buf3,CTEXT("%d"), nOutBufferSize
    invoke  lstrcatA,addr OutBufferSize,addr buf3
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr OutBufferSize

    invoke  lstrcpyA,addr BytesReturned,CTEXT("IpBytesReturned:");
    invoke  wsprintfA,addr buf4,CTEXT("0x%x"), IpBytesReturned
    invoke  lstrcatA,addr BytesReturned,addr buf4
    invoke  SendMessageA, lbHwnd, LB_ADDSTRING, 0, addr BytesReturned

    push IpOverlapped
    push IpBytesReturned
    push nOutBufferSize
    push IpOutBuffer
    push nInbufferSize
    push IpInBuffer
    push dwIoControlCode
    push hDevice
    call NewDeviceIoControl

    ret
myDeviceIoControl endp

hookstart proc

        invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("OpenFile"),offset myOpenFile
mov NewOpenFile, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("CreateFileA"),offset myCreateFileA
mov NewCreateFileA, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("CreateFileW"),offset myCreateFileW
mov NewCreateFileW, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("ReadFile"),offset myReadFile
mov NewReadFile, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("ReadFileEx"),offset myReadFileEx
mov NewReadFileEx, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("WriteFile"),offset myWriteFile
mov NewWriteFile, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("WriteFileEx"),offset myWriteFileEx
mov NewWriteFileEx, eax
invoke HookAPI,CTEXT("KERNEL32.DLL"),CTEXT("DeviceIoControl"),offset myDeviceIoControl
mov NewDeviceIoControl, eax
ret
hookstart Endp

hookend proc

        invoke UnhookAPI,myOpenFile
invoke UnhookAPI,myCreateFileA
invoke UnhookAPI,myCreateFileW
invoke UnhookAPI,myReadFile
invoke UnhookAPI,myReadFileEx
invoke UnhookAPI,myWriteFile
invoke UnhookAPI,myWriteFileEx
invoke UnhookAPI,myDeviceIoControl
ret
hookend Endp

End DllEntry
```



deroko 的那个可能比较完美:

[`bbs.pediy.com/showthread.php?...light=Ultimate`](http://bbs.pediy.com/showthread.php?t=45601&highlight=Ultimate)

```asm
.386  
.model flat, stdcall  
option casemap: none  

include windows.inc  
include user32.inc  
include kernel32.inc  

includelib user32.lib  
includelib kernel32.lib  

public C          Detoured_MessageBoxA  
public C          Detoured_GetModuleHandleA  

.data?  
Detoured_MessageBoxA     dd     ?  
Detoured_GetModuleHandleA dd    ?  

.code  

DllEntry proc hInstance:HINSTANCE, reason:DWORD, reserved1:DWORD  
 .if reason==DLL_PROCESS_ATTACH  
       mov  eax,TRUE  
 .endif   
 ret  
DllEntry Endp  
HOOK_user32_MessageBoxA proc hwnd:DWORD, text:DWORD, about:DWORD, icon:DWORD  

             push icon  
             push about  
             push text  
             push hwnd  
             call Detoured_MessageBoxA  
             ret  

HOOK_user32_MessageBoxA endp  

HOOK_kernel32_GetModuleHandleA proc modulename:dword  

             push modulename  
             call Detoured_GetModuleHandleA  
             ret  

HOOK_kernel32_GetModuleHandleA  endp  

HOOK_kernel32_ExitProcess proc exitcode:dword  

             invoke    TerminateProcess, -1, exitcode  
             ret  

HOOK_kernel32_ExitProcess endp  

End DllEntry  
```

```
-----------------------mydll.Inc-------------  
HOOK_kernel32_GetModuleHandleA proto  
HOOK_user32_MessageBoxA proto  
HOOK_kernel32_ExitProcess proto  
Detoured_GetModuleHandleA proto  
Detoured_MessageBoxA proto  
```

```
------------------------mydll.Def-------------  
EXPORTS  
HOOK_user32_MessageBoxA  
HOOK_kernel32_GetModuleHandleA  
HOOK_kernel32_ExitProcess  
Detoured_GetModuleHandleA  
Detoured_MessageBoxA 
```

4dd466d0e1e0cc123afbe504da2462ca
