
# [原创]`hook_exitwindowsex.asm`

```asm
;本程序不能阻挡调用NtShutdownSystem函数的关机。
;实现本功能还有别的办法：如：
;1.hook WM_QUERYENDSESSION 信息
;2.在驱动中拦截 IRP_MJ_SHUTDOWN
;3.修改函数的物理地址，ssdt等等。
;主程序如下：

.386
.model flat,stdcall
option casemap:none
include windows.inc
include kernel32.inc
include user32.inc
includelib user32.lib
includelib kernel32.lib
includelib dll.lib
InstallHook proto

.data?
hInstance dd ?
hWinMain dd ?
stWndClass WNDCLASSEX <>
stMsg MSG <>

.const
szClassName db "correy",0
szCaptionMain db "made by correy",0

.code
liuchunli proc uses ebx edi esi,hWnd,uMsg,wParam,lParam
.if uMsg == WM_CREATE
  call InstallHook
.elseif uMsg == WM_CLOSE
  invoke DestroyWindow,hWinMain
  invoke PostQuitMessage,NULL
.else
  invoke DefWindowProc,hWnd,uMsg,wParam,lParam
  ret
.endif
xor eax,eax
ret
liuchunli endp

start:
invoke GetModuleHandle,NULL
mov hInstance,eax
mov stWndClass.hInstance,eax
invoke LoadIcon,hInstance,1
mov stWndClass.hIcon,eax
mov stWndClass.cbSize,sizeof WNDCLASSEX
mov stWndClass.style,CS_HREDRAW or CS_VREDRAW
mov stWndClass.lpfnWndProc,offset liuchunli
mov stWndClass.hbrBackground,COLOR_WINDOW + 1
mov stWndClass.lpszClassName,offset szClassName
invoke RegisterClassEx,addr stWndClass
invoke CreateWindowEx,200h,offset szClassName,offset szCaptionMain,0Cf0000h,9,9,99h,99h,0,0,hInstance,0
mov hWinMain,eax
;invoke ShowWindow,hWinMain,1
invoke UpdateWindow,hWinMain
.while TRUE
invoke GetMessage,addr stMsg,0,0,0
.break .if eax == 0
invoke TranslateMessage,addr stMsg
invoke DispatchMessage,addr stMsg
.endw
invoke ExitProcess,NULL
end start
```

### DLL 文件

```asm
;不足之处，敬请指导。
;QQ:112426112
;Email:leguanyuan@126.com
;af2d9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2M7Y4u0W2P5g2)9J5k6i4N6W2j5Y4y4Q4x3X3g2U0L8$3@1`.

.386
.model flat,stdcall
option casemap:none
include windows.inc
include kernel32.inc
includelib kernel32.lib
include user32.inc
includelib user32.lib
hookapi proto :DWORD,:DWORD

.data
hInstance dd 0
hp dd 0
apiadd DWORD ?
oldapi db 6 dup(?)
szjmp db 068h;这三行构造push hookapi
x dd ?       ;ret 指令。
pretn db 0c3h
szdll db "user32.dll",0
szapi db "ExitWindowsEx",0
correy db "made by correy",0
notice db "你确认要注销/关机/重新启动吗？",0

.data?
hHook dd ?

.code
hookapi proc yy:DWORD,zz:DWORD
  invoke MessageBox,0,addr notice,addr correy,MB_YESNO
  .if eax==6
    invoke WriteProcessMemory,hp,apiadd,addr oldapi,6,0
    invoke ExitWindowsEx,yy,zz
  .endif
mov eax,0
ret
hookapi endp

hookproc proc nCode:DWORD,wParam:DWORD,lParam:DWORD
  invoke CallNextHookEx,hHook,nCode,wParam,lParam
  mov eax,TRUE
ret
hookproc endp

InstallHook proc
  invoke SetWindowsHookEx,WH_GETMESSAGE,addr hookproc,hInstance,0
  mov hHook,eax
  ret
InstallHook endp

UninstallHook proc
  invoke UnhookWindowsHookEx,hHook
  invoke WriteProcessMemory,hp,apiadd,addr oldapi,6,0
  ret
UninstallHook endp

DllEntry proc hInst:HINSTANCE, reason:DWORD,aa:DWORD
.if reason==DLL_PROCESS_ATTACH
  push hInst
  pop hInstance
  invoke GetCurrentProcess
  mov hp,eax
  invoke LoadLibrary,addr szdll
  invoke GetProcAddress,eax,addr szapi
  mov apiadd,eax
  invoke ReadProcessMemory,hp,apiadd,addr oldapi,6,0
  mov x,offset hookapi
  invoke WriteProcessMemory,hp,apiadd,addr szjmp,6,0
.endif
.if reason==DLL_PROCESS_DETACH
  invoke WriteProcessMemory,hp,apiadd,addr oldapi,6,0
.endif
mov eax,TRUE
ret
DllEntry Endp
End DllEntry
;made at 2010.10.15
```

### DEF 文件

```asm
LIBRARY dll

EXPORTS InstallHook
```

### RC 文件

```asm
1 ICON  "me.ico"

1 VERSIONINFO
FILEVERSION 0,0,0,0
PRODUCTVERSION 0,0,0,0
FILEOS 0
FILETYPE 0
FILESUBTYPE 0
{
  BLOCK "StringFileInfo"
   {
    BLOCK "040904E4"
     {
     VALUE "Comments", "备注"
     VALUE "CompanyName", "乐观集团"
     VALUE "FileVersion", "文件版本"
     VALUE "FileDescription", "made by correy"  //文件描述
     VALUE "InternalName", "内部名称"
     VALUE "LegalCopyright", "合法版权"
     VALUE "LegalTrademarks", "合法商标"
     VALUE "OriginalFilename", "源文件名"
     VALUE "ProductName", "产品名称"
     VALUE "ProductVersion", "产品版本"
     VALUE "PrivateBuild", "个人用内部版本说明"
     VALUE "SpecialBuild", "特殊内部版本说明"
     }
    }
  BLOCK "VarFileInfo"
  {
   VALUE "Translation", 2052, 1200
  }
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`3.rar`](javascript:void(0);)

  （5.50kb，63次下载）
