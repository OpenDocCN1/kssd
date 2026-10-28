
# [旧帖][原创]自己用汇编写的网页刷新器0.00雪花

闷 我太粗心了 改了下

```
.386

.model flat, stdcall

option casemap :none

include                windows.inc

include         shell32.inc

includelib      shell32.lib

include                user32.inc

includelib        user32.lib

include                kernel32.inc

includelib        kernel32.lib

include         comdlg32.inc

includelib      comdlg32.lib

ICO_MAIN        equ                1000h        ;图标

DLG_MAIN        equ                1

IDC_TIME        equ               2

IDC_WEB         equ                 3

IDC_END         equ                4

IDC_TIMER1       equ             5

IDC_ON          equ            100

IDC_OFF          equ          101

                .data?

hInstance       dd           ?

dwTime          dd           ?

hIDOK           dd           ?

szBuffer       db    256 dup(?)

szBuff         db    256 dup(?)

IDC_FLAGS       dd      ?

Every           dd      ?

hIDC_WEB       dd      ?

hIDC_TIME      dd      ?

               .const

sz1            dd         'wo',0

               .code

_ProcDlg       proc     uses   ebx edi esi hWnd,wMsg,wParam,lParam

                        mov     eax,wMsg

                        .if     eax == WM_TIMER

                                mov eax,Every

                                xchg dwTime,esi

                                inc esi

                                xchg dwTime,esi

                                cmp eax,dwTime

                                jne  over

                                je  baidu

                          baidu:

                                      invoke ShellExecute,NULL,NULL ,addr szBuff,NULL,NULL,SW_SHOW

                                      mov  eax,30h

                                      mov  dwTime,eax

                         over:   xor eax,eax

                        .elseif     eax ==   WM_CLOSE

                                invoke   ExitProcess,NULL

                        .elseif eax ==   WM_INITDIALOG

                                mov      esi,30h

                                mov      dwTime,esi

                                invoke   LoadIcon,hInstance,ICO_MAIN

                                invoke   SendMessage,hWnd,WM_SETICON,ICON_BIG,eax

                        .elseif eax ==   WM_COMMAND

                                mov      eax,wParam

                                .if      ax ==  IDC_END

                                         invoke       EnableWindow,hIDOK,TRUE

                                         invoke  EnableWindow,hIDC_TIME,TRUE

                                          invoke  EnableWindow,hIDC_WEB,TRUE

                                         mov     eax,IDC_OFF

                                         mov     IDC_FLAGS,eax

                                         mov     eax,30h

                                           mov     dwTime,eax

                                         invoke  KillTimer,hWnd,IDC_TIMER1

                                .elseif  ax ==   IDOK

                                         .if   IDC_FLAGS==IDC_ON

                                               xor  eax,eax

                                               jmp  last

                                            .endif

                                           mov     ebx,IDC_ON

                                           mov     IDC_FLAGS,ebx

                                         invoke  GetDlgItemText,hWnd,IDC_TIME,addr szBuffer,sizeof szBuffer

                                         invoke  GetDlgItemText,hWnd,IDC_WEB,addr szBuff,sizeof szBuff

                                         mov     eax,dword ptr szBuffer

                                        \

                                         mov     Every,eax

                                         invoke  SetTimer,hWnd,IDC_TIMER1,60000,NULL

                                         invoke  GetDlgItem,hWnd,IDOK

                                         mov     hIDOK,eax

                                         invoke  EnableWindow,hIDOK,FALSE

                                         invoke  GetDlgItem,hWnd,IDC_TIME

                                         mov     hIDC_TIME,eax

                                         invoke  EnableWindow,hIDC_TIME,FALSE

                                         invoke  GetDlgItem,hWnd,IDC_WEB

                                         mov     hIDC_WEB,eax

                                         invoke  EnableWindow,hIDC_WEB,FALSE

                                         invoke ShellExecute,NULL,NULL ,addr szBuff,NULL,NULL,SW_SHOW

                                .endif

                                .else

                          last:

                                   mov        eax,FALSE

                        ret

                .endif

                mov        eax,TRUE

                ret

_ProcDlg endp

start:         invoke   GetModuleHandle,NULL

               mov      hInstance,eax

               invoke   DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlg,NULL

               invoke   ExitProcess,NULL

               end      start
```

## 资源文件

```
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#include                <resource.h>

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#define        ICO_MAIN                1000

#define        DLG_MAIN                1

#define IDC_TIME                2

#define IDC_END                      4

#define IDC_WEB                3

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

ICO_MAIN        ICON                "Main.ico"

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DLG_MAIN DIALOG 111, 104, 201, 57

STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU

CAPTION "网页刷新器"

FONT 9, "宋体"

{

LTEXT "网站", -1, 7, 8, 25, 8

EDITTEXT IDC_WEB, 35, 5, 160, 12, ES_AUTOHSCROLL | WS_BORDER | WS_TABSTOP

LTEXT "时间m", -1, 7, 25, 25, 8

EDITTEXT IDC_TIME, 35, 22, 160, 12, ES_AUTOHSCROLL | WS_BORDER | WS_TABSTOP

PUSHBUTTON "取消", IDC_END, 115, 38, 40, 14

PUSHBUTTON "执行", IDOK, 155, 38, 40, 14,  WS_TABSTOP

}

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [网页刷新.rar](javascript:void(0);)

  （3.63kb，32次下载）
