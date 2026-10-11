
# 易语言幻真随心听 3.83 反调试+破解分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-516.htm`](https://www.pediy.com/kssd/pediy07/pediy7-516.htm)

【破文标题】 易语言幻真随心听 3.83 反调试+破解分析

【破文作者】 `weiyi75`[Dfcg][D.4S]

【作者邮箱】 `weiyi75@sohu.com`

【作者主页】 Dfcg 官方大本营、龙族联盟论坛

【使用工具】 `Peid`, `Ollydbg`, `Resscope`, `W32Dasm`, `Keymake`

【破解平台】 `Win2000/XP`

【软件名称】 易语言幻真随心听 3.83

【保护方式】 `[Overlay]` 保护，反普通 `Od` 调试器,30 次试用随机 `Nag`。

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

-------------------------------------------------------------

【破解内容]

首先用 `PEID` 查软件为 `Microsoft Visual C++ 6.0` `[Overlay]`,`[Overlay]`?文件尾部有加密数据，你可以试一试用 `Aspack` 加壳，无法运行，必须用 `Winhex` 复制文件尾部数据到加壳文件尾部。`Microsoft Visual C++ 6.0` 编程?感觉就入口是 `C++`。

`Alt+M` 打开内存镜像

内存镜像，项目 `20`

```
地址=00409000
大小=00037000 (225280.)
Owner=hzsxt    00400000
区段=.ecode   //读区段名应该是易语言段。
类型=Imag 01001002
访问=R
初始访问=RWE
```

这个只是猜测。

用 `Resscope` 编辑程序除了文件图标，一个资源都没有发现。

`W32Dasm` 反编译无任何有用信息。

这些和我们见过的资源和调试资源一览无遗的真 `C` 语言程序大不一样，种种现象它是易语言编的。

并且使用中发现大量 `Bug`,运行程序，双击任务栏图标就出错。

如果我的系统没有问题，那么这个软件好意思卖钱?出错位置非常多。

看到幽幽星空`[Dfcg][Tfw]`调试时说这个软件有反调试，我也试了一下没有发现。原来我是用二哥的杀不死的 `OD` 系列，所以没这个问题，想顺利调试还是用修改版吧。

广告一下。

杀不死的 `OD`

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

反软件保护神 `Kill`。

反 `Acprotect` `Kill`。

反未知软件 `Antidbg`。

无入口点多余废话警告，有些软件或壳非常多入口废话警告，点确定就要累死。

`CMPXCHG8G` 危险指令无废话警告,点确定就要累死。

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

于是改用普通 `OD` 调试，运行程序。

我也调试了这个程序。

`OD` 载入程序

命令行下断点

```
bp MessageBoxA
```

注册时候中断。

```
77D36476 >  833D D0C3D677 0>cmp dword ptr ds:[77D6C3D0], 0  //清除断点。
77D3647D    0F85 885B0100   jnz USER32.77D4C00B
77D36483    6A 00           push 0
77D36485    FF7424 14       push dword ptr ss:[esp+14]
77D36489    FF7424 14       push dword ptr ss:[esp+14]
77D3648D    FF7424 14       push dword ptr ss:[esp+14]
77D36491    FF7424 14       push dword ptr ss:[esp+14]
77D36495    E8 03000000     call USER32.MessageBoxExA
77D3649A    C2 1000         retn 10
77D3649D >  55             push ebp
0012F894   1005CBB6  /CALL 到 MessageBoxA 来自 krnln.1005CBB0  //堆栈友好提示。
0012F898   00000000  |hOwner = NULL
0012F89C   0040D0DB  |Text = "对不起,您正在使用非法程序对本程序进行破解操作!幻真随心听被迫关闭.如果有问题请与作者联系.,"
0012F8A0   100D791C  |Title = "信息："
0012F8A4   00002000  \\Style = MB_OK|MB_TASKMODAL
0012F8A8   00E38B30
0012F8AC   100EB300  krnln.100EB300
0012F8B0   00E384A0
0012F8B4   100A6219  返回到 krnln.100A6219 来自 ntdll.RtlSetLastWin32Error
```

我们知道一个复杂程序的判断 `Call` 是层层调用的，全部以 `retn` 结束。

用 `Alt+F9` 返回，程序不停说你在破解，但是并不关闭程序，而是反复打开重复窗口直搞到你死机为止，怒!

快速确定一个确定按钮，然后中断了。

```
1005CBB6    5F            pop edi
1005CBB7    83F8 03       cmp eax, 3
1005CBBA    5E            pop esi
1005CBBB    75 0F         jnz short krnln.1005CBCC
1005CBBD    8B4C24 68     mov ecx, dword ptr ss:[esp+68]
1005CBC1    B8 02000000   mov eax, 2
1005CBC6    8901          mov dword ptr ds:[ecx], eax
1005CBC8    83C4 64       add esp, 64
1005CBCB    C3            retn
```

于是 `Ctrl+F9` 2 次 根据 `Od` 信息直到返回领空为止。

```
100543B5    C3            retn
```

信息框提示你返回到 `0043A73F`

轻轻的 `F8`  //怒!`F8` 被禁用了。

点调试菜单的单步跳过 `F8` 或 `OD` 工具栏按钮。

```
0043A73A    E8 1B380000   call hzsxt.0043DF5A
0043A73F    83C4 28       add esp, 28  //回家的感觉真好,然后当然滚动条往上面看。
0043A742    E9 00000000   jmp hzsxt.0043A747
0043A747    8BE5          mov esp, ebp
0043A749    5D            pop ebp
0043A74A    C3            retn
0043A6F7    55            push ebp
0043A6F8    8BEC          mov ebp, esp
0043A6FA    81EC 0C000000 sub esp, 0C
0043A700    E8 46000000   call hzsxt.0043A74B
0043A705    8945 F8       mov dword ptr ss:[ebp-8], eax  eax=1 就 over
0043A708    837D F8 01    cmp dword ptr ss:[ebp-8], 1
0043A70C    0F85 35000000 jnz hzsxt.0043A747  //这里必须走,可选爆破点,平时不影响使用。
0043A712    6A 00         push 0
0043A714    6A 00         push 0
0043A716    6A 00         push 0
0043A718    68 01030080   push 80000301
0043A71D    6A 00         push 0
0043A71F    68 00000000   push 0
0043A724    68 04000080   push 80000004
0043A729    6A 00         push 0
0043A72B    68 DBD04000   push hzsxt.0040D0DB
0043A730    68 03000000   push 3
0043A735    BB 00030000   mov ebx, 300
0043A73A    E8 1B380000   call hzsxt.0043DF5A  //你在非法调试程序!!!
0043A73F    83C4 28       add esp, 28
0043A742    E9 00000000   jmp hzsxt.0043A747
0043A747    8BE5          mov esp, ebp
0043A749    5D            pop ebp
0043A74A    C3            retn
```

作者是用的 `Timer` 控件判断是否调试。

```
0043A705    8945 F8       mov dword ptr ss:[ebp-8], eax
0043A708    837D F8 01    cmp dword ptr ss:[ebp-8], 1
0043A70C    0F85 35000000 jnz hzsxt.0043A747  //这里必须走。
```

破解方法很多

```
0043A708    837D F8 01    cmp dword ptr ss:[ebp-8], 1
```

修改为

```
0043A708    837D F8 01    cmp dword ptr ss:[ebp-8], 0
```

或

```
0043A70C    0F85 35000000 jnz hzsxt.0043A747
```

修改为

```
0043A70C    0F84 35000000 jz hzsxt.0043A747
```

更简单的就是用杀不死的 `OD` 你可能不会发现有这个问题。

我们爆破上面的位置再次载入程序

下

```
bp MessageBoxA
```

直接注册，填入伪注册码 `9999999`

```
77D36476 >  833D D0C3D677 0>cmp dword ptr ds:[77D6C3D0], 0  中断,清除断点。
77D3647D    0F85 885B0100   jnz USER32.77D4C00B
77D36483    6A 00           push 0
77D36485    FF7424 14       push dword ptr ss:[esp+14]
77D36489    FF7424 14       push dword ptr ss:[esp+14]
77D3648D    FF7424 14       push dword ptr ss:[esp+14]
77D36491    FF7424 14       push dword ptr ss:[esp+14]
77D36495    E8 03000000     call USER32.MessageBoxExA
77D3649A    C2 1000         retn 10
```

堆栈友好提示

```
0012F400   1005CBB6  /CALL 到 MessageBoxA 来自 krnln.1005CBB0
0012F404   00000000  |hOwner = NULL
0012F408   0040D036  |Text = "注册码错误！"
0012F40C   100D791C  |Title = "信息："
0012F410   00002000  \\Style = MB_OK|MB_TASKMODAL
0012F414   00E36AC0
```

还是先 `Alt+F9` 确定注册失败按钮返回。

```
1005CBB6    5F            pop edi
1005CBB7    83F8 03       cmp eax, 3
1005CBBA    5E            pop esi
1005CBBB    75 0F         jnz short krnln.1005CBCC
1005CBBD    8B4C24 68     mov ecx, dword ptr ss:[esp+68]
1005CBC1    B8 02000000   mov eax, 2
1005CBC6    8901          mov dword ptr ds:[ecx], eax
1005CBC8    83C4 64       add esp, 64
1005CBCB    C3            retn
```

于是 `Ctrl+F9` 2 次 根据 `Od` 信息直到返回领空为止。

```
100543B5    C3            retn
```

信息框提示你返回到 `00437513`

轻轻的 `F8`  //怒!`F8` 被禁用了。

```
0043750E    E8 476A0000   call hzsxt.0043DF5A
00437513    83C4 28       add esp, 28   //回家的感觉真好,然后当然滚动条往上面看。
00437516    8BE5          mov esp, ebp
00437518    5D            pop ebp
00437519    C3            retn
0043727E    C3            retn
0043727F    55            push ebp  根据向上返回定律，这个是点注册按钮执行的第一句代码，这里下断，再次注册立即中断。
00437280    8BEC          mov ebp, esp
00437282    81EC 0C000000 sub esp, 0C
00437288    C745 FC 0000000>mov dword ptr ss:[ebp-4], 0
0043728F    6A FF         push -1
00437291    6A 08         push 8
00437293    68 58000116   push 16010058
00437298    68 53000152   push 52010053
0043729D    E8 D06C0000   call hzsxt.0043DF72  //算法 Call
004372A2    83C4 10       add esp, 10   //或得伪注册码
004372A5    8945 F8       mov dword ptr ss:[ebp-8], eax
004372A8    A1 200DE300   mov eax, dword ptr ds:[E30D20]
004372AD    50            push eax    //EAX=k8p8i8=临时注册码//假的只能供你清除 30 次试用，关闭程序又没有注册。
004372AE    FF75 F8       push dword ptr ss:[ebp-8]  //将 999999999 压入堆栈。
004372B1    E8 0620FFFF   call hzsxt.004292BC
004372B6    83C4 08       add esp, 8
004372B9    83F8 00       cmp eax, 0
004372BC    B8 00000000   mov eax, 0
004372C1    0F94C0        sete al
004372C4    8945 F4       mov dword ptr ss:[ebp-C], eax
004372C7    8B5D F8       mov ebx, dword ptr ss:[ebp-8]
004372CA    85DB          test ebx, ebx
```



# 汇编代码分析：启动爆破与校验逻辑

## 代码段：启动校验与注册标记写入

```assembly
004372CC    74 09            je short hzsxt.004372D7  
004372CE    53              push ebx  
004372CF    E8 7A6C0000     call hzsxt.0043DF4E  //经典比较。
004372D4    83C4 04         add esp, 4  
004372D7    837D F4 00      cmp dword ptr ss:[ebp-C], 0  
004372DB    0F84 05020000   je hzsxt.004374E6    //爆破点 1，跳转 Over，启动还有校验,非爆不可了。
004372E1    68 00000000     push 0  
004372E6    BB 44060000     mov ebx, 644  
004372EB    E8 6A6C0000     call hzsxt.0043DF5A  //往下临时注册，怒，你看看它往我们的 C 盘乱写垃圾注册标记。
004372F0    83C4 04         add esp, 4  
004372F3    68 01030080     push 80000301  
004372F8    6A 00           push 0  
004372FA    68 09000000     push 9  
004372FF    68 01030080     push 80000301  
00437304    6A 00           push 0  
00437306    68 00000000     push 0  
0043730B    68 02000000     push 2  
00437310    BB FC050000     mov ebx, 5FC  
00437315    E8 406C0000     call hzsxt.0043DF5A  
0043731A    83C4 1C         add esp, 1C  
0043731D    68 01030080     push 80000301  
00437322    6A 00           push 0  
00437324    68 01000000     push 1  
00437329    68 04000080     push 80000004  
0043732E    6A 00           push 0  
00437330    68 D5A94000     push hzsxt.0040A9D5             ; ASCII "hzyy2.5ws\\"
00437335    68 01030080     push 80000301  
0043733A    6A 00           push 0  
0043733C    68 01000000     push 1  
00437341    68 03000000     push 3  
00437346    BB A4060000     mov ebx, 6A4  
0043734B    E8 0A6C0000     call hzsxt.0043DF5A  
00437350    83C4 28         add esp, 28  
00437353    68 04000080     push 80000004  
00437358    6A 00           push 0  
0043735A    68 B9A94000     push hzsxt.0040A9B9             ; ASCII "C:\\hzrj.com"
0043735F    68 04000080     push 80000004  
00437364    6A 00           push 0  
00437366    68 69CF4000     push hzsxt.0040CF69             ; ASCII "C:\\command.com"
0043736B    68 02000000     push 2  
00437370    BB 3C020000     mov ebx, 23C  
00437375    E8 E06B0000     call hzsxt.0043DF5A  
0043737A    83C4 1C         add esp, 1C  
0043737D    BB 06000000     mov ebx, 6  
00437382    E8 280FFFFF     call hzsxt.004282AF  
00437387    68 04000080     push 80000004  
0043738C    6A 00           push 0  
0043738E    68 B9A94000     push hzsxt.0040A9B9             ; ASCII "C:\\hzrj.com"
00437393    68 03000000     push 3  
00437398    BB 70020000     mov ebx, 270  
0043739D    E8 B86B0000     call hzsxt.0043DF5A  
004373A2    83C4 28         add esp, 28  
004373A5    8945 FC         mov dword ptr ss:[ebp-4], eax  
004373A8    68 01030080     push 80000301  
004373AD    6A 00           push 0  
004373AF    FF75 FC         push dword ptr ss:[ebp-4]  
004373B2    68 01000000     push 1  
004373B7    BB 8C020000     mov ebx, 28C  
004373BC    E8 996B0000     call hzsxt.0043DF5A  
004373C1    83C4 10         add esp, 10  
004373C4    68 01030080     push 80000301  
004373C9    6A 00           push 0  
004373CB    68 95521E00     push 1E5295  
004373D0    68 01030080     push 80000301  
004373D5    6A 00           push 0  
004373D7    FF75 FC         push dword ptr ss:[ebp-4]  
004373DA    68 02000000     push 2  
004373DF    BB 8C060000     mov ebx, 68C  
004373E4    E8 716B0000     call hzsxt.0043DF5A  
004373E9    83C4 1C         add esp, 1C  
004373EC    6A 00           push 0  
004373EE    6A 00           push 0  
004373F0    6A 00           push 0  
004373F2    68 01030080     push 80000301  
004373F7    6A 00           push 0  
004373F9    68 00000000     push 0  
004373FE    68 04000080     push 80000004  
00437403    6A 00           push 0  
00437405    68 78CF4000     push hzsxt.0040CF78  
0043740A    68 03000000     push 3  
0043740F    BB 00030000     mov ebx, 300  
00437414    E8 416B0000     call hzsxt.0043DF5A  
00437419    83C4 28         add esp, 28  
0043741C    6A 00           push 0  
0043741E    68 E8A94000     push hzsxt.0040A9E8  
00437423    6A FF          push -1  
00437425    6A 08          push 8  
00437427    68 00000106     push 6010000  
0043742C    68 01000152     push 52010001  
00437431    E8 1E6B0000     call hzsxt.0043DF54  
00437436    83C4 18         add esp, 18  
00437439    6A 00           push 0  
0043743B    68 00000000     push 0  
00437440    6A FF          push -1  
00437442    6A 06          push 6  
00437444    68 68010116     push 16010168  
00437449    68 01000152     push 52010001  
0043744E    E8 016B0000     call hzsxt.0043DF54  
00437453    83C4 18         add esp, 18  
00437456    6A 00           push 0  
00437458    68 00000000     push 0  
0043745D    6A FF          push -1  
0043745F    6A 06          push 6  
00437461    68 8C000116     push 1601008C  
00437466    68 01000152     push 52010001  
0043746B    E8 E46A0000     call hzsxt.0043DF54  
00437470    83C4 18         add esp, 18  
00437473    BB 06000000     mov ebx, 6  
00437478    E8 320EFFFF     call hzsxt.004282AF  
0043747D    68 01000100     push 10001  
00437482    68 54000106     push 6010054  
00437487    68 53000152     push 52010053  
0043748C    68 03000000     push 3  
00437491    BB 08070000     mov ebx, 708  
00437496    E8 BF6A0000     call hzsxt.0043DF5A  
0043749B    83C4 28         add esp, 28  
0043749E    68 04000080     push 80000004  
004374A3    6A 00           push 0  
004374A5    68 9FA84000     push hzsxt.0040A89F  
004374AA    68 05000080     push 80000005  
004374AF    6A 00           push 0  
004374B1    68 15A44000     push hzsxt.0040A415  
004374B6    68 01000100     push 10001  
004374BB    68 54000106     push 6010054  
004374C0    68 53000152     push 52010053  
004374C5    68 03000000     push 3  
004374CA    BB 08070000     mov ebx, 708  
004374CF    E8 866A0000     call hzsxt.0043DF5A  
004374D4    83C4 28         add esp, 28  
004374D7    C705 840CE300 0>mov dword ptr ds:[E30C84], 2  
004374E1    E9 30000000     jmp hzsxt.00437516  
004374E6    6A 00           push 0  
004374E8    6A 00           push 0  
004374EA    6A 00           push 0  
004374EC    68 01030080     push 80000301  
004374F1    6A 00           push 0  
004374F3    68 00000000     push 0  
004374F8    68 04000080     push 80000004  
004374FD    6A 00           push 0  
004374FF    68 36D04000     push hzsxt.0040D036  
00437504    68 03000000     push 3  
00437509    BB 00030000     mov ebx, 300  
0043750E    E8 476A0000     call hzsxt.0043DF5A    //注册码不对。
00437513    83C4 28         add esp, 28  
00437516    8BE5            mov esp, ebp  
00437518    5D             pop ebp  
00437519    C3             retn
```

## 启动爆破思路

这里是实时校验的一段代码。

经过破解分析许多程序，发现程序员的一些特点。

由于校验注册代码特长，短了不放心啊。所以他们喜欢将启动和实时校验及功能限制等都调用同一个 `Call`，可以节约代码，至少互相有关系。

### 代码段：校验逻辑入口

```assembly
004372B1    E8 0620FFFF     call hzsxt.004292BC  
004372B6    83C4 08         add esp, 8  
004372B9    83F8 00         cmp eax, 0  
004372BC    B8 00000000     mov eax, 0  
004372C1    0F94C0          sete al  
004372C4    8945 F4         mov dword ptr ss:[ebp-C], eax  
004372C7    8B5D F8         mov ebx, dword ptr ss:[ebp-8]  
004372CA    85DB            test ebx, ebx  
004372CC    74 09           je short hzsxt.004372D7  
004372CE    53              push ebx  
004372CF    E8 7A6C0000     call hzsxt.0043DF4E  
004372D4    83C4 04         add esp, 4  
004372D7    837D F4 00      cmp dword ptr ss:[ebp-C], 0  //注意这里是注册成功与否的关键。
```

`dword ptr ss:[ebp-C]=1` 就注册成功。

而 `dword ptr ss:[ebp-C]=X` 是由：

```assembly
004372B1    E8 0620FFFF     call hzsxt.004292BC
```

决定的。

启动判断注册也调用了 `004372B1` 处的 `Call`。

```assembly
004372DB    0F84 05020000   je hzsxt.004374E6
```

我们跟进：

```assembly
004292BC    8B5424 04       mov edx, dword ptr ss:[esp+4]  //跟进 call hzsxt.004292BC  
004292C0    8B4C24 08       mov ecx, dword ptr ss:[esp+8]  
004292C4    85D2            test edx, edx
```



```
004292C6    75 0D           jnz short hzsxt.004292D5  
004292C8    33C0          xor eax, eax  
004292CA    85C9          test ecx, ecx  
004292CC    74 06         je short hzsxt.004292D4  
004292CE    8039 00       cmp byte ptr ds:[ecx], 0  
004292D1    74 01         je short hzsxt.004292D4  
004292D3    48            dec eax  
004292D4    C3            retn  
```

这里面的过程就没有跟踪，也没有兴趣跟。

```
dword ptr ss:[ebp-C]=1
```

程序是不会给你的，难道我们不会主动出击吗?

## 爆破点 2

```
004292BC    8B5424 04     mov edx, dword ptr ss:[esp+4]  //跟进 call hzsxt.004292BC  
004292C0    8B4C24 08     mov ecx, dword ptr ss:[esp+8]  
```

改为

```
004292BC    C745 F4 0100000>mov dword ptr ss:[ebp-C], 1  //主动出击。  
004292C3    C3            retn   //返回上次调用，后面的多余判断执行它们干什么?  
```

...........................................................................

然后去`C`盘清理一下垃圾。

## 破解小结

### 可选爆破点

```
0043A708    837D F8 01      cmp dword ptr ss:[ebp-8], 1
```

修改为

```
0043A708    837D F8 01      cmp dword ptr ss:[ebp-8], 0
```

或

```
0043A70C    0F85 35000000   jnz hzsxt.0043A747  
```

修改为

```
0043A70C    0F84 35000000   jz hzsxt.0043A747  
```

启动和实时校验两处爆破点。

```
004372DB    0F84 05020000   je hzsxt.004374E6  
```

修改为

```
004372DB    0F85 05020000   jne hzsxt.004374E6  
```

```
004292BC    8B5424 04       mov edx, dword ptr ss:[esp+4]   
004292C0    8B4C24 08       mov ecx, dword ptr ss:[esp+8]  
```

改为

```
004292BC    C745 F4 0100000>mov dword ptr ss:[ebp-C], 1   
004292C3    C3              retn    
```

### 伪内存注册机

中断地址：`004372B1`

中断次数：`1`

第一字节：`E8`

指令长度:`5`

### 内存方式

寄存器`EAX`

二哥自己编程的`Mp3`都比它好用，至少没有`Bug+`免费的。

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
