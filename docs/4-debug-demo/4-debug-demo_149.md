
# \[OCN\]Keygen 模板 win32asm 程序

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-676.htm`](https://www.pediy.com/kssd/pediy08/pediy8-676.htm)

附件下载：[\[ocn\]keygen.rar](pediy8-676/keygen.rar)

```asm
;###########################################################

.586

.model flat,stdcall

option casemap:none

include \\masm32\\include\\windows.inc

include \\masm32\\include\\user32.inc

include \\masm32\\include\\kernel32.inc

include \\masm32\\include\\comctl32.inc

include \\masm32\\include\\shell32.inc

include minifmod.inc

includelib \\masm32\\lib\\user32.lib

includelib \\masm32\\lib\\kernel32.lib

includelib \\masm32\\lib\\comctl32.lib

includelib \\masm32\\lib\\shell32.lib

includelib minifmod.lib

;###########################################################

;%%%%%%%%%%%%%%%%%%%%
; 对话框过程定义
;%%%%%%%%%%%%%%%%%%%%

DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD

;%%%%%%%%%%%%%%%%%%%%
; 常数定义
;%%%%%%%%%%%%%%%%%%%%

.const

DLG_MAIN equ 102

DLG_ABOUT equ 104

IDI_ICON equ 103

IDM_MUSIC equ 101

IDC_REG equ 1001

IDC_ABOUT equ 1003

IDC_WEB1 equ 1010

IDC_WEB2 equ 1011

IDC_EMAIL equ 1012

IDC_OK equ 1013

IDE_NAME equ 1000

IDE_SN equ 1002

;%%%%%%%%%%%%%%%%%%%%
; 变量定义
;%%%%%%%%%%%%%%%%%%%%

.data

handleIcon dd 0

handleEdit dd 0

szFormat db "%1d",0

szOpen db "open", 0

szSoftAdd db "http://spaces.msn.com/moodsky/",0 ; 在这里定义 IDC_WEB2

szOCNadd db "http://www.chinaocn.net",0 ; 在这里定义 IDC_WEB2

szEmail db "mailto:moodsky@eyou.com",0 ; 在这里定义 IDC_EMAIL

szNameBuffer db 50 dup(0)

szSnBuffer db 50 dup(0)

szBuffer db 100 dup(0)

.data?

hInstance HINSTANCE ?

CommandLine LPSTR ?

pMusic LPVOID ?

nMusicSize DWORD ?

;###########################################################

.code

start:

    ;%%%%%%%%%%%%%%%%%%%%
    ; 建立对话框
    ;%%%%%%%%%%%%%%%%%%%%

    invoke InitCommonControls ; 初始化自定义控制库

    invoke GetModuleHandle,NULL ; 获取当前模块句柄

    mov hInstance,eax

    push esi

    invoke FindResource, hInstance, IDM_MUSIC, RT_RCDATA

    push eax

    invoke SizeofResource, hInstance, eax

    mov nMusicSize, eax

    pop eax

    invoke LoadResource, hInstance, eax

    invoke LockResource, eax

    mov esi, eax

    mov eax, nMusicSize

    add eax, SIZEOF nMusicSize

    invoke GlobalAlloc, GPTR, eax

    mov pMusic, eax

    mov ecx, nMusicSize

    mov dword ptr [eax], ecx

    add eax, SIZEOF nMusicSize

    mov edi, eax

    rep movsb

    pop esi

    invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,OFFSET DlgProc,FALSE ; 显示定义的对话框

    invoke GlobalFree, pMusic

    invoke ExitProcess,NULL

;%%%%%%%%%%%%%%%%%%%%
; 关于对话框过程
;%%%%%%%%%%%%%%%%%%%%

AboutProc proc hAWin:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM

    .IF uMsg==WM_CLOSE

    invoke EndDialog,hAWin,NULL

    ret

    .ELSEIF uMsg==WM_INITDIALOG

        ;%%%%%%%%%%%%%%%%%%%%
        ; 对话框初始化消息
        ; 设置标题栏的图标
        ;%%%%%%%%%%%%%%%%%%%%

        invoke SetWindowPos,hAWin,HWND_TOPMOST,0,0,0,0,SWP_NOSIZE or SWP_NOMOVE

    .ELSEIF uMsg==WM_COMMAND

        ;%%%%%%%%%%%%%%%%%%%%
        ; 按钮消息处理
        ;%%%%%%%%%%%%%%%%%%%%

    mov eax,wParam

    .IF ax == IDC_WEB1

    ;%%%%%%%%%%%%%%%%%%%%
    ; 软件下载地址按钮
    ;%%%%%%%%%%%%%%%%%%%%

      invoke ShellExecute,NULL,OFFSET szOpen,OFFSET szSoftAdd,NULL,NULL,SW_SHOWNORMAL

    .ELSEIF ax == IDC_WEB2

    ;%%%%%%%%%%%%%%%%%%%%
    ; 我的论坛地址按钮
    ;%%%%%%%%%%%%%%%%%%%%

      invoke ShellExecute,NULL,OFFSET szOpen,OFFSET szOCNadd,NULL,NULL,SW_SHOWNORMAL

        .ELSEIF ax == IDC_EMAIL

    ;%%%%%%%%%%%%%%%%%%%%
    ; 邮件地址按钮
    ;%%%%%%%%%%%%%%%%%%%%

      invoke ShellExecute,NULL,OFFSET szOpen,OFFSET szEmail,NULL,NULL,SW_SHOWNORMAL

        .ELSEIF ax == IDC_OK

    ;%%%%%%%%%%%%%%%%%%%%
    ; OK 按钮
    ;%%%%%%%%%%%%%%%%%%%%

        invoke EndDialog,hAWin,NULL

      ret

    .ENDIF

    .ELSE

    mov eax,FALSE

    ret

    .ENDIF

    mov eax,TRUE

    ret

AboutProc endp

;%%%%%%%%%%%%%%%%%%%%
; 主对话框过程
;%%%%%%%%%%%%%%%%%%%%

DlgProc proc hWin:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM

    mov eax,uMsg

    .IF eax == WM_CLOSE

        ;%%%%%%%%%%%%%%%%%%%%
        ; 点击"x"按钮
        ;%%%%%%%%%%%%%%%%%%%%

    invoke mfmPlay, 0

    invoke EndDialog,hWin,NULL

    .ELSEIF eax == WM_INITDIALOG

        ;%%%%%%%%%%%%%%%%%%%%
        ; 对话框初始化消息
        ; 设置标题栏的图标
        ;%%%%%%%%%%%%%%%%%%%%

        invoke LoadIcon,hInstance,IDI_ICON

        mov handleIcon, eax

        invoke SendMessage,hWin,WM_SETICON,1,handleIcon

        invoke SetWindowPos,hWin,HWND_TOPMOST,0,0,0,0,SWP_NOSIZE or SWP_NOMOVE

    xor eax,eax

        invoke mfmPlay, pMusic

        invoke SetFocus, eax

    .ELSEIF eax == WM_COMMAND

        ;%%%%%%%%%%%%%%%%%%%%
        ; 按钮消息处理
        ;%%%%%%%%%%%%%%%%%%%%

        mov eax,wParam

        .IF eax == IDC_REG

            ;%%%%%%%%%%%%%%%%%%%%
            ; 点击"注册"按钮
            ;%%%%%%%%%%%%%%%%%%%%

            .IF

      .ELSE

      .ENDIF

    .ELSEIF eax == IDC_ABOUT

      ;%%%%%%%%%%%%%%%%%%%%
    ; 点击"关于"按钮
    ;%%%%%%%%%%%%%%%%%%%%

      invoke CreateDialogParam,hInstance,DLG_ABOUT,hWin,offset AboutProc,FALSE

      invoke EnableWindow,hWin,0

    .ENDIF

    .ELSE

    mov eax,FALSE

    ret

    .ENDIF

    mov eax,TRUE

    ret

DlgProc endp

end start

;###########################################################
```

4dd466d0e1e0cc123afbe504da2462ca
