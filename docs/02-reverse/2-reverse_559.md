
# 如何自动和手动定位加密库函数

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-292.htm`](https://www.pediy.com/kssd/pediy08/pediy8-292.htm)

## 【题记】

1. 有些朋友问起这个，所以，我在这里写出自己的方法和大家一起探讨。如果谁有更好的方法，也希望能贴出来和大家分享。
2. 以 `miracl` 库为例。

## 【工具】

1. `IDA 5.0 Adv 5.0.0.879`
2. `CryptoSIG.v2.0`
3. `OllyDbg v1.10`
4. 我的第 14 个 `CrackMe_0014` 及其源代码 `RSA.c`
5. `PdriLl` 的 `KenGenMe nr.2`
6. 《加密与解密 II》配套光盘中 `chap06` 的一个例子 `RSA_CrackMe`

## 【自动定位】

以 `CrackMe_0014` 为例。我的建议是你自己先至少粗略分析一下这 3 个 `CrackMe`，以了解其加密流程，后面可能更好理解我试图要阐述的东西。

1. 把 `CryptoSIG.sig` 文件拷贝到 `IDA` 的 `sig` 目录下；
2. 用 `IDA` 载入程序，等待它分析完后在 `IDA View-A` 窗口内右键切换到 `Text View` 模式；
3. 按 `Shift + F5` 键调出 `List of applied library modules` 窗口；
4. 右键选择 `Apply new signature` 或者按 `Insert` 键弹出如下窗口
5. 选择 `CryptoSIG`，点击 `OK`

总共有 55 个函数被重新标识。

你可能留意到了 `HTBTeam` 小组，感谢他们为我们带来这么好的东东吧 J

6. 在 `IDA` 的 `File` 菜单中选择 `Produce file à Create MAP file...` 保存之；
7. 用 `Ollydbg` 载入程序，在菜单中选择 `插件àLoad MapàLoad Map File` 后，来到 `00401130` 看看。

这是经过 `IDA + CryptoSIG` 分析过后的关键子程序代码：

```
00401130  /$>sub     esp, 550
00401136  |.>push    ebx
00401137  |.>push    ebp
……
0040116D  |.>stos    word ptr es:[edi]
0040116F  |.>push    0
00401171  |.>push    64
00401173  |.>stos    byte ptr es:[edi]
00401174  |.>call    00401AB0
00401179  |.>mov     ebp, eax
……
004011CA  |.>stos    word ptr es:[edi]
004011CC  |.>mov     ebx, [<&USER32.GetDlgItemTextA>] ;  USER32.GetDlgItemTextA
004011D2  |.>add     esp, 8
004011D5  |.>stos    byte ptr es:[edi]
004011D6  |.>lea     eax, [esp+10]
004011DA  |.>push    0C9                         ; /Count = C9 (201.)
004011DF  |.>push    eax                         ; |Buffer
004011E0  |.>push    3E8                         ; |ControlID = 3E8 (1000.)
004011E5  |.>push    esi                         ; |hWnd
004011E6  |.>call    ebx                         ; \GetDlgItemTextA
……
004011FC  |.>lea     ecx, [esp+D8]
00401203  |.>push    0C9                         ; /Count = C9 (201.)
00401208  |.>push    ecx                         ; |Buffer
00401209  |.>push    3E9                         ; |ControlID = 3E9 (1001.)
0040120E  |.>push    esi                         ; |hWnd
0040120F  |.>call    ebx                         ; \GetDlgItemTextA
……
00401260  |>lea     edx, [esp+3F8]
00401267  |.>push    edx
00401268  |.>call    <_shs_init>
0040126D  |.>mov     al, [esp+14]
00401271  |.>add     esp, 4
……
00401298  |>lea     edx, [esp+1A0]
0040129F  |.>lea     eax, [esp+3F8]
004012A6  |.>push    edx
004012A7  |.>push    eax
004012A8  |.>call    <_shs_hash>
……
0040131C  |.>mov     byte ptr [esp+278], 0
00401324  |.>push    0
00401326  |.>mov     dword ptr [ebp+234], 10
00401330  |.>call    <_mirvar>
00401335  |.>push    0
00401337  |.>mov     esi, eax
00401339  |.>call    <_mirvar>
0040133E  |.>push    0
00401340  |.>mov     ebp, eax
00401342  |.>call    <_mirvar>
00401347  |.>push    0
00401349  |.>mov     edi, eax
0040134B  |.>call    <_mirvar>
00401350  |.>mov     ebx, eax
00401352  |.>lea     eax, [esp+E8]
00401359  |.>push    eax
0040135A  |.>push    edi
0040135B  |.>call    <_cinstr>
00401360  |.>push    0040D0DC          ; ASCII "6199855658D504EBC98DF20A2F170CD1"
00401365  |.>push    esi
00401366  |.>call    <_cinstr>
0040136B  |.>push    0040D0D4          ; ASCII "10001"
00401370  |.>push    ebp
00401371  |.>call    <_cinstr>
00401376  |.>push    esi
00401377  |.>push    edi
00401378  |.>call    <_compare>
0040137D  |.>add     esp, 30
00401380  |.>cmp     eax, -1
00401383  |.>jnz     short 004013E6
00401385  |.>push    ebx
00401386  |.>push    esi
00401387  |.>push    ebp
00401388  |.>push    edi
00401389  |.>call    <_powmod>
0040138E  |.>lea     ecx, [esp+340]
00401395  |.>push    0
00401397  |.>push    ecx
00401398  |.>push    ebx
00401399  |.>push    0
0040139B  |.>call    <_big_to_bytes>
004013A0  |.>push    esi
004013A1  |.>call    <_mirkill>
004013A6  |.>push    ebp
004013A7  |.>call    <_mirkill>
004013AC  |.>push    edi
004013AD  |.>call    <_mirkill>
004013B2  |.>push    ebx
004013B3  |.>call    <_mirkill>
004013B8  |.>add     esp, 30
004013BB  |.>call    00402280
004013C0  |.>lea     edx, [esp+330]
004013C7  |.>lea     eax, [esp+268]
004013CE  |.>push    edx                         ; /String2
004013CF  |.>push    eax                         ; |String1
004013D0  |.>call    [<&KERNEL32.lstrcmpA>]        ; \lstrcmpA
004013D6  |.>neg     eax
……
004013EC  |.>add     esp, 550
004013F2  \.>retn
```

蓝颜色的文字就是 `CryptoSIG` 帮助我们分析出来的（其中，`mir` 表示 `miracl` 库）；如果不用 `CryptoSIG`，那么该处只是显示调用某一地址，比如：`call 00403AB0`，这就是 `CryptoSIG` 用和不用的区别。

红颜色的文字是 `CryptoSIG` 没有帮助我们分析出来的一个库函数调用，在 `RSA.c` 源代码文件里对应 `miracl *mip = mirsys(100,0)` 这个语句，但 `miracl *mirsys(nd,nb)` 却是任何使用 `miracl` 库必须调用的一个函数，我们看看 `miracl` 手册对它的描述：

```
Initialise the MIRACL system for the current program thread, as described below. Must be called before attempting to use any other MIRACL routines.

(1) The error tracing mechanism is initialised.

(2) the number of computer words to use for each big/flash number is calculated from nd and nb.

(3) Sixteen big work variables (four of them double length) are initialised.

(4) Certain instance variables are given default initial values.

(5) The random number generator is started by calling irand(0L).
```

好了，我不再对这个函数自身进行过多的描述，更多的东西你可以参考 `miracl` 手册。

继续我们的红颜色文字。在 `IDA View-A` 窗口内双击 `00401174` 处 `call` 后面的 `sub_401AB0`，你看到了什么？如下图中的那个关键字 `_mr_first_alloc`，呵呵，这个函数所处的子程序对应于 `miracl *mirsys(nd,nb)` 函数。

**对自动定位的总结：**

自动定位关键是要加载正确的签名文件（`.sig`）。推荐使用的另一个文件包是 `BigLib Signature Collection 1.1`，它的 `FGint` 很不错（用于 `Delphi` 编写的程序），看雪学院密码学工具栏目底部有下载。

## 【手动定位】

**方法 1：** 对于手动定位，关键是要能识别出库函数反汇编后的代码，这与编译程序时所使用的编译器有关。下面以 `VC` 所带编译器为例；如果程序采用 `GCC` 或者其它编译器，可能会有所差别。

在这里我给出以下几个常用的 `miracl` 库函数的识别特征码（在 `OllyDbg` 中)：

`mirvar(iv)`，`cinstr(x,s)`，`powmod(x,y,z,w)`，`big_to_bytes(max,x,ptr,justify)`

以下以两个例子来说明，你可以和我的 `CrackMe` 相对比来检测。

1. `mirvar(iv)`：记住下面这些代码，以及这几个关键常数（`18`，`17`，`12`，`4`）即可基本确定是否是 `mirvar(iv)`。以 `PdriLl` 的 `KenGenMe nr.2` 为例：

```
004017B0  /$>push   esi
004017B1  |.>push   edi
004017B2  |.>call   00401390
004017B7  |.>mov    esi, eax
004017B9  |.>mov    eax, [esi+230]
004017BF  |.>test   eax, eax
004017C1  |.>je     short 004017C8             //这里有一个je跳转指向retn下一指令
004017C3  |.>pop    edi
004017C4  |.>xor    eax, eax
004017C6  |.>pop    esi
004017C7  |.>retn                               //这里有一个retn
004017C8  |>mov    edx, [esi+1C]
004017CB  |.>inc    edx
004017CC  |.>mov    eax, edx
004017CE  |.>mov    [esi+1C], edx
004017D1  |.>cmp    eax, 18
004017D4  |.>jge    short 004017ED
004017D6  |.>mov    dword ptr [esi+eax*4+20], 17
004017DE  |.>mov    eax, [esi+244]
004017E4  |.>test   eax, eax
004017E6  |.>je     short 004017ED
004017E8  |.>call   004015D0
004017ED  |>mov    eax, [esi+8C]
004017F3  |.>test   eax, eax
004017F5  |.>jnz    short 0040180D
004017F7  |.>push   12
004017F9  |.>call   004013A0
004017FE  |.>mov    eax, [esi+1C]
00401801  |.>add    esp, 4
```

2. 后面几个函数均以 `RSA_CrackMe` 为例

   `cinstr(x,s)`：关键常数（`18`，`4E`，`7FFFFFFF`，`OFFFF`，`10`）

```
004035E0 >/$>push   ebx
004035E1  |.>push   ebp
004035E2  |.>push   esi
004035E3  |.>push   edi
004035E4  |.>call   004012F0
004035E9  |.>mov    esi, eax
004035EB  |.>mov    eax, [esi+218]
004035F1  |.>test   eax, eax
004035F3  |.>jnz    004036B6
004035F9  |.>mov    edx, [esi+1C]
004035FC  |.>inc    edx
004035FD  |.>mov    eax, edx
004035FF  |.>mov    [esi+1C], edx
00403602  |.>cmp    eax, 18
00403605  |.>jge    short 0040361E
```



3. `powmod(x,y,z,w)`：关键常数（`18, 12....`）

请留意一下这个函数和 `cinstr` 函数的区别，很好识别的。

```
004033D0   /$>push    ebx
004033D1   |.>push    ebp
004033D2   |.>push    esi
004033D3   |.>push    edi
004033D4   |.>call    004012F0
004033D9   |.>mov     esi, eax
004033DB   |.>mov     eax, [esi+218]
004033E1   |.>test    eax, eax
004033E3   |.>jnz     004035D5
004033E9   |.>mov     edx, [esi+1C]
004033EC   |.>inc     edx
004033ED   |.>mov     eax, edx
004033EF   |.>mov     [esi+1C], edx
004033F2   |.>cmp     eax, 18
004033F5   |.>jge     short 0040340E
004033F7   |.>mov     dword ptr [esi+eax*4+20], 12  // 区别 cinstr 的常数
```

4. `big_to_bytes(max,x,ptr,justify)`：关键常数（`18, 8D, FFFFFFFC, 0C, 0E`）

```
00402E40   /$>sub     esp, 8
00402E43   |.>push    ebx
00402E44   |.>push    ebp
00402E45   |.>push    esi
00402E46   |.>push    edi
00402E47   |.>call    004012F0
00402E4C   |.>mov     ebp, eax
00402E4E   |.>mov     [esp+10], ebp
00402E52   |.>mov     eax, [ebp+218]
00402E58   |.>test    eax, eax
00402E5A   |.>jnz     00403062
00402E60   |.>mov     ebx, [esp+20]
00402E64   |.>push    ebx
00402E65   |.>call    00402460
00402E6A   |.>add     esp, 4
00402E6D   |.>test    eax, eax
00402E6F   |.>je      00403062
00402E75   |.>mov     edi, [esp+1C]
00402E79   |.>test    edi, edi
00402E7B   |.>jg      short 00402E89
00402E7D   |.>mov     eax, [esp+28]
00402E81   |.>test    eax, eax
00402E83   |.>jnz     00403062
00402E89   |.>mov     edx, [ebp+1C]
00402E8C   |.>inc     edx
00402E8D   |.>mov     eax, edx
00402E8F   |.>mov     [ebp+1C], edx
00402E92   |.>cmp     eax, 18
00402E95   |.>jge     short 00402EAE
00402E97   |.>mov     dword ptr [ebp+eax*4+20], 8D
00402E9F   |.>mov     eax, [ebp+22C]
00402EA5   |.>test    eax, eax
00402EA7   |.>je      short 00402EAE
00402EA9   |.>call    00401530
00402EAE   |.>push    ebx
00402EAF   |.>call    00402090
00402EB4   |.>mov     eax, [ebp]
00402EB7   |.>add     esp, 4
00402EBA   |.>test    eax, eax
00402EBC   |.>jnz     00402FA9
00402EC2   |.>mov     edx, [ebx]
00402EC4   |.>mov     eax, [ebx+4]
00402EC7   |.>and     edx, 7FFFFFFF
00402ECD   |.>xor     ebx, ebx
00402ECF   |.>dec     edx
00402ED0   |.>lea     esi, [edx*4]
00402ED7   |.>mov     eax, [esi+eax]
00402EDA   |.>test    eax, eax
00402EDC   |.>je      short 00402EE7
00402EDE   |.>inc     ebx
00402EDF   |.>shr     eax, 8
00402EE2   |.>inc     esi
00402EE3   |.>test    eax, eax
00402EE5   |.>jnz     short 00402EDE
00402EE7   |.>and     ebx, 80000003
00402EED   |.>jns     short 00402EF4
00402EEF   |.>dec     ebx
00402EF0   |.>or      ebx, FFFFFFFC  // 这个常数你在别的地方见过吗？
```
......
```
00402FFC   |.>push    eax
00402FFD   |.>push    100
00403002   |.>push    eax
00403003   |.>call    00402C80
00403008   |.>add     esp, 0C
```
......
```
00403046   |.>retn
00403047   |.>mov     eax, esi
00403049   |.>pop     edi
0040304A   |.>pop     esi
0040304B   |.>pop     ebp
0040304C   |.>pop     ebx
0040304D   |.>add     esp, 8
00403050   |.>retn
00403051   |.>push    0E  // 这个常数前有一个 retn
00403053   |.>call    00401300
00403058   |.>mov     eax, [ebp+1C]
0040305B   |.>add     esp, 4
0040305E   |.>dec     eax
0040305F   |.>mov     [ebp+1C], eax
00403062   |.>pop     edi
00403063   |.>pop     esi
00403064   |.>pop     ebp
00403065   |.>xor     eax, eax
00403067   |.>pop     ebx
00403068   |.>add     esp, 8
0040306B   |.>retn
```

**方法 2：** 一般说来，往往用连续用好几次 `mirvar(0)` 函数以同时初始化几个变量，如 `RSA.c`：

```
n=mirvar(0);
e=mirvar(0);
m=mirvar(0);
c=mirvar(0);
```

我们来看看在 `OD` 中的代码：

```
00401324   |.>push    0
00401326   |.>mov     dword ptr [ebp+234], 10  //IOBASE=16
00401330   |.>call    00401890  // mirvar(0)，可以用方法 1 来验证
00401335   |.>push    0
00401337   |.>mov     esi, eax
00401339   |.>call    00401890  // 这个函数连续用了4次
0040133E   |.>push    0
00401340   |.>mov     ebp, eax
00401342   |.>call    00401890
00401347   |.>push    0
00401349   |.>mov     edi, eax
0040134B   |.>call    00401890
00401350   |.>mov     ebx, eax
00401352   |.>lea     eax, [esp+E8]
00401359   |.>push    eax
0040135A   |.>push    edi
0040135B   |.>call    00403AB0  // 这个函数连续用了 3 次
00401360   |.>push    0040D0DC ;  ASCII "6199855658D504EBC98DF20A2F170CD1"
00401365   |.>push    esi
00401366   |.>call    00403AB0
0040136B   |.>push    0040D0D4 ;  ASCII "10001"
00401370   |.>push    ebp
00401371   |.>call    00403AB0
```

看见了那个长字符串了，干什么用的呢？当然是供 `cinstr(x,s)` 用的，这个函数的作用是把字符串 `s` 所对应的整数赋值给大整数 `x`。

还有一个就是 `mirkill(x)` 函数的连续调用，其作用是“Securely kills off a big/flash number by zeroising it, and freeing its memory.”，还是来看看它：

```
004013A0   |.>push    esi
004013A1   |.>call    00402260
004013A6   |.>push    ebp
004013A7   |.>call    00402260
004013AC   |.>push    edi
004013AD   |.>call    00402260
004013B2   |.>push    ebx
004013B3   |.>call    00402260
```

**对手动定位的总结：**

其实，这两种方法都不是充分条件，但如果这两种方法结合起来，同时这些数字又能恰好凑在一个个子程序中，那么会是不是某种偶然？我觉得可能性不大。当然，在调试的同时还要能够想起这些东东可真不是件容易的事，这可能就得靠意识和平时的多实践多积累了。

【一点疑问】

这种自动定位方法你可以在看雪论坛精华里面搜索“.sig”，找到相关的事例。而这种手动定位方法则完全是我自己无意中发现的，不知道[Ryosuke](http://bbs.pediy.com/member.php?s=&action=getinfo&userid=26845)通常是如何手动定位的。

【版权声明】 本文纯属技术交流，转载请注明作者及来自[看雪学院](http://www.pediy.com/)，并保持文章的完整性。

引用: *最初由 `happytown` 发布*

此文仅供探讨。

对于 `miracl` 库函数的识别，补充 2 点：

1、`miracl` 的 `api` 都很有特点，就是主要的 `api` 中基本都有这么一句：

`mov dword ptr [ebp+eax*4+20], xx`

其中 `xx` 是一个不同的 16 进制数，每一个 `api` 都不同，但所有版本的相同的 `api` 这个数都是相同的。

BlackEye 大侠曾经写过一片小文，关于用这种方法识别 `miracl` 的 `api`。

等我有空找到那片原文，我把它贴上来。

2、用 `miracl` 的 `C++` wrapper 写的 `crackme`，用 `ida` ＋ `sig` 的方法可能识别不出来，这个可能就需要手动了。最好自己写一个类似的程序、编译、反汇编然后对比着看。

`ror` 的 `crackme` 就是一个例子。

一直都想写一个类似的帖子，却总是懒于动手，让楼主抢先了，呵呵

4dd466d0e1e0cc123afbe504da2462ca
