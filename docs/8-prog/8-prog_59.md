

# enumwindows.asm

> 原文：[`www.pediy.com/kssd/pediy12/131667.html`](https://www.pediy.com/kssd/pediy12/131667.html)

.386  
.model flat,stdcall  
option casemap:none  
include windows.inc  
include kernel32.inc  
include user32.inc  
includelib user32.lib  
includelib kernel32.lib  
include comctl32.inc   
includelib comctl32.lib   
  
.data  
szClassName db "correy",0  
szCaptionMain db "made by correy",0  
ListViewClass db 'SysListView32',0;ListBox  
szstatic db "Static",0  
windowstitle db '主窗口的标题',0  
windowsclass db "主窗口类的名字",0  
szhwindows db "主窗口的句柄（十六进制）",0  
cwindowstitle db '子窗口的标题',0  
cwindowsclass db "子窗口类的名字",0  
cszhwindows db "子窗口的句柄（十六进制）",0  
nowindowstitle db "无",0  
sznochildwindows db "此窗口没有子窗口",0  
notice db "说明：单击主窗口的标题，可以在下面显示她的子窗口的相关信息。",0  
;本程序还有不足自出，敬请指导，联系方式 Email:leguanyuan@126.com,如：选中标题是用颜色加深显示一行。还有更多功能待补充，如：单击子窗口可以增加控制子窗口的相关功能，如获取 QQ 密码登陆框的密码（腾讯公司当然有加密等保护呀！），获取 QQ 发送框的窗口句柄等。还有更多属性没有显示，如窗口的大小，边框等属性等等。  
  
.data?  
hInstance dd ?  
hWinMain dd ?  
hlist dd ?  
hlist2 dd ?  
stWndClass WNDCLASSEX <>  
stMsg MSG <>  
lvc LV\_COLUMN <>  
lvi LV\_ITEM <>  
buffer db 256 dup (?)  
bufferl db 8 dup (?),0  
hstatic dd ?  
  
.code  
  
mulit proc m:dword  
mov ecx,m  
.if ecx==1  
mov ebx,1  
ret  
.endif  
  
mov ebx,1  
  
mul10:  
   imul ebx,16  
   dec ecx  
   cmp ecx,1  
jne mul10  
  
ret  
mulit endp   
  
String2Dword proc uses ecx edi edx esi String:DWORD   
  LOCAL Result:DWORD   
  mov Result,0   
  mov edi,String   
    
  mov ecx,8  
  
calc:  
      mov bl,byte ptr \[edi\]  
      movzx ebx,bl  
    .if bl >= "A" && bl <= "F"  
      sub bl,"A"  
      add bl,10  
    .elseif  
      sub bl,"0"  
    .endif  
        
      push ecx  
        
      push ebx  
      invoke mulit,ecx  
      mov eax,ebx  
      pop ebx  
        
      imul eax,ebx  
      add Result,eax  
      
    pop ecx  
      
  inc edi  
  dec ecx  
  cmp ecx,0  
  jne calc  
    
  mov eax,Result  
    
  ret   
String2Dword endp   
  
showeax proc;坚决拒绝使用 wsprintf 等类似函数。  
pushad  
mov ecx,8  
xor esi,esi  
againp:  
dec ecx  
rol eax,4  
push eax  
and eax,0Fh  
cmp eax,9  
jg big  
add eax,30h  
mov \[bufferl+esi\],al  
pop eax  
inc esi  
cmp ecx,0  
jne againp  
je showp   
big:  
add eax,37h  
mov \[bufferl+esi\],al  
pop eax  
inc esi  
cmp ecx,0  
jne againp  
showp:  
popad  
ret  
showeax endp  
  
EnumChildProc proc eHandle2:DWORD,y2:DWORD  
  LOCAL pbuf2  :DWORD  
  LOCAL Buffer2\[1024\]:BYTE  
  LOCAL tbuf2\[512\]:BYTE  
  LOCAL clName2\[128\]:BYTE  
    
  invoke RtlZeroMemory,addr Buffer2,sizeof Buffer2  
    
  invoke GetClassName,eHandle2,addr clName2,128  
  invoke GetWindowText,eHandle2,addr tbuf2,512  
    
  lea eax,Buffer2  
  mov pbuf2,eax  
    
  mov eax,eHandle2  
  call showeax  
    
  invoke lstrlen,addr tbuf2  
  cmp eax,0  
  je next  
    
  mov lvi.imask,LVIF\_TEXT  
  lea eax,tbuf2  
  mov lvi.pszText,eax  
  mov lvi.iSubItem,0  
  invoke SendMessage,hlist2,LVM\_INSERTITEM,0,addr lvi  
  jmp next2  
    
  next:  
  mov lvi.imask,LVIF\_TEXT  
  mov lvi.pszText,offset nowindowstitle  
  mov lvi.iSubItem,0  
  invoke SendMessage,hlist2,LVM\_INSERTITEM,0,addr lvi  
  
  next2:  
    
  lea eax,clName2  
  mov lvi.pszText,eax  
  mov lvi.iSubItem,1  
  invoke SendMessage,hlist2,LVM\_SETITEM,0,addr lvi  
    
  mov lvi.pszText,offset bufferl  
  mov lvi.iSubItem,2  
  invoke SendMessage,hlist2,LVM\_SETITEM,0,addr lvi  
    
  mov eax,1  
  ret  
EnumChildProc endp  
  
Enmwin proc eHandle:DWORD,y:DWORD  
  LOCAL pbuf  :DWORD  
  LOCAL Buffer\[1024\]:BYTE  
  LOCAL tbuf\[512\]:BYTE  
  LOCAL clName\[128\]:BYTE  
    
  invoke RtlZeroMemory,addr Buffer,sizeof Buffer  
    
  invoke GetClassName,eHandle,addr clName,128  
  invoke GetWindowText,eHandle,addr tbuf,512  
    
  lea eax,Buffer  
  mov pbuf,eax  
    
  mov eax,eHandle  
  call showeax  
    
  invoke lstrlen,addr tbuf  
  cmp eax,0  
  je next  
    
  mov lvi.imask,LVIF\_TEXT  
  lea eax,tbuf  
  mov lvi.pszText,eax  
  mov lvi.iSubItem,0  
  invoke SendMessage,hlist,LVM\_INSERTITEM,0,addr lvi  
  jmp next2  
    
  next:  
  mov lvi.imask,LVIF\_TEXT  
  mov lvi.pszText,offset nowindowstitle  
  mov lvi.iSubItem,0  
  invoke SendMessage,hlist,LVM\_INSERTITEM,0,addr lvi  
  
  next2:  
    
  lea eax,clName  
  mov lvi.pszText,eax  
  mov lvi.iSubItem,1  
  invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi  
    
  mov lvi.pszText,offset bufferl  
  mov lvi.iSubItem,2  
  invoke SendMessage,hlist,LVM\_SETITEM,0,addr lvi  
    
  mov eax,1  
  ret  
Enmwin endp  
  
liuchunli proc uses ebx edi esi,hWnd,uMsg,wParam,lParam  
local p:POINT  
local x:dword  
local y:dword  
.if uMsg == WM\_CLOSE  
  invoke DestroyWindow,hWinMain  
  invoke PostQuitMessage,NULL  
    
.elseif uMsg == WM\_CREATE  
  INVOKE CreateWindowEx,WS\_EX\_CLIENTEDGE,addr ListViewClass,0,WS\_CHILD or WS\_VISIBLE or LVS\_REPORT or LVS\_SHAREIMAGELISTS,\\  
         0,0,700,200,hWnd,0,hInstance,0  
  mov hlist,eax  
    
  mov lvc.imask,LVCF\_TEXT+LVCF\_WIDTH   
  mov lvc.pszText,offset windowstitle  
  mov lvc.lx,150   
  invoke SendMessage,hlist, LVM\_INSERTCOLUMN, 0, addr lvc  
    
  mov lvc.pszText,offset windowsclass  
  mov lvc.lx,150  
  invoke SendMessage,hlist, LVM\_INSERTCOLUMN, 1 ,addr lvc   
    
  mov lvc.pszText,offset szhwindows  
  mov lvc.lx,200   
  invoke SendMessage,hlist,LVM\_INSERTCOLUMN,2,addr lvc  
    
  mov lvi.iItem,0  
  invoke EnumWindows,ADDR Enmwin,0  
    
  INVOKE CreateWindowEx,WS\_EX\_CLIENTEDGE,addr ListViewClass,0,WS\_CHILD or WS\_VISIBLE or LVS\_REPORT or LVS\_SHAREIMAGELISTS,0,200,700,200,hWnd,0,hInstance,0  
  mov hlist2,eax  
    
  mov lvc.imask,LVCF\_TEXT+LVCF\_WIDTH   
  mov lvc.pszText,offset cwindowstitle  
  mov lvc.lx,150   
  invoke SendMessage,hlist2, LVM\_INSERTCOLUMN, 0, addr lvc  
    
  mov lvc.pszText,offset cwindowsclass  
  mov lvc.lx,150  
  invoke SendMessage,hlist2, LVM\_INSERTCOLUMN, 1 ,addr lvc   
    
  mov lvc.pszText,offset cszhwindows  
  mov lvc.lx,200   
  invoke SendMessage,hlist2,LVM\_INSERTCOLUMN,2,addr lvc  
    
  INVOKE CreateWindowEx,WS\_EX\_CLIENTEDGE,addr szstatic,0,WS\_CHILD or WS\_VISIBLE or SS\_LEFT or WS\_GROUP,0,400,700,100,hWnd,0,hInstance,0  
  MOV hstatic,EAX  
  invoke SendMessage,eax,WM\_SETTEXT,0,addr notice  
    
    
.elseif uMsg==WM\_NOTIFY   
  push edi   
  mov edi,lParam   
  assume edi:ptr NMHDR   
  mov eax,\[edi\].hwndFrom   
  .if eax==hlist   
    .if \[edi\].code==NM\_DBLCLK   
      invoke SendMessage,hlist2,LVM\_DELETEALLITEMS,0,0  
      invoke SendMessage,hlist,LVM\_GETNEXTITEM,-1,LVNI\_FOCUSED  
      mov lvi.iItem,eax  
      mov lvi.iSubItem,2  
      mov lvi.cchTextMax,256  
      mov lvi.imask,LVIF\_TEXT  
      mov lvi.pszText,offset buffer  
      invoke SendMessage,hlist,LVM\_GETITEM,0,addr lvi  
      
      invoke String2Dword,addr buffer  
      mov lvi.iItem,0  
      invoke EnumChildWindows,eax,ADDR EnumChildProc,0  
      .if !eax  
        mov lvi.imask,LVIF\_TEXT  
        lea eax,sznochildwindows  
        mov lvi.pszText,eax  
        mov lvi.iSubItem,0  
        invoke SendMessage,hlist2,LVM\_INSERTITEM,0,addr lvi  
      .endif  
    .endif   
  .endif   
  pop edi   
.else  
  invoke DefWindowProc,hWnd,uMsg,wParam,lParam  
  ret  
.endif  
xor eax,eax  
ret  
liuchunli endp  
  
start:  
  
invoke InitCommonControls  
  
invoke GetModuleHandle,NULL  
mov hInstance,eax  
mov stWndClass.hInstance,eax  
  
invoke LoadIcon,hInstance,1  
mov stWndClass.hIcon,eax  
  
invoke LoadCursor,0,32512  
mov dword ptr stWndClass+28,eax  
  
mov stWndClass.cbSize,48  
mov stWndClass.style,3  
mov stWndClass.lpfnWndProc,offset liuchunli  
mov stWndClass.hbrBackground,6  
mov stWndClass.lpszClassName,offset szClassName  
  
invoke RegisterClassEx,addr stWndClass  
invoke CreateWindowEx,200h,offset szClassName,offset szCaptionMain,0Ca0000h,80000000h,80000000h,710,530,0,0,hInstance,0  
mov hWinMain,eax  
invoke ShowWindow,hWinMain,1  
invoke UpdateWindow,hWinMain  
.while TRUE  
invoke GetMessage,addr stMsg,0,0,0  
.break .if eax == 0  
invoke TranslateMessage,addr stMsg  
invoke DispatchMessage,addr stMsg  
.endw  
invoke ExitProcess,0  
end start  
;made at 2010.10.28  
;此文献给我的老婆杜岷娟。

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 9.rar  
 |