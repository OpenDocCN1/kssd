
```
.if uMsg==WM_TIMER

    mov eax,wParam
    .if eax==ID_TIMER1
        mov eax,hIco1
        xchg hIco2,eax
        mov hIco1,eax
        invoke SendDlgItemMessage,hDlg,IDC_SETICON,STM_SETIMAGE,IMAGE_ICON,eax
    .elseif eax==ID_TIMER2
        mov eax,hBmp1
        xchg hBmp2,eax
        mov hBmp1,eax
        invoke SendDlgItemMessage,hDlg,IDC_BMP,STM_SETIMAGE,IMAGE_BITMAP,eax
    .endif

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>对话框初始化.
.elseif uMsg==WM_INITDIALOG
    invoke AnimateWindow,hDlg,500,AW_CENTER
    invoke RtlZeroMemory,addr @Stime,sizeof @Stime
    invoke RtlZeroMemory,addr @Min,sizeof @Min
    mov @Min.wYear,1981
    invoke RtlZeroMemory,addr @Max,sizeof @Max
    mov @Max.wYear,2020
    invoke SetClassLong,hDlg,GCL_HCURSOR,hCur1
    invoke LoadIcon,hInstance,ICO_MAIN
    invoke SendMessage,hDlg,WM_SETICON,ICON_BIG,eax
    invoke LoadBitmap,hInstance,IDB_1
    mov hBmp1,eax
    invoke LoadBitmap,hInstance,IDB_2
    mov hBmp2,eax
    invoke LoadIcon,hInstance,ICO_1
    mov hIco1,eax
    invoke LoadIcon,hInstance,ICO_2
    mov hIco2,eax

    invoke SendDlgItemMessage,hDlg,IDC_TEAMNAME,CB_ADDSTRING,0,addr szText1
    invoke SendDlgItemMessage,hDlg,IDC_TEAMNAME,CB_ADDSTRING,0,addr szText2
    invoke SendDlgItemMessage,hDlg,IDC_TEAMNAME,CB_ADDSTRING,0,addr szText3
    invoke SendDlgItemMessage,hDlg,IDC_TEAMNAME,CB_ADDSTRING,0,addr szText4
    invoke SendDlgItemMessage,hDlg,IDC_TEAMNAME,CB_SETCURSEL,0,0
    invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText9
    invoke LoadIcon,hInstance,ICO_1
    mov hIco1,eax
    invoke LoadIcon,hInstance,ICO_2
    mov hIco2,eax

    invoke SetTimer,hDlg,ID_TIMER1,400,NULL
    invoke SetTimer,hDlg,ID_TIMER2,2000,NULL
    invoke GetDlgItem,hDlg,IDC_WORKDATE
    mov hDtl,eax
    mov eax,hDlg
    mov hWinMain,eax

    invoke SetWindowPos,hDlg,HWND_TOPMOST,0,0,0,0,SWP_NOSIZE or SWP_NOMOVE
    invoke _CalWeek

.elseif uMsg==WM_NOTIFY
    mov eax,wParam
    mov edx,lParam
    .if eax==IDC_WORKDATE
        assume edx:ptr NMDATETIMECHANGE
        .if [edx].nmhdr.code == DTN_DATETIMECHANGE
            invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText9
            invoke _CalWeek
        .endif
        assume edx:nothing
    .endif

.elseif uMsg==WM_COMMAND
    mov eax,wParam

    .if ax==IDC_FIND
        invoke SendDlgItemMessage,hDlg,IDC_TEAMNAME,CB_GETCURSEL,0,0
        mov szIndex,eax
        invoke SendDlgItemMessage,hDlg,IDC_WORKDATE,DTM_GETSYSTEMTIME,0,addr @Stime
        movzx eax,@Stime.wYear
        mov szYear,eax
        movzx eax,@Stime.wMonth
        mov szMonth,eax
        movzx eax,@Stime.wDay
        mov szDay,eax
        movzx eax,@Stime.wDayOfWeek
        mov szDayWeek,eax
        invoke _GetBetweenDays,szYear,szMonth,szDay,1000,0001,0001
        mov szBeet,eax
        mov edx,0
        mov bx,4
        div bx
        mov eax,edx

        .if szIndex==0
            .if eax==0
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText5
            .elseif eax==1
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText6
            .elseif eax==2
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText7
            .elseif eax==3
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText8
            .endif
        .elseif szIndex==1
            .if eax==0
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText6
            .elseif eax==1
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText7
            .elseif eax==2
                invoke SetDlgItemText,hDlg,IDC_SHIFTRESULT,addr szText8
```
