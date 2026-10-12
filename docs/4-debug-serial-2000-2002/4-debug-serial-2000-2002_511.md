
# Transoft's Server All 1.02

> 原文：[`www.pediy.com/kssd/pediy03/forum515.htm`](https://www.pediy.com/kssd/pediy03/forum515.htm)

初学者的东西：Transoft's Server All 1.02 破解

——Passion 在新世纪的第一个破解

以前用过一个叫 Transoft's Server All 的服务器软件，华军下的，能将 `WINNT` 下的任意程序转换成 `NT` 的服务，也就是不需要登录就能自动运行。这号功能听起来不错，只是仍然要注册，于是忍不住又想练练手。——高手可别笑话我这个初学者。

说实话 Transoft's Server All 1.02 在高手眼里只不过是小菜，但我对付它却花了两个多小时，时间就花在寻找用我输入的用户名生成注册码的过程。开始原想分析出注册码计算的算法的，可似乎太复杂，跟了几百个语句后放弃了。早知道这样，不如在内存中寻找假注册码再设内存读写断点，那样八成更快些。

还是老手段，点菜单“Apply Registration Code”弹出注册框，要输入 `name`、`userkey` 和 `Email address`，于是输入 `Passion/1212121212121212/passion@email.com.cn`（不过后来这个 `Email` 地址根本就没用到）。下断 `bpx hmemcpy`，断后回 Server All 领空，`RET` 大概七层后来到此处：

```
:00478C95 E86276FAFF               call 004202FC

//运行到这里，上边这个 CALL 是读取 userkey 的。

:00478C9A 8B45F8                   mov eax, dword ptr [ebp-08]
:00478C9D 50                       push eax
:00478C9E 8D55F4                   lea edx, dword ptr [ebp-0C]
:00478CA1 8B45FC                   mov eax, dword ptr [ebp-04]
:00478CA4 8B80FC010000             mov eax, dword ptr [eax+000001FC]
:00478CAA E84D76FAFF               call 004202FC

//这个 CALL 读取 NAME

:00478CAF 8B45F4                   mov eax, dword ptr [ebp-0C]
:00478CB2 5A                       pop edx
:00478CB3 E850FFFFFF               call 00478C08

//这里肯定是生成注册码并判断的，判断结果在 AL 中。

:00478CB8 84C0                     test al, al
:00478CBA 0F8474030000             je 00479034   //如果跳就完了。
```

我以前把这里的 `0F8474030000` 全用 `90` 来 `NOP` 掉也注册成功了，不过这里要找出注册码，因此在此处按 `F8` 进入：

```
:00478CB3 E850FFFFFF               call 00478C08
```

进入后程序段变得很复杂，不过前边的逻辑还是稍微有些清楚，首先将输入的 `name` 按字符分开，按 `ASCII` 码进行迭代运算（许多乘法和浮点指令弄得我头晕脑胀的）。后来也不知怎么回事就出现了字符串“80219520”，幸亏觉得不对头，因为刚跟那么一点时间就出来注册码，那算法也太不成器了。再跟，又出来一个字符串“10027440”。——当然还得跟。

再跟下去，这两个字符串被作为参数参与了下一轮运算。并且运行到 `00466757` 处出现字符表，估计是计算出来的值通过查表的方式生成对应字符。

具体的流程复杂得很，注册码计算至少经过了三大步骤，迭代运算我也搞不清有多少次，因此代码没法贴出来。

```
:004666CB E810D6F9FF               call 00403CE0

//此处 D EAX 可看到第一个中间结果——字符 80219520

:00466757 E888D7F9FF               call 00403EE4

//此处 D EAX 可看到字符表

:0046676A E871D5F9FF               call 00403CE0

//此处 D EAX 可看到第二个中间结果——字符 10027440
```

最后，在 `00466779` 上下 `D EAX`，可看见最终的注册码。

——这个地方对于我来说真不容易找，几乎每一个 `CALL` 我都跟进去了。

```
:00466779 E87ED3F9FF               call 00403AFC

//这里注册码是 JNZVh，一般长度都在五六个左右。
```

所以我注册时用的就是：

Name: `Passion`

Userkey: `JNZVh`

Email Address: 随便

填入自己的 `Name` 后在 `0167:00466779` 处中断时，来一个 `D EAX` 可看见与自己的注册名对应的、属于自己的注册码。（相信大家都喜欢得到这种结果吧？）

另外我怀疑这个程序是用 `DELPHI` 写的，因为凡是碰到取字符串长度的子程序，都是在 `EBP-04` 还是类似的其他地方取一个值到 `EAX` 里再 `RET` 回去，这是 `Pascal` 的风格，跟 `C` 的字符串遍历的方法不一样。

话说二十一世纪的某年某月某日，Passion 偶然把系统时钟调后了一年，不料运行某软件时连续弹出过期提示共三百六十五次，Passion 给气得七窍生烟，马上动手 `Crack`。开始 `Win32 Dasm` 反汇编，却找不到执行入口点，`Program entry point` 的菜单也是灰色，而且代码开始处的指令也不是通常的 `push ebp` 之类的。Passion 于是心中发虚，“反汇编手段这么高？”

——用 `FileInfo` 等东西看，没有丝毫结果。Passion 于是更发虚：“加壳手段也这么高？”

——用 `SoftIce Loader` 载入，开始倒是能执行，运行一段代码后，居然出现一个 `int 3`，按 `F10` 便直接启动软件，跳过后又碰到一个 `REPZ MOVSB`，执行便同样直接启动。Passion 开始冒冷汗了：“反跟踪手段也太高了吧？”

——于是“两股战战，几欲先走。”

过了数日，Passion 又想起这事，越想越不对劲，可百思不得其解。最后碰上了 `NORTON ANTIVIRUS`……，NORTON 老先生告诉他说：“……WIN95.CIH……”

——“砰！”

Passion 当场晕倒。

…… 4dd466d0e1e0cc123afbe504da2462ca
