
# 突发奇想！让 Windows 不再 Windows

> 原文：[`www.pediy.com/kssd/pediy10/84639.html`](https://www.pediy.com/kssd/pediy10/84639.html)

希望不要随便利用：汇编代码（附件是演示的程序，最多两分钟后会安全返回）

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```asm
.386  

.model flat,stdcall  

option casemap:none  

;****************************************  

include  windows.inc  

include  kernel32.inc  

includelib  kernel32.lib  

include  user32.inc  

includelib  user32.lib  

;****************************************  

.data?  

hwnd dd ?  

hwdesk dd ?  

mesg  MSG <>  

.const  

szClassName  db  'Progman',0  

szCaptionMain  db  'Program Manager',0  

;****************************************  

.code  

start:  

   invoke FindWindow,addr szClassName,addr szCaptionMain  

   mov hwdesk,eax  

   invoke SetTimer,NULL,1,500,0  

   .while 1  

   invoke GetMessage,addr mesg,NULL,0,0  

    .if mesg.message==WM_TIMER  

    invoke GetForegroundWindow  

     .if eax!=hwdesk  

       mov hwnd,eax  

       invoke IsWindowVisible, hwnd       


         .if eax  

      invoke ShowWindow,hwnd,SW_HIDE  

         .endif  

     .endif  

    .endif  

    .endw  

    invoke KillTimer,NULL,1  

    invoke ExitProcess,NULL  

 end start  

 ;****************************************
```

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` || `nowindow.rar` \[解压密码：pediy\]   

 |
