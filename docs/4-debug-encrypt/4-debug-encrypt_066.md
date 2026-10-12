
# AVI Joiner MD5 的暴力破解

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-306.htm`](https://www.pediy.com/kssd/pediy07/pediy7-306.htm)

==============================

AVI/MPEG/RM/WMV Joiner V4.61 汉化版 crack tutorial

==============================

==============================

## 程序说明

AVI/MPEG/RM Joiner 将琐碎影片按顺序作结合整理。

这个软件可以帮助你把零散的影片文件组合成为一个大的影片文件。

这个程序支持 AVI, MPEG, MPEG-4，DivX 以及 RM 格式影片，它可以将不同格式的影片组合。

你可以任意组合或者排列这些片段。

==============================

==============================

## 破解说明

其实本软件的破解，crsky 很早以前就做过了，并提供了注册码:

name = crsky

sn = VFJ377-7LLHOE-14000

不过用别人的名字注册，总有种食人牙秽的感觉，^\_^，(crsky 别介意，玩笑)。

所以决定搞一个自己用户名的注册码出来。

程序加壳 - ASPack 2.12

保护方式 - 试用期+注册码

注册保存 - 注册表

保护算法 - MD5

破解目的：

熟悉 MD5 算法，尝试暴力破解

破解结果：

name = MrBeer\[CCG\]

sn = VFJ431-AA8UKY-51241

==============================

==============================

## 破解分析

1. 第一步是对程序进行查壳/脱壳，这个对于任何破解应该都是不可免除的一步。
    用 PEiD 检测主程序 `EZMerge.exe`，发现 `ASPack 2.12 -> Alexey Solodovnikov`。
    直接使用 PEiD Generic Unpacker 插件进行脱壳。（PS:本人手动只脱过 UPX，^\_^)

2. 第二步是侦测软件的加密算法，这一步大大减小了手动跟踪算法的强度。
    这也是我经过 SUNBIRD 指点，第一次使用 Kanal 工具进行侦测的 ^\_^。
    脱壳后的主程序为 `EZMerge.exe.unpacked\_.exe`，再用 PEiD 检测，用 Kanal 插件检测其中可能的加密算法。
    发现有 MD5 算法，初步认为写注册机是不大可能了。

3. 第三步是运行程序，进行注册，观察程序反应。
    在 `Register` 界面输入 FADE 用户名和注册码，分别为：
    `Name = MrBeer[CCG]`
    `SN = VFJ377-7LLHOE-14000`。
    注册后，程序提示为重启认证。
    开来我输入的 FADE 用户名和注册码不是被写入了注册表，就是被写入了 `KeyFile`。
    简单检查了一下程序所在文件夹，通过"时间日期"，没有发现最近修改或生成的文件。
    估计写入注册表的可能性大一些。
    搜索注册表，发现了 `HKEY_LOCAL_MACHINE\SOFTWARE\BoilSoft\EZMerge\SN` 键下有：
    `name=MrBeer[CCG]` 和 `sn=VFJ377-7LLHOE-14000` 两个值。果然！

4. 第四步是静态分析，这是我个人的习惯，有助于把握软件的整体构架，理清思路。
    我个人一直认为，破解的精髓是用运头脑来分析，而不仅仅是使用工具来分析、跟踪。
    动手之前多动脑，否则破解就变成了一项体力劳动了，呵呵。
    以前一直使用 W32Dasm 做静态分析，今天试用一下 PLL621 大侠的 C32ASM。
    虽然界面有些不习惯，但功能可真不是盖的，起码对中文字符串的支持要比 W32Dasm 和 OllyDbg 都强。
    不过既然是重启认证类型，就没必要仔细查看字符串了。
    直接在输入表中查找 `RegQueryValueA/RegQueryValueExA` 函数，共有 7 处调用。
    其他有用信息就没有了，呵呵，吹嘘了半天静态分析的习惯，这里没用上，^\_^。

5. 第五步就是最后的法宝，动态分析了。XP 系统下，当然是 OllyDbg 了。
    我使用的是 DYK 汉化、修改过的 ODbyDYK，功能更强大。
    通过对 `RegQueryValueA/RegQueryValueExA` 的 API 设断，很容易就跟踪到注册码验证部分。
    验证过程是（逻辑过程，有程序实际的实现过程有出入)：
    a. `name` 全部转换为小写字母；
    b. 验证 `sn` 结构，必须为：`xxxxxx-xxxxxx-xxxxx`。
    c. 对 `sn` 第 1 段进行 MD5 运算；
    d. 以 `sn` 第 1 段的 MD5 结果为常数，对 `name` 进行 MD5 运算；
    e. 对 `sn` 第 2、3 段进行运算；
    f. `sn` 第 1 段的 MD5 结果与程序内置的 256 个 MD5 值进行比较，必须属于其中之一；
    g. 对 `name` 各字符的 ASCII 码进行累加，以累加结果的低位字节为 `Index`，取出一个内置的 MD5 值；
    h. 比较该 MD5 值与 `sn` 第 1 段的 MD5 值，相同，则验证成功。
    因为本文目的主要时针对 MD5，其余验证部分不在此讨论了。

==============================

==============================

## 源码分析

通过对 `RegQueryValueA/RegQueryValueExA` 设断，很容易跟踪到这里：

```
00409FF0    .  E8 B>call EZMerge\_.004110B0             ;  --->从注册表中获得 sn
00409FF5    .  83C4>add esp,8
00409FF8    .  83CF>or edi,FFFFFFFF
00409FFB    .  85C0 test eax,eax
00409FFD    .  74 7>je short EZMerge\_.0040A075
00409FFF    .  8D8C>lea ecx,dword ptr ss:[esp+2E8]
0040A006    .  68 0>push 200
0040A00B    .  51   push ecx
0040A00C    .  E8 1>call EZMerge\_.00411130             ;  --->从注册表中获得 name
0040A011    .  83C4>add esp,8
0040A014    .  85C0 test eax,eax
0040A016    .  74 5>je short EZMerge\_.0040A075
0040A018    .  8D94>lea edx,dword ptr ss:[esp+2E8]
0040A01F    .  8D4C>lea ecx,dword ptr ss:[esp+8]
0040A023    .  52   push edx
0040A024    .  E8 8>call EZMerge\_.004635AB
0040A029    .  8D4C>lea ecx,dword ptr ss:[esp+8]
0040A02D    .  C784>mov dword ptr ss:[esp+5648],0
0040A038    .  E8 0>call EZMerge\_.00463A3F             ;  --->name 字符转换为小写
0040A03D    .  8B54>mov edx,dword ptr ss:[esp+8]
0040A041    .  8D44>lea eax,dword ptr ss:[esp+C]
0040A045    .  8D8C>lea ecx,dword ptr ss:[esp+E8]
0040A04C    .  50   push eax
0040A04D    .  51   push ecx
0040A04E    .  52   push edx
0040A04F    .  E8 0>call EZMerge\_.00410D60             ;  --->验证 sn（关键 CALL)
0040A054    .  83C4>add esp,0C
0040A057    .  85C0 test eax,eax
0040A059    .  74 0>je short EZMerge\_.0040A065         ;  --->不跳，验证成功
```

进入关键 CALL：

```
00410D60  /$  83EC>sub esp,0C                          ;  --->入口
00410D63  |.  8B44>mov eax,dword ptr ss:[esp+10]
........................................................
省略代码若干行
........................................................
00410DD7  |.  B9 0>mov ecx,0E
00410DDC  |.  33C0 xor eax,eax
00410DDE  |.  8BFD mov edi,ebp
00410DE0  |.  6A 2>push 2D                              ;  --->很明显，是查找"-"
00410DE2  |.  F3:A>rep stos dword ptr es:[edi]
00410DE4  |.  53   push ebx
00410DE5  |.  33FF xor edi,edi
00410DE7  |.  8BF3 mov esi,ebx
00410DE9  |.  E8 A>call EZMerge\_.0044DC90
00410DEE  |.  83C4>add esp,8
00410DF1  |.  85C0 test eax,eax
00410DF3  |.  74 3>je short EZMerge\_.00410E2C
00410DF5  |.  8B5C>mov ebx,dword ptr ss:[esp+20]
00410DF9  |>  47   /inc edi
00410DFA  |.  C600>|mov byte ptr ds:[eax],0
00410DFD  |.  8BCF |mov ecx,edi
00410DFF  |.  40   |inc eax
00410E00  |.  49   |dec ecx                              ;  Switch (cases 1..2)
00410E01  |.  74 0>|je short EZMerge\_.00410E10
00410E03  |.  49   |dec ecx
00410E04  |.  75 0>|jnz short EZMerge\_.00410E12
00410E06  |.  8974>|mov dword ptr ss:[esp+10],esi        ;  Case 2 of switch 00410E00
00410E0A  |.  8944>|mov dword ptr ss:[esp+28],eax
00410E0E  |.  EB 0>|jmp short EZMerge\_.00410E12
00410E10  |>  8BDE |mov ebx,esi                           ;  Case 1 of switch 00410E00
00410E12  |>  6A 2>|push 2D                              ;  Default case of switch 00410E00
00410E14  |.  50   |push eax
00410E15  |.  8BF0 |mov esi,eax
00410E17  |.  E8 7>|call EZMerge\_.0044DC90
00410E1C  |.  83C4>|add esp,8
00410E1F  |.  85C0 |test eax,eax
00410E21  |.^ 75 D>\\jnz short EZMerge\_.00410DF9
00410E23  |.  83FF>cmp edi,2                              ;  --->校验 sn 格式(必须为 xxxxxx-xxxxxx-xxxxx)
00410E26  |.  74 1>je short EZMerge\_.00410E3F            ;  --->跳
00410E28  |.  8B5C>mov ebx,dword ptr ss:[esp+24]
00410E2C  |>  53   push ebx
00410E2D  |.  E8 0>call EZMerge\_.00463B36
00410E32  |.  83C4>add esp,4
00410E35  |.  33C0 xor eax,eax
00410E37  |.  5F   pop edi
00410E38  |.  5E   pop esi
00410E39  |.  5D   pop ebp
00410E3A  |.  5B   pop ebx
00410E3B  |.  83C4>add esp,0C
00410E3E  |.  C3   retn
00410E3F  |>  803B>cmp byte ptr ds:[ebx],0               ;  --->来到这里
00410E42  |.  0F84>je EZMerge\_.00410F8F
00410E48  |.  8B44>mov eax,dword ptr ss:[esp+28]
00410E4C  |.  8038>cmp byte ptr ds:[eax],0
00410E4F  |.  0F84>je EZMerge\_.00410F8F
00410E55  |.  8B4C>mov ecx,dword ptr ss:[esp+10]
00410E59  |.  8039>cmp byte ptr ds:[ecx],0
00410E5C  |.  0F84>je EZMerge\_.00410F8F
00410E62  |.  8BFB mov edi,ebx
00410E64  |.  83C9>or ecx,FFFFFFFF
00410E67  |.  33C0 xor eax,eax
00410E69  |.  6A 0>push 0
00410E6B  |.  F2:A>repne scas byte ptr es:[edi]
00410E6D  |.  F7D1 not ecx
00410E6F  |.  49   dec ecx
00410E70  |.  51   push ecx
00410E71  |.  53   push ebx
00410E72  |.  E8 1>call EZMerge\_.00444090                ;  --->得到 sn 第一段的 MD5
00410E77  |.  50   push eax
00410E78  |.  E8 3>call EZMerge\_.00410FB0                ;  转换 MD5 值为字符串格式
00410E7D  |.  83C4>add esp,10
00410E80  |.  8944>mov dword ptr ss:[esp+14],eax
00410E84  |.  BF 9>mov edi,EZMerge\_.0048D990             ;  --->内置 MD5 所在空间
00410E89  |>  8B37 /mov esi,dword ptr ds:[edi]
00410E8B  |.  8B4C>|mov ecx,dword ptr ss:[esp+14]
00410E8F  |>  8A01 |/mov al,byte ptr ds:[ecx]
00410E91  |.  8AD0 ||mov dl,al
00410E93  |.  3A06 ||cmp al,byte ptr ds:[esi]
00410E95  |.  75 1>||jnz short EZMerge\_.00410EB3
```



```
00410E97   |.  84D2 ||test dl,dl  

00410E99   |.  74 1>||je short EZMerge\_.00410EAF  

00410E9B   |.  8A41>||mov al,byte ptr ds:\[ecx+1\]  

00410E9E   |.  8AD0 ||mov dl,al  

00410EA0   |.  3A46>||cmp al,byte ptr ds:\[esi+1\]          ;  --->比较 sn 第 1 段的 MD5 值是否属于程序内置的个 MD5 值之一  

00410EA3   |.  75 0>||jnz short EZMerge\_.00410EB3  

00410EA5   |.  83C1>||add ecx,2  

00410EA8   |.  83C6>||add esi,2  

00410EAB   |.  84D2 ||test dl,dl  

00410EAD   |.^ 75 E>||jnz short EZMerge\_.00410E8F  

00410EAF   |>  33C9 |xor ecx,ecx  

00410EB1   |.  EB 0>|jmp short EZMerge\_.00410EB8  

00410EB3   |>  1BC9 |sbb ecx,ecx  

00410EB5   |.  83D9>|sbb ecx,-1  

00410EB8   |>  85C9 |test ecx,ecx  

00410EBA   |.  74 2>|je short EZMerge\_.00410EE2            ;  --->跳！！！  

00410EBC   |.  83C7>|add edi,4  

00410EBF   |.  81FF>|cmp edi,EZMerge\_.0048DD90  

00410EC5   |.^ 7C C>\\jl short EZMerge\_.00410E89  

00410EC7   |.  8B44>mov eax,dword ptr ss:\[esp+24\]  

00410ECB   |.  8B74>mov esi,dword ptr ss:\[esp+18\]  

00410ECF   |.  50   push eax  

00410ED0   |.  E8 6>call EZMerge\_.00463B36  

00410ED5   |.  83C4>add esp,4  

00410ED8   |.  8BC6 mov eax,esi  

00410EDA   |.  5F   pop edi  

00410EDB   |.  5E   pop esi  

00410EDC   |.  5D   pop ebp  

00410EDD   |.  5B   pop ebx  

00410EDE   |.  83C4>add esp,0C  

00410EE1   |.  C3   retn  

00410EE2   |>  8BFB mov edi,ebx                               ;  --->来到这里  

........................................................  

省略代码若干行，并非不重要，而是因为与 MD5 无关  

........................................................  

00410F5B   |.  56   push esi  

00410F5C   |.  8945>mov dword ptr ss:\[ebp+28\],eax  

00410F5F   |.  8955>mov dword ptr ss:\[ebp+2C\],edx  

00410F62   |.  E8 B>call EZMerge\_.00410D20                   ;  --->name 各字符 ASCII 码相加，取地位字节  

00410F67   |.  55   push ebp  

00410F68   |.  8945>mov dword ptr ss:\[ebp+30\],eax  

00410F6B   |.  8955>mov dword ptr ss:\[ebp+34\],edx  

00410F6E   |.  E8 A>call EZMerge\_.00411020                   ;  --->比较 sn 第 1 段 MD5 与和 name 对应的 MD5  

00410F73   |.  83C4>add esp,28  

00410F76   |.  8BF0 mov esi,eax                               ;  --->eax=1  

00410F78   |.  8B44>mov eax,dword ptr ss:\[esp+24\]  

00410F7C   |.  50   push eax  

00410F7D   |.  E8 B>call EZMerge\_.00463B36  

00410F82   |.  83C4>add esp,4  

00410F85   |.  8BC6 mov eax,esi                                ;  --->eax=1  

00410F87   |.  5F   pop edi  

00410F88   |.  5E   pop esi  

00410F89   |.  5D   pop ebp  

00410F8A   |.  5B   pop ebx  

00410F8B   |.  83C4>add esp,0C  

00410F8E   |.  C3   retn  

00410F8F   |>  8B4C>mov ecx,dword ptr ss:\[esp+24\]  

00410F93   |.  51   push ecx  

00410F94   |.  E8 9>call EZMerge\_.00463B36  

00410F99   |.  83C4>add esp,4  

00410F9C   |>  5F   pop edi  

00410F9D   |.  5E   pop esi  

00410F9E   |.  5D   pop ebp  

00410F9F   |.  33C0 xor eax,eax  

00410FA1   |.  5B   pop ebx  

00410FA2   |.  83C4>add esp,0C  

00410FA5   \\ .  C3   retn  
```

\=============================

## KeyGen

因为程序内置的 255 个 MD5 值已知，MD5 的明文格式已知，我决定使用暴力破解（注意，不是爆破），生成注册机。

破解工具使用的是 `A!Die` 开发的 `MD5Crack V3.1`。

速度飞快，破解 255 个 MD5 明文共用时 1770 秒。

当然，这和明文长度有关。本软件所用明文仅 6 个字符。如果碰上超常的明文，这种暴力破解方式是不可取的。

根据破解出的 255 个明文即可写出注册机。

感觉 `AVI/MPEG/RM/WMV Joiner V4.61` 还是很不错的软件，这里就不提供它的注册机/源码了。

如果希望 FREE 使用，呵呵，请使用 `crsky` 或者我的注册信息吧。

\=============================

## Greets

感谢 `PEiD/C32ASM/OllyDbg` 及其插件的开发、修改者，没有他们的杰出工作也就没有本文。

感谢 `SUNBIRD` 对 `PDiD Kanal` 插件使用的指导。

感谢 `crsky` 提供的注册信息。

\=============================

`MrBeer[CCG]`

03/Jul/2005

\=============================

`4dd466d0e1e0cc123afbe504da2462ca`
