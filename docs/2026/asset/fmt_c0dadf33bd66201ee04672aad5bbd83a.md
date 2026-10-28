
```asm
    invoke CompareString,LOCALE\_USER\_DEFAULT,NORM\_IGNORECASE,\_ProName,-1,addr stProcess.szExeFile,-1

        .if eax==2

        push   stProcess.th32ProcessID

          pop   closepid

      .endif

            invoke   Process32Next,hSnapShot,addr stProcess

    .endw

       invoke OpenProcess,PROCESS\_TERMINATE,FALSE,closepid

      .if        eax

     mov        ebx,eax

     invoke        TerminateProcess,ebx,-1

     .endif

    ret

\_ProcessDel endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

\_ProcDlg        proc        uses ebx edi esi hWnd,wMsg,wParam,lParam

                mov        eax,wMsg

                .if        eax == WM\_CLOSE

                        invoke        EndDialog,hWnd,NULL

                .elseif        eax == WM\_INITDIALOG

                        invoke        LoadIcon,hInstance,ICO\_MAIN

                        invoke        SendMessage,hWnd,WM\_SETICON,ICON\_BIG,eax

                .elseif        eax == WM\_COMMAND

                        mov        eax,wParam

                        .if        eax == IDC\_15

                                invoke \_GetOpenDirectoryName,addr szDirectory

                                invoke        SetDlgItemText,hWnd,IDC\_11,addr szDirectory

                                invoke        GetDlgItemText,hWnd,IDC\_11,addr szName1,MAX\_PATH

                        .elseif        eax == IDC\_16

                                invoke \_GetOpenDirectoryName,addr szDirectory1

                                invoke        SetDlgItemText,hWnd,IDC\_12,addr szDirectory1

                                invoke        GetDlgItemText,hWnd,IDC\_12,addr szName2,MAX\_PATH

                        .elseif        eax == IDC\_17

                                invoke \_GetOpenDirectoryName,addr szDirectory2

                                invoke        SetDlgItemText,hWnd,IDC\_13,addr szDirectory2

                                invoke        GetDlgItemText,hWnd,IDC\_13,addr szName3,MAX\_PATH

                        .elseif        eax == IDC\_18

                                invoke \_GetOpenDirectoryName,addr szDirectory3

                                invoke        SetDlgItemText,hWnd,IDC\_14,addr szDirectory3

                                invoke        GetDlgItemText,hWnd,IDC\_14,addr szName4,MAX\_PATH

                        .elseif eax == IDC\_19

                                invoke lstrlen,addr szName1

                                .if eax!=NULL

                                invoke \_ChangeNotification,addr szName1,CTXT("目录监视程序2.exe"),ASM2

                                invoke ShellExecute,0,0,CTXT("目录监视程序2.exe"),0,0,SW\_SHOW

                                .endif

                                invoke lstrlen,addr szName2

                                .if eax!=NULL

                                invoke \_ChangeNotification,addr szName2,CTXT("目录监视程序3.exe"),ASM3

                                invoke ShellExecute,0,0,CTXT("目录监视程序3.exe"),0,0,SW\_SHOW

                                .endif

                                invoke lstrlen,addr szName3

                                .if eax!=NULL

                                invoke \_ChangeNotification,addr szName3,CTXT("目录监视程序4.exe"),ASM4

                                invoke ShellExecute,0,0,CTXT("目录监视程序4.exe"),0,0,SW\_SHOW

                                .endif

                                invoke lstrlen,addr szName4

                                .if eax!=NULL

                                invoke \_ChangeNotification,addr szName4,CTXT("目录监视程序5.exe"),ASM5

                                invoke ShellExecute,0,0,CTXT("目录监视程序5.exe"),0,0,SW\_SHOW

                                .endif

                        .elseif eax == IDC\_20

                                invoke \_StopDelFile,CTXT("目录监视程序2.exe")

                                invoke \_StopDelFile,CTXT("目录监视程序3.exe")

                                invoke \_StopDelFile,CTXT("目录监视程序4.exe")

                                invoke \_StopDelFile,CTXT("目录监视程序5.exe")

                                invoke        MessageBox,NULL,CTXT("所有监控完全停止,监控程序已经从电脑中移除"),CTXT("title"),MB\_ICONINFORMATION

                        .endif

                .else

                        mov        eax,FALSE

                        ret

                .endif

                mov        eax,TRUE

                ret

\_ProcDlg        endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

\_ProcLook        proc        uses ebx edi esi hWinMain1,wMsg,wParam,lParam

local @szBuffer[MAX\_PATH]:BYTE

                mov        eax,wMsg

                .if        eax == WM\_CLOSE

                        invoke        EndDialog,hWinMain1,NULL

                .elseif        eax == WM\_INITDIALOG

                        invoke        LoadIcon,hInstance,ICO\_MAIN

                        invoke        SendMessage,hWinMain1,WM\_SETICON,ICON\_BIG,eax

                        invoke \_EnablePrivilege,CTXT("SeDebugPrivilege"),TRUE

                .elseif        eax == WM\_COMMAND

                        mov        eax,wParam

                        push hWinMain1

                        pop hWinProce

                        .if eax == IDC\_23

                        invoke CreateThread,NULL,0,addr \_Process,NULL,0,addr ThreadId2

                        .elseif        ax == IDC\_22

                                shr        eax,16

                                .if        ax ==        LBN\_SELCHANGE

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 将鼠标点击结果显示在文本框中

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

                                        invoke        SendMessage,lParam,LB\_GETCURSEL,0,0

                                        lea        ecx,@szBuffer

                                        invoke        SendMessage,lParam,LB\_GETTEXT,eax,ecx

                                        invoke SetDlgItemText,hWinMain1,IDC\_25,addr @szBuffer

                                .endif

                        .elseif eax == IDC\_24

                                invoke GetDlgItemText,hWinMain1,IDC\_25,addr szProcess,sizeof szProcess

                                invoke lstrlen,addr szProcess

                                .if eax==NULL

                                invoke MessageBox,hWinMain1,CTXT("请输入进程名称"),NULL,MB\_ICONSTOP

                                .elseif

                                        invoke \_ProcessDel,addr szProcess

                                        .endif

                                .endif

                .else

                        mov        eax,FALSE

                        ret

                .endif

                mov        eax,TRUE

                ret

\_ProcLook        endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

\_ProcDlgMain        proc        uses ebx edi esi hWnd,wMsg,wParam,lParam

local        @dwTemp,@szBuffer[MAX\_PATH]:BYTE

                local        @szBuffer1[128]:byte

                local        @szTextBuff[2048]:byte

                local  @stFindFile:WIN32\_FIND\_DATA

                mov        eax,wMsg

                .if        eax == WM\_CLOSE

                        invoke        EndDialog,hWnd,NULL

                .elseif        eax == WM\_INITDIALOG

                        push hWnd

                        pop hWinMain

                        push hWnd

                        pop hlistview

                        invoke InsertColumn

                        invoke        LoadIcon,hInstance,ICO\_MAIN
```
