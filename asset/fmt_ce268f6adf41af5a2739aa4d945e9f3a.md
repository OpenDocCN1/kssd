
# ChinaRen 校友录伴侣 (v2.01 build 20807)

> 原文：[`www.pediy.com/kssd/pediy05/pediy50496.htm`](https://www.pediy.com/kssd/pediy05/pediy50496.htm)

**简单算法——ChinaRen 校友录伴侣 (v2.01 build 20807)**

- **下载页面**：`http://regcrap.yeah.net`（最新版是 `v2.02 build 20821`）
- **软件大小**：`932KB`
- **软件语言**：简体中文
- **软件类别**：共享
- **应用平台**：`Win9x/2000/xp`

**【软件简介】**：`ChinaRen` 校友录伴侣（以下简称 `CRAP`）是一款专门针对 `ChinaRen` 校友录设计的共享软件。您或许曾因为 `ChinaRen` 的速度太慢而无法浏览自己班级的最新留言；您或许想看看班级以往的留言，但由于一页一页的翻阅太麻烦而苦苦等待；您或许想查找以往的某一条留言，但由于留言太多而无从下手；您或许为由于 `ChinaRen` 服务器瘫痪所造成的班级留言的丢失而遗憾；您或许……你还是试试吧。

**【软件限制】**：功能限制。每一个用户名进行一次注册。

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`W32Dasm8.93` 黄金版

---

**【过 程】**：

`Crap.exe` 无壳。`DELPHI` 编写。反汇编，查找出错提示，很容易找到核心。

伪装者`[CCG]`大侠曾写过 `v1.01 build 10717` 的注册机，`2.01` 版的算法还是差不多一样的。

有些分支判断就没写了，如果自己追一下就知道了。

- **用户名**：`fly`
- **试炼码**：`13572468`

---

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

```
|:004D61CD(C)

|

:004D61C8 6A00                         push 00000000
:004D61CA 6A00                         push 00000000
:004D61CC 49                           dec ecx
:004D61CD 75F9                         jne 004D61C8
:004D61CF 51                           push ecx
:004D61D0 53                           push ebx
:004D61D1 56                           push esi
:004D61D2 8BD8                         mov ebx, eax
:004D61D4 33C0                         xor eax, eax
:004D61D6 55                           push ebp
:004D61D7 68C6644D00                   push 004D64C6
:004D61DC 64FF30                       push dword ptr fs:[eax]
:004D61DF 648920                       mov dword ptr fs:[eax], esp
:004D61E2 8D55FC                       lea edx, dword ptr [ebp-04]
:004D61E5 8B8300030000                 mov eax, dword ptr [ebx+00000300]
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

```
|:004D6184(C)

|

:004D61EB E8A06EF7FF                   call 0044D090
:004D61F0 837DFC00                     cmp dword ptr [ebp-04], 00000000
:004D61F4 751C                         jne 004D6212
```

*Possible StringData Ref from Code Obj -> "请输入用户名。"*

```
                                    |
:004D61F6 B8DC644D00               mov eax, 004D64DC
:004D61FB E8DCFCF6FF               call 00445EDC
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

```
|:004D6195(C)

|

:004D6200 8B9300030000             mov edx, dword ptr [ebx+00000300]
:004D6206 8BC3                     mov eax, ebx
:004D6208 E81F1CF9FF               call 00467E2C
:004D620D E95A020000               jmp 004D646C
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

```
|:004D61F4(C)

|

:004D6212 8D55F8                   lea edx, dword ptr [ebp-08]
:004D6215 8B8300030000             mov eax, dword ptr [ebx+00000300]
:004D621B E8706EF7FF               call 0044D090
:004D6220 837DF800                 cmp dword ptr [ebp-08], 00000000
:004D6224 751C                     jne 004D6242
```

*Possible StringData Ref from Code Obj -> "请输入用户名。"*

```
                                    |
:004D6226 B8DC644D00               mov eax, 004D64DC
:004D622B E8ACFCF6FF               call 00445EDC
:004D6230 8B9300030000             mov edx, dword ptr [ebx+00000300]
:004D6236 8BC3                     mov eax, ebx
:004D6238 E8EF1BF9FF               call 00467E2C
:004D623D E92A020000               jmp 004D646C
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

```
|:004D6224(C)

|

:004D6242 8D55F0                   lea edx, dword ptr [ebp-10]
:004D6245 8B8300030000             mov eax, dword ptr [ebx+00000300]
:004D624B E8406EF7FF               call 0044D090
:004D6250 8B45F0                   mov eax, dword ptr [ebp-10]
                                    =====>EAX=fly
:004D6253 8D55F4                   lea edx, dword ptr [ebp-0C]
:004D6256 E8A9DF0000               call 004E4204
                                    =====>算法 CALL！进入！
:004D625B 8B45F4                   mov eax, dword ptr [ebp-0C]
                                    =====>EAX=434349016219029972
:004D625E 50                       push eax
:004D625F 8D55EC                   lea edx, dword ptr [ebp-14]
:004D6262 8B8304030000             mov eax, dword ptr [ebx+00000304]
:004D6268 E8236EF7FF               call 0044D090
:004D626D 8B55EC                   mov edx, dword ptr [ebp-14]
:004D6270 58                       pop eax
:004D6271 E8F2E8F2FF               call 00404B68
                                    =====>比较 CALL！进入！
:004D6276 7416                     je 004D628E
                                    =====>不跳则 OVER！
```

*Possible StringData Ref from Code Obj -> "您输入的用户注册码不正确。"*

```
                                    =====>BAD BOY！
:004D6278 B8F4644D00               mov eax, 004D64F4
:004D627D E85AFCF6FF               call 00445EDC
:004D6282 8BC3                     mov eax, ebx
:004D6284 E80335F9FF               call 0046978C
:004D6289 E9DE010000               jmp 004D646C
```

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

```
|:004D6276(C)

|

:004D628E 8D55E8                   lea edx, dword ptr [ebp-18]
:004D6291 8B8300030000             mov eax, dword ptr [ebx+00000300]
:004D6297 E8F46DF7FF               call 0044D090
:004D629C 8B55E8                   mov edx, dword ptr [ebp-18]
:004D629F 8B83F0020000             mov eax, dword ptr [ebx+000002F0]
:004D62A5 8B8018020000             mov eax, dword ptr [eax+00000218]
:004D62AB 8B08                     mov ecx, dword ptr [eax]
:004D62AD FF5154                   call [ecx+54]
:004D62B0 40                       inc eax
:004D62B1 741C                     je 004D62CF
```

*Possible StringData Ref from Code Obj -> "请已经注册过该用户名。"*

```
                                    |
:004D62B3 B818654D00               mov eax, 004D6518
:004D62B8 E81FFCF6FF               call 00445EDC
:004D62BD 8B9300030000             mov edx, dword ptr [ebx+00000300]
:004D62C3 8BC3                     mov eax, ebx
:004D62C5 E8621BF9FF               call 00467E2C
:004D62CA E99D010000               jmp 004D646C
```

…… …… 省 略 …… ……

*Possible StringData Ref from Code Obj -> "注册成功！感谢您的注册。"*

```
                                    =====>呵呵，胜利女神！
:004D640D 68A8654D00               push 004D65A8
```

------------------------------------------------------------------

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

**进入算法 CALL：`4D6256`**

```
:004D6256 E8A9DF0000               call 004E4204
```

*Referenced by a CALL at Addresses:*

```
|:004D60FC   , :004D6256   , :004E3E92

|

:004E4204 55                       push ebp
:004E4205 8BEC                     mov ebp, esp
:004E4207 83C4C0                   add esp, FFFFFFC0
:004E420A 56                       push esi
:004E420B 57                       push edi
:004E420C 33C9                     xor ecx, ecx
:004E420E 894DC0                   mov dword ptr [ebp-40], ecx
:004E4211 894DC4                   mov dword ptr [ebp-3C], ecx
:004E4214 894DC8                   mov dword ptr [ebp-38], ecx
:004E4217 894DCC                   mov dword ptr [ebp-34], ecx
:004E421A 894DF4                   mov dword ptr [ebp-0C], ecx
:004E421D 894DF0                   mov dword ptr [ebp-10], ecx
:004E4220 894DEC                   mov dword ptr [ebp-14], ecx
:004E4223 894DE8                   mov dword ptr [ebp-18], ecx
:004E4226 8955F8                   mov dword ptr [ebp-08], edx
:004E4229 8945FC                   mov dword ptr [ebp-04], eax
:004E422C 8B45FC                   mov eax, dword ptr [ebp-04]
:004E422F E8D809F2FF               call 00404C0C
:004E4234 33C0                     xor eax, eax
:004E4236 55                       push ebp
:004E4237 6888444E00               push 004E4488
:004E423C 64FF30                   push dword ptr fs:[eax]
:004E423F 648920                   mov dword ptr fs:[eax], esp
:004E4242 8D55CC                   lea edx, dword ptr [ebp-34]
```
