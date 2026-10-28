
`LISTBOX` `IDC_LISTBOX`,250,22,50,120,`LBS_STANDARD`                                ;canmeng萌4

`PUSHBUTTON` "复位",`IDC_RESET`,250,150,30,14                                        ;canmeng萌4

;注意:在define哪一行的后面不能加注释啊，

### 4. 在`Client1.asm`中，首先把在`Client.rc`中定义的常量写出来。

①

`IDC_COUNT1`        equ        2008                                                ;canmeng萌4

`IDC_LISTBOX`        equ        2009                                                ;canmeng萌4

`IDC_RESET`        equ        2010                                                        ;canmeng萌4

②定义一个全局变量，用来存储所有在线的用户名。

```asm
.data?
szUserBuffer        db        48 dup (?)                                        ;canmeng萌4，暂存所有的用户名字
```

③在循环接收消息中，对接收的消息头部命令ID进行判断，如果为`CMD_ONLINE_USER`，则说明数据包中包含所有的用户名称，于是把这些用户名称存到全局变量`szUserBuffdr`中，然后再一个个的显示在列表框中。

```
;*********************************************
; 循环接收消息
;*********************************************
```

```
……
.if        @stMsg.MsgHead.dwCmdId == CMD_ONLINE_USER        ;canmeng萌4
        mov        ecx,@stMsg.MsgOnlineUser.dwlength                ;canmeng萌4
        mov        ebx,offset szUserBuffer                                                ;canmeng萌4
        lea        edx,@stMsg.MsgOnlineUser.szcontent                        ;canmeng萌4
        .while        TRUE                                                                                 ;canmeng萌4
                sub        ecx,1                                                                                ;canmeng萌4
                mov        al,[edx]                                                                        ;canmeng萌4
                mov        [ebx],al                                        ;canmeng萌4
                add        edx,1                                                ;canmeng萌4
                add        ebx,1                                                ;canmeng萌4
                .break        .if ecx==0                                ;canmeng萌4
        .endw                                                                ;canmeng萌4
```

;注意:上面几句代码不能用`lstrcpy`代替

```asm
invoke        SendDlgItemMessage,hWinMain,IDC_LISTBOX,LB_RESETCONTENT,0,0
        mov        ecx,@stMsg.MsgOnlineUser.sznumber                ;canmeng萌4
        .while        TRUE                                                                                ;canmeng萌4
                sub ecx,1                                                                                ;canmeng萌4
                mov eax,12                                                                                ;canmeng萌4
                mul ecx                                ;注意,这一步会影响edx的值,非常非常重要
                mov edx,offset szUserBuffer                                                ;canmeng萌4
                add edx,eax                                                                                ;canmeng萌4
                push edx                                                                                ;canmeng萌4
                push ecx                                                                                ;canmeng萌4
                invoke SendDlgItemMessage,hWinMain,IDC_LISTBOX,LB_ADDSTRING,0,edx        ;canmeng萌4
                pop  ecx                                                                                ;canmeng萌4
                pop  edx                                                                                ;canmeng萌4
                .break .if ecx==0                                                                        ;canmeng萌4
        .endw                                                                                                ;canmeng萌4
```

`Invoke SetDlgItemInt,hWinMain,IDC_COUNT1,@stMsg.MsgOnlineUser.sznumber,FALSE`        ;canmeng萌4

`@@:`                                                                                                                ;canmeng萌4

`.elseif        @stMsg.MsgHead.dwCmdId == CMD_MSG_DOWN`

这样，每当有用户登录或者退出时，每个客户端都能及时的显示当前的用户了。

## 第二部分

### 5. 从此步骤开始，就正式实现悄悄话的功能了。在`Message.inc`中，`MSG_UP`和`MSG_DOWN`结构体中都增加爱“本消息的接收对象”字段。

①
```asm
MSG_UP                        struct
  szreceiver                db        12 dup (?)        ;                        ;canmeng萌4
  dwLength                dd        ?                ;后面内容字段的长度
  szContent                db        256 dup (?)        ;内容，不等长，长度由dwLength指定
MSG_UP                        ends
```

②
```asm
MSG_DOWN                struct
  szTime                db        20 dup (?)        ;canmeng萌1
  szSender                db        12 dup (?)        ;消息发送者
  szreceiver                db        12 dup (?)        ;消息接收者                                ;canmeng萌4
  dwLength                dd        ?                ;后面内容字段的长度
  szContent                db        256 dup (?)        ;内容，不等长，长度由dwLength指定
MSG_DOWN                ends
```

### 6. 在`_MsgQueue.asm`中，

①也要改变消息的结构:增加消息的接收对象

```asm
MSG_QUEUE_ITEM        struct                        ;队列中单条消息的格式定义
  dwMessageId        dd        ?                ;消息编号
  szSender        db        12 dup (?)        ;发送者
  szreceiver        db        12 dup (?)        ;接收者                                ;canmeng萌4
  szContent        db        256 dup (?)        ;聊天内容
  szTime        db        20 dup (?)        ;canmeng萌1
MSG_QUEUE_ITEM        ends
```

②在`_InsertMsgQueue`中，需要增加一个参数`_lpszreceiver`并添加相应的语句。

```asm
_InsertMsgQueue        proc        _lpszSender,_lpszContent,_lpszreceiver        ;canmeng萌4
……
invoke        lstrcpy,addr [esi].szreceiver,_lpszreceiver                                                ;canmeng萌4
```

③在`_GetMsgFromQueue`中，需要再增加一个参数`_lpszreceiver`并添加相应的语句。

```asm
_GetMsgFromQueue        proc        _dwMessageId,_lpszSender,_lpszContent,_lpszTime,_lpszreceiver;canmeng萌1,canmeng萌4
……
invoke        lstrcpy,_lpszreceiver,addr [esi].szreceiver                ;canmeng萌4
```

### 7. 在服务器端即`Server.asm`中，当用户发来一个登录数据包或者一个退出数据包，那么此消息的接收者当然是全部的客户端，我把此时的接收对象定义为“Allusers”。所以我在.const段中定义了一个全局变量`dwreceiver`。

①
```asm
.const
dwreceiver        db        'Allusers',0                        ;canmeng萌4
```

②这样当服务器收到客户端的登录数据包时，就把“Allusers”作为消息的接收对象插入到消息队列中去。

```
;*************************************************
; 广播：xxx 进入了聊天室
;*************************************************
```

```asm
invoke        lstrcpy,esi,addr [edi].szUserName
                invoke        lstrcat,esi,addr szUserLogin
                invoke        _InsertMsgQueue,addr szSysInfo,esi,addr dwreceiver                ;canmeng萌4
```

③相似的，当服务器收到客户端的推出数据包时，也把“Allusers”作为消息的接收对象插入到消息队列中。

```
;*************************************************
; 广播：xxx 退出了聊天室
;*************************************************
```

```asm
invoke        lstrcpy,esi,addr [edi].szUserName
invoke        lstrcat,esi,addr szUserLogout
invoke        _InsertMsgQueue,addr szSysInfo,addr @szBuffer,addr dwreceiver        ;canmeng萌4
```

④当收到用户的聊天语句数据包时，直接把数据包中所包含的接收对象字段提取出来就行了。

```
;*************************************************
; 循环处理消息
;*************************************************
```

```asm
invoke        _InsertMsgQueue,addr [edi].szUserName,addr [esi].MsgUp.szContent,\
                                                                        addr [esi].MsgUp.szreceiver                ;canmeng萌4
```
