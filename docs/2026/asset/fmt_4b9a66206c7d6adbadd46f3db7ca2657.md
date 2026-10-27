
# [原创]AMI WINBIOS密码提取器

最近写了一个AMI WINBIOS密码提取器，能方便读取BIOS密码~~~

大鸟请飞过...........

说明：编译环境：`RadASM`

附：资源文件代码未给出

;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

;●                             AMI WINBIOS密码提取器(Ring3)                      ●

;●                                                                                  ●

;● 作者:CoCol   QQ:1174968967                              2010-6                   ●

;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

```asm
.386

.model flat,stdcall

option casemap:none

include KillYou.inc

IDD_DL   equ   1000

IDC_BTN1   equ   1005

IDI_ICON   equ   1009

IDA_SYS   equ   2000

.data

szDriverPath2 db "%SystemRoot%\system32\drivers\SafeComs360.sys",0

szDriverName db "SafeComs360",0

_Table db 2h   dup('?'),'1','2','3','4','5','6','7','8','9','0'

       db 4h   dup('?'),'Q','W','E','R','T','Y','U','I','O','P'

       db 4h   dup('?'),'A','S','D','F','G','H','J','K','L'

       db 5h   dup('?'),'Z','X','C','V','B','N','M'

       db 14h   dup('?'),'g','h','i','?','d','e','f','?','a','b','c','o'

       db 0adh   dup('?'),0

_Cram  db 80h,7h dup(0)

_Pwd   db 7h dup(0)

.data?

hInstance dd ?

dwBytesReturned dd ?

szDriverPath db 260 dup(?)

.code

SearchTable proc pTable:dword,dbNum:byte
  local Asicc:byte
  pushad
  mov ebx,pTable
  mov al,dbNum
  xlat
  mov Asicc,al
  popad
  mov al,Asicc
  ret
SearchTable endp

Decryption proc dbFrontByte:byte,dbBehindByte:byte,pTable:dword
  local Asicc:byte
  pushad
  xor dl,dl
  mov al,byte ptr dbFrontByte
  mov ah,byte ptr dbBehindByte
    De1:mov ch,al
  and ch,0c3h
  jp De2
  stc
  jmp De3
    De2:clc
    De3:rcr al,1
  inc dl
  cmp al,ah
  jnz De1
  invoke SearchTable,pTable,dl
   mov Asicc,al
  popad
  mov al,Asicc
  ret
Decryption endp

Crack proc
  pushad
  mov esi,offset _Cram
  mov edi,offset _Pwd
  mov bl,byte ptr[esi]
    Cra:inc esi
  mov bh,byte ptr[esi]
  cmp bh,0
  jz Disp
  invoke Decryption,bl,bh,offset _Table
  mov [edi],al
  inc edi
  mov bl,bh
  jmp Cra
   Disp:invoke MessageBox,NULL,offset _Pwd ,$CTA0("提取到的本机BIOS密码      "),MB_ICONINFORMATION+MB_OK
  popad
  ret
Crack endp

;--------------------------------------------------------------------------------------
;函数:StartSerivice
;参数说明:No
;功能:启动服务
;--------------------------------------------------------------------------------------

StartSerivice proc
  local hSCM:dword
  local hService:dword
  pushad
        invoke OpenSCManager,0,0,SC_MANAGER_ALL_ACCESS
  .if eax
    mov hSCM,eax
    invoke CreateService,hSCM,addr szDriverName,addr szDriverName,\
    SERVICE_ALL_ACCESS,SERVICE_KERNEL_DRIVER,SERVICE_DEMAND_START,\
    SERVICE_ERROR_NORMAL,addr szDriverPath,0,0,0,0,0
      .if eax
         mov hService,eax
      .else
         invoke OpenService,hSCM,addr szDriverName,SERVICE_ALL_ACCESS
         .if eax
            mov hService,eax
         .else
          invoke CloseServiceHandle,hSCM
      invoke MessageBox,0,$CTA0("初始化失败..."),0,16
      invoke ExitProcess,hInstance
      .endif
    .endif
  invoke StartService,hService,0,0
  invoke CloseServiceHandle,hService
  invoke CloseServiceHandle,hSCM
  .endif
  invoke DeleteFile,offset szDriverPath
   ret
  popad
StartSerivice endp

;--------------------------------------------------------------------------------------
;函数:CreateSys
;参数说明:hinstance：程序句柄、dwResID：资源ID、dwFilePath：生成文件路径
;功能:释放驱动程序
;--------------------------------------------------------------------------------------

CreateSys proc hinstance:dword,dwResID:dword,dwFilePath:dword
        local hResInfo:dword
        local dwResSize:dword
        local lpRes:dword
        local hFile:dword
        pushad
  invoke FindResource,hinstance,dwResID,RT_RCDATA
  .if eax
    mov hResInfo,eax
    invoke SizeofResource,hinstance,hResInfo
    mov dwResSize,eax
    invoke LoadResource,hinstance,hResInfo
    .if eax
      mov lpRes,eax
      invoke SetFileAttributes,dwFilePath,FILE_ATTRIBUTE_NORMAL
      invoke CreateFile,dwFilePath,GENERIC_WRITE,FILE_SHARE_READ,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL
                    mov hFile,eax
                  invoke SetFilePointer,hFile,NULL,NULL,FILE_BEGIN
                  invoke _lwrite,hFile,lpRes,dwResSize
                  invoke SetFileAttributes,dwFilePath,FILE_ATTRIBUTE_NORMAL or  FILE_ATTRIBUTE_SYSTEM or FILE_ATTRIBUTE_HIDDEN
                  invoke _lclose,hFile
    .endif
  .endif
  popad
  ret
CreateSys endp

;--------------------------------------------------------------------------------------
;函数:StartSetRam
;参数说明:No
;功能:连接服务并发送控制代码到驱动
;--------------------------------------------------------------------------------------

StartGetRam proc
        local hDevice:dword
  pushad
  invoke CreateFile,$CTA0("\\\\.\\SetCmosRam"),GENERIC_READ + GENERIC_WRITE,0,NULL,OPEN_EXISTING,0,NULL
  .if eax!=INVALID_HANDLE_VALUE
     mov hDevice,eax
    invoke DeviceIoControl,hDevice,GET_CMOS_RAM,NULL,0,(offset _Cram)+1,sizeof _Cram ,addr dwBytesReturned,NULL
     .if  (eax!=0)&&(dwBytesReturned!=0)
       call Crack
      invoke CloseHandle,hDevice
       jmp Exit
    .else
      invoke MessageBox, NULL, $CTA0("启动失败,尝试重新运行本程序.."), $CTA0("错误"), MB_OK + MB_ICONSTOP
      jmp Exit
    .endif
  .endif
  invoke MessageBox, NULL, $CTA0("打开服务失败.."),$CTA0("错误"), MB_OK + MB_ICONSTOP
  Exit:
  popad
  ret
StartGetRam endp

;--------------------------------------------------------------------------------------
;函数:WndProc
;参数说明:hWnd：uMsg：wParam：lParam
;功能:窗体事件过程
;--------------------------------------------------------------------------------------

WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
  pushad
      .if uMsg == WM_CLOSE
          invoke EndDialog, hWnd, 0
      .elseif uMsg == WM_INITDIALOG
          invoke LoadIcon,hInstance,IDI_ICON
          invoke SendMessage,hWnd,WM_SETICON,ICON_BIG,eax
          invoke ExpandEnvironmentStrings,offset szDriverPath2,offset szDriverPath,260
          invoke CreateSys,hInstance,IDA_SYS,offset szDriverPath
          invoke StartSerivice
      .elseif uMsg == WM_COMMAND
          mov eax, wParam
            mov edx, eax
            shr edx, 16
          movzx eax, ax
            .if edx == BN_CLICKED
                .if eax == IDC_BTN1
                  invoke StartGetRam
                  invoke ExitProcess,hInstance
                .endif
          .endif
      .else
          mov eax, FALSE
                 ret
      .endif
    mov eax, TRUE
    ret
WndProc endp

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

;   程序入口:

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

@Start:
    invoke GetModuleHandle, NULL
    mov hInstance, eax
    invoke DialogBoxParam, eax,IDD_DLG, 0, WndProc, 0
    invoke ExitProcess, eax
end @Start
```

;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

## AMI WINBIOS密码提取器(驱动)

;●                                                                                  ●

;● 作者:CoCol   QQ:1174968967                              2010-6                   ●

;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

```asm
.386

.model flat, stdcall

option casemap:none

include IoCmosRam.inc

.const

CCOUNTED_UNICODE_STRING "\\Device\\DevCMOS",g_usDeviceName,4

CCOUNTED_UNICODE_STRING "\\??\\SetCmosRam",g_usSymbolicLinkName,4

.data

Buff1  db 8h dup (0)

.code

;--------------------------------------------------------------------------------------
;函数:DriverUnload
;功能:卸载例程
;--------------------------------------------------------------------------------------
```
