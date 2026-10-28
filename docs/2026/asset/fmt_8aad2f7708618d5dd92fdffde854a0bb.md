
```
                        invoke        SendMessage,hWnd,WM_SETICON,ICON_BIG,eax

                        invoke  GetDlgItem, hWnd, IDC_PROGRESS

                   mov     hProgress, eax

                   invoke  GetDlgItem,hWnd,IDC_List

                        mov   hlistview, eax

                        invoke InsertColumn

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.asp")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.aspx")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.php")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.jsp")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.htm")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.html")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.jpg")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.js")

                        invoke        SendDlgItemMessage,hWnd,IDC_COMBO1,CB_ADDSTRING,0,CTXT("*.gif")

                .elseif        eax == WM_COMMAND

                        mov        eax,wParam

                        .if        eax == IDC_Look

                        invoke        GetDlgItemText,hWnd,IDC_Directory,addr @szBuffer,MAX_PATH

                        call         GetOpenDirectoryName

                        invoke        SetDlgItemText,hWnd,IDC_Directory,addr szPath

                        .elseif        eax ==        IDOK

                                invoke SendDlgItemMessage,hWnd,IDC_List,0,0,0

                                        invoke        GetDlgItemText,hWnd,IDC_Directory,addr szPath,MAX_PATH

                                        invoke lstrlen,addr szPath

                                        .if eax == NULL

                                                invoke MessageBox,NULL,CTXT("'请填写目录路径"),NULL,MB_ICONSTOP

                                                .elseif

                                        invoke        GetDlgItemText,hWnd,IDC_COMBO1,addr szPath1,MAX_PATH

                                        invoke lstrlen,addr szPath1

                                        .if eax==NULL

                                        invoke SendDlgItemMessage,hWnd,IDC_List,LB_RESETCONTENT,0,0

                     invoke CreateThread,NULL,0,addr _FindFile,addr szPath,0,addr ThreadId1

                     invoke  SendMessage, hProgress, PBM_SETPOS,100, 0

                     .elseif

                     invoke SendDlgItemMessage,hWnd,IDC_List,LB_RESETCONTENT,0,0

                     invoke CreateThread,NULL,0,addr _FindFileTyte,addr szPath,0,addr ThreadId3

                     invoke  SendMessage, hProgress, PBM_SETPOS,100, 0

                     .endif

               .endif

;********************************************************************************

                        .elseif eax == IDC_Check

                        invoke        GetDlgItemText,hWnd,IDC_KeYi,addr szFileLook,MAX_PATH

                        invoke  FindFirstFile,addr szFileLook,addr @stFindFile

         .if eax == INVALID_HANDLE_VALUE

         invoke MessageBox,NULL,CTXT("找不到文件,请确定路径是否正确!"),NULL,MB_ICONSTOP

         .elseif

         invoke CreateThread,NULL,0,addr _ChenLook,addr szFileLook,0,addr ThreadId

                        .endif

                        .elseif eax == IDC_Del

                        invoke        GetDlgItemText,hWnd,IDC_KeYi,addr szFileLook,MAX_PATH

         invoke  FindFirstFile,addr szFileLook,addr @stFindFile

         .if eax == INVALID_HANDLE_VALUE

         invoke MessageBox,NULL,CTXT("找不到文件,请确定路径是否正确!"),NULL,MB_ICONSTOP

         .elseif

                        invoke DeleteFile,addr szFileLook

                        invoke        MessageBox,hWnd,CTXT("文件删除成功!"),CTXT("title"),MB_ICONINFORMATION

                        .endif

                        .elseif        ax == IDC_List

                                shr        eax,16

                                .if        ax ==        LBN_SELCHANGE

;********************************************************************************
; 将鼠标点击结果显示在文本框中
;********************************************************************************

                                        invoke        SendMessage,lParam,LB_GETCURSEL,0,0

                                        lea        ecx,@szBuffer

                                        invoke        SendMessage,lParam,LB_GETTEXT,eax,ecx

                                        invoke        SetDlgItemText,hWnd,IDC_24,addr @szBuffer

                                .endif

                        .elseif eax == IDC_Out

                        invoke MessageBox,NULL,addr szabout,CTXT("关于工具的一点说明"),MB_ICONINFORMATION

                        .elseif eax == IDC_JianShi

                                invoke        DialogBoxParam,hInstance,DLG_EDIT,NULL,offset _ProcDlg,NULL

                                .elseif eax == IDC_ShuoMing

                                        invoke MessageBox,hWnd,addr Shuo,CTXT("说明"),MB_ICONINFORMATION

                                        .elseif eax == IDC_21

                                invoke        DialogBoxParam,hInstance,DLG_LOOK,NULL,offset _ProcLook,NULL

                        .endif

;********************************************************************************

                .else

                        mov        eax,FALSE

                        ret

                .endif

                mov        eax,TRUE

                ret

_ProcDlgMain        endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:

                invoke        GetModuleHandle,NULL

                mov                hInstance,eax

                invoke        DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,NULL

                invoke        ExitProcess,NULL

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

                end        start
```

```
RC:
```
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
#include                <resource.h>
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#define        ICO_MAIN                      1000        //图标

#define        DLG_MAIN                1                //        主窗口  对话框

#define        DLG_EDIT                2                //        编辑规则对话框

#define        DLG_LOOK    3

#define  IDC_Look           1001

#define  IDC_Directory                    1002

#define  IDC_KeYi                  1003

#define  IDC_Check                  1004

#define  IDC_Del           1005

#define  IDC_List            1006

#define  IDC_PROGRESS            1007

#define  IDC_Out              1008

#define  IDC_JianShi              1009

#define  IDC_ShuoMing              1010

#define  IDC_11             1011

#define  IDC_12              1012

#define  IDC_13              1013

#define  IDC_14              1014

#define  IDC_15              1015

#define  IDC_16              1016

#define  IDC_17              1017

#define  IDC_18              1018

#define  IDC_19              1019

#define  IDC_20              1020

#define  IDC_21              1021

#define  IDC_22              1022

#define  IDC_23              1023

#define  IDC_24              1024

#define  IDC_25              1025

#define IDC_Lujing        1026

#define IDC_COMBO1      1027
```
