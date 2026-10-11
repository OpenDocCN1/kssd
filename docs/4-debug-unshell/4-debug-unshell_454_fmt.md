
# VideoSplitter V2.31 脱壳去暗桩+完美爆破

> 原文：[`www.pediy.com/kssd/pediy06/pediy6481.htm`](https://www.pediy.com/kssd/pediy06/pediy6481.htm)

【脱文作者】 `weiyi75[Dfcg]`

【作者邮箱】 `weiyi75@sohu.com`

【作者主页】 `Dfcg` 官方大本营

【使用工具】 `Ollydbg1.10b`, `ImportREC1.42`, `Loadpe`

【破解平台】 `Win2000/XP`

【软件名称】 `VideoSplitter V2.31`

【下载地址】 `http://www.boilsoft.com/videosplitter`

【软件简介】 一个 `AVI/MPEG/ASF/WMV` 分机的分割，合并工具，也包括了 `AVI` 和 `MPEG` 的播放器在里面，你可以很轻松的利用这个播放器把你喜欢的片断截取下来。

【软件大小】 `705KB`

【加壳方式】 `ASProtect 1.23 RC4 - 1.3.08.24 -> Alexey Solodovnikov`

【保护方式】 没有使用 `Asprotcet` 的 `Rsa` 算法保护，采用自行研究的加密算法，启动时校验注册码，`10` 天试用期限制，`SDK` 防脱壳暗桩。

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享。

--------------------------------------------------------------------------------

## 破解内容

### 第一部分，脱去 Asprotect1.23 外壳保护，补上 Stolen Code，修复 IAT。

首先 `Peid` 查壳，为 `ASProtect 1.23 RC4 - 1.3.08.24 -> Alexey Solodovnikov`。通过 `weiyi75` 作品 `Asprotect1.23rc4` 之 `Dephi` 语言篇经验，脱壳修复这个软件易如反掌。

复习一下，寄存器 `EBP=0012FFF0`（//这是未加壳程序停在入口点 `EBP` 的值，其它 `Dephi` 语言或编程语言这里的 `EBP` 值全部相同，这又说明什么，当 `Asprotect` 运行到这里或 `Stolen Code` 做手脚时 `EBP` 的值就为 `EBP=0012FFF0`，我们来具体试试。）

`Ollydbg` 载入程序，隐藏 `OD`，`OD` 异常设置忽略除内存异常外的全部异常。

```
00401000  >  68 01604B00     push VideoSpl.004B6001        //加壳程序入口点，这里先停一下，Asprotect 壳一般都有防脱壳暗桩，我们初学者采用被动式脱壳法比较好，发现问题再想办法解决。那么就先脱壳，修复 IAT，补上 Stolen Code。
00401005    E8 01000000     call VideoSpl.0040100B
0040100A    C3             retn
0040100B    C3             retn
0040100C    9B             wait
0040100D    3E:AE          scas byte ptr es:[edi]
0040100F    81E9 06A552EF  sub ecx, EF52A506
00401015    2D 1FCABC58     sub eax, 58BCCA1F
0040101A    56             push esi
0040101B    BC 9F7BA982     mov esp, 82A97B9F
00401020    4F             dec edi
00401021    40             inc eax
00401022    D6             salc
00401023    4F             dec edi
00401024    6E             outs dx, byte ptr es:[edi]
00401025    F9             stc
00401026    95             xchg eax, ebp
00401027    27             daa
00401028    47             inc edi
00401029    8371 C3 9A      xor dword ptr ds:[ecx-3D], FFFFFF9A
.........................................................................................
```

直接用 `OD` 脚本插件到达地球人都知道的位置。

```
012239EC    3100           xor dword ptr ds:[eax], eax
012239EE    64:8F05 0000000> pop dword ptr fs:[0]
012239F5    58             pop eax
012239F6    833D B07E2201 0> cmp dword ptr ds:[1227EB0], 0
012239FD    74 14         je short 01223A13
012239FF    6A 0C         push 0C
01223A01    B9 B07E2201     mov ecx, 1227EB0
01223A06    8D45 F8         lea eax, dword ptr ss:[ebp-8]
01223A09    BA 04000000     mov edx, 4
01223A0E    E8 2DD1FFFF     call 01220B40
01223A13    FF75 FC         push dword ptr ss:[ebp-4]
01223A16    FF75 F8         push dword ptr ss:[ebp-8]
01223A19    8B45 F4         mov eax, dword ptr ss:[ebp-C]
01223A1C    8338 00         cmp dword ptr ds:[eax], 0
01223A1F    74 02         je short 01223A23
01223A21    FF30           push dword ptr ds:[eax]
01223A23    FF75 F0         push dword ptr ss:[ebp-10]
01223A26    FF75 EC         push dword ptr ss:[ebp-14]
01223A29    C3             retn        //再这里下断点,Shift+F9 中断后清除断点，F8 步过。
..........................................................................................
01237838   /E9 08000000     jmp 01237845  //到这里。
0123783D   |40              inc eax
0123783E  ^|79 BE          jns short 012377FE
01237840   |1F              pop ds
01237841   |6C              ins byte ptr es:[edi], dx
01237842   |35 CA3BE80D     xor eax, 0DE83BCA
01237847    0000            add byte ptr ds:[eax], al
01237849    00B1 961704ED   add byte ptr ds:[ecx+ED041796], dh
0123784F    22B3 70E96E0F   and dh, byte ptr ds:[ebx+F6EE970]
..........................................................................................
```

我这次没有用内存镜像断点到 `OEP`，下面的这个方法可以发现 `Stolen Code`（如果有）+ `Oep` 二合一。

仍然是命令行断点

```
tc ebp==12fff0     //当前 EBP 内容为 12fff0 就被中断，略微看看就可以找到 Stolen Code(如果有)或 Oep
```

```
01237AE8   /3E:EB 02        jmp short 01237AED  //一会就来到这里,呵呵，C 语言就开始有 Stolen Code,往下全部用 F7 步过,转标签 1
01237AEB   |CD20 2EEB01C7   vxdcall C701EB2E
01237AF1    F3:             prefix rep:
01237AF2    EB 02          jmp short 01237AF6
01237AF4    CD20 6A505589   vxdcall 8955506A
01237AFA    74 24         je short 01237B20
01237AFC    04 F3         add al, 0F3
01237AFE    EB 02          jmp short 01237B02
01237B00    CD20 8D642404   vxdcall 424648D
01237B06    2E:EB 02       jmp short 01237B0B
..........................................................................................
```

标签 1

```
01237AED   /2E:EB 01        jmp short 01237AF1
01237AF1    F3:             prefix rep:
01237AF6    6A 50          push 50
01237AF8    55             push ebp
01237AF9    897424 04       mov dword ptr ss:[esp+4], esi
01237AFD    F3:             prefix rep:
01237B02    8D6424 04       lea esp, dword ptr ss:[esp+4]
01237B06    2E:EB 02       jmp short 01237B0B
01237B0B   /EB 01          jmp short 01237B0E
01237B0E    0FBCF1          bsf esi, ecx
01237B11    36:EB 01       jmp short 01237B15
01237B15    C1D6 C6         rcl esi, 0C6
01237B18    EB 01          jmp short 01237B1B
01237B1B    8DB7 F77A2301   lea esi, dword ptr ds:[edi+1237AF7]
01237B21    2BF7           sub esi, edi
01237B23    FF56 32        call dword ptr ds:[esi+32]
01237B30    5E             pop esi                             ; 01237B26
01237B31    2E:EB 02       jmp short 01237B36
01237B36    337424 28      xor esi, dword ptr ss:[esp+28]     ; kernel32.77E68EC8
01237B3A    5E             pop esi
01237B3B    F2:            prefix repne:
01237B3F    55             push ebp          //Stolen Code 第一句
01237B40    8BEC            mov ebp, esp     //执行完它后，EBP=ESP=12FFC0,打破了从跟踪到这里 EBP 的值一直为 12fff0 的情况，同时也让我们也可以知道入口代码就在附近。Stolen Code 第二句。
01237B42    6A FF           push -1          //Stolen Code 第三
01237B44    68 402E4800     push 482E40     //Stolen Code 第四句
01237B49    68 B8E54400     push 44E5B8     //Stolen Code 第五句
01237B4E    64:A1 00000000  mov eax, dword ptr fs:[0]  //Stolen Code 第六句
01237B54    F2:            prefix repne:
01237B58    50             push eax          //Stolen Code 第七句
01237B59    64:8925 0000000> mov dword ptr fs:[0], esp   //Stolen Code 第八句
01237B60    83EC 58        sub esp, 58             //Stolen Code 第九句
01237B63    F2:            prefix repne:
01237B67    53              push ebx            //Stolen Code 第十句
01237B68    F2:            prefix repne:
01237B6C    56              push esi             //Stolen Code 第十一句
01237B6D    F2:            prefix repne:
01237B71    57              push edi       //Stolen Code 第十二句
01237B72    8965 E8         mov dword ptr ss:[ebp-18], esp       //Stolen Code 第十三句
01237B75    3E:EB 02       jmp short 01237B7A
01237B7A    F2:            prefix repne:
01237B7E    68 04E44400     push 44E404
01237B83    68 467A2301     push 1237A46
01237B88    C3              retn
01237A46   /EB 01          jmp short 01237A49
01237A49    51               push ecx
01237A4A    57               push edi
01237A4B    9C               pushfd
01237A4C    FC               cld
01237A4D    BF 8A7A2301      mov edi, 1237A8A
01237A52    B9 5E140000      mov ecx, 145E
01237A57    F3:AA            rep stos byte ptr es:[edi]      //F7 可以看着它循环执行 Stolen Code,真是白痴，我们的 Stolen Code 已经到手了，它还在打扫战场。
01237A59    9D               popfd
01237A5A    5F               pop edi
```



```
01237A5B 59 pop ecx
01237A5C C3 retn //返回到伪 Oep
```

```
0044E3DE 0000 add byte ptr ds:[eax], al //真 Oep
0044E3E0 0000 add byte ptr ds:[eax], al
0044E3E2 0000 add byte ptr ds:[eax], al
0044E3E4 0000 add byte ptr ds:[eax], al
0044E3E6 0000 add byte ptr ds:[eax], al
0044E3E8 0000 add byte ptr ds:[eax], al
0044E3EA 0000 add byte ptr ds:[eax], al
0044E3EC 0000 add byte ptr ds:[eax], al
0044E3EE 0000 add byte ptr ds:[eax], al
0044E3F0 0000 add byte ptr ds:[eax], al
0044E3F2 0000 add byte ptr ds:[eax], al
0044E3F4 0000 add byte ptr ds:[eax], al
0044E3F6 0000 add byte ptr ds:[eax], al
0044E3F8 0000 add byte ptr ds:[eax], al
0044E3FA 0000 add byte ptr ds:[eax], al
0044E3FC 0000 add byte ptr ds:[eax], al
0044E3FE 0000 add byte ptr ds:[eax], al
0044E400 0000 add byte ptr ds:[eax], al
0044E402 0000 add byte ptr ds:[eax], al
0044E404 0000 add byte ptr ds:[eax], al
0044E404 FF15 EC314700 call dword ptr ds:[4731EC] //伪 Oep
0044E40A 33D2 xor edx, edx
0044E40C 8AD4 mov dl, ah
0044E40E 8915 28F44900 mov dword ptr ds:[49F428], edx
0044E414 8BC8 mov ecx, eax
0044E416 81E1 FF000000 and ecx, 0FF
0044E41C 890D 24F44900 mov dword ptr ds:[49F424], ecx
0044E422 C1E1 08 shl ecx, 8
0044E425 03CA add ecx, edx
0044E427 890D 20F44900 mov dword ptr ds:[49F420], ecx
0044E42D C1E8 10 shr eax, 10
0044E430 A3 1CF44900 mov dword ptr ds:[49F41C], eax
0044E435 6A 01 push 1
0044E437 E8 882C0000 call VideoSpl.004510C4
```

..........................................................................................

整理一下 Stolen Code

.......................................................

```
push ebp //Stolen Code
mov ebp, esp
push -1
push 4822F0
push 44E000 //一共 13 句代码
mov eax, dword ptr fs:[0]
push eax
mov dword ptr fs:[0], esp
sub esp, 58
push ebx
push esi
push edi
mov dword ptr ss:[ebp-18], esp //Stolen Code
```

.......................................................

```
0044E3DE 55 push ebp //补好后复制代码到文件中，直接用 Loadpe 脱壳。
0044E3DF 8BEC mov ebp, esp
0044E3E1 6A FF push -1
0044E3E3 68 402E4800 push VideoSpl.00482E40
0044E3E8 68 B8E54400 push VideoSpl.0044E5B8
0044E3ED 64:A1 00000000 mov eax, dword ptr fs:[0]
0044E3F3 50 push eax
0044E3F4 64:8925 0000000>mov dword ptr fs:[0], esp
0044E3FB 83EC 58 sub esp, 58
0044E3FE 53 push ebx
0044E3FF 56 push esi
0044E400 57 push edi
0044E401 8965 E8 mov dword ptr ss:[ebp-18], esp
0044E404 FF15 EC314700 call dword ptr ds:[4731EC]
0044E40A 33D2 xor edx, edx
0044E40C 8AD4 mov dl, ah
0044E40E 8915 28F44900 mov dword ptr ds:[49F428], edx
0044E414 8BC8 mov ecx, eax
0044E416 81E1 FF000000 and ecx, 0FF
0044E41C 890D 24F44900 mov dword ptr ds:[49F424], ecx
0044E422 C1E1 08 shl ecx, 8
0044E425 03CA add ecx, edx
0044E427 890D 20F44900 mov dword ptr ds:[49F420], ecx
0044E42D C1E8 10 shr eax, 10
0044E430 A3 1CF44900 mov dword ptr ds:[49F41C], eax
0044E435 6A 01 push 1
0044E437 E8 882C0000 call VideoSpl.004510C4
```

.......................................................

### IAT 修复

运行 `ImportREC`，`OEP` 填入 `4E3DE`，自动搜索，获得输入信息，先用跟踪等级 1 修复，然后用 `Asprotect1.3` 插件修复剩下的几个指针，修复先脱壳的 `Dumped.exe`。

### 第二部分，去除暗桩

`OD` 载入修复的文件，`F9` 运行。

状态栏/提示内存写错误。

堆栈友好提示

```
0012AFD8 0040DD53 返回到 dumped\_.0040DD53 来自 01221C08 //错误来自 0040DD53
0012AFDC 00474758 ASCII ".RMF"
0012AFE0 00000004
0012AFE4 00000000
0012AFE8 00496850 dumped\_.00496850
```

.......................................................

那么 `Ctrl+G` 去 `0040DD53` 看看。

```
0040DD4D |. FF15 8CD64800 call dword ptr ds:[48D68C] //ds:[48D68C]指向壳中,跟踪原程序发现其作用是简单的防脱壳校验。
0040DD53 |. E8 13F10500 call dumped\_.0046CE6B //无可疑
```

`Od` 载入原程序

```
0040DD4D |. FF15 8CD64800 call dword ptr ds:[48D68C]
```

```
01221C08 55 push ebp //F7 跟进
01221C09 8BEC mov ebp, esp
01221C0B 53 push ebx
01221C0C 56 push esi
01221C0D 8B5D 0C mov ebx, dword ptr ss:[ebp+C]
01221C10 8B75 08 mov esi, dword ptr ss:[ebp+8]
01221C13 8BC3 mov eax, ebx
01221C15 E8 F208FFFF call 0121250C
01221C1A A3 A4632201 mov dword ptr ds:[12263A4], eax
01221C1F 8B15 A4632201 mov edx, dword ptr ds:[12263A4]
01221C25 8BC6 mov eax, esi
01221C27 8BCB mov ecx, ebx
01221C29 E8 EA09FFFF call 01212618
01221C2E 891D 347E2201 mov dword ptr ds:[1227E34], ebx
01221C34 53 push ebx
01221C35 8BCE mov ecx, esi
01221C37 8B15 307E2201 mov edx, dword ptr ds:[1227E30]
01221C3D A1 2C7E2201 mov eax, dword ptr ds:[1227E2C]
01221C42 E8 F9EEFFFF call 01220B40
01221C47 833D E47A2201 0>cmp dword ptr ds:[1227AE4], 0
01221C4E 75 05 jnz short 01221C55
01221C50 E8 4FFFFFFF call 01221BA4
01221C55 5E pop esi
01221C56 5B pop ebx
01221C57 5D pop ebp
01221C58 C2 0800 retn 8 //跟进，晕，好多代码啊，壳是不可能搬回的,它进壳中的过程我们没必有知道，但它从壳中取出代码动态修改程序领空代码我们脱壳后是需要的,可是这个也不是解码 Call。
```

`Ctrl+F` 在程序空间里查找命令：`retn 8` 随便找一个吧：

命令行 `DD 48D68C`

把 `48D68C` 处的 `01221C08` 改为：`00403B74`

复制到程序里，保存覆盖文件。

`Ctrl+F2` 重新加载，`F9` 运行。

程序界面出来了，尝试分割文件立即出错。

状态栏/提示内存写错误。

堆栈友好提示

```
0012ADB8 0041116A 返回到 dumped\_.0041116A 来自 01221BA4
0012ADBC 0012B8E8
0012ADC0 0012B8E8
0012ADC4 0040FF02 返回到 dumped\_.0040FF02 来自 dumped\_.00411160
0012ADC8 0012B8E8
0012ADCC 00474A68 dumped\_.00474A68
```

那么 `Ctrl+G` 去 `0041116A` 看看

```
00411164 . FF15 84D64800 call dword ptr ds:[48D684] //这里搞的飞机,ds:[48D684]指向壳中,跟踪原程序发现其作用是动态解码。
0041116A . E9 76000000 jmp dumped\_.004111E5
```

Btw:用 `Winxp` 脱壳时这里堆栈出错没有提示。

```
01224350 3100 xor dword ptr ds:[eax], eax //只有再硬盘指纹过后的 pre-dip 中分析了。
01224352 EB 01 jmp short 01224355
01224354 68 648F0500 push 58F64
01224359 0000 add byte ptr ds:[eax], al
0122435B 00EB add bl, ch
0122435D 02E8 add ch, al
0122435F 0158 80 add dword ptr ds:[eax-80], ebx
01224362 3D A07E2201 cmp eax, 1227EA0
01224367 007414 E8 add byte ptr ss:[esp+edx-18], dh
0122436B 2D D2FFFF84 sub eax, 84FFFFD2
01224370 C0740B E8 F0 sal byte ptr ds:[ebx+ecx-18], 0F0
01224375 DEFF fdivp st(7), st
01224377 FF84C0 740289EC inc dword ptr ds:[eax+eax*8+EC890274]
0122437E B2 01 mov dl, 1
01224380 A1 140C2201 mov eax, dword ptr ds:[1220C14]
01224385 E8 06C9FFFF call 01220C90
```

..........................................................

```
0012FF3C 0012FF44 指针到下一个 SEH 记录
0012FF40 01224307 SE 句柄
0012FF44 0012FFE0 指针到下一个 SEH 记录
0012FF48 01224C49 SE 句柄
```



```
0012FF4C   0012FF90
0012FF50   01210000
0012FF54   011F0000
0012FF58   01224138
0012FF5C   0123A318  ASCII "ARInMgCAUXY="
```

```
0040D490    8B4424 04       mov eax, dword ptr ss:[esp+4]
0040D494    A3 84D64800     mov dword ptr ds:[48D684], eax //pre-dip4,校验注册码，与硬盘指纹计算后生成一个密匙，在部分壳里，会解压 N 段代码起到防止脱壳的功能。
0040D499    C2 0400         retn 4
0040D49C    90            nop
0040D49D    90            nop
0040D49E    90            nop
0040D49F    90            nop
0040D4A0    8B4424 04       mov eax, dword ptr ss:[esp+4]
0040D4A4    A3 88D64800     mov dword ptr ds:[48D688], eax
0040D4A9    C2 0400         retn 4
0040D4AC    90            nop
0040D4AD    90            nop
0040D4AE    90            nop
0040D4AF    90            nop
0040D4B0    8B4424 04       mov eax, dword ptr ss:[esp+4]
0040D4B4    A3 8CD64800     mov dword ptr ds:[48D68C], eax
```

于是再脱壳程序，`dd 48D684` 下内存访问断点，分割时同样断在我们刚才错误的地方。

```
00411164    FF15 84D64800   call dword ptr ds:[48D684] //原程序和脱壳程序错误地方。
0041116A    E9 76000000     jmp dumped_.004111E5
0041116F    C590 74E92A3B   lds edx, fword ptr ds:[eax+3B2AE974]
00411175    3E:46           inc esi
00411177    15 BD0C6586     adc eax, 86650CBD
0041117C    C7              ???                           ; 未知命令
0041117D    6E            outs dx, byte ptr es:[edi]
0041117E    EB 01           jmp short dumped_.00411181
00411180    D5 C2           aad 0C2
00411182    D4 4A           aam 4A
00411184    D6            salc
00411185    45            inc ebp
00411186    18AA 3E4837A5   sbb byte ptr ds:[edx+A537483E], ch
0041118C    03DB          add ebx, ebx
0041118E    B8 68E13DED     mov eax, ED3DE168
00411193    DF29          fild qword ptr ds:[ecx]
00411195    ED            in eax, dx
00411196    FA            cli
00411197    A4            movs byte ptr es:[edi], byte ptr ds:[esi]
00411198    B2 DA         mov dl, 0DA
0041119A    8817          mov byte ptr ds:[edi], dl
0041119C    AB            stos dword ptr es:[edi]
```

```
01221BA4    833D A4632201 0>cmp dword ptr ds:[12263A4], 0 //跟进 00411164
01221BAB    75 0D         jnz short 01221BBA
01221BAD    A1 407D2201     mov eax, dword ptr ds:[1227D40]
01221BB2    8B15 447D2201   mov edx, dword ptr ds:[1227D44]
01221BB8    EB 0B         jmp short 01221BC5
01221BBA    A1 A4632201     mov eax, dword ptr ds:[12263A4]
01221BBF    8B15 347E2201   mov edx, dword ptr ds:[1227E34]
01221BC5    85C0          test eax, eax
01221BC7    74 0B         je short 01221BD4
01221BC9    85D2          test edx, edx
01221BCB    74 07         je short 01221BD4
01221BCD    52            push edx
01221BCE    50            push eax
01221BCF    E8 60FDFFFF     call 01221934
01221BD4    C3            retn   //具体什么作用返回就知道是动态解码用的。
```

```
00411164    FF15 84D64800   call dword ptr ds:[48D684]
0041116A    E9 01000000     jmp VideoSpl.00411170    //解码后跳转的位置都不一样，早就计划好了的。
0041116F    FD            std
00411170    8D4424 04       lea eax, dword ptr ss:[esp+4]
00411174    50            push eax
00411175    6A 04         push 4
00411177    56            push esi
00411178    68 A0FC4000     push VideoSpl.0040FCA0
0041117D    6A 00         push 0
0041117F    6A 00         push 0
00411181    FF15 5C334700   call dword ptr ds:[47335C]
00411187    85C0          test eax, eax
00411189    8986 D0450000   mov dword ptr ds:[esi+45D0], eax
0041118F    74 4E         je short VideoSpl.004111DF
00411191    6A 01         push 1
00411193    8D8E 64290000   lea ecx, dword ptr ds:[esi+2964]
00411199    C786 E0450000 0>mov dword ptr ds:[esi+45E0], 0
004111A3    E8 5E250500     call VideoSpl.00463706
004111A8    6A 00         push 0
004111AA    8D8E 582E0000   lea ecx, dword ptr ds:[esi+2E58]
004111B0    E8 51250500     call VideoSpl.00463706
004111B5    8B8E 68330000   mov ecx, dword ptr ds:[esi+3368]
004111BB    6A 00         push 0
004111BD    6A 00         push 0
004111BF    68 02040000     push 402
004111C4    51            push ecx
004111C5    FF15 28364700   call dword ptr ds:[473628]             ; USER32.SendMessageA
004111CB    8BCE          mov ecx, esi
004111CD    E8 5EF9FFFF     call VideoSpl.00410B30
004111D2    8B96 D0450000   mov edx, dword ptr ds:[esi+45D0]
004111D8    52            push edx
004111D9    FF15 60334700   call dword ptr ds:[473360]
004111DF    EB 04         jmp short VideoSpl.004111E5
004111E1    F9            stc
004111E2    BA C5305E83     mov edx, 835E30C5
004111E7    C404FF        les eax, fword ptr ds:[edi+edi*8]
004111EA    25 88D64800     and eax, 48D688
004111EF    90            nop
004111F0    56            push esi
004111F1    8BF1          mov esi, ecx
004111F3    E8 45A50500     call VideoSpl.0046B73D
004111F8    F64424 08 01    test byte ptr ss:[esp+8], 1
004111FD    74 09         je short VideoSpl.00411208
004111FF    56            push esi
00411200    E8 A4ED0400     call VideoSpl.0045FFA9
00411205    83C4 04         add esp, 4
00411208    8BC6          mov eax, esi
0041120A    5E            pop esi
0041120B    C2 0400         retn 4      //为了不漏掉解码代码也不多动脑筋，从 0041116A 到这个 Call 全部二进程复制。
```

二进制粘贴到脱壳程序的相应位置。

然后还是从程序里找一个

```
0040102B   .  C3            retn
```

给

```
00411164    FF15 84D64800   call dword ptr ds:[48D684]
```

模拟解码。

命令行 `DD 48D684`

修改 `01221BA4` 为 `0040102B`

复制所有修改到程序里面，保存为一个文件，`Ctrl+F2` 重新载入测试功能。

执行时仍然出错，继续找原因。

```
004111D9   .  FF15 60334700 call dword ptr ds:[<&kernel32.ResumeThre>; ResumeThread
004111DF   >  EB 04          jmp short dumped_.004111E5
004111E1      F9            db F9
004111E2      BA            db BA
004111E3   .  C530          lds esi, fword ptr ds:[eax]
004111E5   >  5E            pop esi
004111E6   .  83C4 04        add esp, 4
004111E9   .- FF25 88D64800 jmp dword ptr ds:[48D688] //刚才复制的代码这里指向壳中。
```

```
01221BD8    833D A4632201 0>cmp dword ptr ds:[12263A4], 0 //跟进
01221BDF    75 0D         jnz short 01221BEE
01221BE1    A1 407D2201     mov eax, dword ptr ds:[1227D40]
01221BE6    8B15 447D2201   mov edx, dword ptr ds:[1227D44]
01221BEC    EB 0B         jmp short 01221BF9
01221BEE    A1 A4632201     mov eax, dword ptr ds:[12263A4]
01221BF3    8B15 347E2201   mov edx, dword ptr ds:[1227E34]
01221BF9    85C0          test eax, eax
01221BFB    74 09         je short 01221C06
01221BFD    85D2          test edx, edx
01221BFF    74 05         je short 01221C06
01221C01    E8 A6FDFFFF     call 012219AC
01221C06    C3            retn  //还是一个 retn 返回程序，简单的验证是否脱壳。
```

于是简单应付它

```
004111E9    - FF25 88D64800 jmp dword ptr ds:[48D688]
004111EF      90            nop
```

改为

```
004111E9      C3            retn
004111EA      90            nop
004111EB      90            nop
004111EC      90            nop
004111ED      90            nop
004111EE      90            nop
004111EF      90            nop
```

保存最后的修改，所有暗桩清除,成功分割文件。

破解分析参考我的 `AVI MPEG RM WMV Joiner4.11` 脱壳+破解,过程查不多。

```
0040DED8  |.  E8 53D5FFFF   call dumped_.0040B430
0040DEDD  |.  8B5424 30     mov edx, dword ptr ss:[esp+30]
0040DEE1  |.  8B4C24 2C     mov ecx, dword ptr ss:[esp+2C]
0040DEE5  |.  8B4424 28     mov eax, dword ptr ss:[esp+28]
0040DEE9  |.  33D1          xor edx, ecx
0040DEEB  |.  83C4 14       add esp, 14
0040DEEE  |.  3BC2          cmp eax, edx
0040DEF0  |.  75 0A         jnz short dumped_.0040DEFC            //爆破点,jz short 0040DEFC
```



`0040DEF2  |.  C786 00460000>mov dword ptr ds:[esi+4600], 1       //全局标志位赋值。`

`0040DEFC  |>  39AE 00460000 cmp dword ptr ds:[esi+4600], ebp`

`0040DF02  |.  75 10        jnz short dumped_.0040DF14`

这样改法，执行分割时功能被`Nop`掉了，晕。

只好将

`0040E8BC    8B86 00460000 mov eax, dword ptr ds:[esi+4600]`

改为

`0040E8BC  .  8986 00460000 mov dword ptr ds:[esi+4600], eax`

`RVA=0000E8BC`处为

`8B`—— 未注册版

`89`—— 注册版

没时间仔细看，这个程序的变态度比它的两个兄长`ASF-AVI-RM-WMV Repair V1.41`、`AVI MPEG RM WMV Joiner4.61`差多了。

`4dd466d0e1e0cc123afbe504da2462ca`
