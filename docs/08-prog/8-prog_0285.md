
# HOOK API 续之模拟覆盖法 实例 AntiDesktop

> 原文：[`www.pediy.com/kssd/pediy10/90303.html`](https://www.pediy.com/kssd/pediy10/90303.html)

【文章标题】HOOK API 续之模拟覆盖法 实例 AntiDesktop

【文章作者】nohacks(非安全,hacker0058)

【作者主页】`blog.nohacks.net`

【文章出处】看雪论坛(`bbs.pediy.com`)

这个小软件的作用是禁止程序创建虚拟桌面,防止此类软件，如防锁专家，幽灵网吧辅助工具等躲避计费软件（注：`HideProcess.dll` 为隐藏进程模块，因为采用病毒技术，可能会被杀毒软件查杀，不过没有它也不影响正常使用，只不过进程不能隐藏。）

软件的原理是勾住 `CreateDesktop` 这个 API 禁止创建虚拟桌面，小软件没啥技术含量，发表在这里只为做个备份，另外给大家参考参考，有什么错误或需要改正的地方请指出，谢谢！

软件 分 DLL 和调用部分，先看 DLL 的主要代码：

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>;
;           Programmed by nohacks, nohacks@163.com                    ;
;               Website: http://blog.nohacks.net                     ;
;            编(MASM):HOOK API 续之模拟覆盖法 实例 AntiDesktop                   ;
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>;

.486
.model flat,stdcall 
option casemap:none 

include hooklib.inc
           
new\_CreateDesktopA  proto  :DWORD, :DWORD , :DWORD, :DWORD , :DWORD, :DWORD 
new\_CreateDesktopW  proto  :DWORD, :DWORD , :DWORD, :DWORD , :DWORD, :DWORD 

; you code

.data

old\_CreateDesktopA      db 090h,090h,090h,090h,090h,090h,090h,090h,\
             090h,090h,090h,090h,090h,090h,090h,090h,\
             0E9h,000h,000h,000h,000h
             
addres\_CreateDesktopA dd 0
 size\_CreateDesktopA  dd 0

old\_CreateDesktopW      db 090h,090h,090h,090h,090h,090h,090h,090h,\
             090h,090h,090h,090h,090h,090h,090h,090h,\
             0E9h,000h,000h,000h,000h
             
 addres\_CreateDesktopW dd 0
 size\_CreateDesktopW  dd 0

;you code

;共用部分         
hInstance dd 0
bakapi dd 0

.data? 
hHook dd ? 
hWnd dd ?

.code

DllEntry proc hInst:HINSTANCE, reason:DWORD, reserved1:DWORD 
     
    .if reason==DLL\_PROCESS\_ATTACH     ;当 DLL 加载时产生此事件
     
     push hInst 
        pop hInstance 
          
      ;下面几行请根据实际 API 修改
          
     ;CreateDesktopA 部分
     
      invoke GetAddress,CTEXT("user32.dll"),CTEXT("CreateDesktopA")   
        mov addres\_CreateDesktopA,eax
       invoke  Hookapi, addres\_CreateDesktopA,addr new\_CreateDesktopA,addr old\_CreateDesktopA 
       mov size\_CreateDesktopA,eax
     
          
     ;CreateDesktopW 部分
     invoke GetAddress,CTEXT("user32.dll"),CTEXT("CreateDesktopW")   
      mov addres\_CreateDesktopW,eax
       invoke  Hookapi, addres\_CreateDesktopW,addr new\_CreateDesktopW,addr old\_CreateDesktopW 
       mov size\_CreateDesktopW,eax
     
     
      ;you code
     
  .elseif   reason==DLL\_PROCESS\_DETACH 
     
  invoke WriteApi,addres\_CreateDesktopA,addr old\_CreateDesktopA,size\_CreateDesktopA
  invoke WriteApi,addres\_CreateDesktopW,addr old\_CreateDesktopW,size\_CreateDesktopW
      
      
      ; you code
      
 
   .endif 
    
    
    mov  eax,TRUE 
    ret 
DllEntry Endp 

GetMsgProc proc nCode:DWORD,wParam:DWORD,lParam:DWORD 
    invoke CallNextHookEx,hHook,nCode,wParam,lParam 
     mov eax,TRUE
     
      ret 
GetMsgProc endp 

InstallHook proc Hwnd:dword

         
    invoke SetWindowsHookEx,WH\_GETMESSAGE,addr GetMsgProc,hInstance,NULL 
     
    mov hHook,eax 
    ret 
InstallHook endp 

UninstallHook proc 
 invoke UnhookWindowsHookEx,hHook 
 
  ret 
UninstallHook endp 

new\_CreateDesktopA proc   uses ebx edi esi,as:DWORD , bs:DWORD,cd:DWORD , ps:DWORD,ys:DWORD , hs:DWORD

mov eax,TRUE 
ret

new\_CreateDesktopA endp

new\_CreateDesktopW proc   uses ebx edi esi,as:DWORD , bs:DWORD,cd:DWORD , ps:DWORD,ys:DWORD , hs:DWORD

mov eax,TRUE 
ret

new\_CreateDesktopW endp

   ; you code

End DllEntry
```

调用部分

```asm
.486 
.model flat,stdcall 
option casemap:none 

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Include 数据
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include windows.inc 
include kernel32.inc 
includelib kernel32.lib 
include user32.inc 
includelib user32.lib 
include   debug.inc

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  数据段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.data 

szMutex db "http://hi.baidu.com/nohacks",0

.data?

hInstance HINSTANCE ?  
CommandLine LPSTR ? 

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  代码段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code 

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

GetApi proc DllNameAddress:DWORD,ApiNameAddress:DWORD

invoke  GetModuleHandle,DllNameAddress     ;取 DLL 模块句柄
   
 .if eax==NULL
 
 invoke LoadLibrary ,DllNameAddress    ;加载 DLL
 
  .endif
 
invoke GetProcAddress,eax,ApiNameAddress  ;取 API 地址
   
mov eax,eax
 
ret

GetApi endp

whileStar PROC

LOCAL @stMsg:MSG

    .while  TRUE
      invoke  GetMessage,addr @stMsg,NULL,0,0
        
      .break  .if (!eax)
           
           invoke  TranslateMessage,addr @stMsg
      invoke  DispatchMessage,addr @stMsg
    .endw

     
ret


whileStar endp


;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  程序开始
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>


start:
  invoke GetModuleHandle,NULL   
   mov  hInstance,eax
   
  invoke GetCommandLine   
    
   mov CommandLine,eax                     
   
    invoke CreateMutex,NULL,FALSE,addr szMutex
     
        invoke GetLastError
             
      
       .IF eax == ERROR\_ALREADY\_EXISTS
         
                 
         invoke  ExitProcess,NULL
         
        .endif 
  
 invoke  GetApi,CTEXT("antidesktop.dll"),CTEXT("InstallHook")
     
    .if eax!=0
         call eax                  
   
    .endif
  
 invoke  GetApi,CTEXT("HideProcess.dll"),CTEXT("HideProcess")
     
    .if eax!=0
       
         call eax             ;隐藏进程
       
    .endif
        
invoke  whileStar  ;进入消息循环,直到收到退出消息
 
invoke  ExitProcess,NULL


;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

end  start
```

编译环境：`RADASM` + `MASM9.0`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `dll_hookapi.rar`
|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `AntiDesktop.rar`
|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `AntiDesktop(免疫所有虚拟桌面).rar` \[解压密码：`pediy`\] |
