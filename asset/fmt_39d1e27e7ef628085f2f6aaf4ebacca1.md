
```
0187:0114EA4E C785F06FFEFF02000000 MOV       DWORD [EBP+FFFE6FF0],02

0187:0114EA58 33C0                  XOR       EAX,EAX

0187:0114EA5A 5A                    POP       EDX

0187:0114EA5B 59                    POP       ECX

0187:0114EA5C 59                    POP       ECX

0187:0114EA5D 648910                MOV       [FS:EAX],EDX

0187:0114EA60 E976010000            JMP       0114EBDB

0187:0114EA65 8B8DF46FFEFF          MOV       ECX,[EBP+FFFE6FF4]

0187:0114EA6B 49                    DEC       ECX

0187:0114EA6C 8B7B04                MOV       EDI,[EBX+04]

0187:0114EA6F 3BCF                  CMP       ECX,EDI

0187:0114EA71 7E25                  JNG       0114EA98 <---一定跳,就是说我们至少有

0187:0114EA73 8D0431                LEA       EAX,[ECX+ESI]  114EA2A<->114EA98 这一段

0187:0114EA76 8BF0                  MOV       ESI,EAX         是可以拿来 Patch 的

0187:0114EA78 8BC1                  MOV       EAX,ECX

0187:0114EA7A 99                    CDQ

0187:0114EA7B F7FF                  IDIV      EDI

0187:0114EA7D 85C0                  TEST      EAX,EAX

0187:0114EA7F 7E17                  JNG       0114EA98

0187:0114EA81 89C7                  MOV       EDI,EAX

0187:0114EA83 2B7304                SUB       ESI,[EBX+04]

0187:0114EA86 56                    PUSH      ESI

0187:0114EA87 8B4308                MOV       EAX,[EBX+08]

0187:0114EA8A 50                    PUSH      EAX

0187:0114EA8B 8B430C                MOV       EAX,[EBX+0C]

0187:0114EA8E 50                    PUSH      EAX

0187:0114EA8F 56                    PUSH      ESI

0187:0114EA90 E8B7F7FFFF            CALL      0114E24C

0187:0114EA95 4F                    DEC       EDI

0187:0114EA96 75EB                  JNZ       0114EA83
```

我用了注册机编写器的内存补丁,补完后运行,哈哈!程序的 NAG 没有了,不过~~~~~~~倒是出现了一个提示框,显示 Access violation at address `0114C6F3`. Write of address `8D07E07F`.唉,写内存错误,于是下 `bpx 114C6F3`

```
0187:0114C6C8  PUSH    EBP
0187:0114C6C9  MOV     EBP,ESP
0187:0114C6CB  PUSH    EBX
0187:0114C6CC  PUSH    ESI
0187:0114C6CD  PUSH    EDI
0187:0114C6CE  MOV     EDI,01153554
0187:0114C6D3  MOV     EBX,[01153654]<---指向 16 位数据运算的结果
0187:0114C6D9  MOV     EAX,[01153658]
0187:0114C6DE  MOV     ECX,0C
0187:0114C6E3  CDQ
0187:0114C6E4  IDIV    ECX
0187:0114C6E6  MOV     ESI,EAX
0187:0114C6E8  TEST    ESI,ESI
0187:0114C6EA  JNG     0114C733
0187:0114C6EC  MOV     EAX,[EDI]
0187:0114C6EE  MOV     EAX,[EAX]    <---eax=40000
0187:0114C6F0  ADD     EAX,[EBX]    <---eax+[ebx]=8D07E07F
0187:0114C6F2  INC     EAX
0187:0114C6F3  MOV     DWORD [EAX],01<---错误在这里,[eax]等于 8D07E07F
0187:0114C6F9  MOV     EAX,[EDI]
0187:0114C6FB  MOV     EAX,[EAX]
0187:0114C6FD  ADD     EAX,[EBX+04]
0187:0114C700  MOV     EDX,[EBX+08]
0187:0114C703  CALL    0114C68C
```

也就是说 Part I 里的结果最后的 16 位数据被程序利用了,如果这里修改一下代码跳过的话出了这个 CALL 就到了 `DAP_Pro` 的领空,分析它的代码,是 `DAP_Pro` 在调用解压后的一个子程序,也许那 16 位数据是指向我们的注册信息或者某个窗体的数据,因为我们的数据和原定的不同,所以指针乱了,可能程序在启动时会显示一些信息,应该影响不大,因为关掉提示框后程序一切正常,只是 About 选项没了,这个情形和 fwnl 脱掉壳以后是一样的,还好这些数据不是拿来做功能限制的.由于我没有 ASProtect 的高版本,这一步只能跳过(谁能提供???),于是想办法跳过这个提示框.

出现提示框后下 `bpx lockmytask`,分析代码:

```
0187:0044E46D  MOV     EDX,[00407C60] <---指向提示框的标题
0187:0044E473  CALL    00402FA4       <---比较是否是发生错误的提示框
0187:0044E478  TEST    AL,AL
0187:0044E47A  JNZ     0044E4BF       <---跳就没事
0187:0044E47C  CMP     WORD [EBX+BA],BYTE +00
```

Patch 的时候把这里也改掉,问题是以后再有错误就没有提示啦!方法如下,用注册机编写器的内存补丁:

| 地址 | 长度 | 原码 | 修改 |
| :--- | :--- | :--- | :--- |
| 114EA2A | 6 | `8B43048A4407` | `C70700000200` |
| 114EA30 | 7 | `FF8B53048A0F88` | `C747042B000100` |
| 114EA37 | 7 | `4C17FF88078B85` | `C7470802000100` |
| 114EA3E | 7 | `FC6FFEFF668B00` | `C7470C10000700` |
| 114EA45 | 7 | `663B0530231501` | `C7471053616D2E` <---此行原码各机不同,请自行修改 |
| 114EA4C | 7 | `7417C785F06FFE` | `C74714636F6D20` 其它的照搬 |
| 114EA53 | 7 | `FF0200000033C0` | `C747180010005C` |
| 114EA5A | 7 | `5A5959648910E9` | `C7471C58849D8A` |
| 114EA61 | 7 | `760100008B8DF4` | `C74720EB7E26E8` |
| 114EA68 | 7 | `6FFEFF498B7B04` | `C74724D376579C` |
| 114EA6F | 7 | `3BCF7E258D0431` | `C74728163325FF` |
| 114EA76 | 7 | `8BF08BC199F7FF` | `C7472CFF000000` |
| 114EA7D | 7 | `85C07E1789C72B` | `C7477C00000000` |
| 114EA84 | 7 | `7304568B430850` | `C6057AE44400EB` <---Patch 提示框 |
| 114EA8B | 2 | `8B43` | `EB0B` |

现在,整个世界清静了,用此方法 `DAP_Pro` 的热键可用,其它功能未发现缺少,虽然此法对此程序破解并不算完美,但相信此方法对 ASProtect 1.2 以上加壳的程序破解时间限制都有效,至少我试过好几个都行.请大家帮忙测试.但此方法有个问题就是难以找到以上要修改的代码,我总结了个方法:先找到程序注册码保存的主键,输入任意大于 8 位的注册码,用 TRW 载入,下 `bpx regqueryvalueexa`,中断后下 `d *(esp+8)`,当看到内存里出现"Key"时,查找 `8A 0F 88 4C 17 FF` 或 `33 C0 5A 59 59 64 89 10 E9`,附近就是了.如果找不到的话就只能慢慢跟了----痛苦呀.如果你想用你自己的注册名可以用 ASProtect 生成,方法参阅 Part I&II

另外在追注册码时经常会碰到一些 SEH,这东西我不大懂,如下:

```
0187:0114FDE9  PUSH    BYTE +10
0187:0114FDEB  PUSH    DWORD 0115366C
0187:0114FDF0  MOV     EAX,[EBX+02]  <---EBX=0
0187:0114FDF3  PUSH    EAX
0187:0114FDF4  LEA     EAX,[EBX+06]
0187:0114FDF7  PUSH    EAX
0187:0114FDF8  CALL    0114BFF4
```

F8 或 F12 的话就完了,如果不幸来到此只能重来,记住这个 Call 不要进入,只能 F10 代过,真是所谓的步步为营呀,所以我对脱壳不感冒:-)

-=End=-

```
 _/ _/ _/
 _/       _/ _/ _/  _/ _/ _/  _/ _/
 _/ _/     _/       _/    _/    _/
       _/  _/  _/  _/      _/      _/
_/ _/ _/     _/ _/ _/  _/      _/      _/
```

Sam.com  
6:15 2002-4-27 `4dd466d0e1e0cc123afbe504da2462ca`
