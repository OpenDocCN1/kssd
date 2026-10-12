
# 答复

> 原文：[`www.pediy.com/kssd/pediy03/forum957.htm`](https://www.pediy.com/kssd/pediy03/forum957.htm)

## 超级解霸 2000 试用版的破解

平常也不怎么看 VCD，但买了电脑报 2000 年合订本，配套光盘里有超级解霸 2000 试用版，就随手安装了，顺便练练手。

工具：`WinIce4.05`、`W32DASM`、`Hiew6.4` 和一只笔、纸若干。

试用版在使用 30 次后会弹出过期报错窗口，运行次数记录在 `WINDOWS` 文件夹下的 `STHSVCD.INI` 配置文件里，`BERUN`=运行次数。可以想象，每次启动 `STHSVCD`，它都先要读取 `BERUN` 项，和 30 比较，小于的话，则该项加 1，写入 `STHSVCD.INI` 并转入主程序执行，否则报错。那么可以下断 `GetPrivateProfileStringA` 或者下断 `WritePrivateProfileStringA`，考虑到 `STHSVCD` 启动时要设置多项参数，可能多次调用 `GetPrivateProfileStringA` 可能多次被调用，不易追踪，最好利用 `WritePrivateProfileStringA` 设断。

`bpx writeprivateprofilestringa`

运行 `STHSVCD.EXE`，果然...

按 `F12`

```
00414767   push 00426724   ->"STHVCD.INI"
0041476C   push eax
0041476D   push 00428C9C   ->"BERUN"
00414772   push 00426710   ->"SETTING"
00414777   call dword ptr [004A0670]  调用 WritePrivateProfileStringA
0041477D   cmp ebx, 0000001E  <-中断返回此处，1E 即 30
00414780   jle 004147A4    不超过 30 次，则跳转，否则
00414782   call 0040FAF0   弹出过期报错窗口
```

向上看

```
00414732    push 00426724   ->"STHVCD.INI"
00414737    push 00000001
00414739    push 00428C9C   ->"BERUN"
0041473E    push 00426710   ->"SETTING"
00414743    Call dword ptr [004A06F8]  调用 GetPrivateProfileIntA
00414749    lea ebx,[eax+1]   将已使用次数加 1，送入 EBX
0041474C    lea eax, dword ptr [esp+78]
```

Crack 方案有两个：

1、`jle 004147A4` 改为 `jmp 004147A4`；
2、取消 `EBX` 加 1 的操作，`lea ebx,[eax+1]` 改为三个 `NOP`。

具体修改过程就不用我再一一罗嗦了吧。

慢着，别以为万事大吉，还有一个陷阱呢！`STHSVCD` 还在 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Setup` 处设置键值 `RunTime`，保存剩余的运行次数，每次运行 `STHSVCD`，`RunTime` 减 1，当 `RunTime` 为 0 时，报错。根据已有的破解成果表明，将此值改为 `FFFFFFFF` 后，再运行 `STHSVCD` 该值将保持不变。确实这里挺难追，用 `DialogBoxParamA`、`RegQueryValueExA` 等均未成功。最后只有请出“依天宝剑”`W32DASM`，虽然反汇编时报错，但不睬它，继续，发现和读取注册表有关的也就一个 `RegQueryValueExA`，查找 `RegQueryValueExA` 字符串，在程序中发现一处调用。

```
0040FBF9    call dword ptr [004A0590]  调用 RegQueryValueExA
0040FBFF    test eax, eax
0040FC01    je 0040FC28  返回值为 0 则跳转，出错
```

试着把 `je 0040FC28` 屏蔽(改为两个 `NOP`，或者 `INC EAX`，`DEC EAX`)，`OK`！无论怎么运行 `STHSVCD`，`RunTime` 将保持 `0x63` 即 99 不变，永远也不会过期了。

用同样的手段修改音频解霸。

好了，今天就说到这儿，还是让我们举起双手拥护正版吧！

南阳 `wxd` 2001-1-5

`E-mail:wxd125@163.net`

`Oicq:21331694`

## 再谈《超级解霸 2000 试用版的破解》

这是对《超级解霸 2000 试用版的破解》的修正版，介绍的较详细，很适合新手入门观摩

前面我写了《超级解霸 2000 试用版的破解》，觉得已经成功了，但后来在解“虚拟光驱 2000”时，把日期向后调了一个月，无意中运行超级解霸，讨厌的过期报错窗口又跳了出来，我靠！还有一个陷阱！！

于是又用 `W32asm` 详细分析了原代码，果然是三道关卡，嘿嘿，国产软件确也费尽心机。运行试用版，先后要通过“`BERUN`”30 次运行次数的检查、30 天运行期限的检查和注册表 64 次运行次数的检查。你可以用“串数据参考”查找串 `"The Time is over !Please Get one Gold Version !"` 找到过期报错窗口代码，如下：

### 突破口

`Referenced by a CALL at Addresses:`

```
:00414782   , :004147E4   , :0041480F

:0040FAF0 81ECD4000000            sub esp, 000000D4
:0040FAF6 8D442454              lea eax, dword ptr [esp+54]
:0040FAFA 53                    push ebx
:0040FAFB 56                    push esi
:0040FAFC 8B0D9C874200          mov ecx, dword ptr [0042879C]
* Possible Reference to Dialog: DialogID_0080
                              |
:0040FB02 6880000000            push 00000080
* Reference To: USER32.LoadStringA, Ord:01A9h
                              |
:0040FB07 8B35D8084A00          mov esi, dword ptr [004A08D8]
:0040FB0D C744240CD2040000      mov [esp+0C], 000004D2
:0040FB15 50                    push eax
* Possible Reference to String Resource ID=50061: "The Time is over !
Please Get one Gold
Version !"
                              |
:0040FB16 688DC30000            push 0000C38D
:0040FB1B 51                    push ecx
:0040FB1C FFD6                  call esi
```

### 突破口

可以看到有三处 `00414782`、`004147E4`、`0041480F` 调用了这段代码，那么一切都昭然若解了。且看着我沿着程序流程娓娓道来：

### 突破防线

```
:004146E8 8D442478              lea eax, dword ptr [esp+78] ；指定返回项字串的缓冲区
* Possible StringData Ref from Data Obj ->"STHVCD.INI" ；初始化文件的名字
                              |
:004146EC 6824674200            push 00426724
* Possible Reference to Dialog: DialogID_0080
                              |
:004146F1 6880000000            push 00000080 ；缓冲区的最大字符数量
:004146F6 50                    push eax ；指定返回项字串的缓冲区
:004146F7 68D0664200            push 004266D0 ；指定的项没有找到时返回的默认值，此处为 NULL
* Possible StringData Ref from Data Obj ->"SOURCEPATH"
                              |
:004146FC 68A48C4200            push 00428CA4 ；欲获取的项名
* Possible StringData Ref from Data Obj ->"INSTALL" ；欲在其中查找项的小节名称
                              |
:00414701 68D0694200            push 004269D0
* Reference To: KERNEL32.GetPrivateProfileStringA, Ord:019Fh
                              |
:00414706 FF1564064A00          Call dword ptr [004A0664]
:0041470C 85C0                  test eax, eax
:0041470E 7422                  je 00414732 ；复制到缓冲区的字节数量为 0，即 STHVCD.INI 无该项
:00414710 0FBE442478            movsx eax, byte ptr [esp+78] ；复制缓冲区的第一个字节到 EAX
:00414715 83F861                cmp eax, 00000061 ；和“A”比较
:00414718 7C05                  jl 0041471F
:0041471A 83F87A                cmp eax, 0000007A ；和“Z”比较
:0041471D 7E0D                  jle 0041472C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00414718(C)                     ；猜测：如果从光盘安装解霸 2000，
|                                 ；SOURCEPATH 的第一个字符被设置为
:0041471F 8A44247B              mov al, byte ptr [esp+7B]  ；小写字母，并且如果第四个字符为
:00414723 84C0                  test al, al                ；NULL(即根目录)时，如 e: \\，则跳
:00414725 B801000000            mov eax, 00000001          ；过 BERUN 值的检查，到日期检测处。
:0041472A 7402                  je 0041472E                ；如果从光盘安装，第一个字符被设
                                                    ；置为大写，需检查 BERUN 值
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041471D(C)
|
:0041472C 33C0                  xor eax, eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041472A(C)
|
:0041472E 85C0                  test eax, eax
:00414730 7572                  jne 004147A4 ；跳到到日期检测处 1.1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041470E(C)
|
* Possible StringData Ref from Data Obj ->"STHVCD.INI"
                              |
:00414732 6824674200            push 00426724
:00414737 6A01                  push 00000001
* Possible StringData Ref from Data Obj ->"BERUN"
                              |
:00414739 689C8C4200            push 00428C9C
* Possible StringData Ref from Data Obj ->"SETTING"
                              |
:0041473E 6810674200            push 00426710
* Reference To: KERNEL32.GetPrivateProfileIntA, Ord:0199h
                              |
:00414743 FF15F8064A00          Call dword ptr [004A06F8]
:00414749 8D5801                lea ebx, dword ptr [eax+01] ；将已使用次数加 1，送入 EBX
:0041474C 8D442478              lea eax, dword ptr [esp+78]
:00414750 53                    push ebx
* Possible StringData Ref from Data Obj ->"%d"
                              |
:00414751 68F8664200            push 004266F8
:00414756 50                    push eax
* Reference To: USER32.wsprintfA, Ord:029Fh
                              |
:00414757 FF15E4084A00          Call dword ptr [004A08E4]
:0041475D 8D842484000000        lea eax, dword ptr [esp+00000084]
:00414764 83C40C                add esp, 0000000C
* Possible StringData Ref from Data Obj ->"STHVCD.INI"
                              |
:00414767 6824674200            push 00426724
```
