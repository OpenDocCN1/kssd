
# HIDEIT 之 PEDIY

> 原文：[`www.pediy.com/kssd/pediy12/139610.html`](https://www.pediy.com/kssd/pediy12/139610.html)

【文章标题】: HIDEIT 之 PEDIY  
【文章作者】: wusha  
【软件名称】: HIDEIT  
【保护方式】: 无壳  
【编写语言】: `Microsoft Visual C++ 4.x`  
【使用工具】: `PEID 0.95`, `Olldbg1.10`, `lordpe`  
【操作平台】: `XP sp3`  
【软件介绍】: 非常好用的一个隐藏窗口的小工具  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

------------------------------------------------------------------------

【详细过程】

### 1、起因

最近在公司偷偷用 `PUTTY` 建立 `SSH` 来访问外网，可又担心被人发现，所以翻出以前的一个非常好用的隐藏窗口的小工具 `HIDEIT`，并设置成启动后在系统托盘，但有一次 `explorer.exe` 崩溃，结果 `HIDEIT` 就不见了，但进程还在，被它隐藏的窗口也没办法正常恢复出来了（强制结束 `HIDEIT` 进程可以恢复所有窗口，但感觉不爽），所以就想动动手术

### 2、分析

首先查找资料，在创建任务栏时，系统会以字符 `TaskbarCreated` 注册一个全局消息，并向所有顶级窗口广播该消息，因此在 `explorer.exe` 重建任务栏时，只要我们的程序捕捉到该消息，重建托盘图标即可，当然首先我们必须知道该消息的 `ID`，这个需要使用函数 `RegisterWindowMessage` 函数，原型如下

```
UINT RegisterWindowMessage(
LPCTSTR lpString   // message string
);
```

因为系统已经注册了该消息，所以我们调用函数返回的是该消息的唯一 `ID`

重建托盘图标需要调用函数 `Shell_NotifyIcon`，原型如下

```
BOOL Shell_NotifyIcon(DWORD dwMessage,
PNOTIFYICONDATA lpdata
);
```

查看程序的输入表，这两个函数都已经有了，所以不用另外添加了

### 3、开始 PEDIY

首先用 `lordpe` 查看节信息，发现 `.text` 节剩余空间很小，只有 `17200-17196=4` 个字节，所以必须要添加新节，如 `.wei`，大小为 `200h`

如图![](img/cc964cd2d705e918e94f0a184f17ceb1.jpg)

保存后报错“不是有效的 `win32` 程序”，查资料了解到还需要更改文件大小，于是用 `winhex` 在文件末尾追加了 `512` 字节 `0`，重新运行正常

使用 `OD` 载入程序，使用消息断点来到消息处理函数 `0040CC12` 处

代码:

```
0040CC12  /.  55            PUSH EBP
0040CC13  |.  B8 01000000   MOV EAX,1
0040CC18  |.  8BEC          MOV EBP,ESP
0040CC1A  |.  817C24 0C 600>CMP DWORD PTR SS:[ESP+C],360
0040CC22  |.  74 1A         JE SHORT HIDeit.0040CC3E
```

更改为

代码:

```
0040CC12   .  55            PUSH EBP
0040CC13   .  B8 01000000   MOV EAX,1
0040CC18   .  8BEC          MOV EBP,ESP
0040CC1A   .- E9 E1130200   JMP HIDEIT.0042E000
0040CC1F      90            NOP
0040CC20      90            NOP
0040CC21      90            NOP
0040CC22   .  74 1A         JE SHORT HIDEIT.0040CC3E
....
0042E000    837C24 0C 01    CMP DWORD PTR SS:[ESP+C],1                   添加代码处理 WM_CREATE 消息，得到任务栏重建消息 ID，并保存在 0042e200 处
0042E005    75 10           JNZ SHORT HIDEIT.0042E017
0042E007    68 90E04200     PUSH HIDEIT.0042E090                         ; ASCII "TaskbarCreated"
0042E00C    FF15 2C484200   CALL DWORD PTR DS:[<&USER32.RegisterWind>;  user32.RegisterWindowMessageA
0042E012    A3 00E24200     MOV DWORD PTR DS:[42E200],EAX
0042E017    8B4424 0C       MOV EAX,DWORD PTR SS:[ESP+C]
0042E01B    3B05 00E24200   CMP EAX,DWORD PTR DS:[42E200]                ;判读消息是否为任务栏重建消息
0042E021    75 0D           JNZ SHORT HIDEIT.0042E030
0042E023    68 00E14200     PUSH HIDEIT.0042E100                         ;NOTIFYICONDATA 结构指针(数据见后面）
0042E028    6A 00           PUSH 0                                      ;添加图标
0042E02A    FF15 C4464200   CALL DWORD PTR DS:[<&SHELL32.Shell_Notif>;  shell32.Shell_NotifyIconA
0042E030    817C24 0C 60030>CMP DWORD PTR SS:[ESP+C],360
0042E038  - E9 E5EBFDFF     JMP HIDEIT.0040CC22
```

这里碰到个问题，怎么去填充 `NOTIFYICONDATA` 结构呢?

其中 `NOTIFYCONDATA` 结构如下

```
typedef struct _NOTIFYICONDATA {
    DWORD  cbSize;
    HWND  hWnd;
    UINT  uID;
    UINT  uFlags;
    UINT  uCallbackMessage;
    HICON  hIcon;
    TCHAR  szTip[64];
    DWORD  dwState;
    DWORD  dwStateMask;
    TCHAR  szInfo[256];
    union {
        UINT  uTimeout;
        UINT  uVersion;
    };
    TCHAR  szInfoTitle[64];
    DWORD  dwInfoFlags;
    GUID  guidItem;
} NOTIFYICONDATA, *PNOTIFYICONDATA;
```

想了想，最好还是直接使用原来的 `NOTIFYICONDATA` 结构数据，于是通过对函数 `Shell_NotifyIcon` 下断，找到程序初次建立托盘图标的代码如下

代码:

```
004017B0  /$  83EC 58       SUB ESP,58
004017B3  |.  8B41 1C       MOV EAX,DWORD PTR DS:[ECX+1C]
004017B6  |.  8B4C24 5C     MOV ECX,DWORD PTR SS:[ESP+5C]
004017BA  |.  56            PUSH ESI
004017BB  |.  894424 08     MOV DWORD PTR SS:[ESP+8],EAX
004017BF  |.  57            PUSH EDI
004017C0  |.  8B7424 68     MOV ESI,DWORD PTR SS:[ESP+68]
004017C4  |.  894C24 10     MOV DWORD PTR SS:[ESP+10],ECX           ;uID
004017C8  |.  8B4424 6C     MOV EAX,DWORD PTR SS:[ESP+6C]         
004017CC  |.  897424 1C     MOV DWORD PTR SS:[ESP+1C],ESI           ;hIcon;    
004017D0  |.  C74424 08 580>MOV DWORD PTR SS:[ESP+8],58            ;cbsize
004017D8  |.  C74424 14 070>MOV DWORD PTR SS:[ESP+14],7             ;uFlags;
004017E0  |.  C74424 18 000>MOV DWORD PTR SS:[ESP+18],500          ;uCallbackMessage
004017E8  |.  85C0          TEST EAX,EAX
004017EA  |.  74 10         JE SHORT hideit.004017FC
004017EC  |.  6A 40         PUSH 40                                 ; /n = 40 (64.)
004017EE  |.  50            PUSH EAX                                ; |String2
004017EF  |.  8D4424 28     LEA EAX,DWORD PTR SS:[ESP+28]           ; |
004017F3  |.  50            PUSH EAX                                ; |String1
004017F4  |.  FF15 68464200 CALL DWORD PTR DS:[<&KERNEL32.lstrcpynA>>; \lstrcpynA
004017FA  |.  EB 05         JMP SHORT hideit.00401801
004017FC  |>  C64424 20 00  MOV BYTE PTR SS:[ESP+20],0
00401801  |>  8D4424 08     LEA EAX,DWORD PTR SS:[ESP+8]
00401805  |.  50            PUSH EAX                                ;PNOTIFYICONDATA
00401806  |.  6A 00         PUSH 0
00401808  |.  FF15 C4464200 CALL DWORD PTR DS:[<&SHELL32.Shell_Notif>;  shell32.Shell_NotifyIconA
```

郁闷的是调试发现这些 `NOTIFYICONDATA` 数据放在堆栈里了，只能想办法把这些数据事先备份出来，所以将上述代码更改如下

代码:

```
004017FC   > \C64424 20 00  MOV BYTE PTR SS:[ESP+20],0
00401801   >- E9 37C80200   JMP HIDEIT.0042E03D
00401806   .  6A 00         PUSH 0
00401808   .  FF15 C4464200 CALL DWORD PTR DS:[<&SHELL32.Shell_Notif>;  shell32.Shell_NotifyIconA
.....
0042E03D    8D4424 08       LEA EAX,DWORD PTR SS:[ESP+8]
0042E041    50             PUSH EAX
0042E042    57             PUSH EDI
0042E043    56             PUSH ESI
0042E044    BF 00E14200    MOV EDI,HIDEIT.0042E100                 ;数据备份到 0042E100
0042E049    8BF0           MOV ESI,EAX
0042E04B    B9 16000000    MOV ECX,16                              ;结构大小 58 字节/4=16
0042E050    66:F3:A5       REP MOVS WORD PTR ES:[EDI],WORD PTR DS:[esi]
0042E053    5E             POP ESI
0042E054    5F             POP EDI
0042E055  - E9 AC37FDFF    JMP HIDEIT.00401806
```

改好后运行测试，结束 `explorer.exe` 进程，呵呵，我们可爱的小图标又出来了。

------------------------------------------------------------------------

【版权声明】: 看雪论坛首发，转载请注明作者并保持文章的完整, 谢谢!

------------------------------------------------------------------------

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `hide.rar` |
