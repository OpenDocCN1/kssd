
# OpenCanvas V2.24

> 原文：[`www.pediy.com/kssd/pediy05/pediy50662.htm`](https://www.pediy.com/kssd/pediy05/pediy50662.htm)

*   下载页面： [`www.skycn.com/soft/11835.html`](http://www.skycn.com/soft/11835.html)
*   软件大小： `1394 KB`
*   软件语言： 简体中文
*   软件类别： 汉化补丁 / 共享版 / 图像处理
*   应用平台： `Win9x/NT/2000/XP`
*   加入时间： `2003-04-24 10:55:33`
*   下载次数： `376`
*   推荐等级： `*****`
*   开发商： [`poo.webfeng.net/poobbs/`](http://poo.webfeng.net/poobbs/)

### 【软件简介】
`OpenCanvas` 让使用者在使用电脑绘图时，就像是在纸上手绘一样，可以画出极为细致的图形。

### 【软件限制】
`NAG`、功能限制。

### 【作者声明】
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 【破解工具】
`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

---

### 【过 程】
从 天空 下载的 笨鸡 汉化的中文版。

`oC224e.exe` 无壳，可能让汉化者代脱了。`Delphi` 编写。

算法不太难，只是太麻烦了。幸好作者没有再来几个循环互相校验。

**试炼码**：`12345678-ABCD-EFGHIJKL`

---

```assembly
:004AD817 E8BCD6FBFF               call 0046AED8
:004AD81C 8D55E0                   lea edx, dword ptr [ebp-20]
:004AD81F 8B83DC020000             mov eax, dword ptr [ebx+000002DC]
:004AD825 E8522DF8FF               call 0043057C
:004AD82A 8B45E0                   mov eax, dword ptr [ebp-20]
                                  ====>EAX=12345678-ABCD-EFGHIJKL
:004AD82D E8EE65F5FF               call 00403E20
                                  ====>取试炼码长度
:004AD832 BA16000000               mov edx, 00000016
                                  ====>EDX=16（H）=22（D）
:004AD837 E86836F7FF               call 00420EA4
                                  ====>比较试炼码是否是 22 位？
:004AD83C 50                       push eax
:004AD83D 8D55DC                   lea edx, dword ptr [ebp-24]
:004AD840 8B83DC020000             mov eax, dword ptr [ebx+000002DC]
:004AD846 E8312DF8FF               call 0043057C
:004AD84B 8B45DC                   mov eax, dword ptr [ebp-24]
:004AD84E E89167F5FF               call 00403FE4
:004AD853 8BD0                     mov edx, eax
:004AD855 8BC6                     mov eax, esi
:004AD857 59                       pop ecx
:004AD858 E8D3D6FBFF               call 0046AF30
:004AD85D 8BC6                     mov eax, esi
                                  ====>EAX=12345678-ABCD-EFGHIJKL
:004AD85F E8E0740400               call 004F4D44
                                  ====>关键 CALL 一！进入！
:004AD864 8983E0020000             mov dword ptr [ebx+000002E0], eax
                                  ====>上面的 CALL 里运算，正确则 EAX 返回 1 或 3。
                                  ====>否则后面 004AD893 处会让你大叫的。^Q^^Q^
:004AD86A 83BBE0020000FF           cmp dword ptr [ebx+000002E0], FFFFFFFF
:004AD871 744C                     je 004AD8BF
                                  ====>跳则 OVER！
:004AD873 8D45FE                   lea eax, dword ptr [ebp-02]
:004AD876 50                       push eax
:004AD877 8D45FF                   lea eax, dword ptr [ebp-01]
:004AD87A 50                       push eax
:004AD87B 56                       push esi
:004AD87C E8D67E0400               call 004F5757
                                  ====>关键 CALL 二！进入！
:004AD881 83C40C                   add esp, 0000000C
:004AD884 84C0                     test al, al
:004AD886 7437                     je 004AD8BF
                                  ====>跳则 OVER！
:004AD888 56                       push esi
:004AD889 E86B7F0400               call 004F57F9
                                  ====>关键 CALL 三！进入！
:004AD88E 59                       pop ecx
:004AD88F 84C0                     test al, al
:004AD891 742C                     je 004AD8BF
                                  ====>跳则 OVER！
:004AD893 8B83E0020000             mov eax, dword ptr [ebx+000002E0]
                                  ====>此处就是检查你 4AD85F 里的选择了，如果你进了陷阱……
:004AD899 83F801                   cmp eax, 00000001
                                  ===>[ebx+000002E0]是否是 1？呵呵，我的是。
:004AD89C 7415                     je 004AD8B3
                                  ====>跳！我的选择！
:004AD89E 83F803                   cmp eax, 00000003
                                  ===>[ebx+000002E0]是否是 3？另个选择！
:004AD8A1 7410                     je 004AD8B3
                                  ====>此处再不跳就 OVER！
:004AD8A3 33C9                     xor ecx, ecx
:004AD8A5 33D2                     xor edx, edx

* Possible StringData Ref from Code Obj ->"Region code mismatch!"
                                  ====>很让人难受的一句话！*o* *o*
:004AD8A7 B82CD94A00               mov eax, 004AD92C
:004AD8AC E82F6D0500               call 005045E0
:004AD8B1 EB40                     jmp 004AD8F3

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004AD89C(C), :004AD8A1(C)
|
:004AD8B3 C7833402000001000000      mov dword ptr [ebx+00000234], 00000001
:004AD8BD EB34                     jmp 004AD8F3

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004AD871(C), :004AD886(C), :004AD891(C)
|
:004AD8BF FF83E4020000             inc dword ptr [ebx+000002E4]
:004AD8C5 8D55D8                   lea edx, dword ptr [ebp-28]

* Possible StringData Ref from Code Obj ->"预 P"
                                  |
:004AD8C8 A14C995000               mov eax, dword ptr [0050994C]
:004AD8CD E8C27FF5FF               call 00405894
:004AD8D2 8B45D8                   mov eax, dword ptr [ebp-28]
:004AD8D5 33C9                     xor ecx, ecx
:004AD8D7 33D2                     xor edx, edx
:004AD8D9 E8026D0500               call 005045E0
:004AD8DE 8BC3                     mov eax, ebx
:004AD8E0 E8F7FEFFFF               call 004AD7DC
:004AD8E5 84C0                     test al, al
:004AD8E7 740A                     je 004AD8F3
:004AD8E9 C7833402000002000000      mov dword ptr [ebx+00000234], 00000002

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004AD8B1(U), :004AD8BD(U), :004AD8E7(C)
|
:004AD8F3 33C0                     xor eax, eax
:004AD8F5 5A                       pop edx
:004AD8F6 59                       pop ecx
:004AD8F7 59                       pop ecx
:004AD8F8 648910                   mov dword ptr fs:[eax], edx
:004AD8FB 681DD94A00               push 004AD91D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AD91B(U)
|
:004AD900 8D45D8                   lea eax, dword ptr [ebp-28]
:004AD903 E89862F5FF               call 00403BA0
:004AD908 8D45DC                   lea eax, dword ptr [ebp-24]
:004AD90B BA02000000               mov edx, 00000002
:004AD910 E8AF62F5FF               call 00403BC4
:004AD915 C3                       ret
```

这些代码用了我一个下午的时间来分析，一个晚上的时间来整理呀。

哎，毅力和细心真的很重要。虽然我很菜，但或许勤能补拙呀。^O^^O^

---

### 关键 CALL 一 开始

进入关键 CALL：`004AD85F` `call 004F4D44`

```assembly
* Referenced by a CALL at Addresses:

|:004AD85F   , :004ADDA2   , :004C5D56   , :004C77B2   , :004C77FA
|:004D2794   , :004D30BC   , :004D3469   , :004D493F   , :004D4BDA
|:004D4E7D
|
:004F4D44 53                       push ebx
:004F4D45 8BD8                     mov ebx, eax
:004F4D47 53                       push ebx
:004F4D48 E894070000               call 004F54E1
                                  ====>呵呵，陷阱呀！
:004F4D4D 59                       pop ecx
:004F4D4E 84C0                     test al, al
:004F4D50 7404                     je 004F4D56
:004F4D52 33C0                     xor eax, eax
:004F4D54 5B                       pop ebx
:004F4D55 C3                       ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004F4D50(C)
|
:004F4D56 53                       push ebx
:004F4D57 E8A2070000               call 004F54FE
                                  ====>关键 CALL！①进入！
:004F4D5C 59                       pop ecx
:004F4D5D 84C0                     test al, al
:004F4D5F 7407                     je 004F4D68
                                  ====>跳则进行其它可能的注册码组合比较！
```



### 004F4D61

`mov eax, 00000001`  
====> 我以第二个组合为目的，所以需要此处置 1！

`jmp 004F4DA1`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004F4D5F(C)`

### 004F4D68

`push ebx`  
`call 004F551B`  
====> 呵呵，陷阱呀！

`pop ecx`  
`test al, al`  
`je 004F4D7A`

`mov eax, 00000002`  
`jmp 004F4DA1`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004F4D71(C)`

### 004F4D7A

`push ebx`  
`call 004F5538`  
====> 呵呵，还有这个可以用。

`pop ecx`  
`test al, al`  
`je 004F4D8C`

`mov eax, 00000003`  
`jmp 004F4DA1`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
`:004F4D83(C)`

### 004F4D8C

`push ebx`  
`call 004F5555`  
====> 呵呵，陷阱呀！

`pop ecx`  
`test al, al`  
`je 004F4D9E`

`mov eax, 00000004`  
`jmp 004F4DA1`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
`:004F4D39(C), :004F4D95(C)`

### 004F4D9E

`or eax, FFFFFFFF`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
`:004F4D66(U), :004F4D78(U), :004F4D8A(U), :004F4D9C(U)`

### 004F4DA1

`pop ebx`  
`ret`

---

### 进入关键 CALL①

`004F4D57` `call 004F54FE`

\* Referenced by a CALL at Address:  
`:004F4D57`

### 004F54FE

`mov eax, dword ptr [esp+04]`  
====> `EAX`=12345678-ABCD-EFGHIJKL 试炼码

`cmp byte ptr [eax+01], 45`  
====> 试炼码第 2 位是否是 `E`？

`jne 004F5518`

`cmp byte ptr [eax+03], 31`  
====> 试炼码第 4 位是否是 `1`？

`jne 004F5518`

`push eax`  
`call 004F5486`  
====> 关键 CALL！②进入！

`add esp, 00000004`  
`ret`

我的试炼码 `12345678-ABCD-EFGHIJKL` 调整为：`1E315678-ABCD-EFGHIJKL`

---

### 进入关键 CALL②

`004F550F` `call 004F5486`

\* Referenced by a CALL at Addresses:  
`:004F54F2, :004F550F, :004F552C, :004F5549, :004F5566`

### 004F5486

`push esi`  
`mov esi, dword ptr [esp+08]`  
`mov al, 2D`  
====> `AL`=2D 即：`-`

`cmp byte ptr [esi+08], al`  
====> 试炼码第 9 位是否是`-`

`jne 004F54DD`  
====> 跳则 OVER！

`cmp byte ptr [esi+0D], al`  
====> 试炼码第 14 位是否是`-`

`jne 004F54DD`  
====> 跳则 OVER！

`movsx eax, byte ptr [esi]`  
====> 取试炼码第 1 位字符的 HEX 值

`cmp eax, 00000041`  
====> 第 1 位不能小于 `41`

`jl 004F54DD`  
====> 跳则 OVER！

`cmp eax, 0000005A`  
====> 第 1 位不能大于 `5A` 即：第 1 位须是大写字母。

`jg 004F54DD`  
====> 跳则 OVER！

`movsx eax, byte ptr [esi+09]`  
====> 取试炼码第 10 位字符的 HEX 值

`cmp eax, 00000041`  
`jl 004F54DD`  
`cmp eax, 0000005A`  
`jg 004F54DD`  
====> 同上面一样，第 10 位应是大写字母！

`movsx eax, byte ptr [esi+0F]`  
====> 取试炼码第 16 位字符的 HEX 值

`cmp eax, 00000041`  
`jl 004F54DD`  
`cmp eax, 0000005A`  
`jg 004F54DD`  
====> 同上面一样，第 16 位应是大写字母！

所以，第 1、10、16 位必须是大写字母！  

我的试炼码 `12345678-ABCD-EFGHIJKL` 调整为：`AE315678-ABCD-EFGHIJKL`

### 004F54C0

`push esi`  
`call 004F5412`  
====> 关键 CALL③！进入！

`movsx ecx, byte ptr [esi+15]`  
====> 取试炼码第 22 位字符的 HEX 值 `4C`

`and eax, 000000FF`  
`add esp, 00000004`  
`xor edx, edx`  
`cmp ecx, eax`  
====> `ECX`=4C 试炼码第 22 位字符的 HEX 值  
====> `EAX`=34 上面 `004F54C1` 处运算得出的值  
====> 所以注册码的第 22 位应是 `4`

因此把试炼码 `AE315678-ABCD-EFGHIJKL` 再次修改为 `AE315678-ABCD-EFGHIJK4`   

当然，我只是在 `Ollydbg` 中修改了寄存器的值，不用再重新 LOAD，那样太麻烦了。^O^^O^

### 004F54D6

`sete dl`  
====> 根据比较结果设置 `DL` 的值 应为 TRUE 即：`01`

`mov al, dl`  
`pop esi`  
`ret`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
`:004F5490(C), :004F5495(C), :004F549D(C), :004F54A2(C), :004F54AB(C), :004F54B0(C), :004F54B9(C), :004F54BE(C)`

### 004F54DD

`xor al, al`  
====> 跳到此处清 0 就 OVER 了！

`pop esi`  
`ret`

---

### 进入关键 CALL③

`004F54C1` `call 004F5412`

\* Referenced by a CALL at Address:  
`:004F54C1`

### 004F5412

`mov eax, dword ptr [esp+04]`  
====> `EAX`=AE315678-ABCD-EFGHIJKL 试炼码

`mov cl, byte ptr [eax+0F]`  
====> `CL`=46 取试炼码第 16 位字符的 HEX 值

`mov dl, byte ptr [eax+09]`  
====> `DL`=41 取试炼码第 10 位字符的 HEX 值

`mov al, byte ptr [eax]`  
====> `AL`=41 取试炼码第 1 位字符的 HEX 值

`add ecx, edx`  
====> `ECX`=46 + 41=00DACE87

`lea ecx, dword ptr [ecx+eax-04]`  
====> `ECX`=00DACE87 + 0082F131 - 04=015DBFB4

`and ecx, 0000000F`  
====> `ECX`=015DBFB4 AND 0000000F=4

`mov al, byte ptr [ecx+00509044]`  
====> 根据 `ECX` 值从`[00509044]`的表中取值！  
====> `AL`=34 将和试炼码的第 22 位比较！

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`[00509044]`内存中是一张表：

```
00509044  30 31 32 33 34 35 36 37 38 39 41 42 43 44 45 46  0123456789ABCDEF
```



# 逆向分析笔记：关键 CALL 序列解析

## 关键 CALL 一 结束
:004F542D C3 ret

**小结**：第 1、10、16 位字母的 HEX 值的低位相加再减 4，以此为指针从 `[00509044]` 内存中的表中取值。

`1+2+6-4=4` `[00509044 + 4]` 处是 `34`，`34` 和第 22 位比较，所以第 22 位是 `4`。

## 关键 CALL 二 开始

*Referenced by a CALL at Addresses:*

- `:004AD87C`, `:004ADE71`, `:004C5E25`, `:004C77C5`, `:004C7812`
- `:004D2863`, `:004D318B`, `:004D3538`, `:004D4A0E`, `:004D4CA9`
- `:004D4F4C`

```assembly
:004F5757 53                 push ebx
:004F5758 56                 push esi
:004F5759 8B74240C           mov esi, dword ptr [esp+0C]
                             ==== >ESI=AE315678-ABCD-EFGHIJK4 试炼码
:004F575D 0FBE460C           movsx eax, byte ptr [esi+0C]
                             ==== >EAX=44 取试炼码第 13 位字符的 HEX 值
:004F5761 83F830             cmp eax, 00000030
:004F5764 7C0A               jl 004F5770
:004F5766 83F839             cmp eax, 00000039
:004F5769 7F05               jg 004F5770
:004F576B 83C0D0             add eax, FFFFFFD0
                             ==== >如果第 13 位字符是数字则此处-30
:004F576E EB0D               jmp 004F577D

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004F5764(C), :004F5769(C)

:004F5770 83F841             cmp eax, 00000041
:004F5773 7C7F               jl 004F57F4
:004F5775 83F85A             cmp eax, 0000005A
:004F5778 7F7A               jg 004F57F4
:004F577A 83C0C9             add eax, FFFFFFC9
                             ==== >如果第 13 位字符是字母则此处-37
                             ==== >EAX=44 - 37=D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F576E(U)

:004F577D 8944240C           mov dword ptr [esp+0C], eax
                             ==== >[esp+0C]=EAX
:004F5781 0FBE4612           movsx eax, byte ptr [esi+12]
                             ==== >EAX=49 取试炼码第 19 位字符的 HEX 值
:004F5785 83F830             cmp eax, 00000030
:004F5788 7C0A               jl 004F5794
:004F578A 83F839             cmp eax, 00000039
:004F578D 7F05               jg 004F5794
:004F578F 8D58D0             lea ebx, dword ptr [eax-30]
                             ==== >如果第 19 位字符是数字则此处-30
:004F5792 EB0D               jmp 004F57A1

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004F5788(C), :004F578D(C)

:004F5794 83F841             cmp eax, 00000041
:004F5797 7C5B               jl 004F57F4
:004F5799 83F85A             cmp eax, 0000005A
:004F579C 7F56               jg 004F57F4
:004F579E 8D58C9             lea ebx, dword ptr [eax-37]
                             ==== >如果第 19 位字符是字母则此处-37
                             ==== >EBX=49 - 37=12

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F5792(U)

:004F57A1 56                 push esi
:004F57A2 E8B3FCFFFF         call 004F545A
                             ==== >关键 CALL！④进入！
:004F57A7 0FBE4E14           movsx ecx, byte ptr [esi+14]
                             ==== >ECX=4B 取试炼码第 21 位字符的 HEX 值
:004F57AB 25FF000000         and eax, 000000FF
:004F57B0 83C404             add esp, 00000004
:004F57B3 3BC8               cmp ecx, eax
                             ==== >ECX=4B 试炼码第 21 位字符的 HEX 值
                             ==== >EAX=44 上面 4F54C1 处运算得出的值
                             ==== >所以注册码的第 21 位应是 D
```
因此把试炼码 `AE315678-ABCD-EFGHIJK4` 再次修改为 `AE315678-ABCD-EFGHIJD4`。

```assembly
:004F57B5 753D               jne 004F57F4
:004F57B7 8B44240C           mov eax, dword ptr [esp+0C]
                             ==== >EAX=0D 取试炼码第 21 位字符的 HEX 值
:004F57BB 83C341             add ebx, 00000041
                             ==== >EBX=12 + 41=53
:004F57BE 83C041             add eax, 00000041
                             ==== >EAX=0D + 41=4E
:004F57C1 50                 push eax
:004F57C2 89442410           mov dword ptr [esp+10], eax
:004F57C6 E8DBFBFFFF         call 004F53A6
                             ==== >CALL⑤！进入！
:004F57CB 83C404             add esp, 00000004
:004F57CE 84C0               test al, al
:004F57D0 7422               je 004F57F4
:004F57D2 53                 push ebx
:004F57D3 E8CEFBFFFF         call 004F53A6
                             ==== >CALL⑥！进入！
:004F57D8 83C404             add esp, 00000004
:004F57DB 84C0               test al, al
:004F57DD 7415               je 004F57F4
:004F57DF 8B542410           mov edx, dword ptr [esp+10]
:004F57E3 8A44240C           mov al, byte ptr [esp+0C]
                             ==== >AL=4E
:004F57E7 8B4C2414           mov ecx, dword ptr [esp+14]
:004F57EB 5E                 pop esi
:004F57EC 8802               mov byte ptr [edx], al
:004F57EE B001               mov al, 01
:004F57F0 8819               mov byte ptr [ecx], bl
                             ==== >[ECX]=BL=53
:004F57F2 5B                 pop ebx
:004F57F3 C3                 ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004F5773(C), :004F5778(C), :004F5797(C), :004F579C(C), :004F57B5(C)
|:004F57D0(C), :004F57DD(C)

:004F57F4 5E                 pop esi
:004F57F5 32C0               xor al, al
:004F57F7 5B                 pop ebx
:004F57F8 C3                 ret
```

---

## 进入关键 CALL④
`004F57A2` `call 004F545A`

*Referenced by a CALL at Address:*
- `:004F57A2`

```assembly
:004F545A 8B442404           mov eax, dword ptr [esp+04]
                             ==== >EAX=AE315678-ABCD-EFGHIJK4 试炼码
:004F545E 8A4812             mov cl, byte ptr [eax+12]
                             ==== >CL=49 取试炼码第 19 位字符的 HEX 值
:004F5461 8A500C             mov dl, byte ptr [eax+0C]
                             ==== >DL=44 取试炼码第 13 位字符的 HEX 值
:004F5464 03CA               add ecx, edx
                             ==== >ECX=0082F149 + 00000044=0082F18D
:004F5466 83E10F             and ecx, 0000000F
                             ==== >ECX=0082F18D AND 0000000F=D
:004F5469 8A8144905000       mov al, byte ptr [ecx+00509044]
                             ==== >根据 ECX 值从[00509044]的表中取值！
                             ==== >AL=44 将和试炼码的第 21 位比较！
```

`[00509044]` 内存中是一张表：

```
00509044  30 31 32 33 34 35 36 37 38 39 41 42 43 44 45 46  0123456789ABCDEF
```

```assembly
:004F546F C3                 ret
```

**小结**：取第 13、19 位试炼码的 HEX 值，同上进行低位相加，再从表中取值。

`4+9=D` `[00509044 + D]` 处是 `44`，`44` 和第 21 位比较，所以第 21 位是 `D`。

---

## 两次进入 CALL⑤、⑥
`004F57C6` `call 004F53A6`

*Referenced by a CALL at Addresses:*
- `:004F57C6`, `:004F57D3`

```assembly
:004F53A6 8B442404           mov eax, dword ptr [esp+04]
                             ==== >EAX=4E
                             ==== >EAX=53
:004F53AA 25FF000000         and eax, 000000FF
:004F53AF 83F841             cmp eax, 00000041
:004F53B2 7C05               jl 004F53B9
:004F53B4 83F85A             cmp eax, 0000005A
:004F53B7 7E0D               jle 004F53C6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004F53B2(C)

:004F53B9 83F861             cmp eax, 00000061
:004F53BC 7C05               jl 004F53C3
:004F53BE 83F87A             cmp eax, 0000007A
```



### 004F53C1 - 004F53C6

`:004F53C1` `7E03` `jle 004F53C6`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:004F53BC(C)`

### 004F53C3 - 004F53C5

`:004F53C3` `32C0` `xor al, al`
`:004F53C5` `C3` `ret`

### 004F53C6 - 004F53C9

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F53B7(C)`, `:004F53C1(C)`

`:004F53C6` `83F841` `cmp eax, 00000041`
`:004F53C9` `7C0F` `jl 004F53DA`

### 004F53CB - 004F53CE

`:004F53CB` `83F85A` `cmp eax, 0000005A`
`:004F53CE` `7F0A` `jg 004F53DA`

### 004F53D0 - 004F53D3

```
:004F53D0 83C0BF                 add eax, FFFFFFBF
                                 ====> EAX=4E - 41=D
                                 ====> EAX=53 - 41=12

:004F53D3 8A8028905000           mov al, byte ptr [eax+00509028]
                                 ====> 根据 EAX 值从 [00509028] 的表中取值！
                                 ====> AL=01
                                 ====> AL=01
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`[00509028]`内存中是一张表：

`00509028` `01 01 00 01 01 01 01 01 01 01 01 00 01 01 01 01`

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

### 004F53D9

`:004F53D9` `C3` `ret`

这 2 次取值运算的作用不太清楚，可能是检测第 13 位、19 位的字符有没有踩上程序预埋的“地雷”吧？
因为此处内存中有的值是 `00`，如果你很走运就碰上此处的话，恭喜你：“`Game Over`”了！~@~~@~

### 004F53DA - 004F53E3

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F53C9(C)`, `:004F53CE(C)`

```
:004F53DA 83C09F                 add eax, FFFFFF9F
:004F53DD 8A8028905000           mov al, byte ptr [eax+00509028]
:004F53E3 C3                      ret
```

关键 CALL 二 结 束

□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□
◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆
关键 CALL 三 开 始

### 进入关键 CALL：`004AD889`

`call 004F57F9`

* Referenced by a CALL at Addresses:
  * `:004AD889`, `:004ADE7F`, `:004C5E33`, `:004C77D2`, `:004C781F`
  * `:004D2871`, `:004D3199`, `:004D3546`, `:004D4A1C`, `:004D4CB7`
  * `:004D4F5A`

### 004F57F9 - 004F57FC

```
:004F57F9 53                      push ebx
:004F57FA 56                      push esi
:004F57FB 57                      push edi
:004F57FC 8B7C2410                mov edi, dword ptr [esp+10]
                                  ====> EDI=AE315678-ABCD-EFGHIJD4
```

因为在下面的运算中第 7 位和第 12 位必须满足几个条件，所以分析了几遍后把第 7 位改为 `4`，第 12 位改为 `3`。

`AE315678-ABCD-EFGHIJD4` 改为 `AE315648-AB3D-EFGHIJD4`

### 004F5800 - 004F5811

```
:004F5800 0FBE4706                movsx eax, byte ptr [edi+06]
                                  ====> EAX=34  取试炼码第 7 位字符的 HEX 值

:004F5804 83F830                 cmp eax, 00000030
:004F5807 7C0A                    jl 004F5813
:004F5809 83F839                 cmp eax, 00000039
:004F580C 7F05                    jg 004F5813
:004F580E 8D58D0                  lea ebx, dword ptr [eax-30]
                                  ====> 如果第 7 位字符是数字则此处-30
                                  ====> EBX=34 - 30=4

:004F5811 EB15                    jmp 004F5828
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F5807(C)`, `:004F580C(C)`

### 004F5813 - 004F5825

```
:004F5813 83F841                  cmp eax, 00000041
:004F5816 0F8C92000000            jl 004F58AE
:004F581C 83F846                  cmp eax, 00000046
:004F581F 0F8F89000000            jg 004F58AE
:004F5825 8D58C9                  lea ebx, dword ptr [eax-37]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:004F5811(U)`

### 004F5828 - 004F5839

```
:004F5828 0FBE470B                movsx eax, byte ptr [edi+0B]
                                  ====> EAX=33  取试炼码第 12 位字符的 HEX 值

:004F582C 83F830                 cmp eax, 00000030
:004F582F 7C0A                    jl 004F583B
:004F5831 83F839                 cmp eax, 00000039
:004F5834 7F05                    jg 004F583B
:004F5836 8D70D0                  lea esi, dword ptr [eax-30]
                                  ====> 如果第 12 位字符是数字则此处-30
                                  ====> ESI=33 - 30=3

:004F5839 EB0D                    jmp 004F5848
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F582F(C)`, `:004F5834(C)`

### 004F583B - 004F5845

```
:004F583B 83F841                  cmp eax, 00000041
:004F583E 7C6E                    jl 004F58AE
:004F5840 83F846                  cmp eax, 00000046
:004F5843 7F69                    jg 004F58AE
:004F5845 8D70C9                  lea esi, dword ptr [eax-37]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:004F5839(U)`

### 004F5848 - 004F585A

```
:004F5848 57                      push edi
:004F5849 E822FCFFFF              call 004F5470
                                  ====> 关键 CALL！⑦进入！

:004F584E 0FBE4F13                movsx ecx, byte ptr [edi+13]
                                  ====> ECX=4A  取试炼码第 20 位字符的 HEX 值

:004F5852 25FF000000              and eax, 000000FF
:004F5857 83C404                  add esp, 00000004
:004F585A 3BC8                    cmp ecx, eax
```

* `ECX=4A` 试炼码第 22 位字符的 `HEX` 值
* `EAX=37` 上面 `4F5849` 处运算得出的值
* 所以注册码的第 20 位应是 `7`

因此把试炼码 `AE315648-AB3D-EFGHIJD4` 再次修改为 `AE315648-AB3D-EFGHI7D4`

好了，`GAME OVER`！不，是终于成功了！！^O^^O^^O^^O^

### 004F585C - 004F589D

```
:004F585C 7550                    jne 004F58AE
:004F585E C1E604                  shl esi, 04
                                  ====> ESI=3 SHL 4=30

:004F5861 03F3                    add esi, ebx
                                  ====> ESI=30 + 4=34

:004F5863 B90D000000             mov ecx, 0000000D
                                  ====> ECX=D

:004F5868 8BC6                    mov eax, esi
:004F586A 99                      cdq
:004F586B F7F9                    idiv ecx
                                  ====> EDX=34 % D=0

:004F586D 85D2                    test edx, edx
                                  ====> 余数是否为 0？此处余数必须为 0！

:004F586F 753D                    jne 004F58AE
                                  ====> 跳则 OVER！

:004F5871 B84FECC44E             mov eax, 4EC4EC4F
:004F5876 F7EE                    imul esi
:004F5878 C1FA02                  sar edx, 02
:004F587B 8BC2                    mov eax, edx
:004F587D C1E81F                  shr eax, 1F
:004F5880 03D0                    add edx, eax
:004F5882 83FA03                  cmp edx, 00000003
:004F5885 7C17                    jl 004F589E
:004F5887 83FA09                  cmp edx, 00000009
:004F588A 7F12                    jg 004F589E

:004F588C B801000000             mov eax, 00000001
:004F5891 33C9                    xor ecx, ecx
:004F5893 3BC0                    cmp eax, eax
:004F5895 5F                      pop edi
:004F5896 0F94C1                  sete cl
:004F5899 5E                      pop esi
:004F589A 8AC1                    mov al, cl
:004F589C 5B                      pop ebx
:004F589D C3                      ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F5885(C)`, `:004F588A(C)`

### 004F589E - 004F58AD

```
:004F589E 33C0                    xor eax, eax
:004F58A0 33C9                    xor ecx, ecx
:004F58A2 83F801                  cmp eax, 00000001
:004F58A5 5F                      pop edi
:004F58A6 0F94C1                  sete cl
:004F58A9 5E                      pop esi
:004F58AA 8AC1                    mov al, cl
:004F58AC 5B                      pop ebx
:004F58AD C3                      ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:004F5816(C)`, `:004F581F(C)`, `:004F583E(C)`, `:004F5843(C)`, `:004F585C(C)`
  * `:004F586F(C)`

### 004F58AE - 004F58B3

```
:004F58AE 5F                      pop edi
:004F58AF 5E                      pop esi
:004F58B0 32C0                    xor al, al
:004F58B2 5B                      pop ebx
:004F58B3 C3                      ret
```

—————————————————————————————————

### 进入关键 CALL⑦：`004F5849`

`call 004F5470`

* Referenced by a CALL at Address:
  * `:004F5849`

### 004F5470

`:004F5470` `8B442404` `mov eax, dword ptr [esp+04]`



#### 汇编代码分析

```
:004F5474 8A480B                 mov cl, byte ptr [eax+0B]
                                  ==> CL=33  取试炼码第 12 位字符的 HEX 值
:004F5477 8A5006                 mov dl, byte ptr [eax+06]
                                  ==> DL=34  取试炼码第 7 位字符的 HEX 值
:004F547A 03CA                   add ecx, edx
                                  ==> ECX=0082F133 + 0082F134=0105E267
:004F547C 83E10F                 and ecx, 0000000F
                                  ==> ECX=0105E267  AND  0000000F=7
:004F547F 8A8144905000           mov al, byte ptr [ecx+00509044]
                                  ==> 根据 ECX 值从[00509044]的表中取值！
                                  ==> AL=37    将和试炼码的第 20 位比较！
```

```
[00509044]内存中是一张表：
00509044  30 31 32 33 34 35 36 37 38 39 41 42 43 44 45 46  0123456789ABCDEF
```

```
:004F5485 C3                    ret
```

#### 小结

取第 12、7 位试炼码的 HEX 值，同上进行低位相加，再从表中取值。

`3+4=7`，`[00509044 + 7]`处是 `37`，`37` 和第 20 位比较，所以第 20 位是 `7`。

#### 算法总结

一、注册码需要 22 位。第 9、14 位固定为 `-`。

二、注册码应该有五种组合的可能，但是第 1、3、5 组合最终是不行的，会弹出“`Region code mismatch!`”。

三、重新按第二种组合的可能进行求逆。注册码的第 2、4 位固定是 `E`、`1`。

四、第 1、10、16 位应是大写字母。

我的试炼码 `12345678-ABCD-EFGHIJKL` 调整为：`AE315678-ABCD-EFGHIJKL`。

五、关键 CALL 一：

第 1、10、16 位字母的 HEX 值的低位相加再减 4，以此为指针从`[00509044]`内存中的表中取值。

```
[00509044]内存中是一张表：
00509044  30 31 32 33 34 35 36 37 38 39 41 42 43 44 45 46  0123456789ABCDEF
```

`1+2+6-4=4`，`[00509044 + 4]`处是 `34`，`34` 和第 22 位比较，所以第 22 位是 `4`。

六、关键 CALL 二：

1. 取第 13、19 位试炼码的 HEX 值，同上进行低位相加，再从表中取值。
   `4+9=D`，`[00509044 + D]`处是 `44`，`44` 和第 21 位比较，所以第 21 位是 `D`。
   我的试炼码 `AE315678-ABCD-EFGHIJKL` 再次调整为：`AE315678-ABCD-EFGHIJD4`。

2. 取第 13、19 位试炼码的 HEX 值，如果是数字则 `-30`，是字母则 `-37`。
   我的 13 位是 `44 - 37=D`；19 位是 `49 - 37=12`。
   `D+程序给的 41=4E`；`12+程序给的 41=53`。这 2 处最后从`[00509028]`内存的表中取值。

七、关键 CALL 三：

1. 取第 12、7 位试炼码的 HEX 值，如果是数字则 `-30`，是字母则 `-37`。
   我的 12 位是 `43 - 37=C`；`C SHL 4=C0`；7 位是 `37 - 30=7`。
   `C0 + 7=C7`。此处的得数（`C7`）和程序给的 `D` 求模，要求余数为 `0`。
   所以简单穷举一下，找到符合条件的第 12、7 位。第 12 位 `3`，第 7 位 `4`。
   `33-30=3`；`3 SHL 4=30`；`34-30=4`；`30+4=34`；`34 % D=0`。
   当然，还有其它可行的选择。
   我的试炼码 `AE315678-ABCD-EFGHIJD4` 再次调整为：`AE315648-AB3D-EFGHIJD4`。

2. 取第 12、7 位试炼码的 HEX 值，同上进行低位相加，再从表中取值。
   `3+4=7`，`[00509044 + 7]`处是 `37`，`37` 和第 20 位比较，所以第 20 位是 `7`。

其它未用到的各位可以是任意数字或字母。

至此求出一组可用的注册码：`AE315648-AB3D-EFGHI7D4`。

#### 注册信息保存

```regedit
REGEDIT4
[HKEY_CLASSES_ROOT\CLSID\{4ACAA712-F2A3-44E6-A749-34C3BD05BECB}\Version]
"idx"=hex:86,1f,52,8d,8d,f9
"id"="AE315648-AB3D-EFGHI7D4"
```

#### 整理

一组可用的注册码：`AE315648-AB3D-EFGHI7D4`

Cracked By 巢水工作坊——fly `[OCN]` `[FCG]`
2003-04-28 0:38 `4dd466d0e1e0cc123afbe504da2462ca`
