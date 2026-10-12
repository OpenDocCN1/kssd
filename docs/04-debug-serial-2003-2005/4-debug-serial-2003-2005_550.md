
# 邮件精灵 V2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50610.htm`](https://www.pediy.com/kssd/pediy05/pediy50610.htm)

简单算法——邮件精灵 V2.0

- 下载地址：`http://gaoasp.diy.163.com/software/EZMails.zip`
- 软件大小：`262K`
- 运行环境：`Windows 9x/Nt/2000/XP`

### 【软件简介】
邮件精灵是一个简单易用且高效的邮件处理软件，集邮件群发、邮件清理、邮箱地址搜索于一体，通过多线程方式，可以快速地向邮件列表文件中的邮箱地址发送邮件，可以按邮件服务器搜索邮箱地址，也可以快速删除指定邮箱的垃圾邮件。

### 【软件限制】
功能限制

### 【作者声明】
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 【破解工具】
`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

---

### 【过程】
呵呵，刚打开《密码截取 V3.1》的压缩包，就听见“叭……”几声“枪响”，晕，瑞星立马就杀了这个有点黑客性质的程序！关了瑞星，去病毒隔离系统恢复出来居然不能运行了。只好又叫醒睡懒觉的小猫，重新去 DOWN。呜呼哀哉，瑞星有点风吹草动就杀掉程序，我现在几乎都不开瑞星了。分析完了这个《邮件精灵》再想看看它的同门兄弟《密码截取 V3.1》，晕，居然算法一模一样，呵呵，我也可以睡觉了，只是可惜我的小猫白费了力气呀。^O^^O^

`EZMails.exe` 无壳。`Visual C++ 6.0` 编写。

- 用户名：`fly`
- 试炼码：`13572468`

反汇编，根据出错提示很容易就找到核心了。

---

```
:0040891F E898280000           Call 0040B1BC  
:00408924 8B542414             mov edx, dword ptr [esp+14]  
                              ====>EDX=fly

:00408928 8B42F8               mov eax, dword ptr [edx-08]  
:0040892B 85C0                 test eax, eax  
:0040892D 0F8480030000         je 00408CB3  
:00408933 8B442410             mov eax, dword ptr [esp+10]  
                              ====>EAX=13572468

:00408937 8B48F8               mov ecx, dword ptr [eax-08]  
:0040893A 85C9                 test ecx, ecx  
:0040893C 0F8471030000         je 00408CB3  
:00408942 8D4C2414             lea ecx, dword ptr [esp+14]

* Reference To: MFC42.Ordinal:106A, Ord:106Ah  
                              |  
:00408946 E8112B0000           Call 0040B45C  
:0040894B 8D4C2434             lea ecx, dword ptr [esp+34]

* Reference To: MFC42.Ordinal:021D, Ord:021Dh  
                              |  
:0040894F E84A280000           Call 0040B19E  
:00408954 8B4C243C             mov ecx, dword ptr [esp+3C]

\====>下面是黑名单比较了。呵呵，看看是哪几位大侠榜上有名？^-^-^-^-^  

* Possible StringData Ref from Data Obj ->"guodong"  
                              |  
:00408958 68E8154100           push 004115E8  
:0040895D 51                   push ecx  
:0040895E 8D4C243C             lea ecx, dword ptr [esp+3C]  
:00408962 C644245802           mov [esp+58], 02

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h  
                              |  
:00408967 E8EA2A0000           Call 0040B456  
:0040896C 8B54243C             mov edx, dword ptr [esp+3C]

* Possible StringData Ref from Data Obj ->"ttian"  
                              |  
:00408970 68E0154100           push 004115E0  
:00408975 52                   push edx  
:00408976 8D4C243C             lea ecx, dword ptr [esp+3C]

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h  
                              |  
:0040897A E8D72A0000           Call 0040B456  
:0040897F 8B44243C             mov eax, dword ptr [esp+3C]

* Possible StringData Ref from Data Obj ->"fpx"  
                              |  
:00408983 68DC154100           push 004115DC  
:00408988 50                   push eax  
:00408989 8D4C243C             lea ecx, dword ptr [esp+3C]

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h  
                              |  
:0040898D E8C42A0000           Call 0040B456  
:00408992 8B4C243C             mov ecx, dword ptr [esp+3C]

* Possible StringData Ref from Data Obj ->"fpxfpx"  
                              |  
:00408996 68D4154100           push 004115D4  
:0040899B 51                   push ecx  
:0040899C 8D4C243C             lea ecx, dword ptr [esp+3C]

* Reference To: MFC42.Ordinal:16E5, Ord:16E5h  
                              |  
:004089A0 E8B12A0000           Call 0040B456  
:004089A5 8B44243C             mov eax, dword ptr [esp+3C]  
:004089A9 33F6                 xor esi, esi  
:004089AB 85C0                 test eax, eax  
:004089AD 7E47                 jle 004089F6  
:004089AF B303                 mov bl, 03

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004089F4(C)  
|  
:004089B1 8D54241C             lea edx, dword ptr [esp+1C]  
:004089B5 56                   push esi  
:004089B6 52                   push edx  
:004089B7 8D4C243C             lea ecx, dword ptr [esp+3C]  
:004089BB E820DAFFFF           call 004063E0  
:004089C0 8D4C241C             lea ecx, dword ptr [esp+1C]  
:004089C4 885C2450             mov byte ptr [esp+50], bl

* Reference To: MFC42.Ordinal:106A, Ord:106Ah  
                              |  
:004089C8 E88F2A0000           Call 0040B45C  
:004089CD 8B442414             mov eax, dword ptr [esp+14]  
:004089D1 8D4C241C             lea ecx, dword ptr [esp+1C]  
:004089D5 50                   push eax

* Reference To: MFC42.Ordinal:0ACC, Ord:0ACCh  
                              |  
:004089D6 E8DB270000           Call 0040B1B6  
:004089DB 85C0                 test eax, eax  
:004089DD 7D74                 jge 00408A53  
                              ==== >跳则 OVER！如果是黑名单中的名字就立即 OVER 了！

:004089DF 8D4C241C             lea ecx, dword ptr [esp+1C]  
:004089E3 C644245002           mov [esp+50], 02

* Reference To: MFC42.Ordinal:0320, Ord:0320h  
                              |  
:004089E8 E845270000           Call 0040B132  
:004089ED 8B44243C             mov eax, dword ptr [esp+3C]  
:004089F1 46                   inc esi  
:004089F2 3BF0                 cmp esi, eax  
:004089F4 7CBB                 jl 004089B1  
                              ==== >循环 4 次！检测用户名是否是黑名单中的某位！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004089AD(C)  
|  
:004089F6 8D4C2424             lea ecx, dword ptr [esp+24]  
:004089FA 6A01                 push 00000001  
:004089FC 51                   push ecx  
:004089FD 8D4C2418             lea ecx, dword ptr [esp+18]

* Reference To: MFC42.Ordinal:1021, Ord:1021h  
                              |  
:00408A01 E8B8280000           Call 0040B2BE  
:00408A06 8B00                 mov eax, dword ptr [eax]

* Reference To: MSVCRT._mbscmp, Ord:0159h  
                              |  
:00408A08 8B35E0D34000         mov esi, dword ptr [0040D3E0]

* Possible StringData Ref from Data Obj ->"00"  
                              |  
:00408A0E 68D0154100           push 004115D0  
:00408A13 50                   push eax  
:00408A14 C644245804           mov [esp+58], 04  
:00408A19 FFD6                 call esi  
                              ==== >检测试炼码第一位字符是否是 0？

:00408A1B 83C408               add esp, 00000008  
:00408A1E 85C0                 test eax, eax  
:00408A20 7454                 je 00408A76  
                              ==== >跳则 OVER！第一位是 0 则 OVER 了！

:00408A22 8D542428             lea edx, dword ptr [esp+28]  
:00408A26 6A01                 push 00000001  
:00408A28 52                   push edx  
:00408A29 8D4C2418             lea ecx, dword ptr [esp+18]

* Reference To: MFC42.Ordinal:164E, Ord:164Eh  
                              |  
:00408A2D E8CE280000           Call 0040B300  
:00408A32 8B00                 mov eax, dword ptr [eax]

* Possible StringData Ref from Data Obj ->"00"  
                              |  
:00408A34 68D0154100           push 004115D0  
:00408A39 50                   push eax  
:00408A3A FFD6                 call esi  
                              ==== >检测试炼码最后一位字符是否是 0？

:00408A3C 83C408               add esp, 00000008  
:00408A3F 8D4C2428             lea ecx, dword ptr [esp+28]  
:00408A43 85C0                 test eax, eax  
:00408A45 0F94C3               sete bl

* Reference To: MFC42.Ordinal:0320, Ord:0320h  
                              |  
```



```
:00408A48 E8E5260000                 Call 0040B132

:00408A4D 84DB                       test bl, bl

:00408A4F 7525                       jne 00408A76

                                   ==>跳则 OVER！最后一位是 0 则 OVER 了！

:00408A51 EB25                       jmp 00408A78

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004089DD(C)

|

:00408A53 6A00                       push 00000000

:00408A55 6A00                       push 00000000

* Possible StringData Ref from Data Obj ->"注册失败！"

                                   ==>BAD BOY！黑名单的都到这儿了。^\*^ 

:00408A57 68C4154100                 push 004115C4

:00408A5C 8BCD                       mov ecx, ebp

* Reference To: MFC42.Ordinal:1080, Ord:1080h

                                   |

:00408A5E E84D270000                 Call 0040B1B0

:00408A63 8D4C241C                   lea ecx, dword ptr [esp+1C]

:00408A67 C644245002                 mov [esp+50], 02

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                                   |

:00408A6C E8C1260000                 Call 0040B132

:00408A71 E92D020000                 jmp 00408CA3

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00408A20(C), :00408A4F(C)

|

:00408A76 B301                       mov bl, 01

                                   ==>爆破点 ①

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00408A51(U)

|

:00408A78 8D4C2424                   lea ecx, dword ptr [esp+24]

:00408A7C C644245002                 mov [esp+50], 02

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                                   |

:00408A81 E8AC260000                 Call 0040B132

:00408A86 84DB                       test bl, bl

:00408A88 7409                       je 00408A93

:00408A8A 6A00                       push 00000000

:00408A8C 6A00                       push 00000000

:00408A8E E904020000                 jmp 00408C97

                                   ==>跳则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00408A88(C)

|

:00408A93 8B542414                   mov edx, dword ptr [esp+14]

:00408A97 33DB                       xor ebx, ebx

:00408A99 33C0                       xor eax, eax

:00408A9B 8B4AF8                     mov ecx, dword ptr [edx-08]

:00408A9E 85C9                       test ecx, ecx

:00408AA0 7E0B                       jle 00408AAD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00408AAB(C)

|

:00408AA2 0FBE3410                   movsx esi, byte ptr [eax+edx]

                                   ==>依次取 fly 字符的 HEX 值

                                   1、   ====>EAX\=66

                                   2、   ====>EAX\=6C

                                   3、   ====>EAX\=79

:00408AA6 03DE                       add ebx, esi

                                   1、   ====>EAX\=66 + 00=66

                                   2、   ====>EAX\=6C + 66=D2

                                   3、   ====>EAX\=79 + D2=14B

:00408AA8 40                         inc eax

:00408AA9 3BC1                       cmp eax, ecx

:00408AAB 7CF5                       jl 00408AA2

                                   ==>循环相加用户名字符的 HEX 值

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00408AA0(C)

|

:00408AAD 8B442410                   mov eax, dword ptr [esp+10]

                                   ==>EAX\=13572468

:00408AB1 8D4C2428                   lea ecx, dword ptr [esp+28]

:00408AB5 8B40F8                     mov eax, dword ptr [eax-08]

                                   ==>取 13572468 位数

:00408AB8 83C0FE                     add eax, FFFFFFFE

                                   ==>EAX\=8 + -2=6

:00408ABB 50                         push eax

:00408ABC 6A00                       push 00000000

:00408ABE 51                         push ecx

:00408ABF 8D4C241C                   lea ecx, dword ptr [esp+1C]

* Reference To: MFC42.Ordinal:10B6, Ord:10B6h

                                   |

:00408AC3 E844280000                 Call 0040B30C

                                   ==>取试炼码的前 6 位

:00408AC8 8B00                       mov eax, dword ptr [eax]

                                   ==>EAX\=135724

* Reference To: MSVCRT.atol, Ord:023Eh

                                   |

:00408ACA 8B3DE4D34000               mov edi, dword ptr [0040D3E4]

:00408AD0 50                         push eax

:00408AD1 FFD7                       call edi

                                   ==>求 135724 的 16 进制值

:00408AD3 83C404                     add esp, 00000004

:00408AD6 8D4C2428                   lea ecx, dword ptr [esp+28]

:00408ADA 8BF0                       mov esi, eax

                                   ==>EAX\=0002122C（H）=135724（D）

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                                   |

:00408ADC E851260000                 Call 0040B132

:00408AE1 8D542428                   lea edx, dword ptr [esp+28]

:00408AE5 6A02                       push 00000002

:00408AE7 52                         push edx

:00408AE8 8D4C2418                   lea ecx, dword ptr [esp+18]

* Reference To: MFC42.Ordinal:164E, Ord:164Eh

                                   |

:00408AEC E80F280000                 Call 0040B300

:00408AF1 8B00                       mov eax, dword ptr [eax]

:00408AF3 50                         push eax

* Reference To: MSVCRT.atoi, Ord:023Dh

                                   |

:00408AF4 FF15ECD34000               Call dword ptr [0040D3EC]

                                   ==>取试炼码的后 2 位 68，并转化成 16 进制值

:00408AFA 83C404                     add esp, 00000004

:00408AFD 8D4C2428                   lea ecx, dword ptr [esp+28]

:00408B01 89442424                   mov dword ptr [esp+24], eax

                                   ==>[esp+24]=EAX\=44（H）=68（D）

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                                   |

:00408B05 E828260000                 Call 0040B132

:00408B0A 33742424                   xor esi, dword ptr [esp+24]

                                   ==>ESI\=0002122C XOR 44=00021268

:00408B0E 3BDE                       cmp ebx, esi

                                   ==>比较了！

                                   ==>EBX\=14B        用户名字符 HEX 值累加的结果

                                   ==>ESI\=00021268   试炼码末 2 位和前几位异或的结果

:00408B10 0F8577010000               jne 00408C8D

                                   ==>跳则 OVER！  爆破点 ②

:00408B16 8D4C2418                   lea ecx, dword ptr [esp+18]

* Reference To: MFC42.Ordinal:021C, Ord:021Ch

                                   |

:00408B1A E82B260000                 Call 0040B14A

* Possible Reference to String Resource ID=00104: "Option.ini"

                                   ==>注册信息保存

                                   |

:00408B1F 6A68                       push 00000068

:00408B21 8D4C241C                   lea ecx, dword ptr [esp+1C]

:00408B25 C644245405                 mov [esp+54], 05

* Reference To: MFC42.Ordinal:1040, Ord:1040h

                                   |

:00408B2A E8AD270000                 Call 0040B2DC

:00408B2F 8D442428                   lea eax, dword ptr [esp+28]

:00408B33 50                         push eax

:00408B34 E8779CFFFF                 call 004027B0

:00408B39 83C404                     add esp, 00000004

:00408B3C 8D4C2428                   lea ecx, dword ptr [esp+28]

* Possible StringData Ref from Data Obj ->"\\\\"

                                   |

:00408B40 6830124100                 push 00411230

:00408B45 8D542434                   lea edx, dword ptr [esp+34]

:00408B49 B306                       mov bl, 06

:00408B4B 51                         push ecx

:00408B4C 52                         push edx

:00408B4D 885C245C                   mov byte ptr [esp+5C], bl

* Reference To: MFC42.Ordinal:039C, Ord:039Ch

                                   |

:00408B51 E880270000                 Call 0040B2D6
```



:00408B56 `8D4C2418` lea `ecx`, dword ptr `[esp+18]`

:00408B5A `8D54242C` lea `edx`, dword ptr `[esp+2C]`

:00408B5E `51` push `ecx`

:00408B5F `50` push `eax`

:00408B60 `52` push `edx`

:00408B61 `C644245C07` mov `[esp+5C]`, `07`

* Reference To: `MFC42.Ordinal:039A`, `Ord:039Ah`

:00408B66 `E865270000` Call `0040B2D0`

:00408B6B `50` push `eax`

:00408B6C `8D4C241C` lea `ecx`, dword ptr `[esp+1C]`

:00408B70 `C644245408` mov `[esp+54]`, `08`

* Reference To: `MFC42.Ordinal:035A`, `Ord:035Ah`

:00408B75 `E83E270000` Call `0040B2B8`

:00408B7A `8D4C242C` lea `ecx`, dword ptr `[esp+2C]`

:00408B7E `C644245007` mov `[esp+50]`, `07`

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

:00408B83 `E8AA250000` Call `0040B132`

:00408B88 `8D4C2430` lea `ecx`, dword ptr `[esp+30]`

:00408B8C `885C2450` mov byte ptr `[esp+50]`, `bl`

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

:00408B90 `E89D250000` Call `0040B132`

:00408B95 `8D442410` lea `eax`, dword ptr `[esp+10]`

:00408B99 `8D4C2420` lea `ecx`, dword ptr `[esp+20]`

:00408B9D `50` push `eax`

* Reference To: `MFC42.Ordinal:0217`, `Ord:0217h`

:00408B9E `E801260000` Call `0040B1A4`

:00408BA3 `6A00` push `00000000`

:00408BA5 `C644245409` mov `[esp+54]`, `09`

* Reference To: `MSVCRT.time`, `Ord:02D0h`

:00408BAA `FF15C8D34000` Call dword ptr `[0040D3C8]`

:00408BB0 `50` push `eax`

* Reference To: `MSVCRT.srand`, `Ord:02B4h`

:00408BB1 `FF15CCD34000` Call dword ptr `[0040D3CC]`

:00408BB7 `83C408` add `esp`, `00000008`

* Reference To: `MSVCRT.rand`, `Ord:02A6h`

:00408BBA `FF15D0D34000` Call dword ptr `[0040D3D0]`

:00408BC0 `8D4C2424` lea `ecx`, dword ptr `[esp+24]`

:00408BC4 `8BF0` mov `esi`, `eax`

* Reference To: `MFC42.Ordinal:021C`, `Ord:021Ch`

:00408BC6 `E87F250000` Call `0040B14A`

:00408BCB `56` push `esi`

:00408BCC `8D4C2428` lea `ecx`, dword ptr `[esp+28]`

* Possible StringData Ref from Data Obj -> `"%d"`

:00408BD0 `68F8114100` push `004111F8`

:00408BD5 `51` push `ecx`

:00408BD6 `C644245C0A` mov `[esp+5C]`, `0A`

* Reference To: `MFC42.Ordinal:0B02`, `Ord:0B02h`

:00408BDB `E82A260000` Call `0040B20A`

:00408BE0 `8B54242C` mov `edx`, dword ptr `[esp+2C]`

:00408BE4 `52` push `edx`

:00408BE5 `FFD7` call `edi`

:00408BE7 `8B4C2434` mov `ecx`, dword ptr `[esp+34]`

:00408BEB `33C6` xor `eax`, `esi`

:00408BED `50` push `eax`

:00408BEE `56` push `esi`

:00408BEF `8B49F8` mov `ecx`, dword ptr `[ecx-08]`

:00408BF2 `8D542438` lea `edx`, dword ptr `[esp+38]`

:00408BF6 `51` push `ecx`

* Possible StringData Ref from Data Obj -> `"%d%d%d"`

:00408BF7 `68BC154100` push `004115BC`

:00408BFC `52` push `edx`

* Reference To: `MFC42.Ordinal:0B02`, `Ord:0B02h`

:00408BFD `E808260000` Call `0040B20A`

:00408C02 `8B44243C` mov `eax`, dword ptr `[esp+3C]`

:00408C06 `8B4C2438` mov `ecx`, dword ptr `[esp+38]`

:00408C0A `83C424` add `esp`, `00000024`

:00408C0D `50` push `eax`

:00408C0E `51` push `ecx`

* Possible StringData Ref from Data Obj -> `"USERNAME"`

:00408C0F `68B0124100` push `004112B0`

* Possible StringData Ref from Data Obj -> `"REGINFO"`

:00408C14 `68A8124100` push `004112A8`

* Reference To: `KERNEL32.WritePrivateProfileStringA`, `Ord:02E5h`

:00408C19 `8B3508D04000` mov `esi`, dword ptr `[0040D008]`

:00408C1F `FFD6` call `esi`

:00408C21 `8B542418` mov `edx`, dword ptr `[esp+18]`

:00408C25 `8B442420` mov `eax`, dword ptr `[esp+20]`

:00408C29 `52` push `edx`

:00408C2A `50` push `eax`

* Possible StringData Ref from Data Obj -> `"PASSWORD"`

:00408C2B `689C124100` push `0041129C`

* Possible StringData Ref from Data Obj -> `"REGINFO"`

:00408C30 `68A8124100` push `004112A8`

:00408C35 `FFD6` call `esi`

:00408C37 `6830100000` push `00001030`

* Possible StringData Ref from Data Obj -> `"注册信息"`

:00408C3C `68B0154100` push `004115B0`

* Possible StringData Ref from Data Obj -> `"您成功注册！"`
====>呵呵，胜利女神！

:00408C41 `68A0154100` push `004115A0`

:00408C46 `8BCD` mov `ecx`, `ebp`

* Reference To: `MFC42.Ordinal:1080`, `Ord:1080h`

:00408C48 `E863250000` Call `0040B1B0`

:00408C4D `8BCD` mov `ecx`, `ebp`

* Reference To: `MFC42.Ordinal:12F5`, `Ord:12F5h`

:00408C4F `E8FC270000` Call `0040B450`

:00408C54 `8D4C2424` lea `ecx`, dword ptr `[esp+24]`

:00408C58 `C644245009` mov `[esp+50]`, `09`

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

:00408C5D `E8D0240000` Call `0040B132`

:00408C62 `8D4C2420` lea `ecx`, dword ptr `[esp+20]`

:00408C66 `885C2450` mov byte ptr `[esp+50]`, `bl`

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

:00408C6A `E8C3240000` Call `0040B132`

:00408C6F `8D4C2428` lea `ecx`, dword ptr `[esp+28]`

:00408C73 `C644245005` mov `[esp+50]`, `05`

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

:00408C78 `E8B5240000` Call `0040B132`

:00408C7D `8D4C2418` lea `ecx`, dword ptr `[esp+18]`

:00408C81 `C644245002` mov `[esp+50]`, `02`

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

:00408C86 `E8A7240000` Call `0040B132`

:00408C8B `EB16` jmp `00408CA3`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408B10(C)

:00408C8D `6830100000` push `00001030`

* Possible StringData Ref from Data Obj -> `"注册信息"`

:00408C92 `68B0154100` push `004115B0`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408A8E(U)

* Possible StringData Ref from Data Obj -> `"注册失败！"`
====>BAD BOY！

:00408C97 `68C4154100` push `004115C4`

:00408C9C `8BCD` mov `ecx`, `ebp`

* Reference To: `MFC42.Ordinal:1080`, `Ord:1080h`

:00408C9E `E80D250000` Call `0040B1B0`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00408A71(U), :00408C8B(U)

:00408CA3 `8D4C2434` lea `ecx`, dword ptr `[esp+34]`

:00408CA7 `C644245001` mov `[esp+50]`, `01`



* Reference To: `MFC42.Ordinal:0321`, `Ord:0321h`

```
:00408CAC E8B7240000              Call 0040B168
:00408CB1 EB10                    jmp 00408CC3
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:0040892D(C)`, `:0040893C(C)`

```
:00408CB3 6A00                    push 00000000
:00408CB5 6A00                    push 00000000
```

* Possible StringData Ref from Data Obj ->"注册失败！"  
====>BAD BOY！

```
:00408CB7 68C4154100             push 004115C4
:00408CBC 8BCD                   mov ecx, ebp
```

* Reference To: `MFC42.Ordinal:1080`, `Ord:1080h`

```
:00408CBE E8ED240000             Call 0040B1B0
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00408CB1(U)`

```
:00408CC3 8D4C2410               lea ecx, dword ptr [esp+10]
:00408CC7 C644245000             mov [esp+50], 00
```

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

```
:00408CCC E861240000             Call 0040B132
:00408CD1 8D4C2414               lea ecx, dword ptr [esp+14]
:00408CD5 C7442450FFFFFFFF       mov [esp+50], FFFFFFFF
```

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

```
:00408CDD E850240000             Call 0040B132
:00408CE2 8B4C2448               mov ecx, dword ptr [esp+48]
:00408CE6 5F                     pop edi
:00408CE7 5E                     pop esi
:00408CE8 5D                     pop ebp
:00408CE9 5B                     pop ebx
:00408CEA 64890D00000000         mov dword ptr fs:[00000000], ecx
:00408CF1 83C444                 add esp, 00000044
:00408CF4 C3                     ret
```

—————————————————————————————————

呵呵，发现程序在启动时还有校验。爆破顺手也就看看。不知是否有网络校验了。

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00403B7B(C)`

```
:00403B72 0FBE3410               movsx esi, byte ptr [eax+edx]
:00403B76 03EE                   add ebp, esi
:00403B78 40                     inc eax
:00403B79 3BC1                   cmp eax, ecx
:00403B7B 7CF5                   jl 00403B72
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00403B70(C)`

```
:00403B7D 8B4C2410               mov ecx, dword ptr [esp+10]
:00403B81 8D542414               lea edx, dword ptr [esp+14]
:00403B85 8B41F8                 mov eax, dword ptr [ecx-08]
:00403B88 8D4C2410               lea ecx, dword ptr [esp+10]
:00403B8C 83C0FE                 add eax, FFFFFFFE
:00403B8F 50                     push eax
:00403B90 6A00                   push 00000000
:00403B92 52                     push edx
```

* Reference To: `MFC42.Ordinal:10B6`, `Ord:10B6h`

```
:00403B93 E874770000             Call 0040B30C
:00403B98 8B00                   mov eax, dword ptr [eax]
:00403B9A 50                     push eax
:00403B9B FFD7                   call edi
:00403B9D 83C404                 add esp, 00000004
:00403BA0 8D4C2414               lea ecx, dword ptr [esp+14]
:00403BA4 8BF0                   mov esi, eax
```

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

```
:00403BA6 E887750000             Call 0040B132
:00403BAB 8D442414               lea eax, dword ptr [esp+14]
:00403BAF 6A02                   push 00000002
:00403BB1 50                     push eax
:00403BB2 8D4C2418               lea ecx, dword ptr [esp+18]
```

* Reference To: `MFC42.Ordinal:164E`, `Ord:164Eh`

```
:00403BB6 E845770000             Call 0040B300
:00403BBB 8B00                   mov eax, dword ptr [eax]
:00403BBD 50                     push eax
:00403BBE FFD3                   call ebx
:00403BC0 83C404                 add esp, 00000004
:00403BC3 8D4C2414               lea ecx, dword ptr [esp+14]
:00403BC7 8BF8                   mov edi, eax
```

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

```
:00403BC9 E864750000             Call 0040B132
:00403BCE 33F7                   xor esi, edi
:00403BD0 C684242004000004       mov byte ptr [esp+00000420], 04
:00403BD8 3BEE                   cmp ebp, esi
====>呵呵，再比较一次！ 爆破点 ③
:00403BDA 0F94C1                 sete cl
:00403BDD 884C2428               mov byte ptr [esp+28], cl
:00403BE1 8B742428               mov esi, dword ptr [esp+28]
:00403BE5 8D4C2410               lea ecx, dword ptr [esp+10]
:00403BE9 81E6FF000000          and esi, 000000FF
```

—————————————————————————————————

### 算法总结

1、用户名不能位居黑名单之列。
2、注册码第一位和最后一位字符不能是 `0`
3、注册码最后 `2` 位数字的 `HEX` 值和前几位数字的 `HEX` 值异或的结果应等于用户名字符 `HEX` 值累加的之和。

简单求逆：

`fly = 66 + 6C + 79 = 14B`

`14B XOR 44 = 10F（H）= 271（D）`

呵呵，所以我的注册码就是 `27168` 当然，还有很多很多……

—————————————————————————————————

### 完美爆破

发现爆破也挺有意思，有些软件或许可以找到注册码却很难爆破。

呵呵，黑名单的地方就不处理了，也没必要。第 `3` 处是后来发现程序在启动时还有校验才“揪”出来的。

另外：不知道这个东东是否会私下去连网校验，我是小猫上网就不去试了。即使有也不会藏的太隐蔽的。

1、
```
00408A76 B301  mov bl, 01
```
改为：
```
B300  mov bl, 00
```

2、
```
00408B10 0F8577010000  jne 00408C8D
```
改为：
```
909090909090  NOP 掉
```

3、
```
00403BD8 3BEE  cmp ebp, esi
```
改为：
```
3BED  cmp ebp, ebp
```

—————————————————————————————————

### 注册信息保存

同目录下的 `Option.ini` 中

```ini
[REGINFO]
USERNAME=fly
PASSWORD=4505231132
```
呵呵，变了点形

—————————————————————————————————

### 整理

用户名：`fly`
注册码：`27168`

—————————————————————————————————

Cracked By 巢水工作坊——`fly` `[OCN]` `[FCG]`
2003-4-18 00:00 4dd466d0e1e0cc123afbe504da2462ca
