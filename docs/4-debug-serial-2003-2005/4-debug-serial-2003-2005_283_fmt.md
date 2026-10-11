
# TouchPro V4.5.0.

> 原文：[`www.pediy.com/kssd/pediy06/pediy6199.htm`](https://www.pediy.com/kssd/pediy06/pediy6199.htm)

`Ollydbg`——`TouchPro` V4.5.0.0

下载页面： [`www.skycn.com/soft/2827.html`](http://www.skycn.com/soft/2827.html)

软件大小： 282 KB

软件语言： 英文

软件类别： 国外软件 / 共享版 / 文件管理

应用平台： `Win9x/NT/2000/XP`

加入时间： 2003-08-15 17:11:57

下载次数： 1652

推荐等级： \*\*\*\*

开 发 商： [`www.jddesign.co.uk/`](http://www.jddesign.co.uk/)

【软件简介】：Windows 下文件的时间属性有三种：创建时间，修改时间，访问时间。在一些特殊情况下我们要修改日期属性。`TouchPro` 就是一款运行于 Windows 下的时间属性修改工具。`Touch` 安装后集成于资源管理器，不占用任何资源，支持多级目录与隐藏文件的日期属性批量修改。选中文件或目录后单击鼠标右键菜单中的 “`TouchPro`”即可按你指定的时间格式快速将创建时间，修改时间，访问时间设置为你指定时间或当前时间。

【软件限制】：功能限制

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg` 1.09、`PEiD`、`W32Dasm` 9.0 白金版

—————————————————————————————————

【过 程】：

`TouchPro.dll` 无壳。 `Visual C++` 编写。

用户名：`fly [FCG]` （要有空格）

试炼码：`13572468`

—————————————————————————————————

这个东东用 `OD` 调试有点麻烦。安装完程序后会提示是否注册，选是则弹出注册框，运行 `Ollydbg`，附加上 `D:\WINDOWS\System32\MsiExec.exe` 这个进程，然后 `ALT+E` 打开可执行模块，选择 `TouchPro` 模块, 双击“`D:\Program Files\JD Design\TouchPro\TouchPro.dll`”，呵呵，终于进入目标程序领空了！其实在 98 下用 `TRW` 下万能断点最方便了。感谢 `fxyang` 兄的协助！

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100011B4(C)
|
:1000127B 833D4C53011000    cmp dword ptr [1001534C], 00000000
:10001282 7568              jne 100012EC
:10001284 8B35284E0110      mov esi, dword ptr [10014E28]
:1000128A 6A64              push 00000064
:1000128C 8D8560FEFFFF      lea eax, dword ptr [ebp+FFFFFE60]
:10001292 50                push eax
:10001293 6835040000        push 00000435
:10001298 FF7570            push [ebp+70]
:1000129B FFD6              call esi
:1000129D 6A64              push 00000064
:1000129F 8D8598FDFFFF      lea eax, dword ptr [ebp+FFFFFD98]
:100012A5 50                push eax
:100012A6 6832040000        push 00000432
:100012AB FF7570            push [ebp+70]
:100012AE FFD6              call esi
:100012B0 68D4020110        push 100102D4
:100012B5 8D8598FDFFFF      lea eax, dword ptr [ebp+FFFFFD98]
:100012BB 50                push eax
:100012BC 8D8560FEFFFF      lea eax, dword ptr [ebp+FFFFFE60]
:100012C2 50                push eax
:100012C3 E844040000        call 1000170C
                        ====>关键 CALL！进入！
:100012C8 83C40C            add esp, 0000000C
:100012CB 84C0              test al, al
:100012CD 741D              je 100012EC
                        ====>跳则 OVER！
:100012CF 8D8598FDFFFF      lea eax, dword ptr [ebp+FFFFFD98]
:100012D5 50                push eax
:100012D6 8D8560FEFFFF      lea eax, dword ptr [ebp+FFFFFE60]
:100012DC 50                push eax
:100012DD E839070000        call 10001A1B
                        ====>保存注册信息！
:100012E2 0FB6C0            movzx eax, al
:100012E5 59                pop ecx
:100012E6 59                pop ecx
:100012E7 A34C530110        mov dword ptr [1001534C], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:100011BB(C), :10001282(C), :100012CD(C)
|
:100012EC 57                push edi
:100012ED FF7570            push [ebp+70]

* Reference To: USER32.EndDialog, Ord:00C6h
                        |
:100012F0 FF1528020110      Call dword ptr [10010228]
:100012F6 E961FFFFFF        jmp 1000125C

…… ……省 略…… ……

* Possible Reference to String Resource ID=00025: "Your registration details are correct.Thank you for regist"
:10004916 6A19              push 00000019
:10004918 57                push edi
:10004919 E8F7EEFFFF        call 10003815
                        ====>呵呵，胜利女神！
```

—————————————————————————————————

进入关键 CALL：`100012C3 call 1000170C`

```
* Referenced by a CALL at Addresses:
|:100012C3    , :1000204B   
|
:1000170C 55                push ebp
:1000170D 8BEC              mov ebp, esp
:1000170F 81ECD4000000      sub esp, 000000D4
:10001715 53                push ebx
:10001716 FF7508            push [ebp+08]
:10001719 8D852CFFFFFF      lea eax, dword ptr [ebp+FFFFFF2C]
:1000171F 50                push eax
:10001720 32DB              xor bl, bl
:10001722 FF15F84D0110      call dword ptr [10014DF8]

* Possible Reference to String Resource ID=00032: "Enter the number of seconds to offset the existing timestamp"
                        |
:10001728 6A20              push 00000020
:1000172A FF7508            push [ebp+08]
:1000172D E8D95C0000        call 1000740B
                        ====>进入用户名检测 CALL  检测 Name 中是否有空格
:10001732 85C0              test eax, eax
:10001734 59                pop ecx
:10001735 59                pop ecx
:10001736 7468              je 100017A0
                        ====>没有空格则跳则 OVER！
:10001738 56                push esi
:10001739 FF7510            push [ebp+10]
:1000173C 8D852CFFFFFF      lea eax, dword ptr [ebp+FFFFFF2C]
                        ====>EAX=fly [FCG]          Name
:10001742 50                push eax
:10001743 8D45F4            lea eax, dword ptr [ebp-0C]
:10001746 50                push eax
:10001747 E8ABFEFFFF        call 100015F7
                        ====>算法 CALL！进入！
:1000174C 8B450C            mov eax, dword ptr [ebp+0C]
                        ====>EAX=13572468           试炼码
:1000174F 83C40C            add esp, 0000000C
:10001752 33F6              xor esi, esi
:10001754 EB3B              jmp 10001791

                        ====>下面其实就是每 2 位比较注册码！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001797(C)
|
:10001756 83FE0C            cmp esi, 0000000C
:10001759 7341              jnb 1000179C
:1000175B 6683F939          cmp cx, 0039
:1000175F 8A08              mov cl, byte ptr [eax]
:10001761 7705              ja 10001768
                        ====>大于 39 则跳下去-37
:10001763 80E930            sub cl, 30
                        ====>是数字则 HEX 值减 30
:10001766 EB03              jmp 1000176B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001761(C)
|
:10001768 80E937            sub cl, 37
                        ====>大于 39 则-37

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001766(U)
|
:1000176B C0E104            shl cl, 04
                        ====>结果左移 4 位
:1000176E 40                inc eax
:1000176F 40                inc eax
:10001770 668B10            mov dx, word ptr [eax]
:10001773 6685D2            test dx, dx
:10001776 7410              je 10001788
:10001778 6683FA39          cmp dx, 0039
:1000177C 7705              ja 10001783
:1000177E 80EA30            sub dl, 30
:10001781 EB03              jmp 10001786

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000177C(C)
|
:10001783 80EA37            sub dl, 37

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001781(U)
|
:10001786 0ACA              or cl, dl
                        ====>试炼码第 1、2 位结果 OR

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001776(C)
|
:10001788 40                inc eax
```



:10001789 `40` inc `eax`
:1000178A `384C35F4` cmp `byte ptr [ebp+esi-0C]`, `cl`
====> 上面运算的结果逐位和注册码比较！
:1000178E `7509` jne `10001799`
====> 跳则 OVER！
:10001790 `46` inc `esi`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:10001754(U)`
:10001791 `668B08` mov `cx`, `word ptr [eax]`
:10001794 `6685C9` test `cx`, `cx`
:10001797 `75BD` jne `10001756`
====> 循环

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:1000178E(C)`
:10001799 `83FE0C` cmp `esi`, `0000000C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:10001759(C)`
:1000179C `0F94C3` sete `bl`
====> 置 1 则 OK！
:1000179F `5E` pop `esi`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:10001736(C)`
:100017A0 `8AC3` mov `al`, `bl`
:100017A2 `5B` pop `ebx`
:100017A3 `C9` leave
:100017A4 `C3` ret

—————————————————————————————————

进入算法 `CALL`：`10001747 call 100015F7`

\* Referenced by a `CALL` at Address:
`:10001747`
:100015F7 `55` push `ebp`
:100015F8 `8D6C2494` lea `ebp`, `dword ptr [esp-6C]`
:100015FC `81ECC8000000` sub `esp`, `000000C8`
:10001602 `56` push `esi`
:10001603 `8B7578` mov `esi`, `dword ptr [ebp+78]`
:10001606 `56` push `esi`
:10001607 `8D45A4` lea `eax`, `dword ptr [ebp-5C]`
:1000160A `50` push `eax`
:1000160B `FF15F84D0110` call `dword ptr [10014DF8]`
:10001611 `C7457800FC0000` mov `[ebp+78]`, `0000FC00`
====> `[ebp+78]=0000FC00`
:10001618 `EB0A` jmp `10001624`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:1000162A(C)`
:1000161A `0FB7C0` movzx `eax`, `ax`
====> 逐位取用户名字符 HEX 值
:1000161D `014578` add `dword ptr [ebp+78]`, `eax`
====> 循环与 `0000FC00` 累加 最后要用到这个值！
====> `[ebp+78]=FEF3`
:10001620 `46` inc `esi`
:10001621 `46` inc `esi`
:10001622 `33C0` xor `eax`, `eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:10001618(U)`
:10001624 `668B06` mov `ax`, `word ptr [esi]`
:10001627 `6685C0` test `ax`, `ax`
:1000162A `75EE` jne `1000161A`
====> 循环
:1000162C `53` push `ebx`
:1000162D `6810050110` push `10010510`
:10001632 `8D45A4` lea `eax`, `dword ptr [ebp-5C]`
:10001635 `50` push `eax`
:10001636 `FF15F44D0110` call `dword ptr [10014DF4]`
====> 连接用户名与 `123456789012`
:1000163C `8B4574` mov `eax`, `dword ptr [ebp+74]`
====> `EAX=fly [FCG]123456789012`
:1000163F `8A5DB0` mov `bl`, `byte ptr [ebp-50]`
====> 取第 7 位 C
:10001642 `885802` mov `byte ptr [eax+02]`, `bl`
:10001645 `8A5DB6` mov `bl`, `byte ptr [ebp-4A]`
====> 取第 10 位 1
:10001648 `885803` mov `byte ptr [eax+03]`, `bl`
:1000164B `8A5DA4` mov `bl`, `byte ptr [ebp-5C]`
====> 取第 1 位 f
:1000164E `8B757C` mov `esi`, `dword ptr [ebp+7C]`
====> `ESI=TouchPropertyPage` 固定参数！
:10001651 `885804` mov `byte ptr [eax+04]`, `bl`
:10001654 `8A5DB2` mov `bl`, `byte ptr [ebp-4E]`
====> 取第 8 位 G
:10001657 `885805` mov `byte ptr [eax+05]`, `bl`
:1000165A `8A5DAA` mov `bl`, `byte ptr [ebp-56]`
====> 取第 4 位 空格
:1000165D `8A4DB4` mov `cl`, `byte ptr [ebp-4C]`
====> 取第 9 位 ]
:10001660 `8A55B8` mov `dl`, `byte ptr [ebp-48]`
====> 取第 11 位 2
:10001663 `885806` mov `byte ptr [eax+06]`, `bl`
:10001666 `8A5DA6` mov `bl`, `byte ptr [ebp-5A]`
====> 取第 2 位 l
:10001669 `885807` mov `byte ptr [eax+07]`, `bl`
:1000166C `8A5DAE` mov `bl`, `byte ptr [ebp-52]`
====> 取第 6 位 F
:1000166F `885808` mov `byte ptr [eax+08]`, `bl`
:10001672 `8A5DBA` mov `bl`, `byte ptr [ebp-46]`
====> 取第 12 位 3
:10001675 `8808` mov `byte ptr [eax]`, `cl`
====> 用第 9 位]替换`[eax]`的第 1 位 30（H）
:10001677 `885809` mov `byte ptr [eax+09]`, `bl`
:1000167A `8A5DAC` mov `bl`, `byte ptr [ebp-54]`
====> 取第 5 位 [
:1000167D `88580A` mov `byte ptr [eax+0A]`, `bl`
:10001680 `8A5DA8` mov `bl`, `byte ptr [ebp-58]`
====> 取第 3 位 y
:10001683 `88580B` mov `byte ptr [eax+0B]`, `bl`
:10001686 `885001` mov `byte ptr [eax+01]`, `dl`
====> 用第 11 位 2 替换`[eax]`的第 2 位 F5（H）

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`fly [FCG]123456789012` 变化为：
`008EF504 5D 32 43 31 66 47 20 6C 46 33 5B 79 ]2C1fG lF3[y`
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`:10001689` `8A1E` mov `bl`, `byte ptr [esi]`
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
`[ESI]`的值：是固定参数 `TouchPropertyPage`
`100102D4 54 00 6F 00 75 00 63 00 68 00 50 00 72 00 6F 00 T.o.u.c.h.P.r.o.`
`100102E4 70 00 65 00 72 00 74 00 79 00 50 00 61 00 67 00 p.e.r.t.y.P.a.g.`
`100102F4 65 00 e.`
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
====> 下面是把 `5D 32 43 31 66 47 20 6C 46 33 5B 79` 和固定参数 `54 6F 75 63 68 50 72 6F 70 65 72 74` 逐位异或
`:1000168B` `32D9` xor `bl`, `cl`
====> `BL=54 XOR 5D=09`
`:1000168D` `8818` mov `byte ptr [eax]`, `bl`
`:1000168F` `8A4E02` mov `cl`, `byte ptr [esi+02]`
`:10001692` `32CA` xor `cl`, `dl`
====> `CL=6F XOR 32=5D`
`:10001694` `884801` mov `byte ptr [eax+01]`, `cl`
`:10001697` `8A4E04` mov `cl`, `byte ptr [esi+04]`
`:1000169A` `324DB0` xor `cl`, `byte ptr [ebp-50]`
====> `CL=75 XOR 43=36`
`:1000169D` `5B` pop `ebx`
`:1000169E` `884802` mov `byte ptr [eax+02]`, `cl`
`:100016A1` `8A4E06` mov `cl`, `byte ptr [esi+06]`
`:100016A4` `324DB6` xor `cl`, `byte ptr [ebp-4A]`
====> `CL=63 XOR 31=52`
`:100016A7` `884803` mov `byte ptr [eax+03]`, `cl`
`:100016AA` `8A4E08` mov `cl`, `byte ptr [esi+08]`
`:100016AD` `324DA4` xor `cl`, `byte ptr [ebp-5C]`
====> `CL=68 XOR 66=0E`
`:100016B0` `884804` mov `byte ptr [eax+04]`, `cl`
`:100016B3` `8A4E0A` mov `cl`, `byte ptr [esi+0A]`
`:100016B6` `324DB2` xor `cl`, `byte ptr [ebp-4E]`
====> `CL=50 XOR 47=17`



# 反汇编代码分析

## 异或运算段

```asm
:100016B9 884805                   mov byte ptr [eax+05], cl
:100016BC 8A4E0C                   mov cl, byte ptr [esi+0C]
:100016BF 324DAA                   xor cl, byte ptr [ebp-56]
                                  ====>CL=72 XOR 20=52

:100016C2 884806                   mov byte ptr [eax+06], cl
:100016C5 8A4E0E                   mov cl, byte ptr [esi+0E]
:100016C8 324DA6                   xor cl, byte ptr [ebp-5A]
                                  ====>CL=6F XOR 6C=03

:100016CB 884807                   mov byte ptr [eax+07], cl
:100016CE 8A4E10                   mov cl, byte ptr [esi+10]
:100016D1 324DAE                   xor cl, byte ptr [ebp-52]
                                  ====>CL=70 XOR 46=36

:100016D4 884808                   mov byte ptr [eax+08], cl
:100016D7 8A4E12                   mov cl, byte ptr [esi+12]
:100016DA 324DBA                   xor cl, byte ptr [ebp-46]
                                  ====>CL=65 XOR 33=56

:100016DD 884809                   mov byte ptr [eax+09], cl
:100016E0 8A4E14                   mov cl, byte ptr [esi+14]
:100016E3 324DAC                   xor cl, byte ptr [ebp-54]
                                  ====>CL=72 XOR 5B=29

:100016E6 88480A                   mov byte ptr [eax+0A], cl
:100016E9 8A4E16                   mov cl, byte ptr [esi+16]
:100016EC 324DA8                   xor cl, byte ptr [ebp-58]
                                  ====>CL=74 XOR 79=0D

:100016EF 33F6                     xor esi, esi
:100016F1 88480B                   mov byte ptr [eax+0B], cl
```

异或的结果：

```text
008EF504  09 5D 36 52 0E 17 52 03 36 56 29 0D                .]6RR6V).
```

## 循环加密段

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10001704(C)

```asm
:100016F4 8BCE                     mov ecx, esi
:100016F6 B201                     mov dl, 01
:100016F8 D2E2                     shl dl, cl
:100016FA 225578                   and dl, byte ptr [ebp+78]
                                  ====>DL=01、02、04、08 分别 AND F3
                                  ====>F3 即是用户名 HEX 值和 FC00 累加=FEF3 的低位

:100016FD 301406                   xor byte ptr [esi+eax], dl
                                  ===>分别与前 4 位 09 5D 36 52 异或
                                  ①、   ===>[esi+eax]=09 XOR 01=08
                                  ②、   ===>[esi+eax]=5D XOR 02=5F
                                  ③、   ===>[esi+eax]=36 XOR 00=36
                                  ④、   ===>[esi+eax]=52 XOR 00=52

:10001700 46                       inc esi
:10001701 83FE04                   cmp esi, 00000004
:10001704 7CEE                     jl 100016F4
                                  ===>循环 4 次
```

`[EAX]` 的值替换成：这就是 `Name` 运算的结果！其 `HEX` 值就是注册码！

```text
008EF504  08 5F 36 52 0E 17 52 03 36 56 29 0D                _6RR6V).
```

```asm
:10001706 5E                       pop esi
:10001707 83C56C                   add ebp, 0000006C
:1000170A C9                       leave
:1000170B C3                       ret
```

## 用户名检测 CALL

进入用户名检测 `CALL`：`:1000172D call 1000740B`

* Referenced by a `CALL` at Address:
  |:1000172D

```asm
:1000740B 8B442404                 mov eax, dword ptr [esp+04]
                                  ===>EAX=fly [FCG]  Name
:1000740F 668B542408               mov dx, word ptr [esp+08]
                                  ===>DX=20  即：空格
:10007414 EB07                     jmp 1000741D
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10007423(C)

```asm
:10007416 663BCA                   cmp cx, dx
                                  ===>比较用户名中是否有空格？
:10007419 7411                     je 1000742C
:1000741B 40                       inc eax
:1000741C 40                       inc eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10007414(U)

```asm
:1000741D 668B08                   mov cx, word ptr [eax]
:10007420 6685C9                   test cx, cx
:10007423 75F1                     jne 10007416
:10007425 663BCA                   cmp cx, dx
                                  ===>比较用户名中是否有空格？
:10007428 7402                     je 1000742C
:1000742A 33C0                     xor eax, eax
                                  ===>如果到这儿清 0 就 OVER 了！
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:10007419(C), :10007428(C)

```asm
:1000742C C3                       ret
```

## 算法总结

1. 用户名须有空格 `fly [FCG]`
2. `Name` 字符的 `HEX` 值逐位和 `FC00` 累加，`=` `FEF3`，取低位 `F3` 后面用
3. 连接用户名和 `123456789012` -> `fly [FCG]123456789012`
4. 对上面字符串的顺序重新排列：`fly [FCG]123456789012` ==> `]2C1fG lF3[y`
5. `]2C1fG lF3[y` 和 固定参数 `TouchPropertyPage` 的前面 12 位字符 `HEX` 值逐位异或

   得出：`09 5D 36 52 0E 17 52 03 36 56 29 0D`
6. 用 `01`、`02`、`04`、`08` 分别 `AND` `F3` （第 2 步所得的值），分别异或上面所得结果的前 4 位 `09 5D 36 52`
7. 最后得出注册码：`085F36520E1752033656290D`

## 注册信息保存

```reg
REGEDIT4

[HKEY_CURRENT_USER\Software\JD Design\TouchPro]

"RegNumber"=hex:30,38,35,46,33,36,35,32,30,45,31,37,35,32,30,33,33,36,35,36,32,\
 39,30,44

"User"=hex:66,6c,79,20,5b,46,43,47,5d
```

## 整理

`Registartion` `Name`：`fly [FCG]`

`Registartion` `Code`：`085F36520E1752033656290D`

```text
         ,     _/
        /| \_.-~/           \_\   ,     青春都一饷
       ( /~   /             \~-.\ |
       \`\\  _/              \  ~\ )    忍把浮名
       _-~~~-.) )__/;;,.      \_  ///
      /'\__, --~   \ ~~~- ,;;\_\_\_ (.-~~~-.    换了破解轻狂
     `~ _( ,_..-\ (  ,;'' /   ~--  /._`
     /~~//'  /' `~\        ) /--..\_, )_  `
     "  `~"  "      `"     /~'`   `\\~~

                           "     "   "~'  ""
```

`Cracked` `By` 巢水工作坊——`fly [OCN][FCG]`

`2003-8-24` `18:43`

`4dd466d0e1e0cc123afbe504da2462ca`
