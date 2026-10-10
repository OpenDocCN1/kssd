

# 汇编学习之网络编程

> 原文：[`www.pediy.com/kssd/pediy10/78072.html`](https://www.pediy.com/kssd/pediy10/78072.html)

在 C 语言中写过网络编程，但是用汇编怎么实现呢？  
  

代码:

```
;winsock
;======================================


include ws2\_32.inc

includelib ws2\_32.lib



;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* const \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

.const

szConnect db "连接",0

szDisConnect db "断开",0

szErr db "error !",0

szAddr db "127.0.0.1",0

szClient db "Client: %s",0dh,0ah,0
szServer db "Server: %s",0dh,0ah,0

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* code \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\_Connect proc uses ebx esi edi ;\_lParam
  
  LOCAL \_szBuffer\[256\]:byte
  LOCAL \_stSin:sockaddr\_in
  
  ;创建流套接字
  invoke socket,AF\_INET,SOCK\_STREAM,0
  .if eax == INVALID\_SOCKET
    invoke MessageBox,NULL,addr szErr,addr szErr,MB\_OK
    ret
  .endif
  mov hSocket,eax
  
  ;设置套接字属性
  invoke WSAAsyncSelect,hSocket,hDlg,WM\_WINSOCK,FD\_CONNECT or FD\_READ or FD\_CLOSE or FD\_WRITE
  
  ;链接到服务器中的监听套接字
  lea esi, \_stSin
  assume esi: PTR sockaddr\_in
  invoke RtlZeroMemory,addr \_stSin,sizeof \_stSin
  invoke GetDlgItemText,hDlg,IDC\_EDIT\_ADDR,addr \_szBuffer,sizeof \_szBuffer
  invoke inet\_addr,addr \_szBuffer ;ip 地址转换
  .if eax ==INADDR\_NONE
    invoke MessageBox,NULL,addr szErr,addr szErr,MB\_OK
    ret
  .endif
  mov \[esi\].sin\_addr,eax
  mov \[esi\].sin\_family,AF\_INET
  invoke htons,TCP\_PORT
  mov \[esi\].sin\_port,ax
  invoke connect,hSocket,addr \_stSin ,sizeof \_stSin
  
  ;返回
  .if eax == SOCKET\_ERROR
    invoke WSAGetLastError
    .if eax != WSAEWOULDBLOCK
      invoke closesocket,hSocket;关闭套接字
      mov hSocket,0
      invoke MessageBox,NULL,addr szErr,addr szErr,MB\_OK
      ret
    .endif
  .endif
  ret

\_Connect endp

\_DisConnect pro
  invoke closesocket,,hSocket
  mov hSocket,0

  mov bCntFlag,NOTCONNECT
  
  invoke SetDlgItemText,hDlg,IDC\_BUTTON\_CONNECT,addr szConnect
  invoke SendDlgItemMessage,hDlg,IDC\_EDIT\_ADDR,EM\_SETREADONLY,FALSE,0
  
  ;enable button
  invoke EnableWindow,hIDOK,FALSE
  ret
\_DisConnect endp

;send text

\_SendData proc
  
  LOCAL \_szBuffer\[1024\]:byte
  
  invoke GetDlgItemText,hDlg,IDC\_EDIT\_SEND,addr \_szBuffer,sizeof \_szBuffer
  invoke lstrlen,addr \_szBuffer
  .if eax
    invoke send,hSocket,addr \_szBuffer,eax,0
  .endif
  
  ;clear the text
  invoke SetDlgItemText,hDlg,IDC\_EDIT\_SEND,NULL
  invoke RtlZeroMemory,addr szBuffer,sizeof szBuffer
  invoke wsprintf,addr szBuffer,addr szClient,addr \_szBuffer
  invoke GetWindowTextLength,hEditShow
  invoke SendMessage,hEditShow,EM\_SETSEL,eax,eax
  invoke SendMessage,hEditShow,EM\_REPLACESEL,FALSE,addr szBuffer
  ret

\_SendData endp

```

代码:

```
;装入并初始化 Winsock 动态链接库

  invoke WSAStartup,02h,addr wsd
  
  
  .else 
    mov eax,FALSE
    ret
  .endif
  mov eax,TRUE
  ret
  
  
;程序入口点
start:
  invoke GetModuleHandle,NULL
  invoke DialogBoxParam,eax,IDD\_DIALOG\_CLIENT,NULL,addr \_DlgProc,NULL
  invoke ExitProcess,NULL
end start

```