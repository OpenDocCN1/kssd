
# WinImage

> 原文：[`www.pediy.com/kssd/pediy03/forum1103.htm`](https://www.pediy.com/kssd/pediy03/forum1103.htm)

破解心得之 WinImage 篇

作者：时空幻影

时间：2001 年 6 月 25 日

使用工具：`Fileinfo` `v2.43`、`W32DSM` 白金版汉化版、`TRW2000` `v1.22`

软件名称：`WinImage`

发布公司：`Gilles Vollant`

最新版本：`v5.0.5009`

操作系统：`Win9x/ME/NT4/2000`

软件说明：

可制作、解压磁盘映像(`iso` `bin` 等)

由于这个软件没有加壳，因此破解相对容易一些，且注册算法也不复杂，很适合初学者破解。

### 破解步骤

1、先把执行文件用 `Fileinfo` 查看一下有没有加壳，结果没有；

2、用 `W32DSM` 反编译该执行文件，查找出错字符串，找出比对点，然后根据比对点找出核心 `CALL`，记下该 `CALL` 的偏移地址，如这个软件核心 `CALL` 的偏移地址为 `0043407F`；

3、运行 `TRW2000`，再运行该软件，填好 `Name` 和 `Registration Code` 后，按 `Ctrl+N` 激活 `TRW2000`，然后键入 `"BPX HMEMCPY"`

按 `F5` 跳回程序，然后点 `OK` 就会被拦下，再键入 `"pmodule"`。

```
* Possible Reference to Dialog: REGISTER, CONTROL_ID:0816, ""

                                |
:0043405C 6816080000                 push 00000816
:00434061 FF7508                     push [ebp+08]
:00434064 FFD6                       call esi
:00434066 BF6CD04400                 mov edi, 0044D06C  <--pmodule 后到这里，D EAX 可看到自己输入的 Name

* Possible Ref to Menu: WINIMAGMENU, Item: "Create directory..."

                                |
:0043406B 6A7F                     push 0000007F
:0043406D 57                       push edi

* Possible Reference to Dialog: REGISTER, CONTROL_ID:0817, ""

                                |
:0043406E 6817080000                 push 00000817
:00434073 FF7508                     push [ebp+08]
:00434076 FFD6                       call esi
:00434078 6840D44400                 push 0044D440  <--D EDI 可看到输入的注册码
:0043407D 57                       push edi
:0043407E 53                       push ebx
:0043407F E89C5C0000                 call 00439D20  <--核心 CALL，按 F8 进入
:00434084 8B0D40D44400               mov ecx, dword ptr [0044D440]
:0043408A 83C40C                     add esp, 0000000C
:0043408D 33D2                       xor edx, edx
:0043408F A334D24400                   mov dword ptr [0044D234], eax
:00434094 3BC2                       cmp eax, edx
:00434096 5F                       pop edi
:00434097 5B                       pop ebx
:00434098 7406                       je 004340A0
:0043409A 890D04D44400               mov dword ptr [0044D404], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00434098(C)
|
:004340A0 391504D44400               cmp dword ptr [0044D404], edx
:004340A6 890D90D64400               mov dword ptr [0044D690], ecx
:004340AC 7505                       jne 004340B3
:004340AE A390D64400                   mov dword ptr [0044D690], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004340AC(C)
|
:004340B3 6A01                       push 00000001
:004340B5 3BC2                       cmp eax, edx
:004340B7 5E                       pop esi
:004340B8 7529                       jne 004340E3  <--暴破的话把这个 JNE 改成 JMP 即可，即把 75 改成 EB
:004340BA 6800200000                 push 00002000

* Possible Reference to String Resource ID=01069: "WinImage Registration"

                                |
:004340BF 682D040000                 push 0000042D
:004340C4 89350CD34400               mov dword ptr [0044D30C], esi
:004340CA 893564D44400               mov dword ptr [0044D464], esi
:004340D0 88156CD04400               mov byte ptr [0044D06C], dl
:004340D6 881568D44400               mov byte ptr [0044D468], dl

* Possible Reference to String Resource ID=01067: "Registering information is invalid"  <--注册失败对话框

                                |
:004340DC 682B040000                 push 0000042B
:004340E1 EB1B                       jmp 004340FE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004340B8(C)
|
:004340E3 6800200000                 push 00002000

* Possible Reference to String Resource ID=01069: "WinImage Registration"

                                |
:004340E8 682D040000                 push 0000042D
:004340ED 89150CD34400               mov dword ptr [0044D30C], edx
:004340F3 891564D44400               mov dword ptr [0044D464], edx

* Possible Reference to String Resource ID=01066: "Your registration code is valid.
                                                You are now a registered us"  <--  <--注册成功对话框

                                |
:004340F9 682A040000                 push 0000042A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004340E1(U)
|
:004340FE FF7508                     push [ebp+08]
:00434101 E84BDEFEFF                 call 00421F51
:00434106 83C410                     add esp, 00000010
:00434109 56                       push esi
:0043410A FF7508                     push [ebp+08]
```

按 `F8` 后会进入如下地方：

```
* Referenced by a CALL at Addresses:
|:0043407F  , :00439225

|
:00439D20 55                           push ebp
:00439D21 8BEC                         mov ebp, esp
:00439D23 81EC00020000                 sub esp, 00000200
:00439D29 56                           push esi
:00439D2A 8B7510                         mov esi, dword ptr [ebp+10]
:00439D2D 85F6                         test esi, esi
:00439D2F 57                           push edi
:00439D30 7403                         je 00439D35
:00439D32 832600                         and dword ptr [esi], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00439D30(C)
|
:00439D35 FF750C                         push [ebp+0C]
:00439D38 8D8500FFFFFF                   lea eax, dword ptr [ebp+FFFFFF00]
:00439D3E 50                           push eax
:00439D3F E8E0FEFFFF                   call 00439C24
:00439D44 FF7508                         push [ebp+08]
:00439D47 E804FFFFFF                   call 00439C50  <--核心 CALL，按 F8 进入
:00439D4C 8BF8                         mov edi, eax  <--EDI 和 EAX 中的内容均为第一个注册码
:00439D4E 83C40C                         add esp, 0000000C
:00439D51 81FF26DDDCB8                   cmp edi, B8DCDD26
:00439D57 0F84FE000000                   je 00439E5B  <--这里一定不能跳转
:00439D5D 8D8500FFFFFF                   lea eax, dword ptr [ebp+FFFFFF00]
:00439D63 50                           push eax
:00439D64 8D8500FEFFFF                   lea eax, dword ptr [ebp+FFFFFE00]
:00439D6A 57                           push edi
:00439D6B 50                           push eax
:00439D6C E862FFFFFF                   call 00439CD3
:00439D71 59                           pop ecx  <--按 D EAX 可以看到第一个正确的注册码
:00439D72 59                           pop ecx
:00439D73 50                           push eax

* Reference To: CRTDLL.strcmp, Ord:01CFh

                                |
:00439D74 E8FF3A0000                   Call 0043D878
:00439D79 59                           pop ecx
:00439D7A 85C0                         test eax, eax
:00439D7C 59                           pop ecx
:00439D7D 0F84A0000000                   je 00439E23
:00439D83 8D8500FFFFFF                   lea eax, dword ptr [ebp+FFFFFF00]
:00439D89 50                           push eax
:00439D8A 8D8748190514                   lea eax, dword ptr [edi+14051948]
:00439D90 50                           push eax
:00439D91 8D8500FEFFFF                   lea eax, dword ptr [ebp+FFFFFE00]
:00439D97 50                           push eax
:00439D98 E836FFFFFF                   call 00439CD3
:00439D9D 59                           pop ecx  <--按 D EAX 可以看到第二个正确的注册码
:00439D9E 59                           pop ecx
:00439D9F 50                           push eax

* Reference To: CRTDLL.strcmp, Ord:01CFh

                                |
:00439DA0 E8D33A0000                   Call 0043D878
:00439DA5 59                           pop ecx
:00439DA6 85C0                         test eax, eax
:00439DA8 59                           pop ecx
:00439DA9 7478                         je 00439E23
:00439DAB 8D8500FFFFFF                   lea eax, dword ptr [ebp+FFFFFF00]
:00439DB1 50                           push eax
```
