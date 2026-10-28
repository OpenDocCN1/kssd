
# [原创]AutoRun病毒的U盘检测部分实现

标 题: 【原创】AutoRun病毒的U盘检测部分实现

作 者: `lonkil`

时 间: 2008-06-24

链 接: http://bbs.pediy.com/showthread.php?p=471393

这是小弟汇编习作，请高手飘过，给像我这等这些新手分享一下。如有错误之处请跟帖，谢谢。![](https://bbs.kanxue.com/view/img/face/1.gif)

本程序主要实现AutoRun病毒的U盘检测部分。由于底层接口MS已经帮我们实现好，我们上层实现起来就容易多了。主要通过在窗口回调中检测`WM_DEVICECHANGE`消息，然后再进一步确认状态。最后再判定是不是移动存储设备。

做这个小程序有一点需要注意的就是，有几个`struct`类型如`DEV_BROADCAST_HDR`、`DEV_BROADCAST_VOLUME`以及 `DBT_DEVICEARRIVAL`，`DBT_DEVICEREMOVECOMPLETE`，`DBT_DEVTYP_VOLUME`，`masm`的头文件中没有。参考一下VC中的相应的.h文件，在文件头定义一下就可以了。其它就没什么了，看代码吧。

``` 
;功能：实现AutoRun病毒的U盘检测部分和写文件部分。

;作者: Lonkil

;网站：46bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6U0k6X3q4F1M7#2)9J5k6h3y4G2L8b7`.`.

;邮箱：lonkil_At_gmail.com

;日期：2008-6-24

;参考资料：MSDN

;转载请保存文章的完整性，谢谢。

;=========================================================

.386

.model flat,stdcall

option casemap:none



include windows.inc

include gdi32.inc

include user32.inc

include kernel32.inc



includelib  gdi32.lib

includelib    user32.lib

includelib    kernel32.lib





DEV_BROADCAST_HDR STRUCT

    dbch_size        DWORD            ?

    dbch_devicetype    DWORD        ?

    dbch_reserved        DWORD        ?

DEV_BROADCAST_HDR ENDS





DEV_BROADCAST_VOLUME STRUCT

    dbcv_size                DWORD            ?

    dbcv_devicetype        DWORD            ?

    dbcv_reserved            DWORD            ?

    dbcv_unitmask            DWORD            ?

    dbcv_flags                DWORD            ?

DEV_BROADCAST_VOLUME ENDS



DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD;申明窗口回调函数

.const 

IDD_UDISK        equ    100

IDC_INFO            equ    101

IDC_CLOSE        equ    102

DBT_DEVICEARRIVAL        equ 8000h

DBT_DEVICEREMOVECOMPLETE equ 8004h

DBT_DEVTYP_VOLUME             equ    2h

.data

szUdiskInserted    db    "检测到U盘已插入。",0

szWriteFile            db    "病毒已写入^-^,打开你的U盘看看。",0

szFileName            db "AutoRun.ini",0

szRemoveDisk        db    "已检测到U盘被拔出了。",0

szMsgTitle            db    "Site:Www.VcFans.Com",0Dh,0Ah

                        db "Author:Lonkil",0Dh,0Ah

                        db "2008-6-24",0

szDrivePath                db "%c:\\%s",0

.data?

hInstance    dd    ?

szBuf            db 16 dup(?)



.code

    ;写入一个文件到刚插入的U盘

    _WriteFile proc

        local dwWrite:DWORD

        pushad

        invoke CreateFile,offset szBuf,GENERIC_ALL,FILE_SHARE_READ or FILE_SHARE_WRITE, NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL

        mov ebx,eax

        .if ebx != -1

        invoke lstrlen,offset szMsgTitle

        invoke WriteFile,ebx,offset szMsgTitle,eax,dwWrite,0

        .endif

        invoke CloseHandle,ebx

        popad

        ret

    _WriteFile endp

    ;根据输入的掩码得到卷标

    GetVolumeName proc flag:DWORD

        push esi

        push ecx

        mov ecx, 26

        mov esi,flag

        @@:

        test esi,1h

        jnz @f

        shr esi,1h

        dec    ecx

        test ecx,ecx

        jnz    @b

        @@:

        mov eax,5Bh

        sub eax,ecx

        pop ecx

        pop esi

        ret

    GetVolumeName endp

    

    CenterWindow proc hwnd:HWND

        local nScreenW :DWORD;屏幕宽

        local nScreenH :DWORD;屏幕高

        local rtWindow :RECT;窗口大小

        local nX:DWORD

        local nY:DWORD

        

        invoke GetSystemMetrics, SM_CXSCREEN

        mov nScreenW,eax

        invoke GetSystemMetrics, SM_CYSCREEN

        mov nScreenH,eax

        invoke GetWindowRect,hwnd,addr rtWindow

        mov ebx,nScreenW

        mov ecx,rtWindow.right

        sub ecx,rtWindow.left

        sub ebx,ecx;x坐标

        mov nX,ebx

        push ecx;将宽压栈

        mov eax,nScreenH

        mov edx,rtWindow.bottom

        sub edx,rtWindow.top

        sub eax,edx;y坐标

        mov nY,eax

        push edx;将高压栈

        mov ecx,2

        xor edx,edx

        mov eax,nX

        div ecx

        mov nX, eax

        xor edx,edx

        mov eax,nY

        div ecx

        mov nY,eax

        pop edx;高

        pop ecx;宽

        invoke MoveWindow, hwnd,nX,nY,ecx,edx,TRUE

        ;将对框移动屏幕中央

        ret

    CenterWindow endp

    ;初始化窗口函数

    

    

    DlgProc proc hWnd,Umsg,wParam,lParam;窗口回调函数程序实现

        local dbt_h : DEV_BROADCAST_HDR

        local dbt_v    : DEV_BROADCAST_VOLUME

        .if Umsg == WM_COMMAND

            mov eax,wParam

            and eax,0ffffh

            .if eax == IDC_CLOSE

                invoke PostMessage,hWnd,WM_CLOSE,0,0

            .endif

        .elseif Umsg == WM_INITDIALOG

            invoke CenterWindow,hWnd

        .elseif Umsg == WM_DEVICECHANGE

            .if wParam == DBT_DEVICEARRIVAL;U盘已插入

                mov    ebx, lParam

                assume ebx:ptr DEV_BROADCAST_HDR

                .if [ebx].dbch_devicetype == DBT_DEVTYP_VOLUME ;检测是不是磁盘类USB设备

                    invoke SetDlgItemText,hWnd, IDC_INFO,offset szUdiskInserted

                    mov edx,lParam

                    assume edx:ptr DEV_BROADCAST_VOLUME

                    invoke GetVolumeName,[edx].dbcv_unitmask

                    push eax

                    invoke wsprintf,offset szBuf,offset szDrivePath,eax,0

                    invoke GetDriveType,offset szBuf

                    .if eax == DRIVE_REMOVABLE;检测是不是硬动磁盘设备

                        invoke Sleep, 1500

                        pop eax

                        invoke wsprintf,offset szBuf,offset szDrivePath,eax,offset szFileName

                        invoke _WriteFile

                        invoke SetDlgItemText,hWnd, IDC_INFO,offset szWriteFile

                    .endif

                    assume edx:nothing

                .endif

                assume ebx:ptr nothing

            .elseif wParam == DBT_DEVICEREMOVECOMPLETE;U盘已拔出

                invoke SetDlgItemText,hWnd, IDC_INFO,offset szRemoveDisk

            .endif

        .elseif Umsg == WM_CLOSE

                invoke EndDialog,hWnd,wParam             

        .else

        mov eax,FALSE

        ret

        .endif

        mov eax,TRUE

        ret

    DlgProc EndP

    

start:

    invoke    GetModuleHandle,NULL

    mov    hInstance,eax

    invoke DialogBoxParam,hInstance, IDD_UDISK, NULL, offset DlgProc, NULL

    invoke ExitProcess,NULL

end start
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [UDisk.zip](javascript:void(0);)

  （2.29kb，16次下载）
