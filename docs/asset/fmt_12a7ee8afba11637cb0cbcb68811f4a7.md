
```asm
                mov     edx,9  
                mov     ax, word ptr Hz_NumStr[4]  
                je      l_007  
                mov     ax, word ptr Hz_NumStr[2]  
                cmp     bl,20  
                je      l_007  
                mov     al,bl  
                dec     al  
                js      l_008  
                movzx   eax,al  
                cdq  
                mov     ecx,10  
                div     ecx  
                mov     ax, word ptr Hz_NumHStr[2*eax]  

l_007:          stosw  
                mov     ax, word ptr Hz_NumStr[2*edx]  
                stosw  

l_008:          mov     al,0  
                stosb  
                popad  
        assume  esi:nothing  
                ret  

_GetLunarStr    endp  

;******************************************************************

Tg_Str          db      '甲乙丙丁戊己庚辛壬癸'

Dz_Str          db      '子丑寅卯辰巳午未申酉戌亥'

_GetGzStr       proc    GzNumber:dword  
                pushad  
                mov     eax,GzNumber  
                mov     cl,10  
                div     cl  
                movzx   ebx,ah  
                mov     eax,GzNumber  
                mov     cl,12  
                div     cl  
                movzx   edx,ah  
                mov     ax, word ptr Tg_Str[2*ebx]  
                mov     word ptr [esp+1Ch],ax  
                mov     ax, word ptr Dz_Str[2*edx]  
                mov     word ptr [esp+1Ch][2],ax  
                popad  
                ret  

_GetGzStr       endp  

;******************************************************************

Sx_Str          db      '鼠牛虎免龙蛇马羊猴鸡狗猪'

Wx_Str          db      '金木水火土'

_GetSxStr       proc    GzNumber:dword  
                xor     eax,eax  
                pushad  
                mov     eax,GzNumber  
                mov     cl,12  
                div     cl  
                movzx   edx,ah  
                mov     ax, word ptr Sx_Str[2*edx]  
                mov     word ptr [esp+1Ch],ax  
                mov     ecx,GzNumber  
                shr     ecx,1  
                cmp     cl,15  
                jb      l001  
                sub     cl,15  

l001:           mov     eax,42304130h  
                cmp     cl,8  
                jb      l002  
                sub     cl,8  
                mov     eax,2134210h  

l002:           shl     cl,2  
                shr     eax,cl  
                and     al,0Fh  
                movzx   eax,al  
                mov     ax,word ptr Wx_Str[eax*2]  
                mov     word ptr [esp+1Ch][2],ax  
                popad  
                ret  

_GetSxStr       endp  

;=======================================

Hd_str  db      '小寒大寒立春雨水惊蛰春分清明谷雨立夏小满芒种夏至',\\
                    '小暑大暑立秋处暑白露秋分寒露霜降立冬小雪大雪冬至'

_GetHDStr       proc    HdNumber:dword  
                mov     eax,HdNumber  
                or      eax,eax  
                jz      l001  
                dec     eax  
                mov     eax,dword ptr Hd_str[eax*4]  

l001:           ret  

_GetHDStr       endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;调试显示变量值

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_DisplayMenuItem  Proc  _dwCommandID  
    local   @szBuffer[256]:byte  
    pushad  
    invoke  wsprintf,addr @szBuffer,addr szFormat,_dwCommandID  
    invoke  MessageBox,hWinMain,addr @szBuffer,addr szMsgCaption,MB_OK  
    popad  
    ret  

_DisplayMenuItem  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;调用星期函数

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>   

_CalWeek  proc  
          pushad  
          invoke  SendDlgItemMessage,hWinMain,IDC_WORKDATE,DTM_GETSYSTEMTIME,0,addr @Stime1  

            ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>计算农历值  
          mov  ax,@Stime1.wYear  
          mov  stLunarData.dwYear,ax  
          mov  ax,@Stime1.wMonth  
          mov  stLunarData.dwMonth,ax  
          mov  ax,@Stime1.wDay  
          mov  stLunarData.dwDay,ax  
          movzx  ebx,ax  
          invoke  _GetLunarData,addr stLunarData  
          invoke  _GetLunarStr,addr stLunarData,addr @szBufLunar  
          invoke  SendDlgItemMessage,hWinMain,IDC_STATIC1,WM_SETTEXT,0,addr  @szBufLunar  
          ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>计算星期几  
          ;invoke  _GetWeekDay,stLunarData.dwYear,stLunarData.dwMonth,stLunarData.dwDay  
          invoke  SendDlgItemMessage,hWinMain,IDC_WORKDATE,DTM_GETSYSTEMTIME,0,addr @Stime  
          mov  ax,@Stime.wDayOfWeek  
          movzx  eax,ax  
          .if  eax==1  
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek1  
          .elseif eax==2  
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek2  
          .elseif eax==3  
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek3  
          .elseif eax==4  
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek4  
          .elseif eax==5  
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek5  
          .elseif eax==6  
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek6  
          .elseif eax==0       
            invoke  SendDlgItemMessage,hWinMain,IDC_STATIC2,WM_SETTEXT,0,addr  szWeek7  
            ;invoke  MessageBox,hWinMain,addr @szBufLunar,addr szMsgCaption,MB_OK  
          .endif  
          popad  
          ret  

_CalWeek  endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 计算两个阳历日之间的天数

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>    

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_ProcDlgMain  proc  uses ebx edi esi hDlg,uMsg,wParam,lParam  
    local   @szBuffer[128]:byte  
    ;local   @szBufLunar[20]:byte  
    ;local   @Stime:SYSTEMTIME  
    local   @Min:SYSTEMTIME  
    local   @Max:SYSTEMTIME  
    ;local   @Stime1:SYSTEMTIME  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>设定两个计时器改变 BMP 和 ICON 显示  

    .if   uMsg==  WM_TIMER  

      mov  eax,wParam  

      .if  eax==  ID_TIMER1  
        mov  eax,hIco1  
        xchg  hIco2,eax  
        mov  hIco1,eax  
        invoke  SendDlgItemMessage,hDlg,IDC_SETICON,STM_SETIMAGE,IMAGE_ICON,eax  

      .elseif  eax==  ID_TIMER2  
        mov  eax,hBmp1  
        xchg  hBmp2,eax  
        mov  hBmp1,eax  
        invoke  SendDlgItemMessage,hDlg,IDC_BMP,STM_SETIMAGE,IMAGE_BITMAP,eax  

      .endif  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>对话框初始化.   

    .elseif  uMsg==  WM_INITDIALOG           ;  
      invoke  AnimateWindow,hDlg,500,AW_CENTER   ;对话框动画显示 AW_SLIDE or AW_HOR_POSITIVE or  
      invoke  RtlZeroMemory,addr @Stime,sizeof @Stime    ;日期控件初始化  
      invoke  RtlZeroMemory,addr @Min,sizeof @Min  
      mov  @Min.wYear,1981  
      invoke  RtlZeroMemory,addr @Max,sizeof @Max  
      mov  @Max.wYear,2020  
```
