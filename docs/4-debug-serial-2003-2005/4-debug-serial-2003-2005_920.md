
# 天天个人助理(DailyPim)注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-473.htm`](https://www.pediy.com/kssd/pediy07/pediy7-473.htm)

【破文标题】天天个人助理(DailyPim)注册算法分析

【破文作者】`xu_wh`

【软件名称】天天个人助理(DailyPim) V3.53

【下载地址】
http://www.haoz.cn/Software/Catalog59/4746.html
http://www.997.cn/SoftView/SoftView_4599.html

【软件简介】DailyPim 是一款个人日常信息管理的软件，具有的功能有日记本、资料管理、文件管理、日程管理、地址簿、网页快抓、收发消息、收发文件、邮箱监视器、查询天气、火车、航班、电话区号、邮政编码、定时关机等。DailyPim 是国内功能最强大的个人信息管理软件。

【调试环境】Win xp、`Ollydbg` 1.10、`PEID` V0.93、`LordPE` 1.4、`Import RE` 1.6

【作者声明】有人曾经写出了注册机，但是发现注册机程序有问题。后来再网上找到了相关的注册算法，但是发现注册算法不完整，至少不能适用于 3.53 这个版本。为了弥补这个缺漏，对这个程序重新分析。其中参考了 snake 大侠的破解说明，节省了我许多时间。在此表示感谢！

【破解过程】

## 一、脱壳

先用 `PEID` 对程序 `DailyPim.exe` 进行检测，检测的结果是“Nothing found *”。只好手工脱壳。用 `OD` 载入：

```
00C52082 d> 60                 pushad                                ；一路 F8，好好走。
00C52083   E8 00000000        call dailypim.00C52088
00C52088   5D                 pop ebp
00C52089   81ED 22A54500      sub ebp,dailypim.0045A522
00C5208F   8DBD 1CA54500      lea edi,dword ptr ss:[ebp+45A51C]
00C52095   81EF 82000000      sub edi,82
00C5209B   89BD 84A84500      mov dword ptr ss:[ebp+45A884],edi
00C520A1   8B4F 18            mov ecx,dword ptr ds:[edi+18]
00C520A4   89FE               mov esi,edi
00C520A6   0377 14            add esi,dword ptr ds:[edi+14]
00C520A9   8B47 10            mov eax,dword ptr ds:[edi+10]
00C520AC   E8 2A060000        call dailypim.00C526DB
00C520B1   8B4F 24            mov ecx,dword ptr ds:[edi+24]
00C520B4   89FE               mov esi,edi
00C520B6   0377 20            add esi,dword ptr ds:[edi+20]
00C520B9   8B47 1C            mov eax,dword ptr ds:[edi+1C]
00C520BC   E8 1A060000        call dailypim.00C526DB
00C520C1   6A 00              push 0
......
00C522E8   39FE               cmp esi,edi
00C522EA   7D 12              jge short dailypim.00C522FE
00C522EC   8B06               mov eax,dword ptr ds:[esi]
00C522EE   83C6 04            add esi,4
00C522F1   09C0               or eax,eax
00C522F3 ^ 74 F3             je short dailypim.00C522E8
00C522F5   50                 push eax
00C522F6   FF95 08A94500      call near dword ptr ss:[ebp+45A908]
00C522FC ^ EB EA             jmp short dailypim.00C522E8            ；不要回跳
00C522FE   8DB5 58A84500      lea esi,dword ptr ss:[ebp+45A858]     ；F4 到这里
......
00C4B0C7   8B46 04            mov eax,dword ptr ds:[esi+4]
00C4B0CA   05 0E010000        add eax,10E
00C4B0CF   6A 04              push 4
00C4B0D1   68 00100000        push 1000
00C4B0D6   50                 push eax
00C4B0D7   6A 00              push 0
00C4B0D9   FF95 4D050000      call near dword ptr ss:[ebp+54D]
00C4B0DF   8985 52010000     mov dword ptr ss:[ebp+152],eax
00C4B0E5   56                 push esi
00C4B0E6   8B1E               mov ebx,dword ptr ds:[esi]
00C4B0E8   039D 22040000     add ebx,dword ptr ss:[ebp+422]
00C4B0EE   FFB5 56010000     push dword ptr ss:[ebp+156]
00C4B0F4   FF76 04           push dword ptr ds:[esi+4]
00C4B0F7   50                 push eax
00C4B0F8   53                 push ebx
00C4B0F9   E8 6E050000        call dailypim.00C4B66C
00C4B0FE   B3 01              mov bl,1
00C4B100   80FB 00            cmp bl,0
00C4B103   75 5E              jnz short dailypim.00C4B163
00C4B105   FE85 EC000000     inc byte ptr ss:[ebp+EC]
00C4B10B   8B3E               mov edi,dword ptr ds:[esi]
00C4B10D   03BD 22040000     add edi,dword ptr ss:[ebp+422]
00C4B113   FF37               push dword ptr ds:[edi]
00C4B115   C607 C3            mov byte ptr ds:[edi],0C3
00C4B118   FFD7               call near edi
00C4B11A   8F07               pop dword ptr ds:[edi]
00C4B11C   50                 push eax
00C4B11D   51                 push ecx
00C4B11E   56                 push esi
00C4B11F   53                 push ebx
00C4B120   8BC8               mov ecx,eax
00C4B122   83E9 06            sub ecx,6
00C4B125   8BB5 52010000     mov esi,dword ptr ss:[ebp+152]
00C4B12B   33DB               xor ebx,ebx
00C4B12D   0BC9               or ecx,ecx
00C4B12F   74 2E              je short dailypim.00C4B15F
00C4B131   78 2C              js short dailypim.00C4B15F
00C4B133   AC                 lods byte ptr ds:[esi]
00C4B134   3C E8              cmp al,0E8
00C4B136   74 0A              je short dailypim.00C4B142
00C4B138   EB 00              jmp short dailypim.00C4B13A
00C4B13A   3C E9              cmp al,0E9
00C4B13C   74 04              je short dailypim.00C4B142
00C4B13E   43                 inc ebx
00C4B13F   49                 dec ecx
00C4B140 ^ EB EB             jmp short dailypim.00C4B12D
00C4B142   8B06               mov eax,dword ptr ds:[esi]
00C4B144   EB 0A              jmp short dailypim.00C4B150
00C4B146   803E 00            cmp byte ptr ds:[esi],0
00C4B149 ^ 75 F3             jnz short dailypim.00C4B13E
00C4B14B   24 00              and al,0
00C4B14D   C1C0 18            rol eax,18
00C4B150   2BC3               sub eax,ebx
00C4B152   8906               mov dword ptr ds:[esi],eax
00C4B154   83C3 05            add ebx,5
00C4B157   83C6 04            add esi,4
00C4B15A   83E9 05            sub ecx,5
00C4B15D ^ EB CE             jmp short dailypim.00C4B12D
00C4B15F   5B                 pop ebx
00C4B160   5E                 pop esi
00C4B161   59                 pop ecx
00C4B162   58                 pop eax
00C4B163   EB 08              jmp short dailypim.00C4B16D
00C4B165   0000               add byte ptr ds:[eax],al
00C4B167   DE01               fiadd word ptr ds:[ecx]
00C4B169   0000               add byte ptr ds:[eax],al
00C4B16B   D001               rol byte ptr ds:[ecx],1
00C4B16D   8BC8               mov ecx,eax
00C4B16F   8B3E               mov edi,dword ptr ds:[esi]
00C4B171   03BD 22040000     add edi,dword ptr ss:[ebp+422]
00C4B177   8BB5 52010000     mov esi,dword ptr ss:[ebp+152]
00C4B17D   C1F9 02            sar ecx,2
00C4B180   F3:A5             rep movs dword ptr es:[edi],dword ptr ds>
00C4B182   8BC8               mov ecx,eax
00C4B184   83E1 03            and ecx,3
00C4B187   F3:A4             rep movs byte ptr es:[edi],byte ptr ds:[>
00C4B189   5E                 pop esi
00C4B18A   68 00800000        push 8000
00C4B18F   6A 00              push 0
00C4B191   FFB5 52010000     push dword ptr ss:[ebp+152]
00C4B197   FF95 51050000     call near dword ptr ss:[ebp+551]
00C4B19D   83C6 08            add esi,8
00C4B1A0   833E 00            cmp dword ptr ds:[esi],0
00C4B1A3 ^ 0F85 1EFFFFFF     jnz dailypim.00C4B0C7                  ；不要往回跳
00C4B1A9   68 00800000        push 8000                               ；F4 到这里
00C4B1AE   6A 00              push 0
00C4B1B0   FFB5 56010000     push dword ptr ss:[ebp+156]
00C4B1B6   FF95 51050000     call near dword ptr ss:[ebp+551]        ; kernel32.VirtualFree
.....................................
```

中间略。坚持一路 F8，不要往回跳的原则，否则就要多走些路了 :)

```
\==============================================================>来到这里：
00C4B389   8946 10            mov dword ptr ds:[esi+10],eax
00C4B38C   83C6 14            add esi,14
00C4B38F   8B95 22040000      mov edx,dword ptr ss:[ebp+422]
00C4B395 ^ E9 EBFEFFFF        jmp dailypim.00C4B285
00C4B39A   B8 F8384200        mov eax,dailypim.004238F8
00C4B39F   50                 push eax
00C4B3A0   0385 22040000     add eax,dword ptr ss:[ebp+422]
00C4B3A6   59                 pop ecx
00C4B3A7   0BC9               or ecx,ecx
00C4B3A9   8985 A8030000     mov dword ptr ss:[ebp+3A8],eax
00C4B3AF   61                 popad                                 ；终于找到了 :)
```



00C4B3B0 `jnz` short `dailypim.00C4B3BA` ；慢慢走  

00C4B3B2 `mov` `eax`,1  

00C4B3B7 `retn` 0C  

00C4B3BA `push` `dailypim.008238F8` ；应该就是你了吧  

00C4B3BF `retn`  

============================================================>找到 EOP  

008238F8 `push` `ebp` ;EOP here  

008238F9 `mov` `ebp`,`esp`  

008238FB `add` `esp`,-10  

008238FE `push` `ebx`  

008238FF `mov` `eax`,`dailypim.00822A68`  

00823904 `call` `dailypim.00407694`  

00823909 `mov` `ebx`,`dword ptr ds:[831F94]` ; `dailypim.00833C3C`  

0082390F `mov` `eax`,`dword ptr ds:[ebx]`  

00823911 `call` `dailypim.0047B4F0`  

00823916 `mov` `eax`,`dword ptr ds:[ebx]`  

00823918 `mov` `edx`,`dailypim.00823988` ; ASCII "DailyPim"  

我们用 `LordPE` `Dump` 下来，生成一个 `Dump.exe` 的文件。然后启用 `Import RE`，输入 `OEP` 为 `4238F8`，获取输入表，没问题。  

脱壳成功，脱壳后的软件能直接运行。我的天，这个脱了壳的软件有 8M 多。 用 `PE` 检测到 `Borland Delphi 6.0 - 7.0`。  

## 二、破解一：  

在这部分，我们通过跟踪程序生成真注册码的过程，完成对部分注册码的破解。  

首先运行脱壳后的软件，对软件有一个基本的了解。在帮助菜单下选择“注册”，有一个弹出框，点击“输入注册码”。  

我的机器码：`U2686167070`  

我的用户名：`xu_wh`  

我的注册码：`111222333444555666777888999000`（随便输的）  

输入完成后，会看到一个提示信息，指出软件在下次启动的时候会对注册码进行校验。在软件的安装目录看到有一个名为 `PIM.INI` 的文件。在这个文件里，有我们输入的用户名和机器码：  

`REGUSER=xu_wh`  

`REGSN=111222333444555666777888999000`  

`U2686167070=111222333444555666777888999000`  

注意到“`REGSN`”和“`REGUSER`”这样的字符串，就是将来我们破解的一个突破口。  

用 `OD` 载入脱壳后的软件，然后搜索字符串：`REGSN` 和 `REGUSER`。在找到的地方下断。`F9`，运行至此：  

```
007CBEAD   |.   B9 B0C97C00      mov ecx,dumped(?007CC9B0          ;  ASCII "REGUSER"
007CBEB2   |.   BA 84C97C00      mov edx,dumped(?007CC984          ;  ASCII "PIM"
007CBEB7   |.   8BC3             mov eax,ebx
007CBEB9   |.   8B38             mov edi,dword ptr ds:[eax]
007CBEBB   |.   FF17             call near dword ptr ds:[edi]
007CBEBD   |.   8B55 D0          mov edx,dword ptr ss:[ebp-30]     ;  edx = 02545478; value here is 'xu_wh'
007CBEC0   |.   A1 741B8300      mov eax,dword ptr ds:[831B74]
007CBEC5   |.   E8 E28EC3FF      call dumped(?00404DAC             ;  user name saved into 02545478 , 008386898 -> 02545478
007CBECA   |.   6A 00            push 0
007CBECC   |.   8D45 FC          lea eax,dword ptr ss:[ebp-4]
007CBECF   |.   50               push eax
007CBED0   |.   8B0D 30238300    mov ecx,dword ptr ds:[832330]     ;  dumped(?00836934
007CBED6   |.   8B09             mov ecx,dword ptr ds:[ecx]
007CBED8   |.   BA 84C97C00      mov edx,dumped(?007CC984          ;  ASCII "PIM"
007CBEDD   |.   8BC3             mov eax,ebx                       ;  edx is machine code 'U2686167070'
007CBEDF   |.   8B38             mov edi,dword ptr ds:[eax]
007CBEE1   |.   FF17             call near dword ptr ds:[edi]      ;  get user password from PIM.ini by U2686167070
007CBEE3   |.   837D FC 00       cmp dword ptr ss:[ebp-4],0        ;  if password = 0, mean's input password once
007CBEE7   |.   75 16            jnz short dumped(?007CBEFF        ;  then jump
007CBEE9   |.   6A 00            push 0
007CBEEB   |.   8D45 FC          lea eax,dword ptr ss:[ebp-4]
007CBEEE   |.   50               push eax
007CBEEF   |.   B9 C0C97C00      mov ecx,dumped(?007CC9C0          ;  ASCII "REGSN"
007CBEF4   |.   BA 84C97C00      mov edx,dumped(?007CC984          ;  ASCII "PIM"
007CBEF9   |.   8BC3             mov eax,ebx
```

如果输过了密码，则跳过我们在 `007CBEEF` 设置的断点，到达这里：  

```
007CBEFD   |.   FF17            call near dword ptr ds:[edi]
007CBEFF   |>   A1 A81D8300     mov eax,dword ptr ds:[831DA8]        
007CBF04   |.   8B55 FC         mov edx,dword ptr ss:[ebp-4]      ;  edx is 0254F9AC, save input password
007CBF07   |.   E8 A08EC3FF     call dumped(?00404DAC
007CBF0C   |.   6A 01           push 1
007CBF0E   |.   B9 D0C97C00     mov ecx,dumped(?007CC9D0          ;  ASCII "SKIN"
007CBF13   |.   BA 84C97C00     mov edx,dumped(?007CC984          ;  ASCII "PIM"
007CBF18   |.   8BC3            mov eax,ebx
007CBF1A   |.   8B38            mov edi,dword ptr ds:[eax]
007CBF1C   |.   FF57 08         call near dword ptr ds:[edi+8]
007CBF1F   |.   8945 F8         mov dword ptr ss:[ebp-8],eax
007CBF22   |.   6A 04           push 4
007CBF24   |.   B9 E0C97C00     mov ecx,dumped(?007CC9E0          ;  ASCII "STS"
007CBF29   |.   BA 84C97C00     mov edx,dumped(?007CC984          ;  ASCII "PIM"
007CBF2E   |.   8BC3            mov eax,ebx
007CBF30   |.   8B38            mov edi,dword ptr ds:[eax]
007CBF32   |.   FF57 08         call near dword ptr ds:[edi+8]
007CBF35   |.   8945 F4         mov dword ptr ss:[ebp-C],eax
007CBF38   |.   6A 00           push 0
007CBF3A   |.   B9 ECC97C00     mov ecx,dumped(?007CC9EC          ;  ASCII "LEFT"
007CBF3F   |.   BA 84C97C00     mov edx,dumped(?007CC984          ;  ASCII "PIM"
```

...（略掉部分都是对 `pim.ini` 进行读操作） 我们一路 `F8`，直到下面：  

```
007CC905   |.   B9 F8CD7C00    mov ecx,dumped(?007CCDF8          ;  ASCII "IEMENU"
007CC90A   |.   BA 84C97C00    mov edx,dumped(?007CC984          ;  ASCII "PIM"
007CC90F   |.   8BC3           mov eax,ebx
007CC911   |.   8B38           mov edi,dword ptr ds:[eax]
007CC913   |.   FF57 08        call near dword ptr ds:[edi+8]
007CC916   |.   48             dec eax
007CC917   |.   75 07          jnz short dumped(?007CC920
007CC919   |.   8BC6           mov eax,esi
007CC91B   |.   E8 C8F0FFFF    call dumped(?007CB9E8
007CC920   |>   8BC3           mov eax,ebx
007CC922   |.   E8 4175C3FF    call dumped(?00403E68
007CC927   |.   33C0           xor eax,eax
007CC929   |.   5A             pop edx
007CC92A   |.   59             pop ecx
007CC92B   |.   59             pop ecx
007CC92C   |.   64:8910        mov dword ptr fs:[eax],edx
007CC92F   |.   68 51C97C00    push dumped(?007CC951
007CC934   |>   8D45 BC        lea eax,dword ptr ss:[ebp-44]
007CC937   |.   BA 0A000000    mov edx,0A
007CC93C   |.   E8 3B84C3FF    call dumped(?00404D7C
007CC941   |.   8D45 FC        lea eax,dword ptr ss:[ebp-4]
007CC944   |.   E8 0F84C3FF    call dumped(?00404D58            
007CC949   \\.   C3             retn
```

可以知道这部分程序的功能是从配置文件中读出所需的配置信息。程序返回到：  

```
007AC2EC   .   E8 17FB0100     call dumped(?007CBE08
007AC2F1   .   8B03            mov eax,dword ptr ds:[ebx]
007AC2F3   .   E8 6C1A0200     call dumped(?007CDD64          ;  在处 007AC2F3，F7 跟进。此时 EDX = 02584330 -> password  
007AC2F8   .   A1 941F8300     mov eax,dword ptr ds:[831F94]
007AC2FD   .   8B00            mov eax,dword ptr ds:[eax]
007AC2FF   .   C640 5B 01      mov byte ptr ds:[eax+5B],1
007AC303   .   8B45 FC         mov eax,dword ptr ss:[ebp-4]
007AC306   .   E8 99B9CCFF     call dumped(?00477CA4
```

我们来到 `007CDD64`：  

```
007CDD64    $   55               push ebp
007CDD65    .   8BEC             mov ebp,esp
007CDD67    .   B9 18000000     mov ecx,18
007CDD6C    >   6A 00           push 0
007CDD6E    .   6A 00           push 0
007CDD70    .   49              dec ecx
007CDD71    .^ 75 F9            jnz short dumped(?007CDD6C
007CDD73    .   51              push ecx
007CDD74    .   53              push ebx
007CDD75    .   56              push esi
007CDD76    .   57              push edi
007CDD77    .   8945 FC         mov dword ptr ss:[ebp-4],eax
007CDD7A    .   8B1D 2C1F8300   mov ebx,dword ptr ds:[831F2C]     ;  dumped(?0083687C
007CDD80    .   8D75 E0         lea esi,dword ptr ss:[ebp-20]
007CDD83    .   33C0            xor eax,eax
007CDD85    .   55              push ebp
007CDD86    .   68 ECF57C00     push dumped(?007CF5EC
007CDD8B    .   64:FF30         push dword ptr fs:[eax]
```

在 `007CDD64` 中，很大一部分的功能是对临时 `EXCEL` 表进行操作，与我们破解无关。我们在这里一路 `F8`，直到  



007CEFEC `. 8B45 FC` `mov eax,dword ptr ss:[ebp-4]`

007CEFEF `. 8B80 940F0000` `mov eax,dword ptr ds:[eax+F94]` `; eax=A01BA81E（它的生成在下面有说明）`

007CEFF5 `. E8 7616FCFF` `call dumped(?00790670` `; 在这里生成伪注册码。不用管它的详细算法`

007CEFFA `. 8B85 6CFFFFFF` `mov eax,dword ptr ss:[ebp-94]` `; edx=c3d8e946a83746`

007CF000 `. 8D95 70FFFFFF` `lea edx,dword ptr ss:[ebp-90]`

007CF006 `. E8 C117FCFF` `call dumped(?007907CC` `; 生成真正的注册码，F7 跟进，下面就是算法`

007CF00B `. 8B95 70FFFFFF` `mov edx,dword ptr ss:[ebp-90]`

007CF011 `. A1 18188300` `mov eax,dword ptr ds:[831818]`

007CF016 `. E8 915DC3FF` `call dumped(?00404DAC`

007CF01B `. 8B15 A81D8300` `mov edx,dword ptr ds:[831DA8]` `; dumped(?0083689C`

007CF021 `. 8B12` `mov edx,dword ptr ds:[edx]` `; EDX 中存放我们输入的注册码 111222333444555666777888999000`

007CF023 `. A1 18188300` `mov eax,dword ptr ds:[831818]`

007CF028 `. 8B00` `mov eax,dword ptr ds:[eax]` `; EAX 中存放在 007907CC 处生成的注册码 b417d593d4834`

007CF02A `. E8 2D63C3FF` `call dumped(?0040535C` `; 对真假注册码进行对比，我们在这里 F7 跟进，看看是怎么比较的。`

007CF02F `. 85C0` `test eax,eax`

007CF031 `/74 0D` `je short dumped(?007CF040` `; 通常，在这里是一个爆破点但是程序还在别的地方对注册码进行判断，所以，仅仅在这里实施爆破是不行的。还要在后面进行爆破。`

下面是真正的算法，该算法根据机器码算出序列号，是我们关注的重点之一。

```
007907CC /$ 55 push ebp
007907CD |. 8BEC mov ebp,esp
007907CF |. 83C4 C0 add esp,-40
007907D2 |. 53 push ebx
007907D3 |. 56 push esi
007907D4 |. 33C9 xor ecx,ecx
007907D6 |. 894D C0 mov dword ptr ss:[ebp-40],ecx
007907D9 |. 894D C4 mov dword ptr ss:[ebp-3C],ecx
007907DC |. 894D C8 mov dword ptr ss:[ebp-38],ecx
007907DF |. 894D F4 mov dword ptr ss:[ebp-C],ecx
007907E2 |. 894D F0 mov dword ptr ss:[ebp-10],ecx
007907E5 |. 8955 F8 mov dword ptr ss:[ebp-8],edx
007907E8 |. 8945 FC mov dword ptr ss:[ebp-4],eax
007907EB |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
007907EE |. E8 154AC7FF call dumped(?00405208
007907F3 |. 8D75 DC lea esi,dword ptr ss:[ebp-24]
007907F6 |. 33C0 xor eax,eax
007907F8 |. 55 push ebp
007907F9 |. 68 F70A7900 push dumped(?00790AF7
007907FE |. 64:FF30 push dword ptr fs:[eax]
00790801 |. 64:8920 mov dword ptr fs:[eax],esp
00790804 |. 8D45 F4 lea eax,dword ptr ss:[ebp-C]
00790807 |. 8B55 FC mov edx,dword ptr ss:[ebp-4]
0079080A |. E8 E145C7FF call dumped(?00404DF0
0079080F |. 8D45 F4 lea eax,dword ptr ss:[ebp-C]
00790812 |. B9 01000000 mov ecx,1
00790817 |. BA 01000000 mov edx,1
0079081C |. E8 974AC7FF call dumped(?004052B8
00790821 |. 8D45 F4 lea eax,dword ptr ss:[ebp-C]
00790824 |. B9 01000000 mov ecx,1
00790829 |. BA 02000000 mov edx,2
0079082E |. E8 854AC7FF call dumped(?004052B8
00790833 |. 8D45 F4 lea eax,dword ptr ss:[ebp-C]
00790836 |. B9 01000000 mov ecx,1
0079083B |. BA 03000000 mov edx,3
00790840 |. E8 734AC7FF call dumped(?004052B8
00790845 |. 8D45 F4 lea eax,dword ptr ss:[ebp-C]
00790848 |. B9 01000000 mov ecx,1
0079084D |. BA 06000000 mov edx,6
00790852 |. E8 614AC7FF call dumped(?004052B8
00790857 |. 8B45 F4 mov eax,dword ptr ss:[ebp-C]
0079085A |. E8 619CC7FF call dumped(?0040A4C0
0079085F |. 8BD8 mov ebx,eax ; eax=e8242862，eax 是由硬盘的序列号得来的
00790861 |. F7D3 not ebx ; ebx=17DBD79D，ebx 是对 eax 取反
00790863 |. 81F3 22211276 xor ebx,76122122 ; ebx=61C9F6BF，ebx 与 76122122 进行异或
```

以下部分是分别取出 `ebx` 中的 4 个 `byte`

```
00790869 |. 8BC3 mov eax,ebx
0079086B |. 25 000000FF and eax,FF000000
00790870 |. C1E8 18 shr eax,18 ; eax=61
00790873 |. 8906 mov dword ptr ds:[esi],eax
00790875 |. 8BC3 mov eax,ebx
00790877 |. 25 0000FF00 and eax,0FF0000
0079087C |. C1E8 10 shr eax,10
0079087F |. 8946 04 mov dword ptr ds:[esi+4],eax ; eax = C9
00790882 |. 8BC3 mov eax,ebx
00790884 |. 25 00FF0000 and eax,0FF00
00790889 |. C1E8 08 shr eax,8 ; eax=F6
0079088C |. 8946 08 mov dword ptr ds:[esi+8],eax
0079088F |. 8BC3 mov eax,ebx
00790891 |. 25 FF000000 and eax,0FF
00790896 |. 8946 0C mov dword ptr ds:[esi+C],eax ; eax=BF
```

提取后的值放分别放在 `ds:[esi]`,`ds:[esi+4]`,`ds:[esi+8]`,`ds:[esi+C]`，下面分别对这四个值进行计算：

```
00790899 |. 8B16 mov edx,dword ptr ds:[esi] ; edx = 61
0079089B |. 81E2 C0000000 and edx,0C0 ; edx = 40
007908A1 |. 8B4E 04 mov ecx,dword ptr ds:[esi+4] ; ecx=C9
007908A4 |. 81E1 C0000000 and ecx,0C0 ; ecx=C0
007908AA |. C1E9 02 shr ecx,2 ; ecx=30
007908AD |. 03D1 add edx,ecx ; edx=70
007908AF |. 8B4E 08 mov ecx,dword ptr ds:[esi+8] ; ecx=f6
007908B2 |. 81E1 C0000000 and ecx,0C0 ; ecx=C0
007908B8 |. C1E9 04 shr ecx,4 ; ecx=0C
007908BB |. 03D1 add edx,ecx ; edx=7C
007908BD |. 25 C0000000 and eax,0C0 ; eax=80
007908C2 |. C1E8 06 shr eax,6 ; eax=02
007908C5 |. 03D0 add edx,eax ; edx=7E
007908C7 |. 8955 CC mov dword ptr ss:[ebp-34],edx
007908CA |. 8B06 mov eax,dword ptr ds:[esi] ; eax=61
007908CC |. 83E0 30 and eax,30 ; eax=20
007908CF |. C1E0 02 shl eax,2 ; eax=80
007908D2 |. 8B56 04 mov edx,dword ptr ds:[esi+4] ; edx=C9
007908D5 |. 83E2 30 and edx,30 ; edx=00
007908D8 |. 03C2 add eax,edx ; eax=80
007908DA |. 8B56 08 mov edx,dword ptr ds:[esi+8] ; edx=F6
007908DD |. 83E2 30 and edx,30 ; edx=30
007908E0 |. C1EA 02 shr edx,2 ; edx=0C
007908E3 |. 03C2 add eax,edx ; eax=8C
007908E5 |. 8B56 0C mov edx,dword ptr ds:[esi+C] ; edx=BF
007908E8 |. 83E2 30 and edx,30 ; edx=30
007908EB |. C1EA 04 shr edx,4 ; edx=03
007908EE |. 03C2 add eax,edx ; eax=8F
007908F0 |. 8945 D0 mov dword ptr ss:[ebp-30],eax
007908F3 |. 8B06 mov eax,dword ptr ds:[esi] ; eax=61
007908F5 |. 83E0 0C and eax,0C ; eax=00
007908F8 |. C1E0 04 shl eax,4 ; eax=00
007908FB |. 8B56 04 mov edx,dword ptr ds:[esi+4] ; edx=C9
007908FE |. 83E2 0C and edx,0C ; edx=08
00790901 |. C1E2 02 shl edx,2 ; edx=20
00790904 |. 03C2 add eax,edx ; eax=20
00790906 |. 8B56 08 mov edx,dword ptr ds:[esi+8]
00790909 |. 83E2 0C and edx,0C
```



## 汇编代码解析

```
0079090C   |.  03C2                 add eax,edx  

0079090E   |.  8B56 0C              mov edx,dword ptr ds:\[esi+C\]  

00790911   |.  83E2 0C              and edx,0C  

00790914   |.  C1EA 02              shr edx,2  

00790917   |.  03C2                 add eax,edx  

00790919   |.  8945 D4              mov dword ptr ss:\[ebp-2C\],eax     ;  eax=27  

0079091C   |.  8B06                 mov eax,dword ptr ds:\[esi\]  

0079091E   |.  83E0 03              and eax,3  

00790921   |.  C1E0 06              shl eax,6  

00790924   |.  8B56 04              mov edx,dword ptr ds:\[esi+4\]  

00790927   |.  83E2 03              and edx,3  

0079092A   |.  C1E2 04              shl edx,4  

0079092D   |.  03C2                 add eax,edx  

0079092F   |.  8B56 08              mov edx,dword ptr ds:\[esi+8\]  

00790932   |.  83E2 03              and edx,3  

00790935   |.  C1E2 02              shl edx,2  

00790938   |.  03C2                 add eax,edx  

0079093A   |.  8B56 0C              mov edx,dword ptr ds:\[esi+C\]  

0079093D   |.  83E2 03              and edx,3  

00790940   |.  03C2                 add eax,edx  

00790942   |.  8945 D8              mov dword ptr ss:\[ebp-28\],eax     ;  eax=5B  

00790945   |.  8B5D CC              mov ebx,dword ptr ss:\[ebp-34\]  

00790948   |.  C1E3 18              shl ebx,18  

0079094B   |.  8B45 D0              mov eax,dword ptr ss:\[ebp-30\]  

0079094E   |.  C1E0 10              shl eax,10  

00790951   |.  03D8                 add ebx,eax  

00790953   |.  8B45 D4              mov eax,dword ptr ss:\[ebp-2C\]  

00790956   |.  C1E0 08              shl eax,8  

00790959   |.  03D8                 add ebx,eax  

0079095B   |.  035D D8              add ebx,dword ptr ss:\[ebp-28\]  

0079095E   |.  8BC3                 mov eax,ebx  

00790960   |.  25 000000FF         and eax,FF000000  

00790965   |.  C1E8 18              shr eax,18  

00790968   |.  8906                 mov dword ptr ds:\[esi\],eax        ;  eax=7E  

0079096A   |.  8BC3                 mov eax,ebx  

0079096C   |.  25 0000FF00         and eax,0FF0000  

00790971   |.  C1E8 10              shr eax,10  

00790974   |.  8946 04              mov dword ptr ds:\[esi+4\],eax  

00790977   |.  8BC3                 mov eax,ebx  

00790979   |.  25 00FF0000         and eax,0FF00  

0079097E   |.  C1E8 08              shr eax,8  

00790981   |.  8946 08              mov dword ptr ds:\[esi+8\],eax  

00790984   |.  81E3 FF000000       and ebx,0FF  

0079098A   |.  895E 0C              mov dword ptr ds:\[esi+C\],ebx  

0079098D   |.  8B06                 mov eax,dword ptr ds:\[esi\]  

0079098F   |.  8BD0                 mov edx,eax  

00790991   |.  81E2 F0000000       and edx,0F0  

00790997   |.  C1EA 04              shr edx,4  

0079099A   |.  83E0 0F              and eax,0F  

0079099D   |.  C1E0 04              shl eax,4  

007909A0   |.  03D0                 add edx,eax  

007909A2   |.  8916                 mov dword ptr ds:\[esi\],edx  

007909A4   |.  8B46 04              mov eax,dword ptr ds:\[esi+4\]  

007909A7   |.  8BD0                 mov edx,eax  

007909A9   |.  81E2 F0000000       and edx,0F0  

007909AF   |.  C1EA 04              shr edx,4  

007909B2   |.  83E0 0F              and eax,0F  

007909B5   |.  C1E0 04              shl eax,4  

007909B8   |.  03D0                 add edx,eax  

007909BA   |.  8956 04              mov dword ptr ds:\[esi+4\],edx  

007909BD   |.  8B46 08              mov eax,dword ptr ds:\[esi+8\]  

007909C0   |.  8BD0                 mov edx,eax  

007909C2   |.  81E2 F0000000       and edx,0F0  

007909C8   |.  C1EA 04              shr edx,4  

007909CB   |.  83E0 0F              and eax,0F  

007909CE   |.  C1E0 04              shl eax,4  

007909D1   |.  03D0                 add edx,eax  

007909D3   |.  8956 08              mov dword ptr ds:\[esi+8\],edx  

007909D6   |.  8B46 0C              mov eax,dword ptr ds:\[esi+C\]  

007909D9   |.  8BD0                 mov edx,eax  

007909DB   |.  81E2 F0000000       and edx,0F0  

007909E1   |.  C1EA 04              shr edx,4  

007909E4   |.  83E0 0F              and eax,0F  

007909E7   |.  C1E0 04              shl eax,4  

007909EA   |.  03D0                 add edx,eax  

007909EC   |.  8956 0C              mov dword ptr ds:\[esi+C\],edx  

007909EF   |.  8B5E 04              mov ebx,dword ptr ds:\[esi+4\]  

007909F2   |.  C1E3 18              shl ebx,18  

007909F5   |.  8B06                 mov eax,dword ptr ds:\[esi\]  

007909F7   |.  C1E0 10              shl eax,10  

007909FA   |.  03D8                 add ebx,eax  

007909FC   |.  C1E2 08              shl edx,8  

007909FF   |.  03DA                 add ebx,edx  

00790A01   |.  035E 08              add ebx,dword ptr ds:\[esi+8\]  

00790A04   |.  8BC3                 mov eax,ebx                       ;  EAX=F8E7B572，它是运算的最终结果。  

00790A06   |.  33D2                 xor edx,edx  

00790A08   |.  52                   push edx                          ; /Arg2 => 00000000  

00790A09   |.  50                   push eax                          ; |Arg1  

00790A0A   |.  8D45 F0              lea eax,dword ptr ss:\[ebp-10\]     ; |  

00790A0D   |.  E8 CA99C7FF         call dumped(?0040A3DC             ; 将 F8E7B572 转换成 10 进制数  

00790A12   |.  8B45 F0              mov eax,dword ptr ss:\[ebp-10\]     ;  eax=4175934834
```

下面对 `EAX` 中的 10 进制数进行运算，生成 3 个单独的字符，这里生成的是 `b`, `d`, `d`：

```
00790A15   |.  0FB600              movzx eax,byte ptr ds:\[eax\]       ;  eax='0034'  

00790A18   |.  8B55 F0              mov edx,dword ptr ss:\[ebp-10\]  

00790A1B   |.  0FB652 01           movzx edx,byte ptr ds:\[edx+1\]     ;  edx='0031'  

00790A1F   |.  03C2                 add eax,edx                       ;  eax='0065'  

00790A21   |.  B9 05000000         mov ecx,5  

00790A26   |.  99                   cdq  

00790A27   |.  F7F9                 idiv ecx                          ;  edx=1;eax=14  

00790A29   |.  80C2 61              add dl,61                         ;  edx=62  

00790A2C   |.  8855 ED              mov byte ptr ss:\[ebp-13\],dl      ;  dl is 'b'  

00790A2F   |.  8B45 F0              mov eax,dword ptr ss:\[ebp-10\]  

00790A32   |.  0FB640 02           movzx eax,byte ptr ds:\[eax+2\]  

00790A36   |.  8B55 F0              mov edx,dword ptr ss:\[ebp-10\]     ;  eax=37  

00790A39   |.  0FB652 03           movzx edx,byte ptr ds:\[edx+3\]     ;  edx=35  

00790A3D   |.  03C2                 add eax,edx                       ;  eax=6C  

00790A3F   |.  B9 05000000         mov ecx,5  

00790A44   |.  99                   cdq  

00790A45   |.  F7F9                 idiv ecx                          ;  eax=15,edx=3  

00790A47   |.  80C2 61              add dl,61                         ;  edx=64  

00790A4A   |.  8855 EE              mov byte ptr ss:\[ebp-12\],dl      ;  dl is 'd'  

00790A4D   |.  8B45 F0              mov eax,dword ptr ss:\[ebp-10\]  

00790A50   |.  0FB640 04           movzx eax,byte ptr ds:\[eax+4\]  

00790A54   |.  8B55 F0              mov edx,dword ptr ss:\[ebp-10\]     ;  eax=39  

00790A57   |.  0FB652 05           movzx edx,byte ptr ds:\[edx+5\]     ;  edx=33  

00790A5B   |.  03C2                 add eax,edx                       ;  eax=6C  

00790A5D   |.  B9 05000000         mov ecx,5  

00790A62   |.  99                   cdq  

00790A63   |.  F7F9                 idiv ecx                          ;  eax=15;edx=3  

00790A65   |.  80C2 61              add dl,61                         ;  dl=64  

00790A68   |.  8855 EF              mov byte ptr ss:\[ebp-11\],dl      ;  dl is 'd'  

00790A6B   |.  8D45 C8              lea eax,dword ptr ss:\[ebp-38\]  

00790A6E   |.  8A55 ED              mov dl,byte ptr ss:\[ebp-13\]  
```

在生成了 `b`, `d`, `d` 后，将这三个字符插入到相应的位置，`b` 放到第一个数字前，`d` 放到第 4 个数字前，`d` 放到第 7 个数字前。

```
00790A71   |.  E8 C244C7FF         call dumped(?00404F38  

00790A76   |.  8B45 C8              mov eax,dword ptr ss:\[ebp-38\]  

00790A79   |.  8D55 F0              lea edx,dword ptr ss:\[ebp-10\]  

00790A7C   |.  B9 01000000         mov ecx,1  

00790A81   |.  E8 7A48C7FF         call dumped(?00405300             ;  insert b  

00790A86   |.  8D45 C4              lea eax,dword ptr ss:\[ebp-3C\]  

00790A89   |.  8A55 EE              mov dl,byte ptr ss:\[ebp-12\]  

00790A8C   |.  E8 A744C7FF         call dumped(?00404F38  

00790A91   |.  8B45 C4              mov eax,dword ptr ss:\[ebp-3C\]  

00790A94   |.  8D55 F0              lea edx,dword ptr ss:\[ebp-10\]  

00790A97   |.  B9 05000000         mov ecx,5  

00790A9C   |.  E8 5F48C7FF         call dumped(?00405300             ;  insert d  
```



00790AA1 `. . 8D45 C0 lea eax,dword ptr ss:\[ebp-40\]`  
00790AA4 `. . 8A55 EF mov dl,byte ptr ss:\[ebp-11\]`  
00790AA7 `. . E8 8C44C7FF call dumped(?00404F38`  
00790AAC `. . 8B45 C0 mov eax,dword ptr ss:\[ebp-40\]`  
00790AAF `. . 8D55 F0 lea edx,dword ptr ss:\[ebp-10\]`  
00790AB2 `. . B9 09000000 mov ecx,9`  
00790AB7 `. . E8 4448C7FF call dumped(?00405300 ; insert d`  
00790ABC `. . 8B45 F8 mov eax,dword ptr ss:\[ebp-8\]`  
00790ABF `. . 8B55 F0 mov edx,dword ptr ss:\[ebp-10\]`  
00790AC2 `. . E8 E542C7FF call dumped(?00404DAC`  
00790AC7 `. . 33C0 xor eax,eax`  
00790AC9 `. . 5A pop edx`  
00790ACA `. . 59 pop ecx`  
00790ACB `. . 59 pop ecx`  
00790ACC `. . 64:8910 mov dword ptr fs:\[eax\],edx`  
00790ACF `. . 68 FE0A7900 push dumped(?00790AFE`  
00790AD4 `> . 8D45 C0 lea eax,dword ptr ss:\[ebp-40\]`  
00790AD7 `. . BA 03000000 mov edx,3`  
00790ADC `. . E8 9B42C7FF call dumped(?00404D7C`  
00790AE1 `. . 8D45 F0 lea eax,dword ptr ss:\[ebp-10\]`  
00790AE4 `. . BA 02000000 mov edx,2`  
00790AE9 `. . E8 8E42C7FF call dumped(?00404D7C`  
00790AEE `. . 8D45 FC lea eax,dword ptr ss:\[ebp-4\]`  
00790AF1 `. . E8 6242C7FF call dumped(?00404D58`  
00790AF6 `\. C3 retn ; 生成真正的注册码后，返回到上面 007CF00B 执行。`

下面是对真假注册码进行比较的部分：

```
0040535C /$ . 85C0 test eax,eax
0040535E |. . 74 40 je short dumped(?004053A0
00405360 |. . 85D2 test edx,edx
00405362 |. . 74 31 je short dumped(?00405395
00405364 |. . 53 push ebx
00405365 |. . 56 push esi
00405366 |. . 57 push edi
00405367 |. . 89C6 mov esi,eax ; esi 中存放真正的注册码
00405369 |. . 89D7 mov edi,edx ; edi 中存放我们输入的注册码
0040536B |. . 8B4F FC mov ecx,dword ptr ds:\[edi-4\] ; ecx 中存放我们输入的注册码的长度
0040536E |. . 57 push edi
0040536F |. . 8B56 FC mov edx,dword ptr ds:\[esi-4\] ; edx 中存放真注册码的长度
00405372 |. . 4A dec edx
00405373 |. . 78 1B js short dumped(?00405390
00405375 |. . 8A06 mov al,byte ptr ds:\[esi\] ; al 中放真注册码的第一个字符
00405377 |. . 46 inc esi
00405378 |. . 29D1 sub ecx,edx
0040537A |. . 7E 14 jle short dumped(?00405390
0040537C |> . F2:AE /repne scas byte ptr es:\[edi\] ; 在我们输入的注册码中找到程序生成的注册码的子串
0040537E |. . 75 10 |jnz short dumped(?00405390
00405380 |. . 89CB |mov ebx,ecx
00405382 |. . 56 |push esi
00405383 |. . 57 |push edi
00405384 |. . 89D1 |mov ecx,edx
00405386 |. . F3:A6 |repe cmps byte ptr es:\[edi\],byte>
00405388 |. . 5F |pop edi
00405389 |. . 5E |pop esi
0040538A |. . 74 0C |je short dumped(?00405398
0040538C |. . 89D9 |mov ecx,ebx
0040538E |.^ EB EC \jmp short dumped(?0040537C
00405390 |> . 5A pop edx
00405391 |. . 31C0 xor eax,eax
00405393 |. . EB 08 jmp short dumped(?0040539D
00405395 |> . 31C0 xor eax,eax
00405397 |. . C3 retn
00405398 |> . 5A pop edx
00405399 |. . 89F8 mov eax,edi
0040539B |. . 29D0 sub eax,edx ; eax = 3, 41-3E = 3
0040539D |> . 5F pop edi
0040539E |. . 5E pop esi
0040539F |. . 5B pop ebx
004053A0 \\> . C3 retn
```

通过对上面代码的分析，我们知道，只要程序生成注册码是我们输入的注册码的一个子串即可。因为程序还有另外一部分对注册码的其它部分进行校验。

我们在这里还应该注意到，真正注册码的生成和一个数有关：`A01BA81E`，这个数的产生和硬盘序列号有关，下面就是有关的算法：

```
00790624 /$ . 83C4 F4 add esp,-0C
00790627 |. . 6A 00 push 0 ; /pFileSystemNameSize = NULL
00790629 |. . 6A 00 push 0 ; |pFileSystemNameBuffer = NULL
0079062B |. . 8D4424 10 lea eax,dword ptr ss:\[esp+10\] ; |
0079062F |. . 50 push eax ; |pFileSystemFlags
00790630 |. . 8D4424 10 lea eax,dword ptr ss:\[esp+10\] ; |
00790634 |. . 50 push eax ; |pMaxFilenameLength
00790635 |. . 8D4424 10 lea eax,dword ptr ss:\[esp+10\] ; |
00790639 |. . 50 push eax ; |pVolumeSerialNumber
0079063A |. . 6A 00 push 0 ; |MaxVolumeNameSize = 0
0079063C |. . 6A 00 push 0 ; |VolumeNameBuffer = NULL
0079063E |. . 68 68067900 push dumped(?00790668 ; |RootPathName = "c:\\"
00790643 |. . E8 DC74C7FF call <jmp.&kernel32.GetVolumeInfo>; \GetVolumeInformationA
00790648 |. . 8B0424 mov eax,dword ptr ss:\[esp\]
0079064B |. . 05 85000000 add eax,85
00790650 |. . B9 22000000 mov ecx,22
00790655 |. . 33D2 xor edx,edx
00790657 |. . F7F1 div ecx
00790659 |. . 6BC0 29 imul eax,eax,29 ; EAX = A01BA81E，下面的计算中要用到这个值
0079065C |. . 890424 mov dword ptr ss:\[esp\],eax
0079065F |. . 8B0424 mov eax,dword ptr ss:\[esp\]
00790662 |. . 83C4 0C add esp,0C
00790665 \. . C3 retn
```

### 三、破解二

通过对上面程序的分析，我们能得到部分注册码：`b417d593d4834`。输入这个注册码。然后重新运行该软件。在经过一段时间后，又会跳出注册的对话框。

在对软件进行反复跟踪分析后，发现程序会对在地址 `00836948` 和 `0083694C` 处的值进行检查，如果是 `00000003`，程序认为是经过注册的，然后再次对注册码进行检查，如果检查正确，保持 `00000003` 不变，否则给这个变量赋 `00000002`。

最后程序又对这个值进行检查，如果是 `00000002` 表示没有完全注册；如果是 `00000003` 表示程序是正确注册的。（复杂呀！）

另外，在对程序的分析中，注册码的长度不能小于 14。程序对注册码的检查分为两种：

1. 注册码的长度为 14，程序使用密钥 `"YiISIvp * kb2W A9iIpC5 M#IbI kxIScYoWI='v2S2b'"` 进行验密；
2. 注册码的长度 > 14，程序使用密钥 `"YqhNrd8K1JOsbfZVUatRgB3McDiExTSu5GjvPekL2w7AXWzlIyHmQn94Co6Fp0#S_"` 和 `"_$#abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"` 作为密码字典。算法如下：

   拿出待校验的注册码的第一字符，在第一个字符串中，查找出它的位置 `X`，然后在第二个字符串中，取出第 `X` 个字符作为新串的第一个字符。然后再取出待校验注册码的第二个字符，在第一个字符串中找到它的位置 `Y`，然后再从第二个字符串找到第 `Y` 个字符，作为新串的第二个字符。生成新的字符串后，用这个长为 2 的新串和待校验注册码的倒数第 3 位开始，长度为 2 进行比较（如果是正确的注册码，应该相等）。然后再用待校验注册码的倒数第四位和 `0` 比，倒数第一位和 `1` 比。如果都相等了，我们的注册码校验就顺利通过了 :)

我们索性输入注册码为：`b417d593d4834Yj1`。

在 `00836948` 和 `0083694C` 处下硬件读断点，重新运行，在等到一段时间后，终于拦下来了 :)

```
007DE1F2 . 8338 03 cmp dword ptr ds:\[eax\],3 ; eax=00836948
007DE1F5 . 0F85 57020000 jnz dumped(?007DE452 ; 因为在程序启动的时候，从.ini 文件中读取了注册码，已经完成了部分检查，所以 ds:\[eax\]=3，在这里不跳转
007DE1FB . A1 A81D8300 mov eax,dword ptr ds:\[831DA8\] ; ds:\[eax\]中存放我们输入的注册码
007DE200 . 8338 00 cmp dword ptr ds:\[eax\],0 ; 当然这里不是 0 了 :)
007DE203 . 0F84 49020000 je dumped(?007DE452
007DE209 . A1 A81D8300 mov eax,dword ptr ds:\[831DA8\]
007DE20E . 8B00 mov eax,dword ptr ds:\[eax\] ; ds:\[eax\]再次拿到我们输入的注册码
007DE210 . E8 0B6EC2FF call dumped(?00405020 ; 取得注册码的长度
007DE215 . 8BF0 mov esi,eax ; 存入到 esi 中
007DE217 . A1 18188300 mov eax,dword ptr ds:\[831818\] ; eax=00836858
007DE21C . 8B00 mov eax,dword ptr ds:\[eax\] ; ds:\[eax\]中存放由程序自动生成的注册码
007DE21E . E8 FD6DC2FF call dumped(?00405020 ; 取得这个注册码的长度
007DE223 . 2BF0 sub esi,eax ; 两个注册码的长度相减，减的值放在 esi 中
007DE225 . 4E dec esi ; esi 减 1
007DE226 . 0F85 2E010000 jnz dumped(?007DE35A ; 如果不等于 0，则跳转
007DE22C . 8D45 F8 lea eax,dword ptr ss:\[ebp-8\] ; 下面是对长度为 14 的密码进行校验的算法。这次我们先不对它进行分析
...
```



# 密码校验算法分析

## 1. 主校验逻辑（长度 > 14）

以下是对长度大于 14 的密码进行校验的汇编代码段：

```assembly
007DE35A    >   8D45 E4         lea eax,dword ptr ss:[ebp-1C]     ;  eax=0012FDD4  
007DE35D    .   50              push eax  
007DE35E    .   8D45 DC         lea eax,dword ptr ss:[ebp-24]     ;  eax=0012FDCC  
007DE361    .   50              push eax  
007DE362    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]     ;  eax=0083689C  
007DE367    .   8B00            mov eax,dword ptr ds:[eax]        ;  eax=025764AC->'b417d593d4834Yj0'  
007DE369    .   B9 02000000     mov ecx,2                         ;  ecx=2  
007DE36E    .   BA 01000000     mov edx,1                         ;  edx=1  
007DE373    .   E8 006FC2FF     call dumped(?00405278             ;  00405278 的作用是从 ds:[eax]中得到起始位置为 edx,长为 ecx 的子串  
007DE378    .   8B45 DC         mov eax,dword ptr ss:[ebp-24]     ;  子串是'b4'  
007DE37B    .   8D55 E0         lea edx,dword ptr ss:[ebp-20]     ;  edx=12FDD0  
007DE37E    .   E8 FD1DFBFF     call dumped(?00790180             ;  在这里，我们 F7 跟进，见下面的分析。  
007DE383    .   8B45 E0         mov eax,dword ptr ss:[ebp-20]     ;  eax=0258694C --> Yj0  
007DE386    .   B9 02000000     mov ecx,2  
007DE38B    .   BA 01000000     mov edx,1  
007DE390    .   E8 E36EC2FF     call dumped(?00405278             ;  after this, eax=0012FDD4,save the first 2 chars of Yj0  
007DE395    .   8B45 E4         mov eax,dword ptr ss:[ebp-1C]     ;  EAX=0254831C -> Yj  
007DE398    .   50              push eax  
007DE399    .   8D45 D8         lea eax,dword ptr ss:[ebp-28]  
007DE39C    .   50              push eax  
007DE39D    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]  
007DE3A2    .   8B00            mov eax,dword ptr ds:[eax]  
007DE3A4    .   E8 776CC2FF     call dumped(?00405020             ;  get the length of password input,it's 16  
007DE3A9    .   8BD0            mov edx,eax  
007DE3AB    .   83EA 02         sub edx,2  
007DE3AE    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]  
007DE3B3    .   8B00            mov eax,dword ptr ds:[eax]        ;  eax=password input with length 16  
007DE3B5    .   B9 02000000     mov ecx,2  
007DE3BA    .   E8 B96EC2FF     call dumped(?00405278             ;  get the substring with begin at edx,length ecx  
007DE3BF    .   8B55 D8         mov edx,dword ptr ss:[ebp-28]     ;  dex='Yj'  
007DE3C2    .   58              pop eax  
007DE3C3    .   E8 9C6DC2FF     call dumped(?00405164             ;  对用注册的前两位生成的'Yj'和注册码的倒数第 3、2 位进行比较  
007DE3C8    .   74 0B           je short dumped(?007DE3D5         ;  如果是正确的注册码，必须跳。  
007DE3CA    .   A1 981F8300     mov eax,dword ptr ds:[831F98]  
007DE3CF    .   C700 02000000   mov dword ptr ds:[eax],2          ;  不能执行到这里，否则，就会给 00836948 赋 2  
007DE3D5    >   8D45 D4         lea eax,dword ptr ss:[ebp-2C]  
007DE3D8    .   50              push eax  
007DE3D9    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]  
007DE3DE    .   8B00            mov eax,dword ptr ds:[eax]        ;  ds:[eax]是待校验的注册码  
007DE3E0    .   E8 3B6CC2FF     call dumped(?00405020             ;  取得注册码的长度  
007DE3E5    .   8BD0            mov edx,eax  
007DE3E7    .   83EA 03         sub edx,3  
007DE3EA    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]  
007DE3EF    .   8B00            mov eax,dword ptr ds:[eax]  
007DE3F1    .   B9 01000000     mov ecx,1  
007DE3F6    .   E8 7D6EC2FF     call dumped(?00405278             ;  从待校验注册码的倒数第四位取出 1 位‘4’  
007DE3FB    .   8B45 D4         mov eax,dword ptr ss:[ebp-2C]  
007DE3FE    .   BA B4E47D00     mov edx,dumped(?007DE4B4          ;  在 007DE4B4 处是‘0’  
007DE403    .   E8 5C6DC2FF     call dumped(?00405164             ;  在这里，这两个数也应该相等  
007DE408    .   74 0B           je short dumped(?007DE415         ;  否则就会给 00836948 处的赋 2  
007DE40A    .   A1 981F8300     mov eax,dword ptr ds:[831F98]     
007DE40F    .   C700 02000000   mov dword ptr ds:[eax],2  
007DE415    >   8D45 D0         lea eax,dword ptr ss:[ebp-30]  
007DE418    .   50              push eax  
007DE419    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]  
007DE41E    .   8B00            mov eax,dword ptr ds:[eax]  
007DE420    .   E8 FB6BC2FF     call dumped(?00405020             ;  取得注册码的长度  
007DE425    .   8BD0            mov edx,eax                       ;  这个长度给 edx，说明要取最后一位了 :)  
007DE427    .   A1 A81D8300     mov eax,dword ptr ds:[831DA8]  
007DE42C    .   8B00            mov eax,dword ptr ds:[eax]  
007DE42E    .   B9 01000000     mov ecx,1                         ;  ecx=1  
007DE433    .   E8 406EC2FF     call dumped(?00405278             ;  取出待校验注册码的最后一位 ‘1’  
007DE438    .   8B45 D0         mov eax,dword ptr ss:[ebp-30]     
007DE43B    .   BA CCE47D00     mov edx,dumped(?007DE4CC          ;  与 007DE4CC 处的‘1’ 比较  
007DE440    .   E8 1F6DC2FF     call dumped(?00405164             ;  这里也必须相等  
007DE445    .   74 0B           je short dumped(?007DE452  
007DE447    .   A1 981F8300     mov eax,dword ptr ds:[831F98]     ;  否则就会给 00836948 处的赋 2  
007DE44C    .   C700 02000000   mov dword ptr ds:[eax],2  
007DE452    >   33C0            xor eax,eax  
007DE454    .   5A              pop edx  
007DE455    .   59              pop ecx  
007DE456    .   59              pop ecx  
007DE457    .   64:8910         mov dword ptr fs:[eax],edx  
007DE45A    .   EB 15           jmp short dumped(?007DE471  
007DE45C    .^  E9 E75EC2FF     jmp dumped(?00404348  
007DE461    .   A1 981F8300     mov eax,dword ptr ds:[831F98]  
007DE466    .   C700 02000000   mov dword ptr ds:[eax],2  
007DE46C    .   E8 0363C2FF     call dumped(?00404774  
007DE471    >   A1 981F8300     mov eax,dword ptr ds:[831F98]  
007DE476    .   8B00            mov eax,dword ptr ds:[eax]  
```

如果要爆破，最简单的方法，就是在 `007DE1F5` 进行跳转，这样就不会对我们输入的注册码做后期校验了。

## 2. 算法细节说明

下面的算法，是取出注册码的第一个字符 `b`，它在 `'Y..'` 字符串的位置是第 13，而在 `'_$'` 中第 13 个字符是 `'j'`。取出注册码的第二个字符 `4`，它在 `'Y..'` 字符串的位置是第 56，而在 `'_$'` 中第 56 个字符是 `'0'`。所以，新的字符串是 `'j0'`，然后在 `'j0'` 前加上 `'Y...'` 字符串的第一个字符 `'Y'`，得到 `'Yj0'`。

## 3. 子函数分析 (00790180)

```assembly
00790180   /$  55              push ebp  
00790181   |.  8BEC            mov ebp,esp  
00790183   |.  33C9            xor ecx,ecx  
00790185   |.  51              push ecx  
00790186   |.  51              push ecx  
00790187   |.  51              push ecx  
00790188   |.  51              push ecx  
00790189   |.  51              push ecx  
0079018A   |.  53              push ebx  
0079018B   |.  56              push esi                      ;  esi=3  
0079018C   |.  8BF2            mov esi,edx  
0079018E   |.  8945 FC         mov dword ptr ss:[ebp-4],eax  ;  ss:0012FD8C 025901F0 -> 'b4'  
00790191   |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]  
00790194   |.  E8 6F50C7FF     call dumped(?00405208         ;  get the length of 'b4' save EDX=2  
00790199   |.  33C0            xor eax,eax  
0079019B   |.  55              push ebp  
0079019C   |.  68 39027900     push dumped(?00790239  
007901A1   |.  64:FF30         push dword ptr fs:[eax]  
007901A4   |.  64:8920         mov dword ptr fs:[eax],esp  
007901A7   |.  BB 01000000     mov ebx,1                     ;  ebx=1  
007901AC   |.  8D45 F8         lea eax,dword ptr ss:[ebp-8]  
007901AF   |.  BA 50027900     mov edx,dumped(?00790250      ;  ASCII "YqhNrd8K1JOsbfZVUatRgB3McDiExTSu5GjvPekL2w7AXWzlIyHmQn94Co6Fp0#S_"  
007901B4   |.  E8 374CC7FF     call dumped(?00404DF0         ;  after this; ss:12FD88 = string above  
007901B9   |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]  
007901BC   |.  BA 9C027900     mov edx,dumped(?0079029C      ;  ASCII "_$#abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"  
007901C1   |.  E8 2A4CC7FF     call dumped(?00404DF0         ;  after this, ss:12FD84 point to string above  
007901C6   |.  8D45 F0         lea eax,dword ptr ss:[ebp-10] ;  eax=0012FD80  
007901C9   |.  50              push eax  
007901CA   |.  B9 01000000     mov ecx,1  
```



```
007901CF   |.  8BD3             mov edx,ebx                     ;  ecx=edx=dbx=1  

007901D1   |.  8B45 F8         mov eax,dword ptr ss:\[ebp-8\]    ;  eax='Yq...' string  

007901D4   |.  E8 9F50C7FF     call dumped(?00405278            ;  got the first ECX chars from eax string. saved into \[ss:ebp-10\]  

007901D9   |.  8D45 F8         lea eax,dword ptr ss:\[ebp-8\]    ;  EAX=0012FD88  

007901DC   |.  8B55 F8         mov edx,dword ptr ss:\[ebp-8\]    ;  EDX='Yq...' string  

007901DF   |.  E8 444EC7FF     call dumped(?00405028  

007901E4   |.  8D45 F8         lea eax,dword ptr ss:\[ebp-8\]    ;  eax=0012FD88  

007901E7   |.  50             push eax  

007901E8   |.  B9 41000000     mov ecx,41                      ;  ecx=41（the length of 'Yq...' string)  

007901ED   |.  8BD3            mov edx,ebx                       ;  ebx=dex=1  

007901EF   |.  8B45 F8         mov eax,dword ptr ss:\[ebp-8\]    ;  eax='Y...' string  

007901F2   |.  E8 8150C7FF     call dumped(?00405278            ;  F7 to enter this one, check 008335C8  

007901F7   |.  8D45 EC         lea eax,dword ptr ss:\[ebp-14\]  

007901FA   |.  50             push eax  

007901FB   |.  8B4D F4         mov ecx,dword ptr ss:\[ebp-C\]    ;  string '\_$...'  

007901FE   |.  8B55 F8         mov edx,dword ptr ss:\[ebp-8\]    ;  string 'Yq...'  

00790201   |.  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]    ;  eax=first 2 chars in input password  

00790204   |.  E8 B3FEFFFF     call dumped(?007900BC  

00790209   |.  8B55 EC         mov edx,dword ptr ss:\[ebp-14\]   ;  edx = 'j0', this was encrypted chars  

0079020C   |.  8D45 F0         lea eax,dword ptr ss:\[ebp-10\]  

0079020F   |.  E8 144EC7FF     call dumped(?00405028  

00790214   |.  8BC6            mov eax,esi  

00790216   |.  8B55 F0         mov edx,dword ptr ss:\[ebp-10\]  

00790219   |.  E8 8E4BC7FF     call dumped(?00404DAC  

0079021E   |.  33C0            xor eax,eax  

00790220   |.  5A             pop edx  

00790221   |.  59             pop ecx  

00790222   |.  59             pop ecx  

00790223   |.  64:8910         mov dword ptr fs:\[eax\],edx  

00790226   |.  68 40027900     push dumped(?00790240  

0079022B   |>  8D45 EC         lea eax,dword ptr ss:\[ebp-14\]  

0079022E   |.  BA 05000000     mov edx,5  

00790233   |.  E8 444BC7FF     call dumped(?00404D7C  

00790238   |\.  C3             retn                              ;  返回到上面  

\============================================================================================================  
```

上面我们完成了对注册码的所有分析，本来想写注册机的，可是我的汇编已经有 8 年没有用过了。我会努力的!! 感兴趣的大侠可以根据我的算法分析自己写吧 ：（

另外提醒，如果是爆破，有两个地方要改的。不要只改一个地方 ：）

`4dd466d0e1e0cc123afbe504da2462ca`
