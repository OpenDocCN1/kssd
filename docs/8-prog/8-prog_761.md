
# 罗云彬《32 位汇编语言》16 章中对 TCP 聊天室程序的扩展 2

> 原文：[`www.pediy.com/kssd/pediy12/98125.html`](https://www.pediy.com/kssd/pediy12/98125.html)

第一步的功能已经实现了。下一步就是限制用户灌水了。

1. 在 `Message.inc` 中，增加一个 `MSG_UP_RESP` 数据包，服务器端收到客户端发来的聊天语句时，如果判断本客户端此次的发言时间和上次的发言时间间隔太短，就向客户端发送带错误代码的 `MSG_UP_RESP` 数据包。

   ①

```assembly
;******************************************************************** ;canmeng 萌 2
; 提示语句（服务器端->客户端）                     ;canmeng 萌 2
;******************************************************************** ;canmeng 萌 2

MSG_UP_RESP    struct             ;canmeng 萌 2
  dbResult     db  ?             ;canmeng 萌 2
MSG_UP_RESP    ends              ;canmeng 萌 2
```

   ②增加这个数据包的数据包头部命令 ID

```assembly
CMD_MSG_UP_RESP  equ    84h  ;服务器端 ->客户端，提示不可以灌水,canmeng 萌 2
```

   ③在 `MSG_STRUCT` 结构体中也增加一个 `MSG_UP_RESP` 类型的字段

```assembly
MSG_STRUCT    struct
  MsgHead     MSG_HEAD <>
  union
    Login     MSG_LOGIN <>
    LoginResp MSG_LOGIN_RESP <>
    MsgUp     MSG_UP <>
    MsgUpResp MSG_UP_RESP <>     ;canmeng 萌 2
    MsgDown   MSG_DOWN <>
  ends
MSG_STRUCT    ends
```

2. 在 `Server.asm` 中，在 `_ServiceThread` 子线程中定义一个局部变量 `@lasttime`，用来记录本子线程和对应的客户端最近一次通话的时间。

   ①

```assembly
local  @lasttime          ;canmeng 萌 2
```

   ②当服务器端的某个线程收到对应的客户端发来的聊天语句时，首先把此用户上次的发言时间赋给 `@lasttime`，然后再调用 `GetTickCount` 函数得到现在的时间并赋给 `[edi].dwLastTime`

   两个进行比较，如果间隔小于设定的时间间隔(我设定为 2 秒)，那么就向对应的客户端发送一个 `MSG_UP_RESP` 数据包，而且不把本次的聊天语句放进消息队列。

```assembly
;********************************************************************
; 使用 select 函数等待 200ms，如果没有接收到数据包则循环
;********************************************************************

invoke  _WaitData,_hSocket,200 * 1000

.break  .if eax == SOCKET_ERROR

.if  eax
  invoke  _RecvPacket,_hSocket,esi,sizeof @szBuffer
  .break  .if eax
  mov eax,[edi].dwLastTime                    ;canmeng 萌 2
  mov   @lasttime,eax                   ;canmeng 萌 2
  invoke  GetTickCount
  mov  [edi].dwLastTime,eax
  .if  [esi].MsgHead.dwCmdId == CMD_MSG_UP
  mov  ebx,[edi].dwLastTime                    ;canmeng 萌 2
  sub  ebx,@lasttime                       ;canmeng 萌 2
  .if ebx<2000                       ;canmeng 萌 2
    mov [esi].MsgHead.dwCmdId,CMD_MSG_UP_RESP        ;canmeng 萌 2
    mov [esi].MsgHead.dwLength,sizeof MSG_HEAD+sizeof MSG_UP_RESP  ;canmeng 萌 2
    mov [esi].MsgUpResp.dbResult,1               ;canmeng 萌 2
    invoke send,_hSocket,esi,[esi].MsgHead.dwLength,0  ;canmeng 萌 2
  .else                       ;canmeng 萌 2
    invoke  _InsertMsgQueue,addr [edi].szUserName,addr [esi].MsgUp.szContent
  .endif            ;canmeng 萌 2
  .endif
.endif
.endw
```

3. 在 `Client1.asm` 中，

   ①在 `.const` 段中定义一个变量 `sztoofast`，当用户发言太快时就显示这个提示。

```assembly
.const
sztoofast  db  '您的发言太快,请休息片刻',0        ;canmeng 萌 2
```

   ②在 `_workThread` 的“循环接收消息”中进行判断，如果接受的数据包命令 ID 为 `CMD_MSG_UP_RESP`，那么就显示一个消息框。

```assembly
.if  @stMsg.MsgHead.dwCmdId == CMD_MSG_DOWN
……
.elseif  @stMsg.MsgHead.dwCmdId == CMD_MSG_UP_RESP     ;canmeng 萌 2
  .if  @stMsg.MsgUpResp.dbResult==1               ;canmeng 萌 2
  invoke  MessageBox,hWinMain,addr sztoofast,NULL,MB_OK     ;canmeng 萌 2
  .endif                       ;canmeng 萌 2
```

这样，第二步限制用户灌水也完成了。

附件是源代码。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 2 防止用户灌水.rar  
|
