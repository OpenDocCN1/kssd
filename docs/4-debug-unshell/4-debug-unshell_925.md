
# API Spy

> 原文：[`www.pediy.com/kssd/pediy03/forum1112.htm`](https://www.pediy.com/kssd/pediy03/forum1112.htm)

破解 `API Spy` for `Windows` 95/98/NT/2000 《=写得不好，初学者看一看吧

破解人：4

日期：七月二日

`API32spy` 是一个非常有用的破解工具！它可以用来侦测系统调用了那些函数，帮助我们决定下什么断点。

下载：`http://madmat.hypermart.net/apis3225.exe`

`http://apis32.virtualave.net/soft/apis3225.exe`

`http://skyscraper.fortunecity.com/emacs/859/soft/apis3225.exe`

首先，用 `Fileinfor2.45`(`FI.exe`) 检查到 `APISpy2.5`(`Apis32.EXE`) 是用 `Petite1.2` 加密。

使用 `Procdump1.62` 脱壳它的壳吧。不是吧？ERROR！错误操作？

运行 `Blast Wave 2000` v0.2（`Blast Wave 2000` 是一个 `windows` 下的脱壳辅助工具。

它能轻易的找到任何加密壳的入口点.包括 `ASProtect` 以及幻影的加密壳.）

按 `Trace` 键，运行 `Apis32.EXE`. 得到 `Entry point: 00406360`

用 `w32dasm` 打开 `Apis32.EXE`，反编译后, 按 `Find Text` 键,打入 `Entry point` 地址：`00406360`

找到: `004????? E989CDFEFF` `JMP 00406360`

哈，写下 `E989CDFEFF` 吧。

不如再一次使用 `Procdump` 来脱壳它的壳吧？就写一个新的 `Script` 来脱壳 `Petite1.2` 的壳。

打开 `Procdump` 的 `Script.ini`

```
\[INDEX\]

.

.

.

P1B=VGCrypt 0.75

P1C=Aspack108.4

P1D=Aspack2000

P1E=Petite1.2    《=这里加入新的脱壳名称

; script by GustawKit

.

.

.

加入新的脱壳资料

\[Petite1.2\]               《=新的脱壳名称

L1=LOOK E9,89,CD,FE,FF    《=先前写下的 E989CDFEFF

L2=BP                     《=如果发现断点

L3=STEP                   《=就进行一步一步的解析操作，解压以后，保存文件

OPTL1=00000000

OPTL2=01000001

OPTL3=01010001

OPTL4=00030000

OPTL5=00000000
```

运行 `Procdump1.62`，选择 `Unpack=》` 选择 `Petite1.2=》` 选择 `Apis32.EXE`

等待 `Apis32.EXE` 的信息窗口弹出，再按 `Procdump1.62` 的 `OK` 键，开始脱壳。成功了。

然后

用 `w32dasm` 打开 `Apis32.EXE`，反编译后, 按 `String Data References` 键。

找那个弹出窗口的信息 `"This copy of APIS32 is UN R E G I S T E R E D"`

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00402EA5(C)

|

\* Possible StringData Ref from Data Obj ->"This copy of APIS32 is U "

                                    ->"N R E G I S T E R E D" "

                                    |

:00402EA9 BF10A24000              mov edi, 0040A210

:00402EAE BA20BD4000              mov edx, 0040BD20

:00402EB3 83C9FF                  or ecx, FFFFFFFF

:00402EB6 33C0                    xor eax, eax
```

`\* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:00402491(U)`

`|`

`:00402494 0AC0` `or al, al` `《=修改这儿。or al,01（修改 0AC0=>0c01）`

`:00402496 7402` `je 0040249A` `《=如果 al=0,就跳（一跳就玩完了）`

`:00402498 EB09` `jmp 004024A3`

果然弹出窗口不见了。

下一步，修改注册。找那个弹出窗口的信息 `"The registration information you.... "`

`\* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:004018B9(U)`

`|`

`:004018BC 0AC0` `or al, al` `《=修改这儿。or al,01（修改 0AC0=>0c01）`

`:004018BE 7402` `je 004018C2` `《=如果 al=0,就跳（一跳就玩完了）`

`:004018C0 EB2C` `jmp 004018EE`

`\* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:004018BE(C)`

`|`

`\* Possible StringData Ref from Data Obj ->"The registration information you "`
`->"provided is incorrect. Please verify "`
`->"that you entered your name and "`
`->"code properly, and try again. If "`
`->"you encounter difficulties, please "`
`->"send mail to APIS32@Biosys.net or "`
`->"visit our web site http://madmat.hypermart.net"`
`|`

```
:004018C2 BFFCA04000              mov edi, 0040A0FC

:004018C7 BA80C74000              mov edx, 0040C780

:004018CC 83C9FF                  or ecx, FFFFFFFF

:004018CF 33C0                    xor eax, eax

:004018D1 F2                      repnz

:004018D2 AE                      scasb
```

怎么啦？这个程序的作者老是使用这个指令去判断是否注册啊。

`or al, al`

`je 004018C2` `《=如果 al=0，就跳（一跳就玩完了）`

算了，把所有的 `or al, al` 更换成 `or al,01` 吧。

使用 `Hex workshop` 的 `Replace` 代替功能把所有 `0AC0` 换成 `0c01` （大约十个左右）

OK，破解 `API Spy` for `Windows` 95/98/NT/2000 成功。 `4dd466d0e1e0cc123afbe504da2462ca`
