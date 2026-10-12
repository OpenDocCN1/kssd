
# 函数图像大师 v4.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50573.htm`](https://www.pediy.com/kssd/pediy05/pediy50573.htm)

- 下载页面：`http://www.380000.com/download/show.asp?id=fgm`
- 软件分类：数学研究
- 运行平台：`Win98/ME/2000/XP`
- 软件大小：`359KB`
- 软件授权：共享软件
- 注册方式：用户名－注册码
- 软件作者：`Moonsoft`
- 出品日期：`2003-1-21`

### 【软件简介】

函数图像大师 `V4.5` 是一款基于函数、方程和不等式（组）的开放式数学研究平台。它能画出任意函数（`Y=` 型函数、`X=` 型函数）、方程（一般方程、自由方程、参数方程）和不等式（组）的图像，支持辅助工具、程序插件和软件换肤。

函数图像大师 `V4.5` 中：
- 可以绘制任意方程和任意不等式（组）的图像；
- 添加了大量实用的标准函数，包括高斯函数 `[]` 和 `{}`；
- 重写了 `3.x` 版本的核心，使计算更加稳定；
- 每一个函数、方程和不等式都有自己的名字，这使得你可以在运算中就像调用标准函数一样调用自己定义的函数；
- 随时可以改变当前坐标的极限；
- 可以自己定义参加运算的常量；
- 可以利用辅助工具和程序插件添加功能，而你所需要做的，只是从作者网站上免费下载最新的部件复制到主程序所在文件夹；
- 支持软件换肤更能适合你的口味和习惯。

函数图像大师 `V4.5` 必将成为你的数学得力助手。

### 【软件限制】

试用 `30` 天。

### 【作者声明】

初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 【破解工具】

`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm` `10` 修改版

### 【过程】

既然已经分析了它的同门软件，索性再看看这个吧。趁着还没忘记算法流程，方便点。呵呵。

虽然注册码很长，但算法基本的流程是一样的，变换了参数而得到其它几组注册码，所以我只是记录了第一组的算法过程。

`函数图像大师` `IV.exe` 无壳。`VC++` `6.0` 编写。

- 用户名：`fly`
- 试炼码：`12345-67890-ABCDE-FGHIJ-KLMNO`

反汇编，看看参考，很容易就能找到下面的核心。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0040A657(C)`
|

```
:0040A66B 8B8E08010000      mov ecx, dword ptr [esi+00000108]
:0040A671 8D54240C          lea edx, dword ptr [esp+0C]
                              ==== >EDX=12345-67890-ABCDE-FGHIJ-KLMNO
:0040A675 8D44242C          lea eax, dword ptr [esp+2C]
                              ==== >EAX=fly        用户名
:0040A679 52                push edx
:0040A67A 50                push eax
:0040A67B E81070FFFF        call 00401690
:0040A680 8B8E08010000      mov ecx, dword ptr [esi+00000108]
:0040A686 E8456EFFFF        call 004014D0
                              ==== >关键 CALL！进入！
:0040A68B 84C0              test al, al
:0040A68D 6A40              push 00000040
:0040A68F 7410              je 0040A6A1
                              ==== >跳则 OVER！
:0040A691 8B4E0C            mov ecx, dword ptr [esi+0C]
```

* Possible Reference to Dialog:

|

```
:0040A694 687CC54100        push 0041C57C
```

* Possible StringData Ref from Data Obj -> "注册将在程序重启后生效。"

```
                              ==== >呵呵，胜利女神！
:0040A699 6860C54100        push 0041C560
:0040A69E 51                push ecx
:0040A69F EB0E              jmp 0040A6AF
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0040A68F(C)`
|

* Possible StringData Ref from Data Obj -> "失败"

|

```
:0040A6A1 6858C54100        push 0041C558
```

* Possible StringData Ref from Data Obj -> "非法注册码！"

```
                              ==== >BAD BOY！
:0040A6A6 6848C54100        push 0041C548
```

进入关键 `CALL`：`40A686` `call` `004014D0`

* Referenced by a CALL at Addresses:

| `:00405A69` , `:00407C6E` , `:0040876D` , `:0040907B` , `:0040A553`
|
| `:0040A650` , `:0040A686`
|

```
…… ……省 略…… ……

:00401583 8A4C2425          mov cl, byte ptr [esp+25]
:00401587 B02D              mov al, 2D
                              ==== >AL=2D     即：-
:00401589 3AC8              cmp cl, al
                              ==== >比较注册码第 6 个字符是否是 -
:0040158B 7572              jne 004015FF
:0040158D 3844242B          cmp byte ptr [esp+2B], al
                              ==== >比较注册码第 12 个字符是否是 -
:00401591 756C              jne 004015FF
:00401593 38442431          cmp byte ptr [esp+31], al
                              ==== >比较注册码第 18 个字符是否是 -
:00401597 7566              jne 004015FF
:00401599 38442437          cmp byte ptr [esp+37], al
                              ==== >比较注册码第 24 个字符是否是 -
:0040159D 7560              jne 004015FF
:0040159F 33FF              xor edi, edi
:004015A1 8D742422          lea esi, dword ptr [esp+22]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004015F4(C)`
|

```
:004015A5 8D4C2418          lea ecx, dword ptr [esp+18]
:004015A9 8D542440          lea edx, dword ptr [esp+40]
                              ==== >EDX=fly
:004015AD 51                push ecx
:004015AE 57                push edi
:004015AF 52                push edx
:004015B0 8BCD              mov ecx, ebp
:004015B2 E859000000        call 00401610
                              ==== >算法 CALL！进入！
                              ==== >下面是逐位比较！有一处不同就 OVER 了！
:004015B7 8A46FE            mov al, byte ptr [esi-02]
                              ==== >[esi-02]=12345
:004015BA 8A4C2418          mov cl, byte ptr [esp+18]
                              ==== >[esp+18]=SKI00
```

第一个大循环得出：`SKI00`
第二个大循环得出：`A3000`
第三个大循环得出：`3D000`
第四个大循环得出：`10000`
第五个大循环得出：`1Z13G`

至此得出我的完整的 `5` 部分注册码：`SKI00-A3000-3D000-10000-1Z13G`

```
:004015BE 3AC1              cmp al, cl
:004015C0 753D              jne 004015FF
:004015C2 8A4EFF            mov cl, byte ptr [esi-01]
:004015C5 8A442419          mov al, byte ptr [esp+19]
:004015C9 3AC8              cmp cl, al
:004015CB 7532              jne 004015FF
:004015CD 8A16              mov dl, byte ptr [esi]
:004015CF 8A44241A          mov al, byte ptr [esp+1A]
:004015D3 3AD0              cmp dl, al
:004015D5 7528              jne 004015FF
:004015D7 8A4601            mov al, byte ptr [esi+01]
:004015DA 8A4C241B          mov cl, byte ptr [esp+1B]
:004015DE 3AC1              cmp al, cl
:004015E0 751D              jne 004015FF
:004015E2 8A4E02            mov cl, byte ptr [esi+02]
:004015E5 8A44241C          mov al, byte ptr [esp+1C]
:004015E9 3AC8              cmp cl, al
:004015EB 7512              jne 004015FF
:004015ED 47                inc edi
:004015EE 83C606            add esi, 00000006
:004015F1 83FF05            cmp edi, 00000005
:004015F4 7CAF              jl 004015A5
:004015F6 5F                pop edi
:004015F7 5E                pop esi
:004015F8 B001              mov al, 01
                              ==== >置 1 则 OK！
:004015FA 5D                pop ebp
:004015FB 83C454            add esp, 00000054
:004015FE C3                ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:00401581(C), :0040158B(C), :00401591(C), :00401597(C), :0040159D(C)`
|
| `:004015C0(C), :004015CB(C), :004015D5(C), :004015E0(C), :004015EB(C)`
|

```
:004015FF 5F                pop edi
:00401600 5E                pop esi
:00401601 32C0              xor al, al
                              ==== >清 0 则 OVER！
:00401603 5D                pop ebp
:00401604 83C454            add esp, 00000054
:00401607 C3                ret
```

进入算法 `CALL`：`4015B2` `call` `00401610`

* Referenced by a CALL at Address:

| `:004015B2`
|

```
:00401610 8B442404          mov eax, dword ptr [esp+04]
                              ==== >EAX=fly
:00401614 8B542408          mov edx, dword ptr [esp+08]
                              ==== >EDX=00000000
:00401618 03D0              add edx, eax
:0040161A 83EC0C            sub esp, 0000000C
:0040161D B901000000        mov ecx, 00000001
:00401622 8A02              mov al, byte ptr [edx]
:00401624 84C0              test al, al
:00401626 740E              je 00401636
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00401634(C)`
|

```
:00401628 0FBEC0            movsx eax, al
```



### 反汇编分析

:0040162B `0FAFC8` `imul ecx, eax`

`ECX=66 * 01=66`  
`ECX=66 * 6C=2B08`  
`ECX=2B08 * 79=1456C8`

:0040162E `8A4201` `mov al, byte ptr [edx+01]`  
依次取 `fly` 字符的 HEX 值

:00401631 `42` `inc edx`  
:00401632 `84C0` `test al, al`  
:00401634 `75F2` `jne 00401628`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00401626(C)`

:00401636 `56` `push esi`  
:00401637 `8B74241C` `mov esi, dword ptr [esp+1C]`  
:0040163B `8BC6` `mov eax, esi`  
:0040163D `33D2` `xor edx, edx`  
:0040163F `6A24` `push 00000024`  
:00401641 `8910` `mov dword ptr [eax], edx`  
:00401643 `66895004` `mov word ptr [eax+04], dx`  
:00401647 `8D542408` `lea edx, dword ptr [esp+08]`  
:0040164B `52` `push edx`  
:0040164C `51` `push ecx`  
:0040164D `E83D680100` `call 00417E8F`  
又是一个子运算 `CALL`！得出下面 `[esp+10]` 处的值。进入！

:00401652 `8D442410` `lea eax, dword ptr [esp+10]`  
`EAX=[esp+10]=ski0`

:00401656 `50` `push eax`

* Possible Reference to Dialog:

:00401657 `687CC04100` `push 0041C07C`  
:0040165C `56` `push esi`  
:0040165D `E859C70000` `call 0040DDBB`  
此 `CALL` 将上面所得字符截取前 5 位！  
`ESI=ski0`

:00401662 `83C418` `add esp, 00000018`  
:00401665 `33C9` `xor ecx, ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00401685(C)`

:00401667 `8A0431` `mov al, byte ptr [ecx+esi]`  
:0040166A `3C61` `cmp al, 61`  
:0040166C `7C0B` `jl 00401679`  
:0040166E `3C7A` `cmp al, 7A`  
:00401670 `7F07` `jg 00401679`  
:00401672 `2C20` `sub al, 20`  
:00401674 `880431` `mov byte ptr [ecx+esi], al`  
:00401677 `EB08` `jmp 00401681`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:0040166C(C), :00401670(C)`

:00401679 `84C0` `test al, al`  
:0040167B `7504` `jne 00401681`  
:0040167D `C6043130` `mov byte ptr [ecx+esi], 30`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:00401677(U), :0040167B(C)`

:00401681 `41` `inc ecx`  
:00401682 `83F905` `cmp ecx, 00000005`  
:00401685 `7CE0` `jl 00401667`  
上面这个小循环是将 `1e9tt` 中的小写字母转换为大写字母！  
`ESI=ski00` 转换为 `SKI00` 不够 5 位的后面补 0！

:00401687 `5E` `pop esi`  
:00401688 `83C40C` `add esp, 0000000C`  
:0040168B `C20C00` `ret 000C`

### 子函数分析

进入子运算 `CALL:0040164D call 00417E8F`

再进入 `:00417EAC call 00417E33`

* Referenced by a CALL at Addresses:
`:00417E26, :00417EAC`

:00417E33 `55` `push ebp`  
:00417E34 `8BEC` `mov ebp, esp`  
:00417E36 `837D1400` `cmp dword ptr [ebp+14], 00000000`  
:00417E3A `8B4D0C` `mov ecx, dword ptr [ebp+0C]`  
:00417E3D `53` `push ebx`  
:00417E3E `56` `push esi`  
:00417E3F `57` `push edi`  
:00417E40 `740B` `je 00417E4D`  
:00417E42 `8B7508` `mov esi, dword ptr [ebp+08]`  
:00417E45 `C6012D` `mov byte ptr [ecx], 2D`  
:00417E48 `41` `inc ecx`  
:00417E49 `F7DE` `neg esi`  
:00417E4B `EB03` `jmp 00417E50`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00417E40(C)`

:00417E4D `8B7508` `mov esi, dword ptr [ebp+08]`  
`ESI=001456C8`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00417E4B(U)`

:00417E50 `8BF9` `mov edi, ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00417E76(C)`

:00417E52 `8BC6` `mov eax, esi`  
:00417E54 `33D2` `xor edx, edx`  
:00417E56 `F77510` `div [ebp+10]`  
`[ebp+10]=24`  
`EDX=001456C8 % 24=00`  
`EDX=000090A2 % 24=12`  
`EDX=00000404 % 24=14`  
`EDX=0000001C % 24=1C`

:00417E59 `8BC6` `mov eax, esi`  
:00417E5B `8BDA` `mov ebx, edx`  
:00417E5D `33D2` `xor edx, edx`  
:00417E5F `F77510` `div [ebp+10]`  
`EAX=001456C8 / 24=000090A2`  
`EAX=000090A2 / 24=00000404`  
`EAX=00000404 / 24=0000001C`  
`EAX=0000001C / 24=00000000`

:00417E62 `83FB09` `cmp ebx, 00000009`  
:00417E65 `8BF0` `mov esi, eax`  
:00417E67 `7605` `jbe 00417E6E`  
:00417E69 `80C357` `add bl, 57`  
`BL=12 + 57=69` 即字符：`i`  
`BL=14 + 57=6B` 即字符：`k`  
`BL=1C + 57=73` 即字符：`s`

:00417E6C `EB03` `jmp 00417E71`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00417E67(C)`

:00417E6E `80C330` `add bl, 30`  
`BL=00 + 30=30` 即字符：`0`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00417E6C(U)`

:00417E71 `8819` `mov byte ptr [ecx], bl`  
:00417E73 `41` `inc ecx`  
:00417E74 `85F6` `test esi, esi`  
:00417E76 `77DA` `ja 00417E52`  
循环！

:00417E78 `802100` `and byte ptr [ecx], 00`  
:00417E7B `49` `dec ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00417E88(C)`

:00417E7C `8A17` `mov dl, byte ptr [edi]`  
:00417E7E `8A01` `mov al, byte ptr [ecx]`  
:00417E80 `8811` `mov byte ptr [ecx], dl`  
:00417E82 `8807` `mov byte ptr [edi], al`  
:00417E84 `49` `dec ecx`  
:00417E85 `47` `inc edi`  
:00417E86 `3BF9` `cmp edi, ecx`  
:00417E88 `72F2` `jb 00417E7C`  
这个小循环是将 `0iks` 倒序为：`ski0`

:00417E8A `5F` `pop edi`  
:00417E8B `5E` `pop esi`  
:00417E8C `5B` `pop ebx`  
:00417E8D `5D` `pop ebp`  
:00417E8E `C3` `ret`

### 完美爆破

完美爆破很简单。

:00401601 `32C0` `xor al, al`

改为：`B001` `mov al, 01` 就 OK 了！与 :004015F8 处相映成趣！

### 注册信息保存

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\FgmIV\]  
"User Name"=hex:66,6c,79,00,34,fb,6d,00,68,fb,6d,00,62,00,00,00,86,01,00,00,7d,\  
 03,00,00,d0,01,00,00,d7,03  
"Register Code"=hex:53,4b,49,30,30,2d,41,33,30,30,30,2d,33,44,30,30,30,2d,31,\  
 30,30,30,34,2d,31,5a,31,33,47,00
```

### 整理

用户名：`fly`



注册码：`SKI00-A3000-3D000-10000-1Z13G`

—————————————————————————————————

Cracked By 巢水工作坊——fly【OCN】

2003-4-12 2:00 `4dd466d0e1e0cc123afbe504da2462ca`
