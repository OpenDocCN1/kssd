
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
