

# AutoRun 病毒的 U 盘检测部分实现

> 原文：[`www.pediy.com/kssd/pediy10/67212.html`](https://www.pediy.com/kssd/pediy10/67212.html)

标 题: 【原创】AutoRun 病毒的 U 盘检测部分实现  
作 者: lonkil  
时 间: 2008-06-24  
链 接: http://bbs.pediy.com/showthread.php?p=471393  
  
这是小弟汇编习作，请高手飘过，给像我这等这些新手分享一下。如有错误之处请跟帖，谢谢。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")   
  
本程序主要实现 AutoRun 病毒的 U 盘检测部分。由于底层接口 MS 已经帮我们实现好，我们上层实现起来就容易多了。主要通过在窗口回调中检测 WM\_DEVICECHANGE 消息，然后再进一步确认状态。最后再判定是不是移动存储设备。  
  
做这个小程序有一点需要注意的就是，有几个 struct 类型如 DEV\_BROADCAST\_HDR、DEV\_BROADCAST\_VOLUME 以及 DBT\_DEVICEARRIVAL，DBT\_DEVICEREMOVECOMPLETE，DBT\_DEVTYP\_VOLUME，masm 的头文件中没有。参考一下 VC 中的相应的.h 文件，在文件头定义一下就可以了。其它就没什么了，看代码吧。  
  
  
  

代码:

```
;功能：实现 AutoRun 病毒的 U 盘检测部分和写文件部分。
;作者: Lonkil
;网站：www.vcfans.com
;邮箱：lonkil\_At\_gmail.com
;日期：2008-6-24
;参考资料：MSDN
;转载请保存文章的完整性，谢谢。
;=========================================================
.386
.model flat,stdcall
option casemap:none

include windows.inc
include gdi32.inc
include user32.inc
include kernel32.inc

includelib  gdi32.lib
includelib    user32.lib
includelib    kernel32.lib


DEV\_BROADCAST\_HDR STRUCT
    dbch\_size        DWORD            ?
    dbch\_devicetype    DWORD        ?
    dbch\_reserved        DWORD        ?
DEV\_BROADCAST\_HDR ENDS


DEV\_BROADCAST\_VOLUME STRUCT
    dbcv\_size                DWORD            ?
    dbcv\_devicetype        DWORD            ?
    dbcv\_reserved            DWORD            ?
    dbcv\_unitmask            DWORD            ?
    dbcv\_flags                DWORD            ?
DEV\_BROADCAST\_VOLUME ENDS

DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD;申明窗口回调函数
.const 
IDD\_UDISK        equ    100
IDC\_INFO            equ    101
IDC\_CLOSE        equ    102
DBT\_DEVICEARRIVAL        equ 8000h
DBT\_DEVICEREMOVECOMPLETE equ 8004h
DBT\_DEVTYP\_VOLUME             equ    2h
.data
szUdiskInserted    db    "检测到 U 盘已插入。",0
szWriteFile            db    "病毒已写入^-^,打开你的 U 盘看看。",0
szFileName            db "AutoRun.ini",0
szRemoveDisk        db    "已检测到 U 盘被拔出了。",0
szMsgTitle            db    "Site:Www.VcFans.Com",0Dh,0Ah
                        db "Author:Lonkil",0Dh,0Ah
                        db "2008-6-24",0
szDrivePath                db "%c:\\\\%s",0
.data?
hInstance    dd    ?
szBuf            db 16 dup(?)

.code
    ;写入一个文件到刚插入的 U 盘
    \_WriteFile proc
        local dwWrite:DWORD
        pushad
        invoke CreateFile,offset szBuf,GENERIC\_ALL,FILE\_SHARE\_READ or FILE\_SHARE\_WRITE, NULL,CREATE\_ALWAYS,FILE\_ATTRIBUTE\_NORMAL,NULL
        mov ebx,eax
        .if ebx != -1
        invoke lstrlen,offset szMsgTitle
        invoke WriteFile,ebx,offset szMsgTitle,eax,dwWrite,0
        .endif
        invoke CloseHandle,ebx
        popad
        ret
    \_WriteFile endp
    ;根据输入的掩码得到卷标
    GetVolumeName proc flag:DWORD
        push esi
        push ecx
        mov ecx, 26
        mov esi,flag
        @@:
        test esi,1h
        jnz @f
        shr esi,1h
        dec    ecx
        test ecx,ecx
        jnz    @b
        @@:
        mov eax,5Bh
        sub eax,ecx
        pop ecx
        pop esi
        ret
    GetVolumeName endp
    
    CenterWindow proc hwnd:HWND
        local nScreenW :DWORD;屏幕宽
        local nScreenH :DWORD;屏幕高
        local rtWindow :RECT;窗口大小
        local nX:DWORD
        local nY:DWORD
        
        invoke GetSystemMetrics, SM\_CXSCREEN
        mov nScreenW,eax
        invoke GetSystemMetrics, SM\_CYSCREEN
        mov nScreenH,eax
        invoke GetWindowRect,hwnd,addr rtWindow
        mov ebx,nScreenW
        mov ecx,rtWindow.right
        sub ecx,rtWindow.left
        sub ebx,ecx;x 坐标
        mov nX,ebx
        push ecx;将宽压栈
        mov eax,nScreenH
        mov edx,rtWindow.bottom
        sub edx,rtWindow.top
        sub eax,edx;y 坐标
        mov nY,eax
        push edx;将高压栈
        mov ecx,2
        xor edx,edx
        mov eax,nX
        div ecx
        mov nX, eax
        xor edx,edx
        mov eax,nY
        div ecx
        mov nY,eax
        pop edx;高
        pop ecx;宽
        invoke MoveWindow, hwnd,nX,nY,ecx,edx,TRUE
        ;将对框移动屏幕中央
        ret
    CenterWindow endp
    ;初始化窗口函数
    
    
    DlgProc proc hWnd,Umsg,wParam,lParam;窗口回调函数程序实现
        local dbt\_h : DEV\_BROADCAST\_HDR
        local dbt\_v    : DEV\_BROADCAST\_VOLUME
        .if Umsg == WM\_COMMAND
            mov eax,wParam
            and eax,0ffffh
            .if eax == IDC\_CLOSE
                invoke PostMessage,hWnd,WM\_CLOSE,0,0
            .endif
        .elseif Umsg == WM\_INITDIALOG
            invoke CenterWindow,hWnd
        .elseif Umsg == WM\_DEVICECHANGE
            .if wParam == DBT\_DEVICEARRIVAL;U 盘已插入
                mov    ebx, lParam
                assume ebx:ptr DEV\_BROADCAST\_HDR
                .if \[ebx\].dbch\_devicetype == DBT\_DEVTYP\_VOLUME ;检测是不是磁盘类 USB 设备
                    invoke SetDlgItemText,hWnd, IDC\_INFO,offset szUdiskInserted
                    mov edx,lParam
                    assume edx:ptr DEV\_BROADCAST\_VOLUME
                    invoke GetVolumeName,\[edx\].dbcv\_unitmask
                    push eax
                    invoke wsprintf,offset szBuf,offset szDrivePath,eax,0
                    invoke GetDriveType,offset szBuf
                    .if eax == DRIVE\_REMOVABLE;检测是不是硬动磁盘设备
                        invoke Sleep, 1500
                        pop eax
                        invoke wsprintf,offset szBuf,offset szDrivePath,eax,offset szFileName
                        invoke \_WriteFile
                        invoke SetDlgItemText,hWnd, IDC\_INFO,offset szWriteFile
                    .endif
                    assume edx:nothing
                .endif
                assume ebx:ptr nothing
            .elseif wParam == DBT\_DEVICEREMOVECOMPLETE;U 盘已拔出
                invoke SetDlgItemText,hWnd, IDC\_INFO,offset szRemoveDisk
            .endif
        .elseif Umsg == WM\_CLOSE
                invoke EndDialog,hWnd,wParam             
        .else
        mov eax,FALSE
        ret
        .endif
        mov eax,TRUE
        ret
    DlgProc EndP
    
start:
    invoke    GetModuleHandle,NULL
    mov    hInstance,eax
    invoke DialogBoxParam,hInstance, IDD\_UDISK, NULL, offset DlgProc, NULL
    invoke ExitProcess,NULL
end start

```

上传的附件
| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || UDisk.zip  
 |