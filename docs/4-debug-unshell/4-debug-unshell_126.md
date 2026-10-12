
# 阿达连连看 3.80

> 原文：[`www.pediy.com/kssd/pediy06/pediy6901.htm`](https://www.pediy.com/kssd/pediy06/pediy6901.htm)

## 破文信息

*   **破文标题**：阿达连连看 3.80 再脱再爆
*   **破文作者**：二哥 weiyi75\[Dfcg\]
*   **作者邮箱**：`weiyi75@sohu.com`
*   **作者主页**：Dfcg 官方大本营+龙族联盟论坛
*   **使用工具**：`peid`, `UnkillOD`
*   **破解平台**：`Win2000/XP`
*   **软件名称**：阿达连连看 3.80
*   **下载地址**：`http://www.chinadfcg.com/viewthread.php?tid=11729`

## 软件简介

漂亮的办公小姐在电脑前埋头苦干的时候，十之八九，她在玩阿达连连看！这是一款很耐玩的桌面游戏，时下办公一族的新宠。

1. 游戏中包含了数套图案关卡，可以测试玩家眼明手快及逻辑判断能力。
2. 包含关卡有可爱的动物、神奇宝贝、星座传说、麻将、甜点饼干、电脑系统、街头霸王、机器猫。
3. 绿色软件，不在系统中留下任何垃圾。
4. 华丽的画面、动人的音效，令人欲罢不能。

**软件大小**：`4.66M`

## 破解目的与声明

**破解目的**：为我的破解之路铺一块小石头。

**破解声明**：我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

上次二哥的阿达连连看 3.58 先脱后爆没有脱衣爆破，随着这个软件流程的熟悉，先脱后爆更容易。

首先 `Peid` 查壳，`Nothing found *!` 再看 `EP` 区段 `.perplex`，原来是 `Acprotect1.X` 版加的壳。

因为是 `VB` 程序，`Stolen Code` 很简单，没有必要苦苦跟踪，`VB` 的 `IAT` 加密目前大部分加密软件都是有心无力的。

`OD` 异常设置不忽略内存异常，全自动隐藏 `OD` 插件帮你隐藏住 `OD`, 载入程序。

### 1. 初跟踪 Stolen Code

```
00451000  a>  60                          pushad //外壳入口,F9 运行。
00451001   66:81C0 EE8F                   add ax,8FEE
00451006   66:13EF                        adc bp,di
00451009   FC                             cld
0045100A   85C8                           test eax,ecx
0045100C   72 03                          jb short adalinks.00451011
0045100E   73 01                          jnb short adalinks.00451011
00451010  - 72 D3                          jb short adalinks.00450FE5
00451012   DDB9 59104500                  fstsw word ptr ds:[ecx+451059]
00451018   68 E860726B                    push 6B7260E8
0045101D   66:81DF BA66                   sbb di,66BA
00451022   58                             pop eax
00451023   BA 1A2B66B2                    mov edx,B2662B1A
00451028   0F81 04000000                  jno adalinks.00451032
0045102E   66:BD 002E                     mov bp,2E00
00451032   81EA 092B66B2                  sub edx,B2662B09
00451038   8B19                           mov ebx,dword ptr ds:[ecx]
..............................................................................
0045FAAD   CD 01                          int 1   //最后一次异常。
0045FAAF   40                             inc eax
0045FAB0   40                             inc eax
0045FAB1   0BC0                           or eax,eax
0045FAB3   75 05                          jnz short adalinks.0045FABA
0045FAB5   90                             nop
0045FAB6   90                             nop
0045FAB7   90                             nop
0045FAB8   90                             nop
0045FAB9   61                             popad
0045FABA   33C0                           xor eax,eax
0045FABC   64:8F00                        pop dword ptr fs:[eax]
0045FABF   58                             pop eax
0045FAC0   60                             pushad
0045FAC1   E8 00000000                    call adalinks.0045FAC6
0045FAC6   5E                             pop esi
0045FAC7   83EE 06                        sub esi,6
0045FACA   B9 57000000                    mov ecx,57
..............................................................................
```

#### Alt+M 打开内存镜像

内存镜像，项目 `21`

*   地址=`00401000` //对准这里 `F2` 断点, `Shift+F9` 飞向光明之巅
*   大小=`00048000` (294912.)
*   Owner=`adalinks` `00400000`
*   区段=`.text`
*   包含=`code`
*   类型=`Imag` `01001002`
*   访问=`R`
*   初始访问=`RWE`

```
004023F6   - FF25 40114000              jmp dword ptr ds:[401140]               ; MSVBVM60.EVENT_SINK_Release
004023FC   - FF25 E4114000              jmp dword ptr ds:[4011E4]  ; MSVBVM60.ThunRTMain
```

**Stolen Code2** `Call` `004023FC` 执行效果

```
00402402   0000                         add byte ptr ds:[eax],al
00402404   41                           inc ecx
00402405   D22D F009800C                shr byte ptr ds:[C8009F0],cl
0040240B   BD 6C6E0000                  mov ebp,6E6C
00402410   48                           dec eax
00402411   0000                         add byte ptr ds:[eax],al
00402413   0030                         add byte ptr ds:[eax],dh
00402415   0000                         add byte ptr ds:[eax],al
00402417   0040 00                      add byte ptr ds:[eax],al
0040241A   0000                         add byte ptr ds:[eax],al
```

堆栈友好提示

```
0012FFBC   0046A1F8  返回到 adalinks.0046A1F8 来自 adalinks.004023FC
0012FFC0   004028C8  adalinks.004028C8 //Stolen Code1 Push 4028C8 执行效果
0012FFC4   77E614C7  返回到 kernel32.77E614C7
0012FFC8   0012CEA8
0012FFCC   004AA38C
0012FFD0   7FFDF000
```

根据五种语言入口熟悉和堆栈提示修复入口如下

```
004023F0   - FF25 F4104000              jmp dword ptr ds:[<&msvbvm60.EVENT_SINK_Add>; msvbvm60.EVENT_SINK_AddRef
004023F6   - FF25 40114000              jmp dword ptr ds:[<&msvbvm60.EVENT_SINK_Rel>; msvbvm60.EVENT_SINK_Release
004023FC   - FF25 E4114000              jmp dword ptr ds:[<&msvbvm60.ThunRTMain>]   ; msvbvm60.ThunRTMain
00402402   0000                         add byte ptr ds:[eax],al //不能在这里处理，否则无法运行
00402404 U>  68 C8284000                push Unpack_.004028C8 //修复如下,用 OD 插件直接修正入口为 2404，重建输入表方式 1 即可运行。
00402409   E8 EEFFFFFF                  call <jmp.&msvbvm60.ThunRTMain>
...............................................................................
```

由于 `Acprotect` 更新 N 代，简单的 `Esp` 定律已经无法找到 `Stolen Code.`

### Fly 大侠的话

在壳把所有的代码解压之后、处理 `Stolen Code` 之前，把进程 `Dump` 出来，补上那段壳代码，模仿构造当时的堆栈和寄存器值环境，这样就由壳自己来解决 `Stolen Code` 的问题啦。推而广之，这个 `Stolen Code` 简便解决方案也适用于某些其他壳。

由于 `Acprotect` 更新 N 代，这个版本已经很强悍了，入口校验我们后面分析。

借 `VB` 的软柿子了解如何能带发修行。

### 用 `push 004028C8` 入手，`ESP` 定律辅助，重起 `OD`

```
00451000  a>  60                          pushad //F8
00451001   66:81C0 EE8F                   add ax,8FEE  //hr esp ,F9 运行
00451006   66:13EF                        adc bp,di
00451009   FC                             cld
0045100A   85C8                           test eax,ecx
0045100C   72 03                          jb short adalinks.00451011
0045100E   73 01                          jnb short adalinks.00451011
...............................................................................
```

还是分对中断

```
004626FF   61                             popad  '1
00462700   893D 8F1E4500                  mov dword ptr ds:[451E8F],edi
00462750   60                             pushad '2
00462751   E8 0ABDFFFF                    call adalinks.0045E460
0046276E   61                             popad
0046276F   8B1C24                         mov ebx,dword ptr ss:[esp]
004627BF   60                             pushad '3
004627C0   E8 9BBCFFFF                    call adalinks.0045E460
004627D0   61                             popad
004627D1   8F05 F31E4500                  pop dword ptr ds:[451EF3]
00462821   60                             pushad '4
00462822   E8 39BCFFFF                    call adalinks.0045E460
0046282E   61                             popad
0046282F   BA EF1E4500                    mov edx,adalinks.00451EEF
0046287F   60                             pushad '5
00462880   E8 15DEFFFF                    call adalinks.0046069A
00462885   61                             popad
00462886   8B3C24                         mov edi,dword ptr ss:[esp]
004628D6   60                             pushad '6
004628D7   E8 5BDBFFFF                    call adalinks.00460437
004628DC   61                             popad
004628DD   8B35 A71E4500                  mov esi,dword ptr ds:[451EA7]
0046292D   60                             pushad '7
0046292E   E8 86FDFFFF                    call adalinks.004626B9
```



### 反汇编代码段 1

```
00462933     61                            popad

00462934     891D 971E4500                 mov dword ptr ds:[451E97],ebx '到这里滚动条往下,不注意下面的 Stolen Code 变形位置根本发现不了，这里代发修行，重建输入表方式 1 即可运行，代码资源已经解压。

0046293A     FF35 971E4500                 push dword ptr ds:[451E97]

00462940     57                            push edi

00462941     BF E31E4500                   mov edi, adalinks.00451EE3

00462946     8BDF                           mov ebx, edi

00462948     5F                            pop edi

00462949     8B3B                           mov edi, dword ptr ds:[ebx]

0046294B     8F05 931E4500                 pop dword ptr ds:[451E93]

00462951     8B1D 931E4500                 mov ebx, dword ptr ds:[451E93]

00462957     8B0C24                        mov ecx, dword ptr ss:[esp]

0046295A     8F05 FF1E4500                 pop dword ptr ds:[451EFF]

00462960     8907                           mov dword ptr ds:[edi], eax

00462962     8F05 071F4500                 pop dword ptr ds:[451F07]

00462968     FF35 071F4500                 push dword ptr ds:[451F07]

0046296E     8B3C24                        mov edi, dword ptr ss:[esp]

00462971     8F05 DF1E4500                 pop dword ptr ds:[451EDF]

00462977     FF35 0F1F4500                 push dword ptr ds:[451F0F]

0046297D     C70424 C8284000               mov dword ptr ss:[esp], adalinks.004028C8 //这个，我是 12ffc0 下硬件断点发现的

00462984     60                            pushad //保存所有寄存器，后面的代码变形处理 Stolen

00462985     E8 D6BAFFFF                   call adalinks.0045E460

0046298A     6A 00                         push 0

0046298C     E8 0A000000                   call adalinks.0046299B //里面有 INT1 中断

00462991     41                            inc ecx

00462992     43                            inc ebx

00462993     50                            push eax

00462994     72 6F                         jb short adalinks.00462A05

00462996     74 65                         je short adalinks.004629FD

00462998     637400 E8                     arpl word ptr ds:[eax+eax\-18], si

0046299C     25 00000050                   and eax, 50000000

004629A1     72 6F                         jb short adalinks.00462A12

...........................................................................................

0045FAAD     CD 01                         int 1 //内存异常

0045FAAF     40                            inc eax

0045FAB0     40                            inc eax

0045FAB1     0BC0                          or eax, eax

0045FAB3     75 05                         jnz short adalinks.0045FABA

0045FAB5     90                            nop

0045FAB6     90                            nop

0045FAB7     90                            nop

0045FAB8     90                            nop

0045FAB9     61                            popad

0045FABA     33C0                          xor eax, eax

0045FABC     64:8F00                       pop dword ptr fs:[eax]

0045FABF     58                            pop eax

0045FAC0     60                            pushad

0045FAC1     E8 00000000                   call adalinks.0045FAC6

0045FAC6     5E                            pop esi

0045FAC7     83EE 06                       sub esi, 6

0045FACA     B9 57000000                   mov ecx, 57

0045FACF     29CE                          sub esi, ecx

0045FAD1     BA D4BB3510                   mov edx, 1035BBD4

0045FAD6     C1E9 02                       shr ecx, 2

0045FAD9     83E9 02                       sub ecx, 2

0045FADC     83F9 00                       cmp ecx, 0

0045FADF     7C 1A                         jl short adalinks.0045FAFB

0045FAE1     8B048E                        mov eax, dword ptr ds:[esi+ecx*4]

0045FAE4     8B5C8E 04                     mov ebx, dword ptr ds:[esi+ecx*4+4]

0045FAE8     03C3                          add eax, ebx

0045FAEA     C1C8 1B                       ror eax, 1B

0045FAED     03C2                          add eax, edx

0045FAEF     81C2 C7E901AD                 add edx, AD01E9C7

0045FAF5     89048E                        mov dword ptr ds:[esi+ecx*4], eax

0045FAF8     49                            dec ecx

0045FAF9    ^ EB E1                        jmp short adalinks.0045FADC

0045FAFB     61                            popad

0045FAFC     61                            popad

0045FAFD     C3                            retn //这里下断，Shift+F9 断下
```

#### 堆栈友好提示

```
0012FF9C    00464141   返回到 adalinks.00464141 来自 adalinks.0045F8BE
0012FFA0    0012CEA8
0012FFA4    004AA38C
0012FFA8    0012FFF0
0012FFAC    0012FFC0
0012FFB0    7FFDF000
0012FFB4    7FFE0304
0012FFB8    0012FFB0
0012FFBC    00000000
0012FFC0    004028C8   adalinks.004028C8 //知道 12ffc0 保存 Stolen,于是启动是硬件写入断点发现前面我提到的位置
```

要找第二句代码处理位置，`Alt+O` 设置异常不忽略 `INT3` 异常。

```
0045FCFF     90                            nop //INT3 中断

0045FD00     64:67:8F06 0000               pop dword ptr fs:[0]

0045FD06     83C4 04                       add esp, 4

0045FD09     60                            pushad

0045FD0A     E8 00000000                   call adalinks.0045FD0F

0045FD0F     5E                            pop esi

0045FD10     83EE 06                       sub esi, 6

0045FD13     B9 5B000000                   mov ecx, 5B

0045FD18     29CE                          sub esi, ecx

0045FD1A     BA 9575B0C3                   mov edx, C3B07595

0045FD1F     C1E9 02                       shr ecx, 2

0045FD22     83E9 02                       sub ecx, 2

0045FD25     83F9 00                       cmp ecx, 0

0045FD28     7C 1A                         jl short adalinks.0045FD44

0045FD2A     8B048E                        mov eax, dword ptr ds:[esi+ecx*4]

0045FD2D     8B5C8E 04                     mov ebx, dword ptr ds:[esi+ecx*4+4]

0045FD31     03C3                          add eax, ebx

0045FD33     C1C0 1E                       rol eax, 1E

0045FD36     33C2                          xor eax, edx

0045FD38     81F2 3E24211C                 xor edx, 1C21243E

0045FD3E     89048E                        mov dword ptr ds:[esi+ecx*4], eax

0045FD41     49                            dec ecx

0045FD42    ^ EB E1                        jmp short adalinks.0045FD25

0045FD44     61                            popad

0045FD45     61                            popad

0045FD46     C3                            retn //这里下 F2 断点,Shift+F9 到这里,F9 运行。

0046A0E2     61                            popad

0046A0E3     57                            push edi                             ; USER32.77D29D31

0046A0E4     890424                        mov dword ptr ss:[esp], eax

0046A0E7     8F05 FF1E4500                 pop dword ptr ds:[451EFF]

0046A0ED     FF35 FF1E4500                 push dword ptr ds:[451EFF]

0046A0F3     890C24                        mov dword ptr ss:[esp], ecx

0046A0F6     8905 E71E4500                 mov dword ptr ds:[451EE7], eax

0046A0FC     FF35 E71E4500                 push dword ptr ds:[451EE7]         ; adalinks.00451F0F

0046A102     C70424 FC234000               mov dword ptr ss:[esp], adalinks.004023FC    ; jmp to MSVBVM60.ThunRTMain

//平时看 OD 载入 VB 程序应该很熟悉这个 jmp to MSVBVM60.ThunRTMain  Stolen2，待处理

0046A109     8F05 071F4500                 pop dword ptr ds:[451F07]          ; USER32.77D29D31

0046A10F     FF35 071F4500                 push dword ptr ds:[451F07]         ; USER32.77D29D31

0046A115     8F05 F31E4500                 pop dword ptr ds:[451EF3]

0046A11B     53                            push ebx
```



### 代码段分析

```
0046A11C     BB F31E4500                       mov ebx,adalinks.00451EF3

0046A121     8B0B                              mov ecx,dword ptr ds:[ebx]

0046A123     5B                                pop ebx

0046A124     891D E31E4500                     mov dword ptr ds:[451EE3],ebx

0046A12A     FF35 E31E4500                     push dword ptr ds:[451EE3]                   ; adalinks.00451F0F

0046A130     51                                push ecx

0046A131     90                                nop

0046A132     90                                nop

0046A133     60                                pushad //系列处理变形，使得本来效率不高的 VB 程序更慢，加密软件可不管你。

0046A134     E8 2743FFFF                       call adalinks.0045E460

0046A139     8B85 7AA24100                     mov eax,dword ptr ss:[ebp+41A27A]

0046A13F     0385 AAD24000                     add eax,dword ptr ss:[ebp+40D2AA]

0046A145     8985 7AA24100                     mov dword ptr ss:[ebp+41A27A],eax

0046A14B     61                                popad
```

......................................................................... //慢慢回到

```
004023F6   - FF25 40114000                     jmp dword ptr ds:[401140]                     ; MSVBVM60.EVENT_SINK_Release

004023FC   - FF25 E4114000                     jmp dword ptr ds:[4011E4]  ; MSVBVM60.ThunRTMain
```

### 带发修行

```
00462934     891D 971E4500                     mov dword ptr ds:[451E97],ebx '到这里滚动条往下,不注意下面的 Stolen Code 变形位置根本发现不了，这里代发修行，重建输入表方式 1 即可运行，代码资源已经解压。

0046293A     FF35 971E4500                     push dword ptr ds:[451E97]

00462940     57                                push edi

00462941     BF E31E4500                       mov edi,adalinks.00451EE3

00462946     8BDF                              mov ebx,edi

00462948     5F                                pop edi
```

..................................................................................

上述两种脱法各位读者不知看懂没有，不知喜欢哪种。

呵呵，带发修行还是好玩些。

### 2. 对付 `SDK` 方式的入口校验

继续载入带发程序，运行，伪注册后确认程序出错，进入程序后退出也出错，显然是作者的 `SDK` 方式阻止脱壳破解。

我们以退出时错误突破

```
00143358     0000                              add byte ptr ds:[eax],al //这里错误

0014335A     0000                              add byte ptr ds:[eax],al

0014335C     0000                              add byte ptr ds:[eax],al

0014335E     0000                              add byte ptr ds:[eax],al

00143360     0000                              add byte ptr ds:[eax],al

00143362     0000                              add byte ptr ds:[eax],al

00143364     0000                              add byte ptr ds:[eax],al

00143366     0000                              add byte ptr ds:[eax],al

00143368     0000                              add byte ptr ds:[eax],al
```

//堆栈友好提示

```
0012F440    0042FA16  返回到 fsfd.0042FA16 来自 0014334A  //直接右键反汇编中跟随

0012F444    0012F5F8

0012F448    0012F67C

0012F44C    00000001

0012F450    7E192002  返回到 GDI32.7E192002 来自 GDI32.7E192033

0012F454    001A82E8

0012F458    00380910

0012F45C    00000001

0012F460    0185A008

0012F464    00000000

0012F468    00000000
```

向上返回定律找事件代码第一句

```
0042F940    $  55                                push ebp  //第 1 句，跟踪原程序对比分析

0042F941    .  8BEC                              mov ebp,esp

0042F943    .  83EC 08                           sub esp,8

0042F946    .  68 66204000                        push fsfd.00402066                        ;  jmp to MSVBVM60.__vbaExceptHandler; SE handler installation

0042F94B    .  64:A1 00000000                     mov eax,dword ptr fs:[0]

0042F951    .  50                                push eax

0042F952    .  64:8925 00000000                   mov dword ptr fs:[0],esp
```

...............................................................................................

```
0042FA0E    .  50                                push eax

0042FA0F    .  FFD7                              call edi

0042FA11    .  E8 3439D1FF                         call 0014334A //问题是这里错误，跟进看看。
```

进去看看就用了我 6-7 个小时，哎，总算是清楚了一些。

```
0045240C    $  60                                pushad //首先用花指令插件清除 34 个小花

0045240D    .  78 01                             js short fsfd.00452410

0045240F    .  FC                                cld

00452410    >  72 03                             jb short fsfd.00452415

00452412    .  73 01                             jnb short fsfd.00452415

00452414       7A                                 db 7A                                     ;  CHAR 'z'

00452415    .  66:2BC5                            sub ax,bp

00452418    .  EB 01                              jmp short fsfd.0045241B
```

这一大段带小花的语句作用是对

```
004525B7     E8 A4BE0000                         call 0045E460 等语句解码

004525BC     8B4424 20                           mov eax,dword ptr ss:[esp+20]

004525C0     33C9                              xor ecx,ecx
```

```
0045240C     60                                pushad

0045240D     78 01                             js short fsfd.00452410

0045240F     FC                                cld

00452410     90                                nop

00452411     90                                nop

00452412     90                                nop

00452413     90                                nop

00452414     90                                nop

00452415     66:2BC5                             sub ax,bp
```

.......................................................................

```
0045259F   ^\0F85 6BFFFFFF                       jnz fsfd.00452510 //循环解码

004525A5     90                                nop //F4 下来可以看到
```

到

```
004525B7
```

到

```
004526E1
```

之间的代码已经解压

```
004525A6     90                                nop

004525A7     90                                nop

004525A8     90                                nop

004525A9     90                                nop

004525AA     90                                nop

004525AB     90                                nop

004525AC     90                                nop

004525AD     90                                nop

004525AE     90                                nop

004525AF     90                                nop

004525B0     90                                nop

004525B1     90                                nop

004525B2     90                                nop

004525B3     90                                nop

004525B4     90                                nop

004525B5     90                                nop

004525B6     90                                nop

004525B7     82AD 343433E6 EC                  sub byte ptr ss:[ebp+E6333434],-14 //一直到这里后面代码没有解压，有未知命令，可以直接 F4 下来。

004525BE     0C 30                            or al,30

004525C0     2C 01                            sub al,1

004525C2     99                                cdq

004525C3     2F                                das

004525C4     863A                              xchg byte ptr ds:[edx],bh

004525C6     C3                                retn

004525C7     FF7F                              ???                                     ; 未知命令

004525C9     B9 1F286FA6                       mov ecx,A66F281F
```

............................................................................................

转到这里继续

```
004525B7     E8 A4BE0000                         call fsfd.0045E460

004525BC     8B4424 20                           mov eax,dword ptr ss:[esp+20]

004525C0     33C9                              xor ecx,ecx

004525C2     8B9C8D E2264000                    mov ebx,dword ptr ss:[ebp+ecx*4+4026E2]

004525C9     039D AAD24000                     add ebx,dword ptr ss:[ebp+40D2AA]

004525CF     3BC3                              cmp eax,ebx

004525D1     74 07                            je short fsfd.004525DA

004525D3     90                                nop

004525D4     90                                nop

004525D5     90                                nop

004525D6     90                                nop

004525D7     41                                inc ecx
```



### 004525D8
```
^ EB E8
```
`jmp short fsfd.004525C2` //这段循环看不懂,好像不重要

### 004525DA
```
C7848D E2264000 00000000
```
`mov dword ptr ss:[ebp+ecx*4+4026E2],0`

### 004525E5
```
8DB5 C2554000
```
`lea esi,dword ptr ss:[ebp+4055C2]`

### 004525EB
```
B8 0A000000
```
`mov eax,0A`

### 004525F0
```
F7E1
```
`mul ecx`

### 004525F2
```
03F0
```
`add esi,eax`

### 004525F4
```
56
```
`push esi`

### 004525F5
```
51
```
`push ecx`

### 004525F6
```
8A85 FC234000
```
`mov al,byte ptr ss:[ebp+4023FC]`

### 004525FC
```
0AC0
```
`or al,al`

### 004525FE
```
75 28
```
`jnz short fsfd.00452628` //慢慢到这里注意了,精神些。

### 00452600
```
90
```
`nop`

### 00452601
```
90
```
`nop`

### 00452602
```
90
```
`nop`

### 00452603
```
90
```
`nop`

### 00452604
```
8B85 AAD24000
```
`mov eax,dword ptr ss:[ebp+40D2AA]` ; `fsfd.00400000`

### 0045260A
```
8B70 3C
```
`mov esi,dword ptr ds:[eax+3C]`

### 0045260D
```
03B5 AAD24000
```
`add esi,dword ptr ss:[ebp+40D2AA]` ; `fsfd.00400000`

### 00452613
```
83C6 28
```
`add esi,28`

### 00452616
```
AD
```
`lods dword ptr ds:[esi]` //就是这个 `dword ptr ds:[esi]`,值 00062934 就是我的入口,读者可以用 `Peid` 看看就知道了，原程序这里是 51000 ，这句执行后 `EAX=00062934`

### 00452617
```
8AD8
```
`mov bl,al`

### 00452619
```
02DC
```
`add bl,ah`

### 0045261B
```
C1E8 10
```
`shr eax,10`

### 0045261E
```
02D8
```
`add bl,al`

### 00452620
```
02DC
```
`add bl,ah` //后面某处和原入口 51000 对比，正确则解压 004526E1 处需要的正确代码，否则是错误的垃圾代码。

### 00452622
```
889D FC234000
```
`mov byte ptr ss:[ebp+4023FC],bl`

### 00452628
```
59
```
`pop ecx`

..................................................................................

### 004526CE
```
C1C0 09
```
`rol eax,9`

### 004526D1
```
2BC2
```
`sub eax,edx`

### 004526D3
```
81C2 820378F5
```
`add edx,F5780382`

### 004526D9
```
89048E
```
`mov dword ptr ds:[esi+ecx*4],eax`

### 004526DC
```
49
```
`dec ecx`

### 004526DD
```
^ EB E1
```
`jmp short fsfd.004526C0`

### 004526DF
```
61
```
`popad`

### 004526E0
```
61
```
`popad`

### 004526E1
```
C3
```
`retn` //返回 14XXXX 低内存段，此时是错误的垃圾代码。

### 0014334A
```
FD
```
`std` //执行 `OVER`

### 0014334B
```
33DA
```
`xor ebx,edx`

### 0014334D
```
25 FD7E2DB5
```
`and eax,B52D7EFD`

### 00143352
```
9F
```
`lahf`

### 00143353
```
A0 00071C00
```
`mov al,byte ptr ds:[1C0700]`

### 00143358
```
0000
```
`add byte ptr ds:[eax],al` //空地址

### 0014335A
```
0000
```
`add byte ptr ds:[eax],al`

### 0014335C
```
0000
```
`add byte ptr ds:[eax],al`

..................................................................................

基本分析清楚，可以用 `Fly` 大侠的方法修改为原入口然后跳到脱壳程序入口。

也可以这样，麻烦但是可以锻炼技术。

### 修改 1

#### 0045240C
```
60
```
`pushad`

#### 0045240D
```
78 01
```
`js short fsfd.00452410`

#### 0045240F
```
FC
```
`cld`

#### 00452410
```
72 03
```
`jb short fsfd.00452415`

#### 00452412
```
73 01
```
`jnb short fsfd.00452415`

#### 00452414
```
7A 66
```
`jpe short fsfd.0045247C`

修改为

#### 0045240C
```
60
```
`pushad`

#### 0045240D
```
E9 A5010000
```
`jmp fsfd.004525B7`

#### 00452412
```
73 01
```
`jnb short fsfd.00452415`

#### 00452414
```
7A 66
```
`jpe short fsfd.0045247C`

下面的代码是通过跟踪原程序得到，可以通过二进制复制粘贴覆盖原代码，这个二哥的

`ASF-AVI-RM-WMV Repair V1.41` 脱壳去暗桩+汉化完美爆破

`ASProtect 1.1b Registered SDK` 之神奇挂挂 `3.7` 版脱壳+去暗桩

都详细写过，是 `OD` 的基本操作，必须掌握，不再赘述。

### 修改 2,动态代码

#### 004525B7
```
E8 A4BE0000
```
`call fsfd.0045E460`

#### 004525BC
```
8B4424 20
```
`mov eax,dword ptr ss:[esp+20]`

#### 004525C0
```
33C9
```
`xor ecx,ecx`

#### 004525C2
```
8B9C8D E2264000
```
`mov ebx,dword ptr ss:[ebp+ecx*4+4026E2]`

#### 004525C9
```
039D AAD24000
```
`add ebx,dword ptr ss:[ebp+40D2AA]`

#### 004525CF
```
3BC3
```
`cmp eax,ebx`

#### 004525D1
```
74 07
```
`je short fsfd.004525DA`

#### 004525D3
```
90
```
`nop`

#### 004525D4
```
90
```
`nop`

#### 004525D5
```
90
```
`nop`

#### 004525D6
```
90
```
`nop`

#### 004525D7
```
41
```
`inc ecx`

#### 004525D8
```
^ EB E8
```
`jmp short fsfd.004525C2`

#### 004525DA
```
C7848D E2264000 00000000
```
`mov dword ptr ss:[ebp+ecx*4+4026E2],0`

#### 004525E5
```
8DB5 C2554000
```
`lea esi,dword ptr ss:[ebp+4055C2]`

#### 004525EB
```
B8 0A000000
```
`mov eax,0A`

#### 004525F0
```
F7E1
```
`mul ecx`

#### 004525F2
```
03F0
```
`add esi,eax`

#### 004525F4
```
56
```
`push esi`

#### 004525F5
```
51
```
`push ecx`

#### 004525F6
```
8A85 FC234000
```
`mov al,byte ptr ss:[ebp+4023FC]`

#### 004525FC
```
0AC0
```
`or al,al`

#### 004525FE
```
75 28
```
`jnz short fsfd.00452628`

#### 00452600
```
90
```
`nop`

#### 00452601
```
90
```
`nop`

#### 00452602
```
90
```
`nop`

#### 00452603
```
90
```
`nop`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

#### 00452604
```
8B85 AAD24000
```
`mov eax,dword ptr ss:[ebp+40D2AA]`

#### 0045260A
```
8B70 3C
```
`mov esi,dword ptr ds:[eax+3C]`

#### 0045260D
```
03B5 AAD24000
```
`add esi,dword ptr ss:[ebp+40D2AA]`

#### 00452613
```
83C6 28
```
`add esi,28`

#### 00452616
```
AD
```
`lods dword ptr ds:[esi]` //需要动外科手术,针对 `dword ptr ds:[esi]`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

修改为

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

#### 00452604
```
B8 00100500
```
`mov eax,51000` //原入口

#### 00452609
```
90
```
`nop`

#### 0045260A
```
90
```
`nop`

#### 0045260B
```
90
```
`nop`

#### 0045260C
```
90
```
`nop`

#### 0045260D
```
90
```
`nop`

#### 0045260E
```
90
```
`nop`

#### 0045260F
```
90
```
`nop`

#### 00452610
```
90
```
`nop`

#### 00452611
```
90
```
`nop`

#### 00452612
```
90
```
`nop`

#### 00452613
```
90
```
`nop`

#### 00452614
```
90
```
`nop`

#### 00452615
```
90
```
`nop`

#### 00452616
```
90
```
`nop`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

#### 00452617
```
8AD8
```
`mov bl,al`

#### 00452619
```
02DC
```
`add bl,ah`



0045261B     C1E8 10                     shr eax,10  

0045261E     02D8                         add bl,al  

00452620     02DC                         add bl,ah  

00452622     889D FC234000                mov byte ptr ss:[ebp+4023FC],bl  

00452628     59                           pop ecx  

00452629     5E                           pop esi  

0045262A     60                           pushad  

0045262B     B8 02000000                  mov eax,2  

00452630     E8 B5BB0000                  call fsfd.0045E1EA  

00452635     0BC0                         or eax,eax  

00452637     75 24                        jnz short fsfd.0045265D  

00452639     90                           nop  

0045263A     90                           nop  

0045263B     90                           nop  

0045263C     90                           nop  

0045263D     61                           popad  

0045263E     8BBD AED24000                mov edi,dword ptr ss:[ebp+40D2AE]  

00452644     B8 0A000000                  mov eax,0A  

00452649     F7E1                         mul ecx  

0045264B     03F8                         add edi,eax  

0045264D     B9 0A000000                  mov ecx,0A  

00452652     8A9D FC234000                mov bl,byte ptr ss:[ebp+4023FC]  

00452658     EB 11                        jmp short fsfd.0045266B  

0045265A     90                           nop  

0045265B     90                           nop  

0045265C     90                           nop  

0045265D     61                           popad  

0045265E     8BFE                         mov edi,esi  

00452660     B9 0A000000                  mov ecx,0A  

00452665     8A9D FC234000                mov bl,byte ptr ss:[ebp+4023FC]  

0045266B     AC                           lods byte ptr ds:[esi]  

0045266C     32C3                         xor al,bl  

0045266E     AA                           stos byte ptr es:[edi]  

0045266F   ^ E2 FA                       loopd short fsfd.0045266B  

00452671     83EF 0A                      sub edi,0A  

00452674     57                           push edi  

00452675     8B7424 24                   mov esi,dword ptr ss:[esp+24]  

00452679     83EE 04                     sub esi,4  

0045267C     AD                           lods dword ptr ds:[esi]  

0045267D     81EF 0C244000               sub edi,fsfd.0040240C               ; ASCII "ln"  

00452683     2BFD                         sub edi,ebp  

00452685     03C7                         add eax,edi  

00452687     8946 FC                       mov dword ptr ds:[esi-4],eax  

0045268A     5F                           pop edi  

0045268B     57                           push edi  

0045268C     33C9                         xor ecx,ecx  

0045268E     83F9 08                     cmp ecx,8  

00452691     74 0E                        je short fsfd.004526A1  

00452693     90                           nop  

00452694     90                           nop  

00452695     90                           nop  

00452696     90                           nop  

00452697     8B448C 04                   mov eax,dword ptr ss:[esp+ecx*4+4]  

0045269B     89048C                       mov dword ptr ss:[esp+ecx*4],eax  

0045269E     41                           inc ecx  

.................................................................................................  

保存所有修改，仍然无法运行。  

原因是这里  

```
0045268E     83F9 08                       cmp ecx,8  

00452691     74 0E                        je short fsfd.004526A1  

00452693     90                           nop  

00452694     90                           nop  

00452695     90                           nop  

00452696     90                           nop  

00452697     8B448C 04                   mov eax,dword ptr ss:[esp+ecx*4+4]  

0045269B     89048C                       mov dword ptr ss:[esp+ecx*4],eax  

0045269E     41                           inc ecx  

0045269F   ^ EB ED                       jmp short fsfd.0045268E  

004526A1     893C8C                       mov dword ptr ss:[esp+ecx*4],edi //到这里既然我们已经手动解码了，当然要功成身退了。  
```

```
004526A4     60                           pushad  
```
注意,下面这段代码是解压我们复制的二进制代码，我们已经提前修改了，这里再次解码就会冲突，具体原因向下看标签 1，所以直接跳到 `004526E0`

### 外科手术

```
004526A4     60                           pushad  

004526A4   - E9 3700FBFF                jmp fsfd.004026E0  

004526A9     90                           nop  
```

这次就没问题了。

```
004526A5     E8 00000000                  call fsfd.004526AA  

004526AA     5E                           pop esi  

004526AB     83EE 06                     sub esi,6  

004526AE     B9 ED000000                  mov ecx,0ED  

004526B3     29CE                         sub esi,ecx  

004526B5     BA 73F68FC7                  mov edx,C78FF673  

004526BA     C1E9 02                      shr ecx,2  

004526BD     83E9 02                     sub ecx,2  

004526C0     83F9 00                      cmp ecx,0  

004526C3     7C 1A                        jl short fsfd.004526DF  

004526C5     8B048E                       mov eax,dword ptr ds:[esi+ecx*4]  

004526C8     8B5C8E 04                    mov ebx,dword ptr ds:[esi+ecx*4+4]  

004526CC     2BC3                         sub eax,ebx  

004526CE     C1C0 09                      rol eax,9  

004526D1     2BC2                         sub eax,edx  

004526D3     81C2 820378F5               add edx,F5780382  

004526D9     89048E                       mov dword ptr ds:[esi+ecx*4],eax  

004526DC     49                           dec ecx  

004526DD   ^ EB E1                       jmp short fsfd.004526C0  

004526DF     61                           popad //注意  

004526E0     61                           popad  

004526E1     C3                           retn  
```

### 标签 1

我们回头一看,这里都是红色动态解压代码

```
004525B5     87C1                         xchg ecx,eax  

004525B7     11C4                         adc esp,eax  //这你不 Over 吗?  

004525B9     AA                           stos byte ptr es:[edi]  

004525BA     B3 27                        mov bl,27  

004525BC     2BAD C50DCC24               sub ebp,dword ptr ss:[ebp+24CC0DC5]  

004525C2     1F                           pop ds  

004525C3     5F                           pop edi  

004525C4     74 77                        je short fsfd.0045263D  

004525C6     D4 88                        aam 88  

004525C8   ^ E1 B0                       loopde short fsfd.0045257A  

004525CA     07                           pop es  

004525CB     78 73                        js short fsfd.00452640  

004525CD     32AA 1AC199D6               xor ch,byte ptr ds:[edx+D699C11A]  

004525D3     06                           push es  

004525D4     3AB2 CD2137CD               cmp dh,byte ptr ds:[edx+CD3721CD]  

004525DA     A1 5540B60C                  mov eax,dword ptr ds:[CB64055]  

004525DF     15 2A75925E                  adc eax,5E92752A  

004525E4   ^ 79 CA                       jns short fsfd.004525B0  
```

.................................................................................................

也许你会问以壳解壳的作用，它和加壳程序的区别是。

你不用找 Stolen Code 即可修复程序运行，最重要的是破解需要的代码和资源都可以提取修改。

运筹帷幄完毕，去带发修行的程序修改代码到我上面写的。

运行程序，注册和退出都是共用动态解码，退出时候有友好提示，这时你可以知道你已经彻底脱壳了。

### 3. 爆破程序

PEID 算法侦测到 MD5 算法，这个作者没采取 Acprotect 的 RsaKey，不知是盗版的的 AC，还是信不过 AC,而是采用自己的保护方法。

既然外壳已经攻破，当然要爆了。

OD 载入带发程序

```
00462934 f>  891D 971E4500                 mov dword ptr ds:[451E97],ebx //特殊入口，呵呵，这里还是壳，我们动态调试它，Shift+F9 直到运行为止。  

0046293A     FF35 971E4500                 push dword ptr ds:[451E97]  

00462940     57                             push edi  
```



## 汇编代码片段

```
00462941     BF E31E4500                   mov edi,fsfd.00451EE3  
00462946     8BDF                          mov ebx,edi  
00462948     5F                            pop edi  
00462949     8B3B                          mov edi,dword ptr ds:[ebx]  
0046294B     8F05 931E4500                 pop dword ptr ds:[451E93]  
00462951     8B1D 931E4500                 mov ebx,dword ptr ds:[451E93]  
00462957     8B0C24                        mov ecx,dword ptr ss:[esp]  
0046295A     8F05 FF1E4500                 pop dword ptr ds:[451EFF]  
00462960     8907                          mov dword ptr ds:[edi],eax  
00462962     8F05 071F4500                 pop dword ptr ds:[451F07]  
00462968     FF35 071F4500                 push dword ptr ds:[451F07]  
0046296E     8B3C24                        mov edi,dword ptr ss:[esp]  
00462971     8F05 DF1E4500                 pop dword ptr ds:[451EDF]  
00462977     FF35 0F1F4500                 push dword ptr ds:[451F0F]  
0046297D     C70424 C8284000               mov dword ptr ss:[esp],fsfd.004028C8  
```

直接`Ctrl+G` `401000` 去程序领空

老罗的插件搜索双字节

### Ultra 字符串参考

| 地址 | 反汇编 | 文本字符串 |
| :--- | :--- | :--- |
| `0042C048` | `push fsfd.00408DE0` | `\iwinada.dll` //这里应该是注册生成的 |
| `0042C056` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042C1A3` | `push fsfd.00408E00` | `@t@` |
| `0042CD4C` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042CDE9` | `push fsfd.00408E0C` | `@` |
| `0042CE15` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042CE48` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042D197` | `push fsfd.00408E14` | `\0O0O0O0.dll` |
| `0042D1B0` | `push fsfd.00408E30` | `\O0O0O0.dll` |
| `0042D26B` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042D29E` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042D6DC` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042D779` | `push fsfd.00408E4C` | `.` |
| `0042D7A5` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042D7D8` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `0042DEE1` | `push fsfd.00408E64` | `www.myadasoft.com/gift.htm` |
| `00437C28` | `push fsfd.00408DE0` | `\iwinada.dll` //看了一下就发现这里 |
| `00437C3F` | `push fsfd.00408DE0` | `\iwinada.dll` |
| `00438ED1` | `push fsfd.00408EAC` | `www.ayesoftware.com/adagame/` |
| `00438EDD` | `push fsfd.00408E54` | `Open` |
| `00439577` | `push fsfd.00409B34` | `www.suous.com/readgjq.asp?user=` |

```
00437C28     68 E08D4000                   push fsfd.00408DE0                 ; UNICODE "\iwinada.dll"  
00437C2D     66:C705 3E904400 0000        mov word ptr ds:[44903E],0        
00437C36     FFD6                          call esi                          
00437C38     8945 C8                       mov dword ptr ss:[ebp-38],eax    
00437C3B     8B45 E4                       mov eax,dword ptr ss:[ebp-1C]     
00437C3E     50                            push eax                          
00437C3F     68 E08D4000                   push fsfd.00408DE0                 ; UNICODE "\iwinada.dll"  
00437C44     C745 C0 08000000              mov dword ptr ss:[ebp-40],8       
00437C4B     FFD6                          call esi                          
00437C4D     6A 00                         push 0                            
00437C4F     8D4D B0                       lea ecx,dword ptr ss:[ebp-50]     
00437C52     51                            push ecx                          
00437C53     8945 B8                       mov dword ptr ss:[ebp-48],eax    
00437C56     C745 B0 08000000              mov dword ptr ss:[ebp-50],8       
00437C5D     FF15 98114000                 call dword ptr ds:[401198]        ; MSVBVM60.rtcDir  
00437C63     8BD0                          mov edx,eax                       
00437C65     8D4D D4                       lea ecx,dword ptr ss:[ebp-2C]     
00437C68     FFD7                          call edi                          
00437C6A     50                            push eax                          
00437C6B     68 F8864000                   push fsfd.004086F8                
00437C70     FF15 00114000                 call dword ptr ds:[401100]        ; MSVBVM60.__vbaStrCmp  
00437C76     8BF0                          mov esi,eax                       
00437C78     F7DE                          neg esi                           
00437C7A     1BF6                          sbb esi,esi                       
00437C7C     6A 00                         push 0                            
00437C7E     8D55 C0                       lea edx,dword ptr ss:[ebp-40]     
00437C81     F7DE                          neg esi                           
00437C83     52                            push edx                          
00437C84     F7DE                          neg esi                           
00437C86     FF15 98114000                 call dword ptr ds:[401198]        ; MSVBVM60.rtcDir  
00437C8C     8BD0                          mov edx,eax                       
00437C8E     8D4D D8                       lea ecx,dword ptr ss:[ebp-28]     
00437C91     FFD7                          call edi                          
00437C93     50                            push eax                          
00437C94     68 F8864000                   push fsfd.004086F8                
00437C99     FF15 00114000                 call dword ptr ds:[401100]        ; MSVBVM60.__vbaStrCmp  
00437C9F     F7D8                          neg eax                           
00437CA1     1BC0                          sbb eax,eax                       
00437CA3     F7D8                          neg eax                           
00437CA5     F7D8                          neg eax                           
00437CA7     23F0                          and esi,eax                       
00437CA9     8D45 D4                       lea eax,dword ptr ss:[ebp-2C]     
00437CAC     50                            push eax                          
00437CAD     8D4D D8                       lea ecx,dword ptr ss:[ebp-28]     
00437CB0     51                            push ecx                          
00437CB1     6A 02                         push 2                            
00437CB3     FFD3                          call ebx                          
00437CB5     8D55 B0                       lea edx,dword ptr ss:[ebp-50]     
00437CB8     52                            push edx                          
00437CB9     8D45 C0                       lea eax,dword ptr ss:[ebp-40]     
00437CBC     50                            push eax                          
00437CBD     6A 02                         push 2                            
00437CBF     FF15 38104000                 call dword ptr ds:[401038]        ; MSVBVM60.__vbaFreeVarList  
00437CC5     83C4 18                       add esp,18                        
00437CC8     66:85F6                       test si,si                        
00437CCB     74 09                         je short fsfd.00437CD6 //爆破点 1  
00437CCD     66:C705 3E904400 FFFF        mov word ptr ds:[44903E],0FFFF //这里一看就是标志位，FFFF 就是真的意思  
00437CD6     66:833D 3E904400 00          cmp word ptr ds:[44903E],0        
00437CDE     75 05                         jnz short fsfd.00437CE5          
00437CE0     E8 4B84FFFF                   call fsfd.00430130              
```

这里

在`44903E`处下内存读取断点

监视程序哪里想修改它为`0`

不久就发现

```
0042D23F    .   FF15 38104000              call dword ptr ds:[401038]        ;   MSVBVM60.__vbaFreeVarList  
0042D245    .   83C4 0C                    add esp,0C                        
0042D248    .   0FBF4D 8C                  movsx ecx,word ptr ss:[ebp-74]    
0042D24C    .   85C9                       test ecx,ecx                      
0042D24E    .   74 76                      je short fsfd.0042D2C6    //爆破点 2  
0042D250    .   C745 FC 09000000           mov dword ptr ss:[ebp-4],9        
0042D257    .   66:C705 3E904400 0000      mov word ptr ds:[44903E],0 //改写标志位  
0042D260    .   C745 FC 0A000000           mov dword ptr ss:[ebp-4],0A       
0042D267    .   8B55 DC                    mov edx,dword ptr ss:[ebp-24]     
0042D26A    .   52                         push edx                          
0042D26B    .   68 E08D4000                push fsfd.00408DE0              ;   UNICODE "\iwinada.dll"  
0042D270    .   FF15 70104000              call dword ptr ds:[401070]        ;   MSVBVM60.__vbaStrCat  
```



## 汇编代码片段

```
0042D276    .   8945 BC                       mov dword ptr ss:[ebp-44],eax  
0042D279    .   C745 B4 08000000             mov dword ptr ss:[ebp-4C],8  

0042D280    .   8D45 B4                       lea eax,dword ptr ss:[ebp-4C]  

0042D283    .   50                            push eax  

0042D284    .   FF15 04114000                 call dword ptr ds:[401104]                   ;   MSVBVM60.rtcKillFiles  

0042D28A    .   8D4D B4                       lea ecx,dword ptr ss:[ebp-4C]  

0042D28D    .   FF15 1C104000                 call dword ptr ds:[40101C]                   ;   MSVBVM60.__vbaFreeVar  

0042D293    .   C745 FC 0B000000             mov dword ptr ss:[ebp-4],0B  

0042D29A    .   8B4D D8                       mov ecx,dword ptr ss:[ebp-28]  

0042D29D    .   51                            push ecx  

0042D29E    .   68 E08D4000                   push fsfd.00408DE0                            ;   UNICODE "\iwinada.dll"  

0042D2A3    .   FF15 70104000                 call dword ptr ds:[401070]                   ;   MSVBVM60.__vbaStrCat  

....................................................................................  
```

共两处，你可以用这个作品去讨好 MM 了，成了好事别忘记二哥我啊. ^\_^  

## 破解总结

### 启动校验

```
00437CCB      /74 09                          je short fsfd.00437CD6  
```

修改为

```
00437CCB       90                             nop  
00437CCC       90                             nop  
```

### 实时校验

```
0042D24E    . /74 76                          je short fsfd.0042D2C6  
```

修改为

```
0042D24E     /EB 76                          jmp short fsfd.0042D2C6  
```

2004 年 12 月 2 日 13 点 25 分完成文章，历时 7-8 小时。  

\--------------------------------------------------------------------------------   

## 版权声明 

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
