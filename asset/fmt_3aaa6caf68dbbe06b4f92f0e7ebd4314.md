
# WinImage v5.00.5007

> 原文：[`www.pediy.com/kssd/pediy03/forum981.htm`](https://www.pediy.com/kssd/pediy03/forum981.htm)

WinImage v5.00.5007 注册码破解（转载请保持完整，谢谢！）

作者：`BinC`

破解工具：`Trw2000` 或者 `W32DASM`（特别感谢这两款软件的作者^\_^）

软件名称：`WinImage`

整理日期: 2001.05.10

最新版本：5.00.5007

软件授权：共享软件

使用平台：`Win95/98/NT`

软件下载：`http://download.21cn.com/file/utility/cipan/wibt5007.zip`

软件简介：

据说是最好的 `WIN` 下的映像文件处理工具，但我没试过，因为纯粹是拿来练手^\_^

前言：早上 6：30 分到 `21CN` 下载了它，试用一下后 7：00 开始破解，7：15 破解成功，然后上学，中途逃课来写这篇文章^\_^

首先打开 `WinImage` 观察一下，发现注册方法是用注册码方式的，而且此软件有两种模式：`Standard`（一般）和 `Professional`（专业）（这些可看 `Options/WinImage mode selection` 菜单），应该有两种模式的注册码。打开注册窗口，随便输入注册名和注册码。这里注册名用 `BinC`，注册码用 `12345678`

要注意的是，注册名最好不要多于 8 个字符（其实多于 8 个字符也没关系，下文会讲到）。而且此软件一个注册名有多达 6 个注册码，其中两个是专业模式的注册码，下文也会讲到。

启动 `TRW`，然后设一断点。这里用的是拦载注册码文本框的 `WM_GETTEXT` 消息（当然，你可以用 `Hmemcpy`、`GETDLGITEMTEXTA` 等常用的，这里就不班门弄斧了）。回到注册窗口，单击 `OK`，程序被拦。然后按 `F10` 来到以下代码（以下代码是从 `W32SASM` 上 `COPY` 来的，在 `TRW` 中代码是一样的）：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043926E(C)

|

:00439273 FF750C               push [ebp+0C]
:00439276 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:0043927C 50                   push eax
:0043927D E8E0FeFFFF           call 00439162  <-----注意此 CALL
:00439282 FF7508               push [ebp+08]
:00439285 E804FFFFFF           call 0043918E
:0043928A 8BF8                 mov edi, eax
:0043928C 83C40C               add esp, 0000000C
:0043928F 81FF26DDDCB8         cmp edi, B8DCDD26
:00439295 0F84FE000000         je 00439399
:0043929B 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:004392A1 50                   push eax
:004392A2 8D8500FEFFFF         lea eax, dword ptr [ebp+FFFFFE00]
:004392A8 57                   push edi
:004392A9 50                   push eax
:004392AA E862FFFFFF           call 00439211
                                 ~~~~~~~~~~~~~~此 CALL 过后下 d eax 可看到一般模式的注册码
:004392AF 59                   pop ecx
:004392B0 59                   pop ecx
:004392B1 50                   push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
                                 |
:004392B2 E8AD3A0000           Call 0043CD64
:004392B7 59                   pop ecx
:004392B8 85C0                 test eax, eax
:004392BA 59                   pop ecx
:004392BB 0F84A0000000         je 00439361
:004392C1 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:004392C7 50                   push eax
:004392C8 8D8748190514         lea eax, dword ptr [edi+14051948]
:004392CE 50                   push eax
:004392CF 8D8500FEFFFF         lea eax, dword ptr [ebp+FFFFFE00]
:004392D5 50                   push eax
:004392D6 E836FFFFFF           call 00439211
                                 ~~~~~~~~~~~~~此 CALL 过后下 d eax 可看到一般模式的注册码
:004392DB 59                   pop ecx
:004392DC 59                   pop ecx
:004392DD 50                   push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
                                 |
:004392DE E8813A0000           Call 0043CD64
:004392E3 59                   pop ecx
:004392E4 85C0                 test eax, eax
:004392E6 59                   pop ecx
:004392E7 7478                 je 00439361
:004392E9 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:004392EF 50                   push eax
:004392F0 8D8754190617         lea eax, dword ptr [edi+17061954]
:004392F6 50                   push eax
:004392F7 8D8500FEFFFF         lea eax, dword ptr [ebp+FFFFFE00]
:004392FD 50                   push eax
:004392FE E80EFFFFFF           call 00439211
                                 ~~~~~~~~~~~~~此 CALL 过后下 d eax 可看到一般模式的注册码
:00439303 59                   pop ecx
:00439304 59                   pop ecx
:00439305 50                   push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
                                 |
:00439306 E8593A0000           Call 0043CD64
:0043930B 59                   pop ecx
:0043930C 85C0                 test eax, eax
:0043930E 59                   pop ecx
:0043930F 7450                 je 00439361
:00439311 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:00439317 50                   push eax
:00439318 8D8781190510         lea eax, dword ptr [edi+10051981]
:0043931E 50                   push eax
:0043931F 8D8500FEFFFF         lea eax, dword ptr [ebp+FFFFFE00]
:00439325 50                   push eax
:00439326 E8E6FEFFFF           call 00439211
                                 ~~~~~~~~~~~~~ 此 CALL 过后下 d eax 可看到专业模式的注册码
:0043932B 59                   pop ecx
:0043932C 59                   pop ecx
:0043932D 50                   push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
                                 |
:0043932E E8313A0000           Call 0043CD64
:00439333 59                   pop ecx
:00439334 85C0                 test eax, eax
:00439336 59                   pop ecx
:00439337 7455                 je 0043938E
:00439339 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:0043933F 50                   push eax
:00439340 8D8795190104         lea eax, dword ptr [edi+04011995]
:00439346 50                   push eax
:00439347 8D8500FEFFFF         lea eax, dword ptr [ebp+FFFFFE00]
:0043934D 50                   push eax
:0043934E E8BEFEFFFF           call 00439211
                                 ~~~~~~~~~~~~~此 CALL 过后下 d eax 可看到一般模式的注册码
:00439353 59                   pop ecx
:00439354 59                   pop ecx
:00439355 50                   push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
                                 |
:00439356 E8093A0000           Call 0043CD64
:0043935B 59                   pop ecx
:0043935C 85C0                 test eax, eax
:0043935E 59                   pop ecx
:0043935F 7505                 jne 00439366
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004392BB(C), :004392E7(C), :0043930F(C)
|
:00439361 6A01                 push 00000001
:00439363 58                   pop eax
:00439364 EB35                 jmp 0043939B
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043935F(C)
|
:00439366 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
:0043936C 81C797190602         add edi, 02061997
:00439372 50                   push eax
:00439373 8D8500FEFFFF         lea eax, dword ptr [ebp+FFFFFE00]
:00439379 57                   push edi
:0043937A 50                   push eax
:0043937B E891FEFFFF           call 00439211
                                 ~~~~~~~~~~~~~此 CALL 过后下 d eax 可看到专业模式的注册码
:00439380 59                   pop ecx
:00439381 59                   pop ecx
:00439382 50                   push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
.........
......
```

在 `0043927D` 的那个 `CALL`（你可以按 `F8` 进去看一下）就是判断注册码个数是不是小于等于 8，如果大于 8 就截取前 8 位，然后进行大写转换，再下来可能就是注册码计算了（这……实在不是我等未学过汇编的菜鸟所能理解的^\_^）。

至此，破解结束！：）

后记：小弟刚学破解，也是第一次写破解文章，其中难免有错误之处，请各位高手斧正！还有一件事请各位帮帮忙：哪里有汇编的入门教程可以下载（没学过汇编，看那些注册码算法时如雾里看花）？谢谢！

`BinC`

10:18 10.05.2001 `4dd466d0e1e0cc123afbe504da2462ca`
