
# EMEDITOR V3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum510.htm`](https://www.pediy.com/kssd/pediy03/forum510.htm)

破解 `EMEDITOR v3.0`

小弟第一次写过程，有不对的地方大家指正~~~~呵呵~~

`EMEDITOR 3.0` 是一个完全可以代替 `WINDOWS` 下 `NOTEBOOK` 的小玩意~~~~

这东西使用 30 天就需要注册，现将注册过程写一下

这个软件在我的主页有下载 `http://zop.yeah.net/download/crem.zip` 660K 左右

我使用的工具：`TRW2000`+`W32DASM89`+`ULTRAEDIT`

安装完 `EMEDITOR` 后运行并选说明中的关于，关于注册信息点那个输入序列号：

这时弹出一对话窗口，需要填一个分为四部分的序列号，我们设这四个部分为 `X1`，`X2`，`X3`，`X4`

我第一次填的是 `X1`=1234，`X2`=4321，`X3`=1324，`X4`=1423

运行 `TRW2000` 下 `BPX HMEMCPY`（小弟暂时只会用这个断点~~~~呵呵~~~~~）

顺利拦到，我们 `PMODULE` 程序蹦到 `417A01` 处，见下：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00417A0A(C)

|

:004179EF 53 push ebx

:004179F0 8D86F2030000 lea eax, dword ptr [esi+000003F2]

:004179F6 53 push ebx

:004179F7 50 push eax

:004179F8 FF7508 push [ebp+08]

\* Reference To: `USER32.GetDlgItemInt`, Ord:0103h

|

:004179FB FF1588434300 Call dword ptr [00434388]

:00417A01 668907 mov word ptr [edi], ax ---->就是蹦到这里了~~呵呵，看一眼 `EAX`

:00417A04 46 inc esi |，里面装的就是我们的 `X1`，往下走……

:00417A05 47 inc edi |

:00417A06 47 inc edi |

:00417A07 83FE04 cmp esi, 00000004 | 这里比较 `ESI` 和 4 如果小于则往上蹦（就是

:00417A0A 7CE3 jl 004179EF | 上面那段代码~~~直到我们序列号的四个部分

:00417A0C 8D45F4 lea eax, dword ptr [ebp-0C] | 完全存进 `EBP-C` 中（不知道这么说合不合适~）

:00417A0F 50 push eax |

:00417A10 E8F5FFFF call 0041790A | 呵呵~~~~~一个 CALL，看看底下，一个对比，如果

:00417A15 3BC3 cmp eax, ebx | 是，怎么怎么样，不是又怎么怎么样，很显然这是

:00417A17 5F pop edi ----> 关键 CALL，我们按 F8 跟进去 HAVE A LOOK ~~~~~

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\* Referenced by a CALL at Addresses:

|:00417A10 , :00417B90

|

:0041790A 56 push esi --->这段代码比较作用是

:0041790B 8B742408 mov esi, dword ptr [esp+08] 将我们输入的 `X1`，即 1234 除以 10 后

:0041790F 57 push edi 和 171 比较，171 哪来得？就是那个

:00417910 6A0A push 0000000A AB 转换成十进制得到的，如果是呢

:00417912 0FB706 movzx eax, word ptr [esi] 就继续，不是的话~~~~嘿嘿嘿嘿~~

:00417915 99 cdq 你就等着出序列号错误吧~~~由于我

:00417916 59 pop ecx 们输入的是 1234，不符合要求，所以

:00417917 F7F9 idiv ecx 这时我们对 `X1` 进行第一次修正，现在

:00417919 3DAB000000 cmp eax, 000000AB 新的 `X1`=171x 其中“x”为任意数，

:0041791E 7405 je 00417925 我的是 `X1`=1710，接着往下走~~~~~~

:00417920 6A01 push 00000001 走…………………………………………

:00417922 58 pop eax 走……………………………………

:00417923 EB15 jmp 0041793A ---->走………………………………

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: 走…………………………

|:0041791E(C)

|

:00417925 668B7E06 mov di, word ptr [esi+06] 走到此处，程序将我们的 `X4` 移到 `DI` 中

:00417929 56 push esi 走……………………

:0041792A E834FFFFFF call 00417863 走到此处又有一个 CALL，不要着急，看看

:0041792F 85C0 test eax, eax 下面，呵呵~~~~又是一个关键 CALL 我们跟

:00417931 7507 jne 0041793A 进去瞧瞧~~~~~~~~

###################################################################################################################

\* Referenced by a CALL at Address:

|:0041792A

|

:00417863 56 push esi ---->到目前 `ESI` 中存放的还是我们输入的序列号

:00417864 8B742408 mov esi, dword ptr [esp+08] 走………………………………………………

:00417868 668B4602 mov ax, word ptr [esi+02] 这行将我们的 `X2` 移入 `AX` 中

:0041786C 6683660600 and word ptr [esi+06], 0000

:00417871 663D0F27 cmp ax, 270F 比较 `X2` 和 9999

:00417875 0F8788000000 ja 00417903 大于的话蹦到 417903，不过不大可能大于

:0041787B 668B4E04 mov cx, word ptr [esi+04] 这行将我们的 `X3` 移入 `CX` 中

:0041787F 6681F90F27 cmp cx, 270F 比较 `X2` 和 9999

:00417884 777D ja 00417903 大于的话蹦到 417903，不过不大可能大于

:00417886 6685C0 test ax, ax

:00417889 7474 je 004178FF

:0041788B 663DAE08 cmp ax, 08AE 比较 `X2` 和 2222，`X2` 和 5678，我们的 `X2`=4321

:0041788F 746E je 004178FF 没有蹦~~后来发现蹦了就 DIE 了

:00417891 663D2E16 cmp ax, 162E

:00417895 7468 je 004178FF

:00417897 53 push ebx

:00417898 55 push ebp

:00417899 57 push edi

:0041789A 6A0A push 0000000A 下面会提到这个 A

:0041789C 0FB7F9 movzx edi, cx 把 `X3` 放入 `EDI`

:0041789F 0FB70E movzx ecx, word ptr [esi] 把 `X1` 放入 `ECX`

:004178A2 0FB7C0 movzx eax, ax

:004178A5 89442418 mov dword ptr [esp+18], eax 把 `X2` 放入 `ESP+18` 中

:004178A9 8BC1 mov eax, ecx 把 `X1` 放入 `EAX` 中

:004178AB 99 cdq

:004178AC 5B pop ebx 把上面那个 A 放到 `EBX` 中

:004178AD F7FB idiv ebx 用 `X1`/10 1710÷10=171 整数放入 `EAX`，余数放入 `EDX`

:004178AF 6A64 push 00000064 下面会提到这个 64

:004178B1 5D pop ebp

:004178B2 55 push ebp

:004178B3 8BD8 mov ebx, eax 把结果的整数放入 `EBX`

:004178B5 8BC7 mov eax, edi 把 `X3` 放入 `EAX` 中

:004178B7 99 cdq

:004178B8 F7FD idiv ebp 用 `X3` 除以 100（就是上面那个 64）

:004178BA 8B542418 mov edx, dword ptr [esp+18] 把 `X2` 放入 `EDX` 中

:004178BE 03D3 add edx, ebx 把 `X2`+（`X1`/10 的整数）放入 `EDX`

:004178C0 03C2 add eax, edx 把 `X2`+（`X1`/10 的整数）+（`X3`/100 的整数）放入 `EAX`

:004178C2 03C7 add eax, edi 用 `X2`+（`X1`/10 的整数）+（`X3`/100 的整数）+`X3` 放入

:004178C4 5F pop edi `EAX` 中

:004178C5 99 cdq 用 `X2`+（`X1`/10 的整数）+（`X3`/100 的整数）+`X3` 除以

:004178C6 F7FF idiv edi 100，整数放入 `EAX`，余数放入 `EDX`
