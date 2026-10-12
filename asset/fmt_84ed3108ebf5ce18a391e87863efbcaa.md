
# 天天理财、出差理财好帮手 3.3

> 原文：`www.pediy.com/kssd/pediy08/pediy8-373.htm`

【破文标题】天天理财、出差理财好帮手 3.3 注册算法分析  
【破文作者】`XXNB`  
【作者邮箱】  
【作者主页】  
【破解工具】`OD`  
【破解平台】`xpsp2`  
【软件名称】天天理财、出差理财好帮手 3.3  
【软件大小】`3005KB`  
【原版下载】`www.newhua.com/soft/52835.htm`  
【保护方式】  
【软件简介】天天理财、出差理财好帮手是一款个人理财软件，适用于日常理财、出差理财及平常一些重要项目的理财如装修理财等适用。本软件小巧精致，简单易用。

主要功能有：收支项目设定，收入管理，支出管理，收支查询，报表生成等。理财涉及个人隐私因而本软件有密码保护功能。为了满足不同用户的喜好，本软件有四种皮肤供朋友们选择。

【破解声明】向大侠们学习！！！

------------------------------------------------------------------------

【破解过程】

------------------------------------------------------------------------

1、字符串查找“您输入的 `cdkey` 和当前的机器码不匹配，您不能用这个 `cdkey` 在这台机器上使用!”，可以找到下面关键代码：

```
00447DD4   .  55            push    ebp
00447DD5   .  8BEC          mov     ebp, esp
00447DD7   .  83C4 B0       add     esp, -50
00447DDA   .  53            push    ebx
00447DDB   .  56            push    esi
00447DDC   .  57            push    edi
00447DDD   .  8945 B4       mov     [ebp-4C], eax
00447DE0   .  B8 E8925700   mov     eax, 005792E8
00447DE5   .  E8 82D61100   call    0056546C
00447DEA   .  66:C745 C8 08>mov     word ptr [ebp-38], 8
00447DF0   .  33D2          xor     edx, edx
00447DF2   .  8955 F8       mov     [ebp-8], edx
00447DF5   .  8D55 F8       lea     edx, [ebp-8]
00447DF8   .  FF45 D4       inc     dword ptr [ebp-2C]
00447DFB   .  8B45 B4       mov     eax, [ebp-4C]
00447DFE   .  8B80 FC020000 mov     eax, [eax+2FC]
00447E04   .  E8 239E0E00   call    00531C2C
00447E09   .  8D45 F8       lea     eax, [ebp-8]
00447E0C   .  33D2          xor     edx, edx
00447E0E   .  8955 F4       mov     [ebp-C], edx
00447E11   .  8D55 F4       lea     edx, [ebp-C]
00447E14   .  FF45 D4       inc     dword ptr [ebp-2C]
00447E17   .  E8 40781200   call    0056F65C
00447E1C   .  8D45 F4       lea     eax, [ebp-C]
00447E1F   .  8B08          mov     ecx, [eax]                   ;  假码
00447E21   .  A1 E8095800   mov     eax, [5809E8]
00447E26   .  51            push    ecx                            ; /Arg2
00447E27   .  8B10          mov     edx, [eax]                     ; |85w
00447E29   .  52            push    edx                            ; |Arg1
00447E2A   .  E8 CDA7FBFF   call    004025FC                       ; \\关键 Call  《－－－－跟进
00447E2F   .  83C4 08       add     esp, 8
00447E32   .  BA 02000000   mov     edx, 2
00447E37   .  50            push    eax
00447E38   .  8D45 F4       lea     eax, [ebp-C]
00447E3B   .  FF4D D4       dec     dword ptr [ebp-2C]
00447E3E   .  E8 8D761200   call    0056F4D0
00447E43   .  FF4D D4       dec     dword ptr [ebp-2C]             ; |
00447E46   .  8D45 F8       lea     eax, [ebp-8]                   ; |
00447E49   .  BA 02000000   mov     edx, 2                         ; |
00447E4E   .  E8 7D761200   call    0056F4D0                       ; \\A7Money.0056F4D0
00447E53   .  59            pop     ecx
00447E54   .  84C9          test    cl, cl
00447E56     0F84 5E010000 je      00447FBA                         ；关键跳转。
00447E5C   .  B2 01         mov     dl, 1
00447E5E   .  A1 E4A54C00   mov     eax, [4CA5E4]
00447E63   .  E8 7C280800   call    004CA6E4
00447E68   .  8945 B0       mov     [ebp-50], eax                   ；下面是注册成功时，写注册表信息的操作
00447E6B   .  BA 020000080  mov     edx, 80000002
00447E70   .  8B45 B0       mov     eax, [ebp-50]
00447E73   .  E8 10751200   call    0056F388
00447E78   .  66:C745 C8 14>mov     word ptr [ebp-38], 14
00447E7E   .  BA 57915700   mov     edx, 00579157                   ;  \\software\\xcsoft\\a7money
```

2、跟进 `call 004025FC` 可以找到下面关键代码：

```
00402635  |.  53            push    ebx                              ; /Arg2
00402636  |.  33D2          xor     edx, edx                         ; |
00402638  |.  8955 FC       mov     [ebp-4], edx                     ; |
0040263B  |.  8D4D FC       lea     ecx, [ebp-4]                     ; |
0040263E  |.  51            push    ecx                              ; |Arg1
0040263F  |.  FF46 1C       inc     dword ptr [esi+1C]               ; |
00402642  |.  E8 D5FDFFFF   call    0040241C                         ; \\A7Money.0040241C
00402647  |.  66:C746 10 08>mov     word ptr [esi+10], 8
0040264D  |.  66:C746 10 20>mov     word ptr [esi+10], 20
00402653  |.  83C4 08       add     esp, 8
00402656  |.  8B45 FC       mov     eax, [ebp-4]                     ;  这里会出现机器码
00402659  |.  33D2          xor     edx, edx
0040265B  |.  8D4D F8       lea     ecx, [ebp-8]
0040265E  |.  50            push    eax
0040265F  |.  53            push    ebx
00402660  |.  8955 F8       mov     [ebp-8], edx
00402663  |.  51            push    ecx
00402664  |.  FF46 1C       inc     dword ptr [esi+1C]
00402667  |.  E8 04FCFFFF   call    00402270                         ;  上面机器码出现了,这个 Call 肯定要 F7 了。《－－－跟进
```

3、跟进 `call 00402270`，可以找到如下关键代码：

```
004022CB  |.  52            push    edx                              ; /Arg3
004022CC  |.  53            push    ebx                              ; |Arg2
004022CD  |.  894D F8       mov     [ebp-8], ecx                     ; |
004022D0  |.  50            push    eax                              ; |Arg1
004022D1  |.  FF45 E8       inc     dword ptr [ebp-18]               ; |
004022D4  |.  E8 5FFEFFFF   call    00402138                         ; \\A7Money.《－－－－这里时新机器码字符串生产的算法（1）
004022D9  |.  66:C745 DC 08>mov     word ptr [ebp-24], 8
004022DF  |.  8D83 10040000 lea     eax, [ebx+410]
004022E5  |.  83C4 0C       add     esp, 0C
004022E8  |.  8338 00       cmp     dword ptr [eax], 0
004022EB  |.  74 04         je      short 004022F1
004022ED  |.  8B30          mov     esi, [eax]                       ;  固定字符串“a7moneyisoneofthesoftwareofxccnoas”
004022EF  |.  EB 05         jmp     short 004022F6
004022F1  |>  BE FE125700   mov     esi, 005712FE
004022F6  |>  837D F8 00    cmp     dword ptr [ebp-8], 0             ;  新的机器码字符串。由上面的 call 00402138 生成
004022FA  |.  74 05         je      short 00402301
004022FC  |.  8B7D F8       mov     edi, [ebp-8]                     ;  计算后得到的新字符串。
004022FF  |.  EB 05         jmp     short 00402306
00402301  |>  BF FF125700   mov     edi, 005712FF
00402306  |>  33DB          xor     ebx, ebx
00402308  |>  8A0E          /mov     cl, [esi]                        ;  cl 存放的是固定字符串的第 n 个字符。n＝1，2，...16
0040230A  |.  46            |inc     esi
0040230B  |.  66:C745 DC 08>|mov     word ptr [ebp-24], 8
00402311  |.  8A07          |mov     al, [edi]                        ;  al 存放经过计算机器码后的新字符串，取 16 个
00402313  |.  47            |inc     edi
00402314  |.  8845 CB       |mov     [ebp-35], al
00402317  |.  0FBE55 CB     |movsx   edx, byte ptr [ebp-35]
0040231B  |.  0FBEC1        |movsx   eax, cl
0040231E  |.  03C2          |add     eax, edx                         ;  +
00402320  |.  B9 1A000000   |mov     ecx, 1A                          ;  定值 1A＝26
00402325  |.  99            |cdq
00402326  |.  F7F9          |idiv    ecx                              ;  两个字符的 ASCII 码相加的结果 除以 1A
00402328  |.  8BC2          |mov     eax, edx                         ;  得到的余数到 eax
0040232A  |.  8D55 C9       |lea     edx, [ebp-37]
0040232D  |.  04 41         |add     al, 41                           ;  余数+41
0040232F  |.  8845 C9       |mov     [ebp-37], al                     ;  这个是转成字符
00402332  |.  8D45 F4       |lea     eax, [ebp-C]                     ;  以上得到的字符每四个连成一组。然后用“－”连接得到最后的注册码
00402335  |.  C645 CA 00    |mov     byte ptr [ebp-36], 0
```
