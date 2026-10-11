
# [B]UsbTrace v1.2 注册算法简单分析[/B]

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-807.htm`](https://www.pediy.com/kssd/pediy07/pediy7-807.htm)

**UsbTrace v1.2 注册算法简单分析**

**【作者】winndy[FCG][DFCG][PYG]**

**【目标】**UsbTrace v1.2

**【官方网址】http://www.sysnucleus.com**

**【破解说明】** 这个工具是前不久在 exetools 下载的，有 SLZ 提供的注册码，但是没有注册机，于是自己跟了一下。
非常简单，写了一个注册机，然后发到 exetools 了。还好，有人乐意与我分享我的成果，有个人还在 exetools PM 我，让我把 `keygen` 发到他邮箱，他说他只做了个补丁出来。
昨天逛 wasm.ru 时发现上面提供 UsbTrace v1.2 下载，看了一下，我的 `keygen` 在里面，有点意外的惊喜。
**人生诸多不如意，瞬间的惊喜值得留念。**
于是把算法跟踪简要写出来，以鼓舞自己。
原本想用它来做 FCG 学员区的教程的，可是觉得太简单了，对不起学员，所以一直没写破文。
难一点的又没搞出来，跟踪 `usbMonitor` 后发现要取跟踪驱动，诸多原因，没做下去了，等有时间再看看。
在这里先向学员区的成员道个歉。
破文中有不妥之处，还请指正。

**【破解过程】**

### 一、搜索提示字符串，确定注码验证过程

```
代码:

0043EC30   .  64:A1 00>mov eax,dword ptr fs:[0]
0043EC36   .  6A FF    push -1
0043EC38   .  68 B8144>push USBTrace.004814B8
0043EC3D   .  50       push eax
0043EC3E   .  64:8925 >mov dword ptr fs:[0],esp
0043EC45   .  83EC 08  sub esp,8
0043EC48   .  53       push ebx
0043EC49   .  55       push ebp
0043EC4A   .  56       push esi
0043EC4B   .  57       push edi
0043EC4C   .  8BF1     mov esi,ecx
0043EC4E   .  6A 01    push 1
0043EC50   .  E8 2A370>call USBTrace.0046237F
0043EC55   .  E8 08600>call USBTrace.00464C62
0043EC5A   .  85C0     test eax,eax
0043EC5C   .  74 0B    je short USBTrace.0043EC69               ===>
0043EC5E   .  8B10     mov edx,dword ptr ds:[eax]
0043EC60   .  8BC8     mov ecx,eax
0043EC62   .  FF52 74  call dword ptr ds:[edx+74]
0043EC65   .  8BE8     mov ebp,eax
0043EC67   .  EB 02    jmp short USBTrace.0043EC6B
0043EC69   >  33ED     xor ebp,ebp
0043EC6B   >  51       push ecx
0043EC6C   .  8D7E 5C  lea edi,dword ptr ds:[esi+5C]
0043EC6F   .  8BCC     mov ecx,esp
0043EC71   .  896424 1>mov dword ptr ss:[esp+14],esp
0043EC75   .  57       push edi
0043EC76   .  E8 17460>call USBTrace.00463292
0043EC7B   .  51       push ecx
0043EC7C   .  8D5E 60  lea ebx,dword ptr ds:[esi+60]
0043EC7F   .  8BCC     mov ecx,esp
0043EC81   .  896424 1>mov dword ptr ss:[esp+1C],esp
0043EC85   .  53       push ebx
0043EC86   .  C74424 2>mov dword ptr ss:[esp+2C],0
0043EC8E   .  E8 FF450>call USBTrace.00463292
0043EC93   .  8BCD     mov ecx,ebp                                ; |
0043EC95   .  C74424 2>mov dword ptr ss:[esp+28],-1               ; |
0043EC9D   .  E8 AEF0F>call USBTrace.0043DD50                    ; \USBTrace.0043DD50
0043ECA2   .  85C0     test eax,eax                               ===>eax 是 call 的返回值，eax＝1 则成功，eax＝0 则失败
0043ECA4   .  74 60    je short USBTrace.0043ED06               ===>不能跳
0043ECA6   .  E8 45830>call USBTrace.00476FF0
0043ECAB   .  8B68 04  mov ebp,dword ptr ds:[eax+4]
0043ECAE   .  8B03     mov eax,dword ptr ds:[ebx]
0043ECB0   .  50       push eax                                   ; /Arg3
0043ECB1   .  68 ACB74>push USBTrace.004AB7AC                     ; |Arg2 = 004AB7AC ASCII "UserName"
0043ECB6   .  68 A4B74>push USBTrace.004AB7A4                     ; |Arg1 = 004AB7A4 ASCII "RegInfo"
0043ECBB   .  8BCD     mov ecx,ebp                                ; |
0043ECBD   .  E8 F8B90>call USBTrace.0046A6BA                    ; \USBTrace.0046A6BA
0043ECC2   .  8B0F     mov ecx,dword ptr ds:[edi]
0043ECC4   .  51       push ecx                                   ; /Arg3
0043ECC5   .  68 9CB74>push USBTrace.004AB79C                     ; |Arg2 = 004AB79C ASCII "RegCode"
0043ECCA   .  68 A4B74>push USBTrace.004AB7A4                     ; |Arg1 = 004AB7A4 ASCII "RegInfo"
0043ECCF   .  8BCD     mov ecx,ebp                                ; |
0043ECD1   .  E8 E4B90>call USBTrace.0046A6BA                    ; \USBTrace.0046A6BA
0043ECD6   .  6A 40    push 40
0043ECD8   .  68 A4B14>push USBTrace.004AB1A4                     ;  ASCII "USBTrace"
0043ECDD   .  68 64B84>push USBTrace.004AB864                     ;  ASCII "Congratulations. You have successfully registered
0043ECE2   .  8BCE     mov ecx,esi
0043ECE4   .  E8 912A0>call USBTrace.0046177A
0043ECE9   .  8B16     mov edx,dword ptr ds:[esi]
0043ECEB   .  8BCE     mov ecx,esi
0043ECED   .  FF92 C40>call dword ptr ds:[edx+C4]
0043ECF3   .  8B4C24 1>mov ecx,dword ptr ss:[esp+18]
0043ECF7   .  64:890D >mov dword ptr fs:[0],ecx
0043ECFE   .  5F       pop edi
0043ECFF   .  5E       pop esi
0043ED00   .  5D       pop ebp
0043ED01   .  5B       pop ebx
0043ED02   .  83C4 14  add esp,14
0043ED05   .  C3       retn
0043ED06   >  6A 30    push 30
0043ED08   .  68 50B84>push USBTrace.004AB850                     ;  ASCII "Registration Failed"
0043ED0D   .  68 34B84>push USBTrace.004AB834                     ;  ASCII "Invalid registration code"
0043ED12   .  8BCE     mov ecx,esi
0043ED14   .  E8 612A0>call USBTrace.0046177A
0043ED19   .  8B06     mov eax,dword ptr ds:[esi]
0043ED1B   .  8BCE     mov ecx,esi
0043ED1D   .  FF90 C80>call dword ptr ds:[eax+C8]
0043ED23   .  8B4C24 1>mov ecx,dword ptr ss:[esp+18]
0043ED27   .  5F       pop edi
0043ED28   .  5E       pop esi
0043ED29   .  5D       pop ebp
0043ED2A   .  64:890D >mov dword ptr fs:[0],ecx
0043ED31   .  5B       pop ebx
0043ED32   .  83C4 14  add esp,14
0043ED35   .  C3       retn
```

### 二、0043DD50 ，也不知道取什么标题好，反正就是顺藤摸瓜的思想，追进来看看

```
代码:

0043DD50  /$  6A FF    push -1
0043DD52  |.  68 60134>push USBTrace.00481360                     ;  SE handler installation
0043DD57  |.  64:A1 00>mov eax,dword ptr fs:[0]
0043DD5D  |.  50       push eax
0043DD5E  |.  64:8925 >mov dword ptr fs:[0],esp
0043DD65  |.  83EC 0C  sub esp,0C
0043DD68  |.  56       push esi
0043DD69  |.  57       push edi
0043DD6A  |.  8BF9     mov edi,ecx
0043DD6C  |.  C74424 1>mov dword ptr ss:[esp+1C],0
0043DD74  |.  A1 A0DD4>mov eax,dword ptr ds:[4ADDA0]
0043DD79  |.  894424 0>mov dword ptr ss:[esp+C],eax
0043DD7D  |.  894424 0>mov dword ptr ss:[esp+8],eax
0043DD81  |.  8D4424 0>lea eax,dword ptr ss:[esp+C]
0043DD85  |.  8D5424 2>lea edx,dword ptr ss:[esp+24]
0043DD89  |.  50       push eax
0043DD8A  |.  51       push ecx
0043DD8B  |.  8BCC     mov ecx,esp
0043DD8D  |.  896424 1>mov dword ptr ss:[esp+18],esp
0043DD91  |.  52       push edx
0043DD92  |.  C64424 2>mov byte ptr ss:[esp+28],3
0043DD97  |.  E8 F6540>call USBTrace.00463292
0043DD9C  |.  8BCF     mov ecx,edi                                ; |
0043DD9E  |.  E8 DD000>call USBTrace.0043DE80                    ; \\USBTrace.0043DE80 ＝＞这个 call 要跟进
0043DDA3  |.  8BF0     mov esi,eax                                ；eax 是标志，应该为 1
0043DDA5  |.  85F6     test esi,esi
0043DDA7  |.  74 3F    je short USBTrace.0043DDE8
0043DDA9  |.  8D4424 2>lea eax,dword ptr ss:[esp+28]
0043DDAD  |.  8D4C24 0>lea ecx,dword ptr ss:[esp+8]
0043DDB1  |.  50       push eax
0043DDB2  |.  E8 9F580>call USBTrace.00463656
0043DDB7  |.  8D4C24 0>lea ecx,dword ptr ss:[esp+C]
0043DDBB  |.  51       push ecx
0043DDBC  |.  8BCF     mov ecx,edi
0043DDBE  |.  E8 7D000>call USBTrace.0043DE40
```

F8 过来之后，`EAX` `003F7028` ASCII `"41*1-8*50-*43*"`，把 Tls 字符串都用`**`取代了。

```
代码:

0043DDC3  |.  8D5424 0>lea edx,dword ptr ss:[esp+8]
0043DDC7  |.  8BCF     mov ecx,edi
0043DDC9  |.  52       push edx
0043DDCA  |.  E8 71000>call USBTrace.0043DE40
```

F8 过来，`EAX` `003F6F38` ASCII `"1234567890"`，这是假注册码。



```assembly
0043DDCF   |.  8B4424 0>mov eax,dword ptr ss:[esp+8]
0043DDD3   |.  8B4C24 0>mov ecx,dword ptr ss:[esp+C]
0043DDD7   |.  50       push eax                       ; /Arg2＝假注册码
0043DDD8   |.  51       push ecx                       ; |Arg1＝真注册码
0043DDD9   |.  E8 E8FC0>call USBTrace.0044DAC6         ; \USBTrace.0044DAC6
```

上面的 `call` 就是注册码比较过程。我没仔细看，试了几次，可以得出这样的结论：`Tls` 的字符串没有参加验证过程。

```assembly
0043DDDE   |.  8BF0     mov esi,eax                     ；eax 为 1，正确；为 0，错误
0043DDE0   |.  83C4 08  add esp,8
0043DDE3   |.  F7DE     neg esi
0043DDE5   |.  1BF6     sbb esi,esi
0043DDE7   |.  46       inc esi
0043DDE8   |>  8D4C24 0>lea ecx,dword ptr ss:[esp+8]
0043DDEC   |.  C64424 1>mov byte ptr ss:[esp+1C],2
0043DDF1   |.  E8 27570>call USBTrace.0046351D
0043DDF6   |.  8D4C24 0>lea ecx,dword ptr ss:[esp+C]
0043DDFA   |.  C64424 1>mov byte ptr ss:[esp+1C],1
0043DDFF   |.  E8 19570>call USBTrace.0046351D
0043DE04   |.  8D4C24 2>lea ecx,dword ptr ss:[esp+24]
0043DE08   |.  C64424 1>mov byte ptr ss:[esp+1C],0
0043DE0D   |.  E8 0B570>call USBTrace.0046351D
0043DE12   |.  8D4C24 2>lea ecx,dword ptr ss:[esp+28]
0043DE16   |.  C74424 1>mov dword ptr ss:[esp+1C],-1
0043DE1E   |.  E8 FA560>call USBTrace.0046351D
0043DE23   |.  8B4C24 1>mov ecx,dword ptr ss:[esp+14]
0043DE27   |.  8BC6     mov eax,esi
0043DE29   |.  5F       pop edi
0043DE2A   |.  64:890D >mov dword ptr fs:[0],ecx
0043DE31   |.  5E       pop esi
0043DE32   |.  83C4 18  add esp,18
0043DE35   \.  C2 0800  retn 8
```

### 三、`0043DE80` 生成注册码的 `call`

```assembly
0043DE80   /$  6A FF    push -1
0043DE82   |.  68 A4134>push USBTrace.004813A4           ;  SE handler installation
0043DE87   |.  64:A1 00>mov eax,dword ptr fs:[0]
0043DE8D   |.  50       push eax
0043DE8E   |.  64:8925 >mov dword ptr fs:[0],esp
0043DE95   |.  81EC 300>sub esp,130
0043DE9B   |.  53       push ebx
0043DE9C   |.  55       push ebp
0043DE9D   |.  56       push esi
0043DE9E   |.  57       push edi
0043DE9F   |.  A1 A0DD4>mov eax,dword ptr ds:[4ADDA0]
0043DEA4   |.  33F6     xor esi,esi
0043DEA6   |.  89B424 4>mov dword ptr ss:[esp+148],esi
0043DEAD   |.  33ED     xor ebp,ebp
0043DEAF   |.  894424 3>mov dword ptr ss:[esp+34],eax
0043DEB3   |.  897424 2>mov dword ptr ss:[esp+28],esi
0043DEB7   |.  897424 2>mov dword ptr ss:[esp+24],esi
0043DEBB   |.  897424 3>mov dword ptr ss:[esp+38],esi
0043DEBF   |.  33FF     xor edi,edi
0043DEC1   |.  894424 1>mov dword ptr ss:[esp+18],eax
0043DEC5   |.  894424 1>mov dword ptr ss:[esp+1C],eax
0043DEC9   |.  894424 1>mov dword ptr ss:[esp+14],eax
0043DECD   |.  8B8424 5>mov eax,dword ptr ss:[esp+150]
0043DED4   |.  C68424 4>mov byte ptr ss:[esp+148],4
0043DEDC   |.  8B40 F8  mov eax,dword ptr ds:[eax-8]        ；用户名的长度
0043DEDF   |.  83F8 04  cmp eax,4                            ；长度要大于等于 4
0043DEE2   |.  0F8C F40>jl USBTrace.0043E0DC
0043DEE8   |.  E8 03910>call USBTrace.00476FF0
0043DEED   |.  8B40 08  mov eax,dword ptr ds:[eax+8]        ；eax＝00400000，上面 call 的返回值
0043DEF0   |.  8D4C24 3>lea ecx,dword ptr ss:[esp+3C]
0043DEF4   |.  68 04010>push 104                             ; /BufSize = 104 (260.)
0043DEF9   |.  51       push ecx                               ; |PathBuffer＝0012F228
0043DEFA   |.  50       push eax                               ; |hModule = 00400000 (USBTrace)
0043DEFB   |.  FF15 EC4>call dword ptr ds:[<&KERNEL32.GetModuleF>; \GetModuleFileNameA
```

`GetModuleFileName` function retrieves the full path and filename for the executable file containing the specified module.

```text
0012F228  43 3A 5C 50 72 6F 67 72  C:\Progr
0012F230  61 6D 20 46 69 6C 65 73  am Files
0012F238  5C 55 53 42 54 72 61 63  \USBTrac
0012F240  65 5C 55 53 42 54 72 61  e\USBTra
0012F248  63 65 2E 65 78 65 00     ce.exe.
```

`eax=0026`，是上面的完整路径的长度。

下面的几个 API 我就不解释了，可以对照 `win32 programmer's reference` 去理解。

前面一小段代码的意思是，得到 `USBTrace.exe` 文件的版本信息。

```assembly
0043DF01   |.  85C0     test eax,eax
0043DF03   |.  0F84 D30>je USBTrace.0043E0DC
0043DF09   |.  8D5424 2>lea edx,dword ptr ss:[esp+24]
0043DF0D   |.  8D4424 3>lea eax,dword ptr ss:[esp+3C]
0043DF11   |.  52       push edx                       ; /pHandle
0043DF12   |.  50       push eax                       ; |FileName
0043DF13   |.  E8 84700>call <jmp.&VERSION.GetFileVersionInfoSiz>; \GetFileVersionInfoSizeA
0043DF18   |.  8BD8     mov ebx,eax
0043DF1A   |.  85DB     test ebx,ebx
0043DF1C   |.  0F84 BA0>je USBTrace.0043E0DC
0043DF22   |.  53       push ebx
0043DF23   |.  E8 27040>call USBTrace.0044E34F
0043DF28   |.  83C4 04  add esp,4
0043DF2B   |.  894424 2>mov dword ptr ss:[esp+20],eax
0043DF2F   |.  85C0     test eax,eax
0043DF31   |.  0F84 A50>je USBTrace.0043E0DC
0043DF37   |.  8B4C24 2>mov ecx,dword ptr ss:[esp+24]
0043DF3B   |.  50       push eax                       ; /Buffer＝003F82B0
0043DF3C   |.  53       push ebx                       ; |BufSize
0043DF3D   |.  8D5424 4>lea edx,dword ptr ss:[esp+44]          ; |
0043DF41   |.  51       push ecx                       ; |Reserved
0043DF42   |.  52       push edx                       ; |FileName
0043DF43   |.  E8 4E700>call <jmp.&VERSION.GetFileVersionInfoA>  ; \GetFileVersionInfoA
0043DF48   |.  85C0     test eax,eax
0043DF4A   |.  0F84 7F0>je USBTrace.0043E0CF
0043DF50   |.  8B5424 2>mov edx,dword ptr ss:[esp+20]        ；003F82B0，Buffer
0043DF54   |.  8D4424 3>lea eax,dword ptr ss:[esp+38]
0043DF58   |.  8D4C24 2>lea ecx,dword ptr ss:[esp+28]
0043DF5C   |.  50       push eax                       ; /pValueSize
0043DF5D   |.  51       push ecx                       ; |ppValue
0043DF5E   |.  68 00E94>push USBTrace.0049E900         ; |pSubBlock = "\
0043DF63   |.  52       push edx                       ; |pBlock＝003F82B0
0043DF64   |.  E8 27700>call <jmp.&VERSION.VerQueryValueA>     ; \VerQueryValueA
```

`pValueSize` 所指的 size 为 0，可见并未取出数据。

`pBlock＝003F82B0`，指向取出的版本信息。

下面是部分版本信息，可用资源查看工具去看。

```text
003F82B0  84 03 34 00 00 00 56 00  ?4...V.
003F82B8  53 00 5F 00 56 00 45 00  S.\_.V.E.
003F82C0  52 00 53 00 49 00 4F 00  R.S.I.O.
003F82C8  4E 00 5F 00 49 00 4E 00  N.\_.I.N.
003F82D0  46 00 4F 00 00 00 00 00  F.O.....
003F82D8  BD 04 EF FE 00 00 01 00  ?稔...
003F82E0  00 00 01 00 02 00 00 00  ......
003F82E8  00 00 01 00 02 00 00 00  ......
003F82F0  3F 00 00 00 00 00 00 00  ?.......
003F82F8  04 00 00 00 01 00 00 00  ......
003F8300  00 00 00 00 00 00 00 00  ........
003F8308  00 00 00 00 E4 02 00 00  ....?..
```

下面是版本数据的结构：

```c
VS_VERSION_INFO {
    WORD  wLength;
    WORD  wValueLength;
    WORD  wType;
    WCHAR szKey[];
    WORD  Padding1[];
    VS_FIXEDFILEINFO Value;
    WORD  Padding2[];
    WORD  Children[];
};

typedef struct _VS_FIXEDFILEINFO {  // vsffi
    DWORD dwSignature;               //Contains the value 0xFEEFO4BD
    DWORD dwStrucVersion;
    DWORD dwFileVersionMS;
    DWORD dwFileVersionLS;
    DWORD dwProductVersionMS;
    DWORD dwProductVersionLS;
    DWORD dwFileFlagsMask;
    DWORD dwFileFlags;
    DWORD dwFileOS;
    DWORD dwFileType;
    DWORD dwFileSubtype;
    DWORD dwFileDateMS;
    DWORD dwFileDateLS;
} VS_FIXEDFILEINFO;
```

对照上面的数据可以很好的去理解结构的各个字段。

```assembly
0043DF69   |.  85C0     test eax,eax
0043DF6B   |.  0F84 5E0>je USBTrace.0043E0CF
```

下面是关键部分了，取出版本中的信息。



下面一句执行后 `eax` 为 `003F82D8`，指向 `VS_FIXEDFILEINFO` 结构，通过 `dwSignature` 可认出来。

```
0043DF71   |.  8B4424 2>mov eax,dword ptr ss:[esp+28]   ；
0043DF75   |.  33C9     xor ecx,ecx
0043DF77   |.  8B50 14  mov edx,dword ptr ds:[eax+14]   ；取出 dwProductVersionLS，00000002
0043DF7A   |.  66:8B48 >mov cx,word ptr ds:[eax+14]     ；cx＝0002，取低 16 位
0043DF7E   |.  C1EA 10  shr edx,10                      ；右移 16 位，实际上是取高 16 位
0043DF81   |.  83C1 02  add ecx,2
0043DF84   |.  83C2 05  add edx,5
0043DF87   |.  51       push ecx                        ；压入参数 4
0043DF88   |.  52       push edx                        ；压入参数 5
0043DF89   |.  8B50 10  mov edx,dword ptr ds:[eax+10]   ；取出 dwProductVersionMS，00010000
0043DF8C   |.  33C9     xor ecx,ecx
0043DF8E   |.  66:8B48 >mov cx,word ptr ds:[eax+10]     ；cx＝0000，取低 16 位
0043DF92   |.  8D4424 2>lea eax,dword ptr ss:[esp+20]
0043DF96   |.  C1EA 10  shr edx,10                      ；取高 16 位
0043DF99   |.  41       inc ecx
0043DF9A   |.  83C2 03  add edx,3
0043DF9D   |.  51       push ecx                        ；压入参数 1
0043DF9E   |.  52       push edx                        ；压入参数 4
0043DF9F   |.  68 90B74>push USBTrace.004AB790          ;  ASCII "%d%d%d%d"
0043DFA4   |.  50       push eax
0043DFA5   |.  E8 53E90>call USBTrace.0045C8FD
```

这个 `call` 之后，观察寄存器：`ECX 003F89E8` ASCII "4154"

上面那个 `call` 的作用就是把压入的 4 个十进制整数连接为字符串。

```
0043DFAA   |.  8BAC24 6>mov ebp,dword ptr ss:[esp+168]  ;EBP 003F7078 ASCII "winndy"
0043DFB1   |.  83C4 18  add esp,18
0043DFB4   |.  33C9     xor ecx,ecx                     ;计数器
0043DFB6   |.  8B55 F8  mov edx,dword ptr ss:[ebp-8]    ；edx＝6，这是用户名（winndy）的长度，
0043DFB9   |.  85D2     test edx,edx
0043DFBB   |.  7E 14    jle short USBTrace.0043DFD1
0043DFBD   |>  0FBE0429 /movsx eax,byte ptr ds:[ecx+ebp]
0043DFC1   |.  8D1C80   |lea ebx,dword ptr ds:[eax+eax*4]
0043DFC4   |.  8D04D8   |lea eax,dword ptr ds:[eax+ebx*8]
0043DFC7   |.  03F0     |add esi,eax
0043DFC9   |.  41       |inc ecx
0043DFCA   |.  3BCA     |cmp ecx,edx
0043DFCC   |.  8D3446   |lea esi,dword ptr ds:[esi+eax*2]
0043DFCF   |.^ 7C EC    \jl short USBTrace.0043DFBD
```

这段循环的代码用来处理用户名，结果保存在 `esi` 中。

用 `win32 asm` 做注册机的话，直接拿去用就是了。

```
0043DFD1   |>  8BC6     mov eax,esi
0043DFD3   |.  33D2     xor edx,edx
0043DFD5   |.  B9 0F270>mov ecx,270F                   ；270F 的十进制是 9999，对 9999 取模，保证长度为 4
0043DFDA   |.  F7F1     div ecx
0043DFDC   |.  52       push edx                        ；压入余数
0043DFDD   |.  8D5424 2>lea edx,dword ptr ss:[esp+20]
0043DFE1   |.  68 8CB74>push USBTrace.004AB78C          ;  ASCII "%4d"
0043DFE6   |.  52       push edx
0043DFE7   |.  E8 11E90>call USBTrace.0045C8FD          ；函数功能：整数变为字符串
0043DFEC   |.  83C4 0C  add esp,0C
0043DFEF   |.  8D4C24 1>lea ecx,dword ptr ss:[esp+14]
0043DFF3   |.  68 84B74>push USBTrace.004AB784          ;  ASCII "\*\*\*\*"
0043DFF8   |.  E8 A9560>call USBTrace.004636A6
0043DFFD   |.  6A 00    push 0
0043DFFF   |.  E8 28040>call USBTrace.0044E42C
0043E004   |.  50       push eax
0043E005   |.  E8 8DFA0>call USBTrace.0044DA97
0043E00A   |.  83C4 08  add esp,8
0043E00D   |.  33F6     xor esi,esi
0043E00F   |.  B3 5A    mov bl,5A
0043E011   |>  E8 8EFA0>/call USBTrace.0044DAA4
0043E016   |.  99       |cdq
0043E017   |.  B9 1A000>|mov ecx,1A
0043E01C   |.  F7F9     |idiv ecx
0043E01E   |.  80C2 41  |add dl,41
0043E021   |.  3AD3     |cmp dl,bl
0043E023   |.  885424 1>|mov byte ptr ss:[esp+10],dl
0043E027   |.  7E 04    |jle short USBTrace.0043E02D
0043E029   |.  885C24 1>|mov byte ptr ss:[esp+10],bl
0043E02D   |>  8B5424 1>|mov edx,dword ptr ss:[esp+10]
0043E031   |.  8D4C24 1>|lea ecx,dword ptr ss:[esp+14]
0043E035   |.  52       |push edx
0043E036   |.  56       |push esi
0043E037   |.  E8 275A0>|call USBTrace.00463A63
0043E03C   |.  46       |inc esi
0043E03D   |.  83FE 04  |cmp esi,4
0043E040   |.^ 7C CF    \jl short USBTrace.0043E011
```

上面这段循环代码过后，`eax` 指向一个长为 4 的字符串，不是固定值。

跟进 `0044DAA4`，再跟进：发现调用了 `TlsGetValue`，

```
0045116B   |.  FF35 E0F>push dword ptr ds:[4AF1E0]     ; /TlsIndex = C
00451171   |.  8BF8     mov edi,eax                     ; |
00451173   |.  FF15 E84>call dword ptr ds:[<&KERNEL32.TlsG>; \TlsGetValue
```

查看 `api`：

The `TlsGetValue` function retrieves the value in the calling thread's thread local storage (`TLS`) slot for a specified `TLS` index. Each thread of a process has its own slot for each `TLS` index.

If the function succeeds, the return value is the value stored in the calling thread's `TLS` slot associated with the specified index.

If the function fails, the return value is zero. To get extended error information, call `GetLastError`.

这段代码取出 `TLS` 中的数据然后连接成字符串。但不固定。

所以这段代码不需看，即可写注册机。提高效率！想学习的话，可以研究一下。

```
0043E042   |.  8BAC24 5>mov ebp,dword ptr ss:[esp+154]
0043E049   |.  68 74B74>push USBTrace.004AB774          ;  ASCII "\*\*\*\*\*\*\*\*\*\*\*"，注意长为 12
0043E04E   |.  8BCD     mov ecx,ebp
0043E050   |.  E8 51560>call USBTrace.004636A6          ；先 F8
0043E055   |.  33F6     xor esi,esi
0043E057   |>  8B4424 1>/mov eax,dword ptr ss:[esp+18]  ；esp＋18 指向版本信息字符串
0043E05B   |.  8A0C30   |mov cl,byte ptr ds:[eax+esi]
0043E05E   |.  8BC7     |mov eax,edi
0043E060   |.  884C24 1>|mov byte ptr ss:[esp+10],cl
0043E064   |.  8BCD     |mov ecx,ebp
0043E066   |.  8B5424 1>|mov edx,dword ptr ss:[esp+10]
0043E06A   |.  47       |inc edi
0043E06B   |.  52       |push edx
0043E06C   |.  50       |push eax
0043E06D   |.  E8 F1590>|call USBTrace.00463A63         ；先 F8，猜不出来再进去看，注意观察面板和寄存器
0043E072   |.  8B4424 1>|mov eax,dword ptr ss:[esp+1C]  ；esp＋1C 指向用户名得来的字符串
0043E076   |.  8A0C30   |mov cl,byte ptr ds:[eax+esi]
0043E079   |.  8BC7     |mov eax,edi
0043E07B   |.  884C24 2>|mov byte ptr ss:[esp+2C],cl
0043E07F   |.  8BCD     |mov ecx,ebp
0043E081   |.  8B5424 2>|mov edx,dword ptr ss:[esp+2C]
0043E085   |.  47       |inc edi
0043E086   |.  52       |push edx
0043E087   |.  50       |push eax
0043E088   |.  E8 D6590>|call USBTrace.00463A63
0043E08D   |.  8B4424 1>|mov eax,dword ptr ss:[esp+14]  ；esp＋14 指向 Tls 字符串
0043E091   |.  8A0C30   |mov cl,byte ptr ds:[eax+esi]
0043E094   |.  8BC7     |mov eax,edi
0043E096   |.  884C24 3>|mov byte ptr ss:[esp+30],cl
0043E09A   |.  8BCD     |mov ecx,ebp
0043E09C   |.  8B5424 3>|mov edx,dword ptr ss:[esp+30]
0043E0A0   |.  47       |inc edi
0043E0A1   |.  52       |push edx
0043E0A2   |.  50       |push eax
0043E0A3   |.  E8 BB590>|call USBTrace.00463A63
0043E0A8   |.  46       |inc esi
0043E0A9   |.  83FE 04  |cmp esi,4
0043E0AC   |.^ 7C A9    \jl short USBTrace.0043E057
```

通过观察，可以发现上面的循环依次取三个串中的字符，连接起来。

`EAX 003F7028` ASCII "41O18Z50N43Y"



```
代码:

* * *

0043E0AE   |.  68 14E74>push USBTrace.0049E714
0043E0B3   |.  6A 04    push 4
0043E0B5   |.  8BCD     mov ecx,ebp
0043E0B7   |.  E8 D6E00>call USBTrace.0045C192
0043E0BC   |.  68 14E74>push USBTrace.0049E714
0043E0C1   |.  6A 09    push 9
0043E0C3   |.  8BCD     mov ecx,ebp
0043E0C5   |.  E8 C8E00>call USBTrace.0045C192
0043E0CA   |.  BD 01000>mov ebp,1
0043E0CF   |>  8B4424 2>mov eax,dword ptr ss:\[esp+20\]
0043E0D3   |.  50       push eax
0043E0D4   |.  E8 2E020>call USBTrace.0044E307
0043E0D9   |.  83C4 04  add esp,4
0043E0DC   |>  8D4C24 1>lea ecx,dword ptr ss:\[esp+14\]
0043E0E0   |.  C68424 4>mov byte ptr ss:\[esp+148\],3
0043E0E8   |.  E8 30540>call USBTrace.0046351D
0043E0ED   |.  8D4C24 1>lea ecx,dword ptr ss:\[esp+1C\]
0043E0F1   |.  C68424 4>mov byte ptr ss:\[esp+148\],2
0043E0F9   |.  E8 1F540>call USBTrace.0046351D
0043E0FE   |.  8D4C24 1>lea ecx,dword ptr ss:\[esp+18\]
0043E102   |.  C68424 4>mov byte ptr ss:\[esp+148\],1
0043E10A   |.  E8 0E540>call USBTrace.0046351D
0043E10F   |.  8D4C24 3>lea ecx,dword ptr ss:\[esp+34\]
0043E113   |.  C68424 4>mov byte ptr ss:\[esp+148\],0
0043E11B   |.  E8 FD530>call USBTrace.0046351D
0043E120   |.  8D8C24 5>lea ecx,dword ptr ss:\[esp+150\]
0043E127   |.  C78424 4>mov dword ptr ss:\[esp+148\],-1
0043E132   |.  E8 E6530>call USBTrace.0046351D
0043E137   |.  8B8C24 4>mov ecx,dword ptr ss:\[esp+140\]
0043E13E   |.  5F       pop edi
0043E13F   |.  8BC5     mov eax,ebp
0043E141   |.  5E       pop esi
0043E142   |.  5D       pop ebp
0043E143   |.  5B       pop ebx
0043E144   |.  64:890D >mov dword ptr fs:\[0\],ecx
0043E14B   |.  81C4 3C0>add esp,13C
0043E151   \.  C2 0800  retn 8

* * *

```

一路 `F8` 过来，停到 `retn 8`

看堆栈：

```
0012F338    0043DDA3         返回到 USBTrace.0043DDA3 来自 USBTrace.0043DE80

0012F33C    003F7078         ASCII "winndy"

0012F340    0012F350

0012F344    004B04C0         ASCII "8o?"

0012F348    004B0464         USBTrace.004B0464

0012F34C    004ADDB4         USBTrace.004ADDB4

0012F350    003F7028         ASCII "41O1-8Z50-N43Y"
```

注册码出现了。

```
0043E0B3   |.  6A 04    push 4

0043E0C1   |.  6A 09    push 9
```

注意到两个－在第 4 和第 9 个位置，有兴趣的可以跟进去看看。

`retn 8` 执行后，返回继续看。

### 算法总结

1. 根据版本号确定一个字符串。
2. 根据用户名确定一个字符串。
3. 用 `TlsGetValue` 得到一个字符串，但是没用的。
4. 把字符串连接起来。

字符串的具体产生过程见代码分析，都很简单。

### 感谢

all of you！～

### 文章完成时间、地点

2005.12.28，武汉

### 注册机

win32 asm source

`4dd466d0e1e0cc123afbe504da2462ca`
