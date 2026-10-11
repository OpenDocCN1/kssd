
# 网软眼保 2003 修正版(第六版)

> 原文：[`www.pediy.com/kssd/pediy05/pediy50656.htm`](https://www.pediy.com/kssd/pediy05/pediy50656.htm)

简单算法——网软眼保 2003 修正版(第六版)

下载页面： [`www.skycn.com/soft/10438.html`](http://www.skycn.com/soft/10438.html)

软件大小： 784 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 开关定时

加入时间： 2003-04-21 15:57:27

下载次数： 3917

推荐等级： ***

开 发 商： [`go.6to23.com/nie173/`](http://go.6to23.com/nie173/)

## 软件简介
如果你家有贪玩电脑的孩子，或你是个经常过度用电脑的人，那么该软件便是你最好的选择。有了它，你再也不必为你或你的小孩的用眼健康担心了。该软件具有十分大的强制性。当软件运行时没有密码是不可以退出程序的。软件强制保护用户眼睛时，任何没有密码的企图中止软件的行为都是徒劳的。软件使用的保护眼睛的办法是目前最有效的办法，在中国约有 1 亿多的中小学生都在使用该办法保护眼睛。一句话，选择了该软件你一定不会后悔的。

## 软件限制
试用 50 次

## 作者声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具
`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm` 9.0 白金版

---

## 过 程

`eye.exe` 无壳。`Visual C++` 6.0 编写。

机器码：`858278001eye261584`

试炼码：`123456789-ABCDEF`

---

程序启动时的运算部分！或者点“注册”时也可以拦截。

```
* Possible StringData Ref from Data Obj ->"c:\\"

                    |
:0040DE11 6874414200             push 00424174

* Reference To: KERNEL32.GetVolumeInformationA, Ord:0177h

                    |
:0040DE16 FF1560D14100           Call dword ptr [0041D160]
                    ==== > 取我的硬盘序列号

:0040DE1C 8B44242C               mov eax, dword ptr [esp+2C]
                    ==== > EAX=211C1E09

:0040DE20 53                     push ebx
:0040DE21 3578563412             xor eax, 12345678
                    ==== > EAX=211C1E09 XOR 12345678=33284871
                    ==== > 33284871（H）=858278001（D）  得出机器码的前部分

:0040DE26 8BCD                   mov ecx, ebp
:0040DE28 89442430               mov dword ptr [esp+30], eax
:0040DE2C 8BF0                   mov esi, eax

* Reference To: MFC42.Ordinal:18BE, Ord:18BEh

                    |
:0040DE2E E84DBC0000             Call 00419A80
:0040DE33 8D4C2414               lea ecx, dword ptr [esp+14]

* Reference To: MFC42.Ordinal:021C, Ord:021Ch

                    |
:0040DE37 E8EABB0000             Call 00419A26
:0040DE3C 8D542434               lea edx, dword ptr [esp+34]
:0040DE40 C644245C02             mov [esp+5C], 02
:0040DE45 52                     push edx
:0040DE46 C744243820000000       mov [esp+38], 00000020

* Reference To: KERNEL32.GlobalMemoryStatus, Ord:018Dh

                    |
:0040DE4E FF1564D14100           Call dword ptr [0041D164]
                    ==== > GlobalMemoryStatus 取我的内存大小？

:0040DE54 8B44243C               mov eax, dword ptr [esp+3C]
                    ==== > EAX=0FF74000

:0040DE58 8D4C2414               lea ecx, dword ptr [esp+14]
:0040DE5C C1E80A                 shr eax, 0A
                    ==== > EAX=0FF74000 SHR A=0003FDD0
                    ==== > 0003FDD0（H）=261584（D）      得出机器码的后部分

:0040DE5F 50                     push eax

* Possible StringData Ref from Data Obj ->"%lu"

                    |
:0040DE60 686C414200             push 0042416C
:0040DE65 51                     push ecx

* Reference To: MFC42.Ordinal:0B02, Ord:0B02h

                    |
:0040DE66 E8B5BB0000             Call 00419A20
:0040DE6B 83C40C                 add esp, 0000000C
:0040DE6E 8D4C2410               lea ecx, dword ptr [esp+10]

* Reference To: MFC42.Ordinal:021C, Ord:021Ch

                    |
:0040DE72 E8AFBB0000             Call 00419A26
:0040DE77 56                     push esi
:0040DE78 8D542414               lea edx, dword ptr [esp+14]
:0040DE7C B303                   mov bl, 03

* Possible StringData Ref from Data Obj ->"%ld"

                    |
:0040DE7E 6870414200             push 00424170
:0040DE83 52                     push edx
:0040DE84 885C2468               mov byte ptr [esp+68], bl

* Reference To: MFC42.Ordinal:0B02, Ord:0B02h

                    |
:0040DE88 E893BB0000             Call 00419A20
:0040DE8D 83C408                 add esp, 00000008
:0040DE90 8D442418               lea eax, dword ptr [esp+18]
:0040DE94 8BCC                   mov ecx, esp
:0040DE96 89642434               mov dword ptr [esp+34], esp
:0040DE9A 50                     push eax

* Reference To: MFC42.Ordinal:0217, Ord:0217h

                    |
:0040DE9B E874BB0000             Call 00419A14
:0040DEA0 8B742434               mov esi, dword ptr [esp+34]
:0040DEA4 8D4C2434               lea ecx, dword ptr [esp+34]
:0040DEA8 51                     push ecx
:0040DEA9 8BCE                   mov ecx, esi
:0040DEAB E80061FFFF             call 00403FB0
                    ==== > 算法 CALL！进入！以 261584 为参数运算注册码的后部分！

:0040DEB0 50                     push eax
:0040DEB1 8D4C2418               lea ecx, dword ptr [esp+18]
:0040DEB5 C644246004             mov [esp+60], 04

* Reference To: MFC42.Ordinal:035A, Ord:035Ah

                    |
:0040DEBA E873BB0000             Call 00419A32
:0040DEBF 8D4C2430               lea ecx, dword ptr [esp+30]
:0040DEC3 885C245C               mov byte ptr [esp+5C], bl

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                    |
:0040DEC7 E842BB0000             Call 00419A0E
:0040DECC 51                     push ecx
:0040DECD 8D542414               lea edx, dword ptr [esp+14]
:0040DED1 8BCC                   mov ecx, esp
:0040DED3 89642434               mov dword ptr [esp+34], esp
:0040DED7 52                     push edx

* Reference To: MFC42.Ordinal:0217, Ord:0217h

                    |
:0040DED8 E837BB0000             Call 00419A14
:0040DEDD 8D442434               lea eax, dword ptr [esp+34]
:0040DEE1 8BCE                   mov ecx, esi
:0040DEE3 50                     push eax
:0040DEE4 E8C760FFFF             call 00403FB0
                    ==== > 算法 CALL！以 858278001 为参数运算注册码的前部分！
```

---

进入算法 CALL：`00403910` `call` `00403FB0`

因为 2 次运算的流程都是一样的，只是参数不同，所以我只是记录了第一次运算的数据。

```
* Referenced by a CALL at Addresses:

|:00403910   , :0040394D   , :00405A26   , :0040DEAB   , :0040DEE4   |

:00403FB0 6AFF                   push FFFFFFFF
:00403FB2 6827A74100             push 0041A727
:00403FB7 64A100000000           mov eax, dword ptr fs:[00000000]
:00403FBD 50                     push eax
:00403FBE 64892500000000         mov dword ptr fs:[00000000], esp
:00403FC5 83EC3C                 sub esp, 0000003C
:00403FC8 55                     push ebp
:00403FC9 56                     push esi
:00403FCA 57                     push edi
:00403FCB C744241000000000       mov [esp+10], 00000000
:00403FD3 8D4C240C               lea ecx, dword ptr [esp+0C]
:00403FD7 C744245001000000       mov [esp+50], 00000001

* Reference To: MFC42.Ordinal:021C, Ord:021Ch

                    |
:00403FDF E8425A0100             Call 00419A26
:00403FE4 8B6C245C               mov ebp, dword ptr [esp+5C]
                    ==== > EBP=261584             机器码的后部分

:00403FE8 83C9FF                 or ecx, FFFFFFFF
:00403FEB 8BFD                   mov edi, ebp
:00403FED 33C0                   xor eax, eax
:00403FEF 33D2                   xor edx, edx
:00403FF1 C644245002             mov [esp+50], 02
:00403FF6 F2                     repnz
:00403FF7 AE                     scasb
:00403FF8 F7D1                   not ecx
:00403FFA 49                     dec ecx
:00403FFB 85C9                   test ecx, ecx
:00403FFD 7E7E                   jle 0040407D
```



:00403FFF `8BF5` `mov esi, ebp`

:00404001 `8D442414` `lea eax, dword ptr [esp+14]`

:00404005 `53` `push ebx`

:00404006 `2BF0` `sub esi, eax`

:00404008 `B36C` `mov bl, 6C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040407A(C)

|

:0040400A `8D0C16` `lea ecx, dword ptr [esi+edx]`

:0040400D `0FBE440C18` `movsx eax, byte ptr [esp+ecx+18]`

`====>这个循环就是根据机器码的数字值 261584 从下面的位置取值！`

`====>其实下面相当于一张表。`

:00404012 `83C0D5` `add eax, FFFFFFD5`

:00404015 `83F80E` `cmp eax, 0000000E`

:00404018 `7751` `ja 0040406B`

:0040401A `FF2485E4404000` `jmp dword ptr [4*eax+004040E4]`

:00404021 `C644141861` `mov [esp+edx+18], 61`

`3、 ====>1 取 61 即：字符 a`

:00404026 `EB43` `jmp 0040406B`

:00404028 `C644141863` `mov [esp+edx+18], 63`

`1、 ====>2 取 63 即：字符 c`

:0040402D `EB3C` `jmp 0040406B`

:0040402F `C644141868` `mov [esp+edx+18], 68`

:00404034 `EB35` `jmp 0040406B`

:00404036 `C64414186A` `mov [esp+edx+18], 6A`

`6、 ====>4 取 6A 即：字符 j`

:0040403B `EB2E` `jmp 0040406B`

:0040403D `C64414186D` `mov [esp+edx+18], 6D`

`4、 ====>5 取 6D 即：字符 m`

:00404042 `EB27` `jmp 0040406B`

:00404044 `C64414186B` `mov [esp+edx+18], 6B`

`2、 ====6 取 6B 即：字符 k`

:00404049 `EB20` `jmp 0040406B`

:0040404B `C64414187A` `mov [esp+edx+18], 7A`

:00404050 `EB19` `jmp 0040406B`

:00404052 `C644141878` `mov [esp+edx+18], 78`

`5、 ====>8 取 78 即：字符 x`

:00404057 `EB12` `jmp 0040406B`

:00404059 `C644141877` `mov [esp+edx+18], 77`

:0040405E `EB0B` `jmp 0040406B`

:00404060 `885C1418` `mov byte ptr [esp+edx+18], bl`

:00404064 `EB05` `jmp 0040406B`

:00404066 `C64414186E` `mov [esp+edx+18], 6E`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00404018(C), :00404026(U), :0040402D(U), :00404034(U), :0040403B(U)

|:00404042(U), :00404049(U), :00404050(U), :00404057(U), :0040405E(U)

|:00404064(U)

|

:0040406B `8BFD` `mov edi, ebp`

:0040406D `83C9FF` `or ecx, FFFFFFFF`

:00404070 `33C0` `xor eax, eax`

:00404072 `42` `inc edx`

:00404073 `F2` `repnz`

:00404074 `AE` `scasb`

:00404075 `F7D1` `not ecx`

:00404077 `49` `dec ecx`

:00404078 `3BD1` `cmp edx, ecx`

:0040407A `7C8E` `jl 0040400A`

`====>循环从表中取值`

:0040407C `5B` `pop ebx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403FFD(C)

|

:0040407D `C644141400` `mov [esp+edx+14], 00`

:00404082 `8D542414` `lea edx, dword ptr [esp+14]`

`第一次大循环结果 ====>EDX=ckamzj 注册码的后部分`

`第二次大循环结果 ====>EDX=zmzclzwwa 注册码的前部分`

:00404086 `52` `push edx`

:00404087 `8D442410` `lea eax, dword ptr [esp+10]`

\* Possible StringData Ref from Data Obj ->`"%s"`

|

:0040408B `6858424200` `push 00424258`

:00404090 `50` `push eax`

\* Reference To: `MFC42.Ordinal:0B02`, `Ord:0B02h`

|

:00404091 `E88A590100` `Call 00419A20`

:00404096 `8B742464` `mov esi, dword ptr [esp+64]`

:0040409A `83C40C` `add esp, 0000000C`

:0040409D `8D4C240C` `lea ecx, dword ptr [esp+0C]`

:004040A1 `51` `push ecx`

:004040A2 `8BCE` `mov ecx, esi`

\* Reference To: `MFC42.Ordinal:0217`, `Ord:0217h`

|

:004040A4 `E86B590100` `Call 00419A14`

:004040A9 `C744241001000000` `mov [esp+10], 00000001`

:004040B1 `8D4C240C` `lea ecx, dword ptr [esp+0C]`

:004040B5 `C644245001` `mov [esp+50], 01`

\* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

|

:004040BA `E84F590100` `Call 00419A0E`

:004040BF `8D4C245C` `lea ecx, dword ptr [esp+5C]`

:004040C3 `C644245000` `mov [esp+50], 00`

\* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

|

:004040C8 `E841590100` `Call 00419A0E`

:004040CD `8B4C2448` `mov ecx, dword ptr [esp+48]`

:004040D1 `8BC6` `mov eax, esi`

:004040D3 `5F` `pop edi`

:004040D4 `5E` `pop esi`

:004040D5 `5D` `pop ebp`

:004040D6 `64890D00000000` `mov dword ptr fs:[00000000], ecx`

:004040DD `83C448` `add esp, 00000048`

:004040E0 `C20800` `ret 0008`

—————————————————————————————————

`程序启动后注册时的比较部分！`

\* Reference To: `MFC42.Ordinal:021C`, `Ord:021Ch`

|

:004038B2 `E86F610100` `Call 00419A26`

:004038B7 `8D4C2410` `lea ecx, dword ptr [esp+10]`

:004038BB `C784249000000000000000` `mov dword ptr [esp+00000090], 00000000`

\* Reference To: `MFC42.Ordinal:021C`, `Ord:021Ch`

|

:004038C6 `E85B610100` `Call 00419A26`

:004038CB `8D442414` `lea eax, dword ptr [esp+14]`

:004038CF `8BCE` `mov ecx, esi`

:004038D1 `50` `push eax`

:004038D2 `68FF030000` `push 000003FF`

:004038D7 `C684249800000001` `mov byte ptr [esp+00000098], 01`

\* Reference To: `MFC42.Ordinal:0C19`, `Ord:0C19h`

|

:004038DF `E872610100` `Call 00419A56`

`====>取试炼码的前半部分`

:004038E4 `8D4C2410` `lea ecx, dword ptr [esp+10]`

:004038E8 `51` `push ecx`

:004038E9 `6855040000` `push 00000455`

:004038EE `8BCE` `mov ecx, esi`

\* Reference To: `MFC42.Ordinal:0C19`, `Ord:0C19h`

|

:004038F0 `E861610100` `Call 00419A56`

`====>取试炼码的后半部分`

:004038F5 `51` `push ecx`

`====>ECX=ABCDEF`

:004038F6 `8D7E64` `lea edi, dword ptr [esi+64]`

:004038F9 `8BCC` `mov ecx, esp`

:004038FB `8964241C` `mov dword ptr [esp+1C], esp`

:004038FF `57` `push edi`

\* Reference To: `MFC42.Ordinal:0217`, `Ord:0217h`

|

:00403900 `E80F610100` `Call 00419A14`

:00403905 `8B5C241C` `mov ebx, dword ptr [esp+1C]`

:00403909 `8D54241C` `lea edx, dword ptr [esp+1C]`

:0040390D `52` `push edx`

:0040390E `8BCB` `mov ecx, ebx`

:00403910 `E89B060000` `call 00403FB0`

:00403915 `50` `push eax`

:00403916 `8BCF` `mov ecx, edi`

:00403918 `C684249400000002` `mov byte ptr [esp+00000094], 02`

\* Reference To: `MFC42.Ordinal:035A`, `Ord:035Ah`

|

:00403920 `E80D610100` `Call 00419A32`

:00403925 `8D4C2418` `lea ecx, dword ptr [esp+18]`

:00403929 `C684249000000001` `mov byte ptr [esp+00000090], 01`

\* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`



### 反汇编代码片段

```
:00403931 E8D8600100                Call 00419A0E
:00403936 51                       push ecx
:00403937 8D6E68                   lea ebp, dword ptr [esi+68]
:0040393A 8BCC                     mov ecx, esp
:0040393C 8964241C                 mov dword ptr [esp+1C], esp
:00403940 55                       push ebp

* Reference To: MFC42.Ordinal:0217, Ord:0217h

:00403941 E8CE600100                Call 00419A14
:00403946 8D44241C                 lea eax, dword ptr [esp+1C]
:0040394A 8BCB                     mov ecx, ebx
:0040394C 50                       push eax
:0040394D E85E060000                call 00403FB0
:00403952 50                       push eax
:00403953 8BCD                     mov ecx, ebp
:00403955 C684249400000003         mov byte ptr [esp+00000094], 03

* Reference To: MFC42.Ordinal:035A, Ord:035Ah

:0040395D E8D0600100                Call 00419A32
:00403962 8D4C2418                 lea ecx, dword ptr [esp+18]
:00403966 C684249000000001         mov byte ptr [esp+00000090], 01

* Reference To: MFC42.Ordinal:0320, Ord:0320h

:0040396E E89B600100                Call 00419A0E
:00403973 8B4C2414                 mov ecx, dword ptr [esp+14]

====>ECX=123456789

:00403977 8B3F                     mov edi, dword ptr [edi]

====>EDI=zmzclzwwa                 注册码的前部分

:00403979 51                       push ecx
:0040397A 57                       push edi

* Reference To: MSVCRT._mbscmp, Ord:0159h

:0040397B 8B3DC0D64100             mov edi, dword ptr [0041D6C0]
:00403981 FFD7                     call edi

====>比较注册码的前部分！

:00403983 83C408                   add esp, 00000008
:00403986 85C0                     test eax, eax
:00403988 0F8516010000             jne 00403AA4

====>跳则 OVER！

:0040398E 8B542410                 mov edx, dword ptr [esp+10]

====>EDX=ABCDEF

:00403992 8B6D00                   mov ebp, dword ptr [ebp+00]

====>EBP=ckamzj                    注册码的后部分

:00403995 52                       push edx
:00403996 55                       push ebp
:00403997 FFD7                     call edi

====>比较注册码的后部分！

:00403999 83C408                   add esp, 00000008
:0040399C 85C0                     test eax, eax
:0040399E 0F8500010000             jne 00403AA4

====>跳则 OVER！

:004039A4 6A20                     push 00000020

* Possible StringData Ref from Data Obj -> "网软眼保 2003"

:004039A6 6820414200               push 00424120

* Possible StringData Ref from Data Obj -> "祝贺你!你已经注册成功，重新进入设置项即可看到?"
  -> "ЧＧ胛獯蔚淖⒉崧牒突髀胱鞲霰阜荩，员阆麓"
  -> "蔚拿夥焉妒褂谩Ｔ俅胃行ag霄氖gg?"

====>呵呵，胜利女神！

…… 省略……

* Possible StringData Ref from Data Obj -> ".DEFAULT\\Software\\sharesoft\\NetSoft\\EyeSafeGur"
  -> "ad v2.0"

====>保存注册信息！

:00403A4A 6830414200               push 00424130
:00403A4F 6803000080               push 80000003
:00403A54 C644347800               mov [esp+esi+78], 00

* Reference To: ADVAPI32.RegCreateKeyExA, Ord:015Fh

:00403A59 FF1510D04100             Call dword ptr [0041D010]
:00403A5F 8B542418                 mov edx, dword ptr [esp+18]

* Reference To: ADVAPI32.RegSetvalueExA, Ord:0186h

:00403A63 8B3D0CD04100             mov edi, dword ptr [0041D00C]
:00403A69 8D4C2420                 lea ecx, dword ptr [esp+20]
:00403A6D 55                       push ebp
:00403A6E 51                       push ecx
:00403A6F 6A01                     push 00000001
:00403A71 6A00                     push 00000000

* Possible StringData Ref from Data Obj -> "Enrol"

:00403A73 68A8414200               push 004241A8
:00403A78 52                       push edx
:00403A79 FFD7                     call edi
:00403A7B 8B4C2418                 mov ecx, dword ptr [esp+18]
:00403A7F 8D442454                 lea eax, dword ptr [esp+54]
:00403A83 56                       push esi
:00403A84 50                       push eax
:00403A85 6A01                     push 00000001
:00403A87 6A00                     push 00000000

* Possible StringData Ref from Data Obj -> "Enrol2"

:00403A89 68A0414200               push 004241A0
:00403A8E 51                       push ecx
:00403A8F FFD7                     call edi
:00403A91 8B542418                 mov edx, dword ptr [esp+18]
:00403A95 52                       push edx

* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh

:00403A96 FF1508D04100             Call dword ptr [0041D008]
:00403A9C 6A00                     push 00000000

* Reference To: MSVCRT.exit, Ord:0249h

:00403A9E FF15C4D64100             Call dword ptr [0041D6C4]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00403988(C), :0040399E(C)

:00403AA4 6A20                     push 00000020

* Possible StringData Ref from Data Obj -> "网软眼保 2003"

:00403AA6 6820414200               push 00424120

* Possible StringData Ref from Data Obj -> "抱歉，注册码不对，需要帮助请参阅帮助。"

====>BAD BOY！

:00403AAB 6878414200               push 00424178
:00403AB0 8BCE                     mov ecx, esi

* Reference To: MFC42.Ordinal:1080, Ord:1080h

:00403AB2 E8B75F0100               Call 00419A6E
```

---

### 算法总结

1. 取硬盘序列号 `211C1E09` XOR `12345678`=`33284871`（`H`）=`858278001`（`D`），得出机器码的前部分。
2. 取内存大小 `0FF74000` SHR `A`=`0003FDD0`（`H`）=`261584`（`D`），得出机器码的后部分。
3. 机器码中的 `eye` 不参与运算。
4. 根据机器码的值从表（`a`、`c`、`h`、`j`、`m`、`k`、`z`、`x`、`w`、`n`）中不同位置取值。

---

### KeyMake 之 `{72th}` 内存注册机

**中断地址**：`0040397A`  
**中断次数**：`1`  
**第一字节**：`57`  
**指令长度**：`1`  
**内存方式**：`EDI` 后插入 - 作为前后部分的分野。  
**修改内存**：`00403986 85C0 test eax, eax` 改为 `33C0`

**中断地址**：`00403996`  
**中断次数**：`1`  
**第一字节**：`55`  
**指令长度**：`1`  
**内存方式**：`EBP`

---

### 注册信息保存

```
REGEDIT4

[HKEY_USERS\.DEFAULT\Software\sharesoft\NetSoft\EyeSafeGurad v2.0]

"Enrol"="zmzclzwwa"
"Enrol2"="ckamzj"
```

---

### 整理

**机器码**：`858278001eye261584`  
**注册码**：`zmzclzwwa - ckamzj`

---

```
        ,      _/
       /| \_.-~/             \_     ,       青春都一饷
      ( /~   /               \~-.\ |
      \`\\   _/                \   ~\ )        忍把浮名
     _-~~~-.)  )\__/;;,.           \   //'
  /'\_,\   --~   \ ~~~-  ,;;\\__(  (.-~~~-.      换了破解轻狂
\`~ \_( ,\..--\\ (      ,;'' /    ~--   /.\\
  /~~//'    /' \`\\           ) /--..\_, )\   \`~
  "  \`~"  "       \`"       /~'\`\\    \`\\~~\\

                         "     "   "~'  ""
```

Cracked By 巢水工作坊——fly `[OCN][FCG]`  
`2003-04-26  16:45`  
`4dd466d0e1e0cc123afbe504da2462ca`
