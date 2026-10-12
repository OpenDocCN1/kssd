
# 电脑故障、软件一查通 V1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6101.htm`](https://www.pediy.com/kssd/pediy06/pediy6101.htm)

*   下载页面：[`www.skycn.com/soft/10483.html`](http://www.skycn.com/soft/10483.html)
*   软件大小：6660 KB
*   软件语言：简体中文
*   软件类别：国产软件 / 共享版 / 电脑学习
*   应用平台：`Win9x/NT/2000/XP`
*   加入时间：2003-05-30 11:16:30
*   下载次数：20983
*   推荐等级：`***`
*   开发商：[`www.simble.com/`](http://www.simble.com/)

### 软件简介
该软件汇集了电脑常用软件使用说明，电脑经常出现的问题，以及解决方法，并提供分类查询。现有资料 400 余篇。
1. 故障类有两种查询方法：分别是按故障现象和组成部分，自定义查询分类，故障现象又分为无法启动，系统黑屏，运行异常，组成部分分为硬盘，内存，声卡，显卡，主板，其他。
2. 我们将常用软件分为系统、网络、图形图像、光盘工具、黑客工具五大类。
3. 自定义查询分为标题查询，全文查询。您还可以自己添加资料，《电脑故障、软件一查通》是解决电脑故障、查找资料的好帮手。

### 软件限制
21 天试用、NAG

### 作者声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 破解工具
`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

---

### 过程

碰到了一个怪事，这个东东安装时需要替换我的几个系统 `DLL` 文件，重启后我的 `Win98SE` 就非法操作而“壮烈牺牲”了。^O^ ^O^ 恢复注册表也不行。虽然用 `GHOST` 恢复了系统，却也丢失了一些资料。唉，记得其 `V1.1` 版本没有这个问题的。

其实这个程序仅仅爆破的话是很简单的，^O^ ^O^ 但是追注册码和算法就有点麻烦了。因为我的水平太低，再加上我也不想再次 `GHOST` 恢复系统（痛苦），所以我只是简单跟踪了一下程序运算的流程，找到了注册码，算法就只有期待高手来完成了。 ^O^ ^O^

`fix.exe` 无壳。 `VB` 编写。

*   序列号：`7A52D81D93844813`
*   试炼码：`13572468`

---

### 启动时比较是否注册及过期

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00441563(C)
|
:00441576 8B10                   mov edx, dword ptr [eax]
:00441578 50                     push eax
:00441579 FF921C030000           call dword ptr [edx+0000031C]
:0044157F 50                     push eax
:00441580 8D8560FFFFFF           lea eax, dword ptr [ebp+FFFFFF60]
:00441586 50                     push eax
* Reference To: MSVBVM60.__vbaObjSet, Ord:0000h
                                                  |
:00441587 FF1584104000           Call dword ptr [00401084]
:0044158D 8B8D60FFFFFF           mov ecx, dword ptr [ebp+FFFFFF60]
* Reference To: MSVBVM60.__vbaLateIdCallLd, Ord:0000h
                                                  |
:00441593 8B3D04114000           mov edi, dword ptr [00401104]
:00441599 6A00                   push 00000000
:0044159B 6806000368             push 68030006
:004415A0 8D55DC                 lea edx, dword ptr [ebp-24]
:004415A3 51                     push ecx
:004415A4 52                     push edx
:004415A5 FFD7                   call edi
:004415A7 83C410                 add esp, 00000010
:004415AA 50                     push eax
* Reference To: MSVBVM60.__vbaBoolVar, Ord:0000h
                                                  |
:004415AB FF15A4104000           Call dword ptr [004010A4]
                                                  ====>是否已注册？
* Reference To: MSVBVM60.__vbaFreeVar, Ord:0000h
                                                  |
:004415B1 8B1D14104000           mov ebx, dword ptr [00401014]
:004415B7 8D4DDC                 lea ecx, dword ptr [ebp-24]
:004415BA 668BF0                 mov si, ax
:004415BD FFD3                   call ebx
:004415BF 6685F6                 test si, si
:004415C2 0F8490000000           je 00441658
                                                  ====>爆破点！
…… ……省略…… ……
* Reference To: MSVBVM60.__vbaI4Var, Ord:0000h
                                                  |
:00441670 FF15BC114000           Call dword ptr [004011BC]
:00441676 33D2                   xor edx, edx
:00441678 83F815                 cmp eax, 00000015
                                                  ====>比较是否已超过 21 天试用期！
:0044167B 0F9CC2                 setl dl
:0044167E F7DA                   neg edx
:00441680 8D4DDC                 lea ecx, dword ptr [ebp-24]
:00441683 668BF2                 mov si, dx
:00441686 FFD3                   call ebx
:00441688 6685F6                 test si, si
:0044168B 0F84A0000000           je 00441731
                                                  ====>跳则 OVER！
:00441691 A188304600             mov eax, dword ptr [00463088]
:00441696 85C0                   test eax, eax
:00441698 7510                   jne 004416AA
:0044169A 6888304600             push 00463088
:0044169F 6838354000             push 00403538
```

---

### 注册时的比较

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00441B3D(C)
|
:00441B57 8B55E8                   mov edx, dword ptr [ebp-18]
                                                  ===>EAX=13572468           试炼码
:00441B5A 52                       push edx
:00441B5B 6860A54000               push 0040A560
* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
                                                  |
:00441B60 FF15D4104000           Call dword ptr [004010D4]
                                                  ===>比较是否输入注册码？
:00441B66 8BF8                   mov edi, eax
:00441B68 8D4DE8                 lea ecx, dword ptr [ebp-18]
:00441B6B F7DF                   neg edi
:00441B6D 1BFF                   sbb edi, edi
:00441B6F 47                     inc edi
:00441B70 F7DF                   neg edi
* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h
                                                  |
:00441B72 FF1534124000           Call dword ptr [00401234]
:00441B78 8D4DE4                 lea ecx, dword ptr [ebp-1C]
* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
                                                  |
:00441B7B FF1538124000           Call dword ptr [00401238]
:00441B81 6685FF                 test di, di
:00441B84 747D                   je 00441C03
                                                  ===>有试炼码则跳！
* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
                                                  |
:00441B86 8B35CC114000           mov esi, dword ptr [004011CC]
:00441B8C B904000280             mov ecx, 80020004
:00441B91 894DA8                 mov dword ptr [ebp-58], ecx
:00441B94 B80A000000             mov eax, 0000000A
:00441B99 894DB8                 mov dword ptr [ebp-48], ecx
:00441B9C BF08000000             mov edi, 00000008
:00441BA1 8D5580                 lea edx, dword ptr [ebp-80]
:00441BA4 8D4DC0                 lea ecx, dword ptr [ebp-40]
:00441BA7 8945A0                 mov dword ptr [ebp-60], eax
:00441BAA 8945B0                 mov dword ptr [ebp-50], eax
* Possible StringData Ref from Code Obj ->"?l?Q"
                                                  |
:00441BAD C745884CC14000         mov [ebp-78], 0040C14C
:00441BB4 897D80                 mov dword ptr [ebp-80], edi
:00441BB7 FFD6                   call esi
:00441BB9 8D5590                 lea edx, dword ptr [ebp-70]
:00441BBC 8D4DD0                 lea ecx, dword ptr [ebp-30]
* Possible StringData Ref from Code Obj ->"????eQ?l?Qx"
                                                  |
:00441BBF C7459838C14000         mov [ebp-68], 0040C138
:00441BC6 897D90                 mov dword ptr [ebp-70], edi
:00441BC9 FFD6                   call esi
:00441BCB 8D45A0                 lea eax, dword ptr [ebp-60]
:00441BCE 8D4DB0                 lea ecx, dword ptr [ebp-50]
:00441BD1 50                     push eax
:00441BD2 8D55C0                 lea edx, dword ptr [ebp-40]
:00441BD5 51                     push ecx
:00441BD6 52                     push edx
:00441BD7 8D45D0                 lea eax, dword ptr [ebp-30]
:00441BDA 6A40                   push 00000040
:00441BDC 50                     push eax
* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
                                                  |
:00441BDD FF1588104000           Call dword ptr [00401088]
                                                  ===>没有则提示输入注册码！
:00441BE3 8D4DA0                 lea ecx, dword ptr [ebp-60]
:00441BE6 8D55B0                 lea edx, dword ptr [ebp-50]
:00441BE9 51                     push ecx
:00441BEA 8D45C0                 lea eax, dword ptr [ebp-40]
:00441BED 52                     push edx
```
