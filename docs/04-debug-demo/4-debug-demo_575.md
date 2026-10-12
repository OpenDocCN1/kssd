
# vTuner Plus 3.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50166.htm`](https://www.pediy.com/kssd/pediy05/pediy50166.htm)

`vTuner Plus 3.0`在线注册的破解方法一：爆破篇

工具：`TRW2K`, `W32dasm`, `System Mechanic`, `Regshot`

介绍：新版的 `RealOne player` 爽极了，是一个极好的在线播放器！`vTuner Plus` 是 `RealOne Player` 伴侣。

利用 `vTuner` 可列出成千的电视与广播频道，你可以迅速的在网络上找到现场节目、音乐、新闻节目等任何你喜欢的广播内容。`vTuner` 的主窗口非常酷。注册费`$29.95`

下载：`RealOne` 的下载地址最好用 `Google` 搜搜；`vTuner Plus 3.0` 怕是不好找了。

主页：`http://www.vtuner.com/vtunerApp.html`

大小：和 `RealOne` 一起下载的，忘记了 `^\_^`

限制：`VB`，`15` 天试用，在线注册，`NAG` 窗口提示！

破解者: `moonlite[FCG][BCG]`

**[过程]**

1) 首先运行 `vtuner`，提示注册的 `NAG` 就出来了：) 如果输入注册码的话，点击 `Confirm Number` 按钮后，它还会去网上去验证，好让人烦！那就先去掉这个 `NAG` 吧：

启动 `TRW`，点击 `NAG` 窗口中的 `Run vTuner` 按钮，`Ctrl+D` 后，会来到 `TRW` 的领空，再 `pmodule` 一次：

```
:004B8D18 50                   push eax
:004B8D19 FF92B0020000         call dword ptr [edx+000002B0]//b8119  <----NAG 窗口
:004B8D1F 898508FFFFFF         mov dword ptr [ebp+FFFFFF08], eax     <----光标在这里！
:004B8D25 83BD08FFFFFF00       cmp dword ptr [ebp+FFFFFF08], 00000000
:004B8D2C 7D23                 jge 004B8D51
:004B8D2E 68B0020000           push 000002B0

.........
```

将 `004B8D19/offset: b8119` 处的 `CALL` `NOP` 掉，讨厌的提示窗口就没有了：-) `**********` **补丁之一**

2) 将时间往后调 `30` 天，哈哈，讨厌的窗口又来了，这次所提示过期，让上网注册云云。。。可见前面有暗桩。用 `Regshot` 比较发现:

`[HKEY_LOCAL_MACHINE\Software\NEMS\vTuner]` 处的键值从

`"eDate"="06/23/2002 8:30:13 PM"` ->变成了 `"WWeWantToGetYouLikeACrackDealerWould"`

很明显，这是注册表中过期的标志键值。

用 `W32dasm` 反汇编并查找该字符串，共找到三处。通过设断点，不难找到这里：

```
:004B51C9 8B55A0               mov edx, dword ptr [ebp-60] <-----注册表中 eDate KEY 值字串
:004B51CC 52                   push edx                     <---------入栈
* Possible StringData Ref from Code Obj ->"WWeWantToGetYouLikeACrackDealerWould"
                               |
:004B51CD 6844754500           push 00457544                <-----过期的标志字符串
* Reference To: MSVBVM50.__vbaStrTextCmp, Ord:0000h
                               |
:004B51D2 FF1588C47D00         Call dword ptr [007DC488]    <-----两字符串比较的 CALL;
                                                      不相同的话 EAX=FFFFFFFF，否则 EAX=0;
:004B51D8 8BF0                 mov esi, eax
:004B51DA F7DE                 neg esi
:004B51DC 1BF6                 sbb esi, esi
:004B51DE F7DE                 neg esi          <--------字符串不相同时 ESI=1; 否则 ESI=0
:004B51E0 8B45A0               mov eax, dword ptr [ebp-60] <-------仍为过期的标志字符串
:004B51E3 50                   push eax
:004B51E4 68B00E4500           push 00450EB0    <-------d 450EB0 看看！
* Reference To: MSVBVM50.__vbaStrTextCmp, Ord:0000h
                               |
:004B51E9 FF1588C47D00         Call dword ptr [007DC488]<-----又是两字符串的比较;
                                               eDate 键值不为空的话 EAX=1;否则 EAX=0
:004B51EF F7D8                 neg eax
:004B51F1 1BC0                 sbb eax, eax
:004B51F3 F7D8                 neg eax
:004B51F5 23F0                 and esi, eax <-------ESI 和 EAX 与， 得到标志 ESI；
:004B51F7 85F6                 test esi, esi
:004B51F9 0F8554020000         jne 004B5453 <-------正确的话应该跳转！
:004B51FF C745FC18000000       mov [ebp-04], 00000018
:004B5206 8B4DA0               mov ecx, dword ptr [ebp-60]
:004B5209 51                   push ecx
:004B520A 68B00E4500           push 00450EB0
* Reference To: MSVBVM50.__vbaStrTextCmp, Ord:0000h
                               |
:004B520F FF1588C47D00         Call dword ptr [007DC488]
:004B5215 85C0                 test eax, eax
:004B5217 0F85BB000000         jne 004B52D8<-------过期的话应该在这里跳转！
:004B521D C745FC19000000       mov [ebp-04], 00000019 <-------eDate 键值为空的话
                                               会从这里接着向下走下去。。。
:004B5224 66C78510FFFFFFFFFF   mov word ptr [ebp+FFFFFF10], FFFF
* Possible StringData Ref from Code Obj ->"WWeWantToGetYouLikeACrackDealerWould"
                               |
:004B522D BA44754500           mov edx, 00457544
:004B5232 8D8D78FFFFFF         lea ecx, dword ptr [ebp+FFFFFF78]
* Reference To: MSVBVM50.__vbaStrCopy, Ord:0000h
                               |
:004B5238 FF15DCC57D00         Call dword ptr [007DC5DC]
* Possible StringData Ref from Code Obj ->"eeDate"
                               |
:004B523E BA34754500           mov edx, 00457534
:004B5243 8D8D7CFFFFFF         lea ecx, dword ptr [ebp+FFFFFF7C]
* Reference To: MSVBVM50.__vbaStrCopy, Ord:0000h
                               |
:004B5249 FF15DCC57D00         Call dword ptr [007DC5DC]
* Possible StringData Ref from Code Obj ->"SSOFTWARE\nems\vTuner" <-------看出作者的意图了吧？！
                               |
:004B524F BAA8334500           mov edx, 004533A8
:004B5254 8D4D80               lea ecx, dword ptr [ebp-80]
***************
```

4) 看看从 `004B51F9` 跳过来的代码--------->

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B51F9(C)
|
:004B5453 C745FC24000000       mov [ebp-04], 00000024
:004B545A 6A09                 push 00000009
:004B545C 8B45A0               mov eax, dword ptr [ebp-60]  <-----注册表中 eDate 键值
:004B545F 50                   push eax
* Reference To: MSVBVM50.rtcLeftCharBstr, Ord:0268h
                               |
:004B5460 FF1538C67D00         Call dword ptr [007DC638]
:004B5466 8BD0                 mov edx, eax
:004B5468 8D4D80               lea ecx, dword ptr [ebp-80]
* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
                               |
:004B546B FF1554C67D00         Call dword ptr [007DC654]
:004B5471 50                   push eax <-----注册表中 eDate 键值
* Possible StringData Ref from Code Obj ->"ssetupdate" <------这里俺搞不懂作者有什么意图
                               |
:004B5472 68B0754500           push 004575B0
* Reference To: MSVBVM50.__vbaStrTextCmp, Ord:0000h
                               |
:004B5477 FF1588C47D00         Call dword ptr [007DC488] <-----注册表中 eDate KEY 值和字符串
                                               "setupdate" 比较；不相同的话 EAX=FFFFFFFF，否则 EAX=0
:004B547D F7D8                 neg eax
:004B547F 1BC0                 sbb eax, eax
:004B5481 40                   inc eax
:004B5482 F7D8                 neg eax
:004B5484 66898508FFFFFF       mov word ptr [ebp+FFFFFF08], ax <-----ax 值送存；
:004B548B 8D4D80               lea ecx, dword ptr [ebp-80]
* Reference To: MSVBVM50.__vbaFreeStr, Ord:0000h
                               |
:004B548E FF15A4C67D00         Call dword ptr [007DC6A4]
:004B5494 0FBF8D08FFFFFF       movsx ecx, word ptr [ebp+FFFFFF08]
:004B549B 85C9                 test ecx, ecx
:004B549D 0F84CE010000         je 004B5671 //  b489d <-----不要在这里跳！！
:004B54A3 C745FC25000000       mov [ebp-04], 00000025
```

从这里再往下走，就走到罗马了。。。`^\_^`

。。。。

在 `004B549D/offset: B489D` 处改成不跳，或者 `eDate` 键值改成 `"setupdate"` `**********` **补丁之二**

\=>哈哈，搞定!

如果想完美注册的话, 请看它的续篇-->

\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(待续)...

\-->(续上篇)

`vTuner Plus 3.0` 在线注册的破解方法二：在线注册篇

**[上网验证]**：

随便输入注册码，点击 `Confirm Number` 按钮，上网后不久，出错窗口就跳了出来：

`"Invalid vTuner Plus Serial Number. If you have any problems press the help button in the upper right hand corner or e-mail us at support@vtuner.com."`

好，就看 `TRW` 的了....

点击 `OK` 按钮的同时并 `Ctrl+D` 后，会来到 `TRW` 的领空，`pmodule` 返回后可以找到那个验证出错的 `CALL`。

在这个 `CALL` 处设端点。重新在线注册一次，会来到--->>>>>



* Reference To: `MSVBVM50.__vbaFreeObj`, Ord:0000h

```
|

:006240F4 FF15A8C67D00             Call dword ptr [007DC6A8]

:006240FA 0FBF4584                 movsx eax, word ptr [ebp-7C]

:006240FE 85C0                     test eax, eax

:00624100 0F845E0F0000             je 00625064

:00624106 C745FC0C000000           mov [ebp-04], 0000000C

|

....(很长的验证代码，略)...

* Possible StringData Ref from Code Obj ->"CCool"

|

:00624384 6828E54500               push 0045E528 <-----作者是说"要是能来到这里， 你就很 Cool 的了" ^_^

* Reference To: `MSVBVM50.__vbaStrCmp`, Ord:0000h

|

:00624389 FF15CCC47D00             Call dword ptr [007DC4CC]

:0062438F 85C0                     test eax, eax

:00624391 0F85C9090000             jne 00624D60

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00624100(U)

|

:00624397 C745FC15000000           mov [ebp-04], 00000015 <-----这里呢， 就 Cool 呆了！

:0062439E 8B4508                   mov eax, dword ptr [ebp+08]

:006243A1 8B08                     mov ecx, dword ptr [eax]

:006243A3 8B5508                   mov edx, dword ptr [ebp+08]

:006243A6 52                       push edx

:006243A7 FF9120030000             call dword ptr [ecx+00000320]

:006243AD 50                       push eax

:006243AE 8D45BC                   lea eax, dword ptr [ebp-44]

:006243B1 50                       push eax

* Reference To: `MSVBVM50.__vbaObjSet`, Ord:0000h

|

:006243B2 FF1558C47D00             Call dword ptr [007DC458]

:006243B8 89458C                   mov dword ptr [ebp-74], eax

* Possible StringData Ref from Code Obj ->"SSerial Number Confirmed" <-----我喜欢这个字符串 ：-

|

:006243BB 6890EA4500               push 0045EA90

:006243C0 8B4D8C                   mov ecx, dword ptr [ebp-74]

:006243C3 8B11                     mov edx, dword ptr [ecx]

:006243C5 8B458C                   mov eax, dword ptr [ebp-74]

:006243C8 50                       push eax

:006243C9 FF5254                   call [edx+54]

:006243CC 894588                   mov dword ptr [ebp-78], eax

:006243CF 837D8800                 cmp dword ptr [ebp-78], 00000000

:006243D3 7D1D                     jge 006243F2

:006243D5 6A54                     push 00000054

...............

* Reference To: `MSVBVM50.__vbaObjSet`, Ord:0000h

|

:00624420 FF1558C47D00             Call dword ptr [007DC458]

:00624426 89458C                   mov dword ptr [ebp-74], eax

* Possible StringData Ref from Code Obj ->"vvTuner Plus Serial Number Confirmed. "

                                       ->"Downloading vTuner Plus please "

                                       ->"wait. This should take a few minutes."

|

:00624429 6810EB4500               push 0045EB10

:0062442E 8B4D8C                   mov ecx, dword ptr [ebp-74]

............
```

可见，要是强迫跳到`00624397`是否就大功告成了???

动手试试运气----

将`00624100`处改`jmp 624397`，重新再注册一遍：

WooOOO!!! 成功啦。。。

接下来，就是程序提示自动更新注册表和文件。。。从此，`vTuner Plus`就是注册版了！！

出于好奇, 用`System Mechanic`全面的比较了一下注册前后的变化, 结果如下:

```
\--------------------------------\*

FILES CHANGED: (7)

REGISTRY KEYS ADDED: (2)

REGISTRY KEY VALUES CHANGED: (16)

REGISTRY KEY VALUES ADDED: (5)
```

看来, 要想不上网就完美注册它的话, 其难度可想而知!

```
*-----------------------------------

\--------------------\*

{注释}

(I)此方法适用于 `vTuner Plus` 最新版 4.0;

(II)但是并非总好使! 为了安全起见, 尤其不要在国产软件上使用;

{后记}

在线验证的软件真是越来越多了. 写此破解的目的是为了"抛砖引玉".

望高手们能把自己的经验贴出来交流:-)

\---------------------\*

moonlite 写于 2002-6-15  4dd466d0e1e0cc123afbe504da2462ca
```
