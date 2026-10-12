
# EO Video v1.3

> 原文：[`www.pediy.com/kssd/pediy05/pediy50258.htm`](https://www.pediy.com/kssd/pediy05/pediy50258.htm)

> 原有信息:

> 序号: 235933

> 标题: `pll621` 超人，你的 `pe` 很利害！你可否试一下把一个单线程的软件，修改成多线程？ (151 字)

> 发信人: `winamp`

> 时间: 2002-8-26 0:23:11

> 阅读次数: 8

> 详细信息:

> `http://www.onlinedown.net/eovideo.htm`

> 就像 `word` 一样，同时可以打开多个软件界面！

> 这样转换就方便与快多了！

> 这样才是很利害的 `pe` `diy` 实例！

> 请超人试一下！

> 更改方法和原理：

> 关于怎么打开多个程序：

> 昨天晚上 `winapm` 问了我这个问题，今天抽十分钟空看了一下，具体实施的方法

> 这个程序是判断一个标志位来达到检测自己是否已经运行了，而不是用 `findwindow` 等方法

> `\* Possible Reference to Dialog: DialogID_01EE, CONTROL_ID:00FF, ""`
> `|`

> `:00438665 6AFF                          push FFFFFFFF`
> `:00438667 6821C34D00                     push 004DC321`
> `:0043866C 64A100000000                   mov eax, dword ptr fs:[00000000]`
> `:00438672 50                             push eax`
> `:00438673 64892500000000                 mov dword ptr fs:[00000000], esp`
> `:0043867A 81EC68010000                   sub esp, 00000168`
> `:00438680 898DACFEFFFF                   mov dword ptr [ebp+FFFFFEAC], ecx    `
> `:00038680      898D ACFEFFFF                 MOV [DWORD SS:EBP-154],ECX`
> `:00438686 833D0040520000                 cmp dword ptr [00524000], 00000000` 改成 `00038686       C605 00405200 00                 MOV [BYTE DS:524000],0     `

> `这个就是程序的判断标志位的地方，如果为零就证明没有运行，为 1 就已经运行，他然后就会 GetDesktopWindow 把自己的窗口弹出`

> `:0043868D 0F8EF2000000                   jle 00438785                          改成 0003868D       E9 F3000000                    JMP 00038785              `
> `:00438693 8D8D10FFFF                     lea ecx, dword ptr [ebp+FFFFFF10]     00038692       90                           NOP                          `
> `                                                                            `
> `\* Reference To: MFC42.Ordinal:021C, Ord:021Ch`
> `|`

> `:00438699 E846D80900                     Call 004D5EE4`
> `:0043869E C745FC00000000                 mov [ebp-04], 00000000`

> `\* Reference To: USER32.GetDesktopWindow, Ord:00FFh＝＝＝＝》这个断点很有用，因为如果程序要把自己弹出自己窗口就需要这个 GetDesktopWindow 的 api，下这个断点很容易知道程序为什么不能让自己多次运行`

> `\* Reference To: MFC42.Ordinal:0320, Ord:0320h`
> `|`

> `:00438780 E859D70900                     Call 004D5EDE`

> `\* Referenced by a (U)nconditional or (C)onditional Jump at Address:                `
> `|:0043868D(C)                                                                               `
> `|                                                                     00038780   E8 59D70900                    CALL 000D5EDE                        `
> `:00438785 C7050040520001000000 mov dword ptr [00524000], 00000001 ====>改成 00038785   C705 00405200 00000000   MOV [DWORD DS:524000],0             `
> `:0043878F C7851CFFFFFF94000000 mov dword ptr [ebp+FFFFFF1C], 00000094     0003878F   C785 1CFFFFFF 94000000    MOV [DWORD SS:EBP-E4],94 ,0                    `
> `:00438799 8D8D1CFFFF                   lea ecx, dword ptr [ebp+FFFFFF1C]       00038799   8D8D 1CFFFF                LEA ECX,[DWORD SS:EBP-E4]                    `
> `:0043879F 51                             push ecx                                    0003879F   51                        PUSH ECX                        `
> `                                                                            `
> `\* Reference To: KERNEL32.GetVersionExA, Ord:0175h                  `
> `|`

> `:004387A0 FF15EC414E00                     Call dword ptr [004E41EC]`
> `:004387A6 E8D0460100                       call 0044CE7B`

> 关于注册：判断在下面
> `:00420E5D 8D4DF0                           lea ecx, dword ptr [ebp-10]`
> `:00420E60 51                               push ecx`
> `:00420E61 E852AC0200                       call 0044BAB8`
> `:00420E66 85C0                             test eax, eax`
> `:00420E68 7426                             je 00420E90===>跳就完蛋`

> `call 0044BAB8` 到这里：
> `:0044BADD E834A50800                       Call 004D6016`
> `:0044BAE2 8945A4                           mov dword ptr [ebp-5C], eax`

> `\* Possible StringData Ref from Data Obj ->"X5ME"＝＝》这个就是注册码，本来就没有什么注册码的计算，把 X5ME 填入前面四个，后边几个随便你添`
> `|`

> `:0044BAE5 68A8D35000                       push 0050D3A8`
> `:0044BAEA 8B4DA4                           mov ecx, dword ptr [ebp-5C]`
> `:0044BAED 8B11                             mov edx, dword ptr [ecx]`
> `:0044BAEF 52                               push edx`

> `\* Reference To: MSVCRT.strcmp, Ord:02B8h＝＝＝＝》我就是下断点 strcmp 找到的，本来 mfc42 的机理和 vb 差不多，字符串比较也就这么几个函数，假如让我写的话我肯定自己写比较函数，而不会用公用的，太不安全了`
> `|`

> `:0044BAF0 E8ABB60800                       Call 004D71A0`
> `:0044BAF5 83C408                            add esp, 00000008`
> `:0044BAF8 F7D8                              neg eax`

> `pll621`
> ` 4dd466d0e1e0cc123afbe504da2462ca`
