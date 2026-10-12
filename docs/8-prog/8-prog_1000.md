
# 托盘图标伪装大师（超级防锁专家配套程序）

> 原文：[`www.pediy.com/kssd/pediy10/70762.html`](https://www.pediy.com/kssd/pediy10/70762.html)

**标题**: 【原创】托盘图标伪装大师(汇编)
**作者**: 非安全(nohacks)
**时间**: 2008-08-15,17:05
**链接**: `http://bbs.pediy.com/showthread.php?t=70762`

这篇文章是“[多桌面切换程序-通杀所有网管程序](http://bbs.pediy.com/showthread.php?t=55658)”的延续，上篇文章我们提到了用虚拟桌面的方法绕过网管软件的方法，但还有个小缺陷，新桌面的任务栏上没有网管程序的图标，较易被路过的网管发现。这不上班闲着无聊，用汇编写了个小软件：托盘图标伪装大师 1.0，解决了这个问题，虽然没什么技术含量，但作为一种思路，还是值得一看的，主要是 API 函数 `LoadImage` 和 `GetPrivateProfileString` 的使用，前者用来装载图标文件，后者用来读取配置文件。程序主要功能如下：

读取配置文件 `config.ini`，在托盘区显示配置文件指定的程序图标。

```ini
[SETUP]
ICO="WX2004.ICO"
```

程序代码如下：

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Programmed by nohacks, nohacks@163.com
;  Website: http://hi.baidu.com/nohacks
;            Win32 ASM is Masm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  版本信息
;  Icon camouflage Masters V1.0   - 托盘图标伪装大师
;    
;                      2008 年 8 月 15 日
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.386 
.model flat,stdcall 
option casemap:none 
include windows.inc 
include debug.inc
include user32.inc 
include kernel32.inc 
include shell32.inc 
includelib user32.lib 
includelib kernel32.lib 
includelib shell32.lib   
WM\_SHELLNOTIFY equ WM\_USER+5 
ICO\_INDEX equ 1200
IDI\_TRAY equ 0 

WinMain proto :DWORD,:DWORD,:DWORD,:DWORD 

.DATA             
ClassName db "SimpleWinClass",0       
AppName db "nohacks",0 
Section db "SETUP",0
keyname db "ICO",0
ininame db  ".\config.ini",0
mypath  db  "wx2004.ico",0

 
.DATA?        

hInstance dd ? 
note NOTIFYICONDATA <> 
CommandLine LPSTR ? 

buffer db 512 dup(?)

fileErr dd ?


.CODE        
start: 
invoke GetModuleHandle, NULL          
                                     
mov hInstance,eax 

invoke GetCommandLine

mov CommandLine,eax

invoke  GetPrivateProfileString,addr Section,addr keyname,addr mypath,addr buffer ,512,addr ininame

invoke GetFileAttributes,addr buffer  ;快速判断文件是否存在
   
  mov fileErr,eax
         
invoke WinMain, hInstance,NULL,CommandLine, SW\_SHOWDEFAULT     
invoke ExitProcess, eax                 

WinMain proc hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD 
    LOCAL wc:WNDCLASSEX                      
    LOCAL msg:MSG 
    LOCAL hwnd:HWND 

    mov   wc.cbSize,SIZEOF WNDCLASSEX         
    mov   wc.style, CS\_HREDRAW or CS\_VREDRAW 
    mov   wc.lpfnWndProc, OFFSET WndProc 
    mov   wc.cbClsExtra,NULL 
    mov   wc.cbWndExtra,NULL 
    push  hInstance 
    pop   wc.hInstance 
    mov   wc.hbrBackground,COLOR\_WINDOW+1 
    mov   wc.lpszMenuName,NULL 
    mov   wc.lpszClassName,OFFSET ClassName 
            
    invoke LoadIcon,NULL,NULL
       
    mov   wc.hIcon,eax 
    mov   wc.hIconSm,eax 
    invoke LoadCursor,NULL,IDC\_ARROW 
       
    mov   wc.hCursor,eax 
    invoke RegisterClassEx, addr wc             
    invoke CreateWindowEx,NULL,\
         ADDR ClassName,\
         ADDR AppName,\
         WS\_OVERLAPPEDWINDOW,\
         CW\_USEDEFAULT,\
         CW\_USEDEFAULT,\
         CW\_USEDEFAULT,\
         CW\_USEDEFAULT,\
         NULL,\
         NULL,\
         hInst,\
         NULL 
    mov   hwnd,eax 
   ; invoke ShowWindow, hwnd,CmdShow       
   ;invoke UpdateWindow, hwnd               

    .WHILE TRUE                    
            invoke GetMessage, ADDR msg,NULL,0,0 
            .BREAK .IF (!eax) 
            invoke TranslateMessage, ADDR msg 
            invoke DispatchMessage, ADDR msg 
   .ENDW 
    mov     eax,msg.wParam               
    ret 
WinMain endp 

WndProc proc hWnd:HWND, wMsg:UINT, wParam:WPARAM, lParam:LPARAM 

 
         mov  eax,wMsg


  cmp  eax,WM\_CREATE             
  je  boxStart
  cmp  eax,WM\_CLOSE
  je  boxClose  
   
       invoke DefWindowProc,hWnd,wMsg,wParam,lParam     
        ret
     

    boxStart:
     
            mov note.cbSize,sizeof NOTIFYICONDATA 
            push hWnd 
            pop note.hwnd
            mov note.uID,IDI\_TRAY 
            mov note.uFlags,NIF\_ICON+NIF\_MESSAGE+NIF\_TIP 
            

            mov note.uCallbackMessage,WM\_SHELLNOTIFY
               
        
        .if fileErr!=-1
           
           
        invoke LoadImage,hInstance,addr buffer,IMAGE\_ICON,0,0,LR\_LOADFROMFILE ;载入图标文件
       
        .else
           
          invoke LoadIcon,hInstance,ICO\_INDEX
                         
        .endif
                              
            
            mov note.hIcon,eax 
            invoke lstrcpy,addr note.szTip,NULL   ;这里可以设置托盘文字提示，本处留空
           
            invoke ShowWindow,hWnd,SW\_HIDE 
            invoke Shell\_NotifyIcon,NIM\_ADD,addr note 

            jmp return
    boxClose:
         invoke Shell\_NotifyIcon,NIM\_DELETE,addr note  ;删除托盘图标
         
          invoke PostQuitMessage,NULL    
           
           jmp return
           
   return:

    xor eax,eax 
    ret 
     
WndProc endp 

end start 
```

开发环境：RadASM + MASM 最新版

声明：本文原创于看雪软件安全论坛(bbs.pediy.com)，转载请注明出处！

PS:搞木马或在网吧工作的兄弟有福了，比如干掉了某杀软或者切换到了虚拟桌面，用这个程序模拟杀软或网管软件的托盘图标可迷惑网管，你只要提取目标程序的图标放在程序目录，然后修改配置文件即可！

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || writeico.rar  
|
