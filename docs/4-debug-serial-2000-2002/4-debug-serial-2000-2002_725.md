
# 虚拟光驱 2000 V5.1 网络版

> 原文：[`www.pediy.com/kssd/pediy03/forum978.htm`](https://www.pediy.com/kssd/pediy03/forum978.htm)

**软件名称**：`Virtual Drive 2000 网络版`

**整理日期**：`2000.8.12`

**最新版本**：`5.1c`

**文件大小**：`4824KB`

**软件授权**：共享软件

**使用平台**：`Win95/98`

**发布公司**：[`http://www.farstone.com/`](http://www.farstone.com/)

**下载地址**：[`http://www.newhua.com.cn/down/virtdnet.zip`](http://www.newhua.com.cn/down/virtdnet.zip)

**软件简介**：一个非常不错 `WIN95/98` 下的 `CD-ROM` 模拟程序，可以把 `CD-ROM` 复制为硬盘的一个文件，这个版本连 `CD` 音轨、保护都可以模拟，最多模拟出二十部光驱。此版为多语言版，包括：中文版、英文版、和日文版，安装时会自动检测操作系统的语言版本并自动安装。

**破解工具**：`W32Dasm`，`exescope 6.0`

**说明**：这是我将近一年前破解的软件，当时由于工作需要，想找一个网络版的虚拟光驱，找是找到了，但为评估版，上网搜索也没有找到关于网络版的完整破解，于是只好自力更生。在破解过程中，我还发现了在番外地里介绍的修改 `setup.sys` 后试用版变正式版的过程，而且还发现，如果使用该网络版的是中国的用户，则需要软件狗，我全给他解了。这个软件的全部破解过程很长，有 50 多 K，这里仅介绍注册码的计算过程。另外，网络版与单机版的序列号计算过程基本一样，其中，网络版的序列号为 15 位，而单机版为 11 位，就差在最后 4 位数字，最后的 4 位数字表示的是允许使用的用户数。

这个软件可以自动判别操作系统的类型，根据操作系统的类型自动选择用繁体、简体、日语、法语、德语及英语来显示安装界面，虽然这给用户的使用上简便了，但用 `pwdasm` 反汇编时，却出现了很多一些看不懂的字符串——因为缺省是显示繁体中文，虽然不影响程序流程，但这给破解造成了麻烦，为了更好地破解这个软件，我对该软件做一下处理，让 `w32dasm` 能够识别字符串，方法是用 `exescope` 对程序进行处理，将繁体中文和简体中文均改为英文字符显示，嘿嘿，本来用作汉化的常用工具，反其道而行之，可以称为反汉化，反汉化的相应字符串提示不用担心，软件公司已经给做好了，用 `exescope` 改好后，用 `w32dasm` 进行反汇编，竟取得了很好的效果，这回，`w32dasm` 将字符全部显示出来了，当然是英文的，现在我们开始破解吧。

目标程序：`SerSetup.exe`，大小 `331776 Bytes`

运行 `SerSetup.exe`，安装程序要求输入公司名称和序列号等，下面为序列号的计算部分的流程

* Referenced by a CALL at Address:

`:00401F5C`

```
:00402890 64A100000000                mov eax, dword ptr fs:[00000000]
:00402896 6AFF                        push FFFFFFFF
:00402898 68F8834100                 push 004183F8
:0040289D 50                         push eax
:0040289E 64892500000000             mov dword ptr fs:[00000000], esp
:004028A5 81EC00010000               sub esp, 00000100
:004028AB 53                         push ebx
:004028AC 55                         push ebp
:004028AD 56                         push esi
:004028AE 57                         push edi
:004028AF 8BD9                       mov ebx, ecx
:004028B1 8BBC2420010000             mov edi, dword ptr [esp+00000120]    ====> 指向注册码
:004028B8 83CDFF                     or ebp, FFFFFFFF
:004028BB 8BCD                       mov ecx, ebp
:004028BD 33C0                       xor eax, eax
:004028BF F2                         repnz
:004028C0 AE                         scasb                ====> 检测注册码的字符串长度
:004028C1 F7D1                       not ecx
:004028C3 2BF9                       sub edi, ecx
:004028C5 8D542410                   lea edx, dword ptr [esp+10]
:004028C9 8BC1                       mov eax, ecx
:004028CB 8BF7                       mov esi, edi
:004028CD 8BFA                       mov edi, edx
:004028CF C1E902                     shr ecx, 02
:004028D2 F3                         repz
:004028D3 A5                         movsd                  ====> 将注册码中的字符串送入[esp+10]，为后面计算做准备
:004028D4 8BC8                       mov ecx, eax
:004028D6 33C0                       xor eax, eax
:004028D8 83E103                     and ecx, 00000003
:004028DB F3                         repz
:004028DC A4                         movsb
:004028DD 8D7C2410                   lea edi, dword ptr [esp+10]
:004028E1 8BCD                       mov ecx, ebp
:004028E3 F2                         repnz
:004028E4 AE                         scasb                   ====> 再次检测新的字符串长度
:004028E5 F7D1                       not ecx
:004028E7 49                         dec ecx
:004028E8 83F90F                     cmp ecx, 0000000F      ====> 字符串长度是否为 15 个
:004028EB 7515                       jne 00402902           ====> 不是，则序列号错误
:004028ED 8D4C2410                   lea ecx, dword ptr [esp+10]
:004028F1 8844241B                   mov byte ptr [esp+1B], al     ====> [esp+1B]处置 0，对注册码进行截取到 11 位，为后面的计算做准备
:004028F5 51                         push ecx
:004028F6 E8B5EFFFFF                 call 004018B0                  ====> 关键的 CALL
:004028FB 83C404                     add esp, 00000004
:004028FE 85C0                       test eax, eax               ====> EAX 作为旗标
:00402900 751A                       jne 0040291C           ====> 如果 EAX=0 则序列号错误
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:004028EB(C), :0040294D(C), :00402953(C)`

```
:00402902 8D8C2420010000             lea ecx, dword ptr [esp+00000120]
:00402909 89AC2418010000             mov dword ptr [esp+00000118], ebp
:00402910 E81EDB0000                 call 00410433
:00402915 33C0                       xor eax, eax      ====> 失败旗标
:00402917 E9B1000000                 jmp 004029CD
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00402900(C)`

```
:0040291C 8BBC2420010000             mov edi, dword ptr [esp+00000120]    ====> 指向原始的字符串
:00402923 8BCD                       mov ecx, ebp
:00402925 33C0                       xor eax, eax
:00402927 8D542410                   lea edx, dword ptr [esp+10]          ====> 指向刚才合成的字符串
:0040292B F2                         repnz
:0040292C AE                         scasb
:0040292D F7D1                       not ecx
:0040292F 2BF9                       sub edi, ecx
:00402931 8BC1                       mov eax, ecx
:00402933 8BF7                       mov esi, edi
:00402935 8BFA                       mov edi, edx
:00402937 B230                       mov dl, 30
:00402939 C1E902                     shr ecx, 02
:0040293C F3                         repz
:0040293D A5                         movsd                           ====> 向[ESP+10]处复制"0"
:0040293E 8BC8                       mov ecx, eax
:00402940 83E103                     and ecx, 00000003
:00402943 F3                         repz
:00402944 A4                         movsb
:00402945 33C9                       xor ecx, ecx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040295F(C)`

```
:00402947 8A440C1B                   mov al, byte ptr [esp+ecx+1B]        ====> 将最后的 4 个数字分别置入 al 中
:0040294B 3C39                         cmp al, 39                            ====> 是否大于"9"
:0040294D 7FB3                         jg 00402902                         ====> 大于则完蛋
:0040294F 38540C10                     cmp byte ptr [esp+ecx+10], dl        ====> 注册码的第 1 加 ecx 位字符是否小于"0"
:00402953 7CAD                         jl 00402902                         ====> 小于则完蛋
:00402955 2AC2                         sub al, dl                          ====> 将最后的 4 个数字的 ascii 取"0"改为相应的机器码
:00402957 88440C1B                     mov byte ptr [esp+ecx+1B], al        ====> 结果送回[esp+ecx+1B]中
:0040295B 41                           inc ecx                              ====> +1
:0040295C 83F904                       cmp ecx, 00000004                    ====> 是否转换了 4 个
:0040295F 7CE6                         jl 00402947                         ====> 不是则继续
:00402961 0FBE4C241B                   movsx ecx, byte ptr [esp+1B]      \\
:00402966 0FBE44241E                   movsx eax, byte ptr [esp+1E]      \\ ====> 将刚才的结果分别送入这四个中
:0040296B 0FBE7C241C                   movsx edi, byte ptr [esp+1C]      /
:00402970 0FBE74241D                   movsx esi, byte ptr [esp+1D]      /
:00402975 03C1                         add eax, ecx                        ====> 累加起来
:00402977 BD0A000000                   mov ebp, 0000000A
:0040297C 03C7                         add eax, edi
:0040297E 03C6                         add eax, esi
:00402980 99                           cdq                                  ====> mod 0Ah
:00402981 F7FD                         idiv ebp
:00402983 85D2                         test edx, edx                        ====> 余数是否为 0
:00402985 741B                         je 004029A2                          ====> 是，则走，否则等死吧
:00402987 8D8C2420010000               lea ecx, dword ptr [esp+00000120]
:0040298E C7842418010000FFFFFFFF       mov dword ptr [esp+00000118], FFFFFFFF
:00402999 E895DA0000                   call 00410433
:0040299E 33C0                         xor eax, eax                         ====> 置失败标志
:004029A0 EB2B                         jmp 004029CD
```



### Referenced by a (U)nconditional or (C)onditional Jump at Address

`00402985(C)`

|

:004029A2 `8D0C89`    lea ecx, dword ptr [ecx+4*ecx]    ====> 第 12 位数字乘以 5   

:004029A5 `C7842418010000FFFFFFFF`  mov dword ptr [esp+00000118], FFFFFFFF    ====> 这也应该是与 `setup.sys` 某个特殊标志   

:004029B0 `8D044F`    lea eax, dword ptr [edi+2*ecx]    ====> 再乘以 2 加上第 13 位数字   

:004029B3 `8D8C2420010000`    lea ecx, dword ptr [esp+00000120]   

:004029BA `8D1480`    lea edx, dword ptr [eax+4*eax]    ====> 再乘以 5   

:004029BD `8D0456`    lea eax, dword ptr [esi+2*edx]    ====> 再乘以 2 加上第 14 位数字   

:004029C0 `894360`    mov dword ptr [ebx+60], eax        ====> 结果放到[ebx+60]中，应该也是在 `setup.sys` 中，经试验发现，最后 4 位数字表示授权使用的人数   

:004029C3 `E86BDA0000`    call 00410433   

:004029C8 `B801000000`    mov eax, 00000001                 ====> 置成功标志   

### Referenced by a (U)nconditional or (C)onditional Jump at Addresses

`00402917(U)`, `004029A0(U)`

|

:004029CD `8B8C2410010000`    mov ecx, dword ptr [esp+00000110]   

:004029D4 `5F`        pop edi   

:004029D5 `5E`        pop esi   

:004029D6 `5D`        pop ebp   

:004029D7 `5B`        pop ebx   

:004029D8 `64890D00000000`    mov dword ptr fs:[00000000], ecx   

:004029DF `81C40C010000`    add esp, 0000010C   

:004029E5 `C20400`    ret 0004   

### Referenced by a CALL at Address

`004028F6`

|

:004018B0 `81EC00010000`    sub esp, 00000100   

:004018B6 `83C9FF`      or ecx, FFFFFFFF   

:004018B9 `33C0`        xor eax, eax   

:004018BB `53`        push ebx   

:004018BC `8B9C2408010000`    mov ebx, dword ptr [esp+00000108]   

:004018C3 `55`        push ebp   

:004018C4 `56`        push esi   

:004018C5 `57`        push edi   

:004018C6 `8BFB`      mov edi, ebx      ====> [esp+00000108]指向输入的注册码   

:004018C8 `F2`        repnz           ====> 检测注册码的字符串长度   

:004018C9 `AE`        scasb   

:004018CA `F7D1`      not ecx   

:004018CC `49`        dec ecx   

:004018CD `8BFB`      mov edi, ebx   

:004018CF `8BD1`      mov edx, ecx      ====> edx 为字符串长度   

:004018D1 `83C9FF`    or ecx, FFFFFFFF   

:004018D4 `F2`        repnz   

:004018D5 `AE`        scasb   

:004018D6 `F7D1`      not ecx   

:004018D8 `2BF9`      sub edi, ecx   

:004018DA `8D6C2410`  lea ebp, dword ptr [esp+10]   

:004018DE `8BC1`      mov eax, ecx   

:004018E0 `8BF7`      mov esi, edi   

:004018E2 `8BFD`      mov edi, ebp   

:004018E4 `C1E902`    shr ecx, 02   

:004018E7 `F3`        repz   

:004018E8 `A5`        movsd   

:004018E9 `8BC8`      mov ecx, eax   

:004018EB `83E103`    and ecx, 00000003   

:004018EE `83FA0B`    cmp edx, 0000000B       ====> 比较字符串长度是否为 11   

:004018F1 `F3`        repz   

:004018F2 `A4`        movsb   

:004018F3 `0F85A5000000`    jne 0040199E          ====> 不相等，那么就等死吧   

:004018F9 `B803000000`    mov eax, 00000003   

### Referenced by a (U)nconditional or (C)onditional Jump at Address

`00401922(C)`

|

:004018FE `8A0C18`    mov cl, byte ptr [eax+ebx]    \   

:00401901 `80F939`    cmp cl, 39                  \   

:00401904 `0F8794000000`    ja 0040199E                 |   

:0040190A `80F930`    cmp cl, 30                  |   

:0040190D `0F828B000000`    jb 0040199E                 |   

:00401913 `8A4C0410`  mov cl, byte ptr [esp+eax+10]  | ====> 是否为数字，如果是，则转换为机器码数字，否则完蛋   

:00401917 `80C1D0`    add cl, D0                  |   

:0040191A `884C0410`  mov byte ptr [esp+eax+10], cl  |   

:0040191E `40`        inc eax                     |   

:0040191F `83F80B`    cmp eax, 0000000B           /   

:00401922 `7CDA`      jl 004018FE                 /   

:00401924 `8B4C2418`  mov ecx, dword ptr [esp+18]     ====> 第 9 位数字置入 ecx   

:00401928 `81E1FF000000`    and ecx, 000000FF   

:0040192E `41`        inc ecx                     ====> +1   

:0040192F `8BC1`      mov eax, ecx   

:00401931 `C1E005`    shl eax, 05                 ====> 乘以 32   

:00401934 `2BC1`      sub eax, ecx                ====> 减去原数，相当于乘以 31   

:00401936 `8B4C241A`  mov ecx, dword ptr [esp+1A]     ====> 第 11 位数字置入 ecx   

:0040193A `81E1FF000000`    and ecx, 000000FF   

:00401940 `8BD1`      mov edx, ecx   

:00401942 `C1E204`    shl edx, 04                 ====> 乘以 16   

:00401945 `03D1`      add edx, ecx                ====> 加上原数，相当于乘以 17   

:00401947 `8B4C2419`  mov ecx, dword ptr [esp+19]     ====> 第 10 位数字置入 ecx   

:0040194B `81E1FF000000`    and ecx, 000000FF   

:00401951 `03C2`      add eax, edx                ====> 加入到 eax 中   

:00401953 `8D1449`    lea edx, dword ptr [ecx+2*ecx]   

:00401956 `C1E203`    shl edx, 03   

:00401959 `2BD1`      sub edx, ecx                ====> 乘以 7   

:0040195B `B963000000`    mov ecx, 00000063   

:00401960 `03C2`      add eax, edx                ====> 加入到 eax 中，这样我们可以算出，eax 的值=([esp+18]+1)*31+[esp+1A]*17+[esp+19]*7   

:00401962 `99`        cdq   

:00401963 `F7F9`      idiv ecx                  ====> mod 63h，十进制为 99   

:00401965 `8B442413`  mov eax, dword ptr [esp+13]     ====> 将第 4 位数字置入 eax 中   

:00401969 `8B4C2414`  mov ecx, dword ptr [esp+14]     ====> 将第 5 位数字置入 ecx 中   

:0040196D `25FF000000`    and eax, 000000FF   

:00401972 `81E1FF000000`    and ecx, 000000FF   

:00401978 `8D0480`    lea eax, dword ptr [eax+4*eax]  ====> 第 4 位数字乘以 5   

:0040197B `8D0441`    lea eax, dword ptr [ecx+2*eax]  ====> 得到的结果乘以 2 再加上第 5 位数字的值   

:0040197E `3BC2`      cmp eax, edx                ====> 余数是否等于 eax 的值   

:00401980 `751C`      jne 0040199E                ====> 不等，注册码错误   

:00401982 `8A442412`  mov al, byte ptr [esp+12]   

:00401986 `3C58`      cmp al, 58                  ====> 序列号中的第 3 个字母是否为 X   

:00401988 `7404`      je 0040198E   

:0040198A `3C78`      cmp al, 78                  ====> 或者 x   

:0040198C `7510`      jne 0040199E                ====> 二者都不等，去死吧   

### Referenced by a (U)nconditional or (C)onditional Jump at Address

`00401988(C)`

|

:0040198E `5F`        pop edi   

:0040198F `5E`        pop esi   

:00401990 `5D`        pop ebp   

:00401991 `B802000000`    mov eax, 00000002   

:00401996 `5B`        pop ebx   

:00401997 `81C400010000`    add esp, 00000100   

:0040199D `C3`        ret   

### Referenced by a (U)nconditional or (C)onditional Jump at Addresses

`004018F3(C)`, `00401904(C)`, `0040190D(C)`, `00401980(C)`, `0040198C(C)`

|

:0040199E `5F`        pop edi   

:0040199F `5E`        pop esi   

:004019A0 `5D`        pop ebp   

:004019A1 `33C0`      xor eax, eax   

:004019A3 `5B`        pop ebx   

:004019A4 `81C400010000`    add esp, 00000100   

:004019AA `C3`        ret   

从上面的程序中，我们可以知道网络版注册码的情况：



1. 注册码必须为15位(单机版前11位)，第3个字符必须为`x`或`X`，从第4位开始，必须全是数字。
2. 第1、2个字符必须大于等于`"0"`(即`ascii`中的`30`)。
3. `((第 9 位数字+1)*31+第 10 位*7+第 11 位*17) mod 63h=第 4 位乘以 10+第 5 位数字`
4. 最后4位数字相加应等于`10`或者`0`，其中，前3个数字表示授权使用的人数。

根据上面的结果我们可以给出一个注册码：`FSX310000009993`(网上常见的) 或`FSX000000045555`，其他的自己去造吧 `4dd466d0e1e0cc123afbe504da2462ca`
