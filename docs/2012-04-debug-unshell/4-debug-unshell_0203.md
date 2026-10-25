
# Virtual Personality 4.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6347.htm`](https://www.pediy.com/kssd/pediy06/pediy6347.htm)

程序名: `Virtual Personality 4.0`

**※软件简介※**: A professional telephone voice changer utility. High quality natural voice.

Works with your PC speakers as well as over a telephone line. Needs only a standard voice

modem. 24 different voices. Supports voice recording of the PC (or any standard phone

device on the same line). Supports Windows 95/98/Me/NT/2000/Xp…………

软件语言：英文

软件类别：国外软件/音频处理

运行环境：Windows 98/NT/2000/ME/XP

软件大小：2.24M

软件作者：Triplebit

下载页面：`http://www.sharewareorder.com/Virtual-Personality-download-21449.htm`

**※软件限制※**: 采用了 `ARMADILLO` 的 `KEY` 加密，启动要你输入 `KEY`，还有 30 天试用期，所以，如果脱了壳，上述现象都将没有了。现在开始脱壳历程。

**※破解工具※**: `DriverSuite2.7` 中的 `softice4.2.7`、`PEiD`、`LordPE`

**※调试平台※**: Windows XP

`Armadillo` 是当今猛壳之一，加壳方式有两种，一种是标准方式(`Standard Protections Only`)加壳，另一种是 `CopyMem-II+Debug-Blocker` 的加壳，其标准加壳方式相对来说则容易的多。用 `COPYMEMII` 的一般可用 `DILLODUMP` 脱，而用标准加壳用 `DILLODUMP` 是脱不了的。

首先确信你已经牢固的理解了 `PE` 文件格式，因为那是重建 `PE` 映象的基础，并且清楚 `Armadillo` 保护的理论和方法。

**理论**: 首先一点是 `Armadillo` 是通过类似于调试的方法来运行程序的。新版本 `Armadillo` 脱壳的一个难点是它的代码段是分页解密的，我估计它的做法是将代码段锁定，即将其属性设为不可读、写、执行，当被执行到时会产生一个页面保护错误，而调试程序就会捕获这个错误，将对应的页面解锁并解密，然后返回执行。`Armadillo 2.xx` 只是附加了一些新特性，使得逆向工程变得更有趣。`Armadillo` 调试由它自身保护的软件，这一点使得我们这些 crackers 在跟踪调试目标时，遇到了很大的困难。

**※开始吧※**

`PEiD` 侦测知其用 `Armadillo 1.xx - 2.xx` 加壳，`Virtual Personality` 很狡猾，

在运行时，它有两个进程，一开始我还以为是 `CopyMem-II` 方式加壳，其实是标准方式加壳。

**\[一\], DUMP 出程序**

标准的流程如下，下 `bpx VirtualProtect` 去找 `OEP` 一旦 `softice` 中断了，检查 `softice` 是否在

`Virtual Personality` 的领空，如果不在则按 `F5` 直到 `softice` 在 `Virtual Personality` 的领空中断。

然后按 `F12` 辅之以 `F10` 跟踪过去一些指令，直到你看到一个 `"call EDI"` 指令，停在那儿，然后写下 `EDI` 的值。

那就是 `OEP`。

具体如下：

运行 `softice4.2.7`，开启 `Virtual Personality` 直到出现注册框，`ctrl+D` 调出 `softice`，

在按 `OK` 前 `BPX VirtualProtect`。到中断 38 次时程序运行了，重新来一遍，当在 37 次出现中断

在 `VirtualProtect` 时停下，`BC *` 清除所有断点，然后按 `F10` 单步返回到主程序。

```
001B:005FA207  83C404          ADD      ESP,04
001B:005FA20A  8945FC          MOV      [EBP-04],EAX
001B:005FA20D  837DECFF        CMP      DWORD PTR [EBP-14],-01
001B:005FA211  740B           JZ       005FA21E
001B:005FA213  8B55EC          MOV      EDX,[EBP-14]
001B:005FA216  891518C06200    MOV      [0062C018],EDX
001B:005FA21C  EB10           JMP      005FA22E
001B:005FA21E  837DFC01        CMP      DWORD PTR [EBP-04],01
001B:005FA222  740A           JZ       005FA22E
001B:005FA224  C70518C0620001000000MOV   DWORD PTR [0062C018],00000001
001B:005FA22E  837DB000        CMP      DWORD PTR [EBP-50],00
001B:005FA232  740A           JZ       005FA23E
001B:005FA234  8B45B0          MOV      EAX,[EBP-50]
001B:005FA237  50             PUSH     EAX
001B:005FA238  FF1504626200    CALL     [USER32!DestroyWindow]
001B:005FA23E  8B45FC          MOV      EAX,[EBP-04]
001B:005FA241  8BE5           MOV      ESP,EBP
001B:005FA243  5D             POP      EBP
001B:005FA244  C3             RET
```

再耐心地用 `F10` 单步走，因为有一堆的 `JMP`,`JNZ`,`JO` 等，用不了多久的就可来到这：

```
001B:005FB6F8  7511           JNZ      005FB70B       
001B:005FB6FA  6878BC6200     PUSH     0062BC78
001B:005FB6FF  FF159CBC6200   CALL     [0062BC9C]     ,这个 CALL 用 F8 跟进   
001B:005FB705  83C404         ADD      ESP,04
```

在 `005FB6FF` 这个 `CALL` 用 `F8` 跟进，然后继续 `F10`，几次后就来到这：

```
001B:00F8EEE9  334824         XOR      ECX,[EAX+24]
001B:00F8EEEC  334808         XOR      ECX,[EAX+08]
001B:00F8EEEF  2BF9           SUB      EDI,ECX
001B:00F8EEF1  FFD7           CALL     EDI    =====>EDI=0041657E
```

看到最后这个 `CALL EDI` 吗，`0041657E` 就是程序的 `OEP` 了，我们 `F8` 跟进：

```
001B:0041657E  55             PUSH     EBP
001B:0041657F  8BEC            MOV      EBP,ESP
001B:00416581  6AFF           PUSH     FF
001B:00416583  68B0794400     PUSH     004479B0
001B:00416588  6820B54100     PUSH     0041B520
001B:0041658D  64A100000000   MOV      EAX,FS:[00000000]
001B:00416593  50             PUSH     EAX
001B:00416594  64892500000000 MOV      FS:[00000000],ESP
001B:0041659B  83EC58         SUB      ESP,58
```

dump 取内存中己脱壳的文件

```
00F8EEF1    FFD7           CALL     EDI
```

现在这一行，键入以下命令：

```
a eip (然后按回车)

jmp eip (然后按回车)
```

按下 `F5`

这样将改变 `001B:00F8EEF1` 行的代码。你会注意到在键入 `"jmp eip"` 并按下回车后，`00F8EEF1` 的指令现在

是一个 `jmp`。这将有效地使程序 `"暂停"`（有点类似 `TRW2000` 的 `suspend` 命令）。按下 `F5` 使你回到 `window XP`。

运行 `LordPE`，在进程列表中选择 `"Virtual Personality.exe"`（注意要选择下面的那个进程）然后点击鼠标

右键选中 `"Dump (Full)"`，给脱壳的程序起名 dump 存盘。

**\[二\], 修复引入表**

一般步骤如下：

1. 找被脱壳的入口点（`OEP`）；
2. 完全 Dump 目标文件；
3. 运行 `Import REConstructor` 和需要脱壳的应用程序；
4. 在 `Import REConstructor` 下拉列表框中选择应用程序进程；
5. 在左下角填上应用程序的真正入口点偏移（`OEP`）；
6. 按 `"IAT AutoSearch"` 按钮，让其自动检测 `IAT` 位置，出现 `"Found address which may be in the Original IAT.Try 'Get Import'"` 对话框，这表示输入的 `OEP` 发挥作用了。
7. 按 `"Get Import"` 按钮，让其分析 `IAT` 结构得到基本信息；
8. 如发现某个 `DLL` 显示 `"valid :NO"` ，按 `"Show Invalids"` 按钮将分析所有的无效信息，在 `Imported Function Found` 栏中点击鼠标右键，选择 `"Trace Level1 (Disasm)"`，再按 `"Show Invalids"` 按钮。如果成功，可以看到所有的 `DLL` 都为 `"valid:YES"` 字样；
9. 再次刷新 `"Show Invalids"` 按钮查看结果，如仍有无效的地址，继续手动用右键的 `Level 2` 或 `3` 修复；
10. 如还是出错，可以利用 `"Invalidate function(s)"`、`"Delete thunk(s)"`、编辑 `Import` 表（双击函数）等功能手动修复。
11. 开始修复已脱壳的程序。选择 `Add new section (缺省是选上的)` 来为 Dump 出来的文件加一个 Section（虽然文件比较大，但避免了许多不必要的麻烦） 。
12. 按 `"Fix Dump"` 按钮，并选择刚在（2）步 Dump 出来的文件，在此不必要备份。如修复的文件名是 `"Dump.exe"`，它将创建一个 `"Dump\_.exe"`，此外 `OEP` 也被修正。
13. 生成的文件可以跨平台运行。

1. 先用 `importREC` 试试，输入我们找到的正确的 `OEP:0041657E`，注意输入 `0041657E-00400000=0001657E`

有很多指针无效。

运行 `Virtual Personality`，先等提示注册框出来，再下断 `bpx GetModuleHandleA`，再按 `OK`，

断下 20 次左右，每次断下都 `F12` 看看回到哪儿了，直到到了如下：

```
001B:00F75B87  FF15AC80F900    CALL     [KERNEL32!GetModuleHandleA]
001B:00F75B8D  394508          CMP      [EBP+08],EAX
001B:00F75B90  7507           JNZ      00F75B99
001B:00F75B92  B968B4F900      MOV      ECX,00F9B468
001B:00F75B97  EB52           JMP      00F75BEB
001B:00F75B99  393D68BAF900    CMP      [00F9BA68],EDI
001B:00F75B9F  B968BAF900      MOV      ECX,00F9BA68
001B:00F75BA4  0F8491000000    JZ       00F75C3B  =====>magic jump!
001B:00F75BAA  8B35581CFA00    MOV      ESI,[00FA1C58]
001B:00F75BB0  A1085DFA00      MOV      EAX,[00FA5D08]
001B:00F75BB5  F6410801        TEST     BYTE PTR [ECX+08],01
001B:00F75BB9  740E           JZ       00F75BC9
001B:00F75BBB  8B507C         MOV      EDX,[EAX+7C]
001B:00F75BBE  335078         XOR      EDX,[EAX+78]
001B:00F75BC1  335064         XOR      EDX,[EAX+64]
001B:00F75BC4  F6C280         TEST     DL,80
001B:00F75BC7  7513           JNZ      00F75BDC
001B:00F75BC9  8B507C         MOV      EDX,[EAX+7C]
001B:00F75BCC  33506C         XOR      EDX,[EAX+6C]
001B:00F75BCF  335048         XOR      EDX,[EAX+48]
001B:00F75BD2  33502C         XOR      EDX,[EAX+2C]
001B:00F75BD5  3316           XOR      EDX,[ESI]
001B:00F75BD7  395508         CMP      [EBP+08],EDX
001B:00F75BDA  740C           JZ       00F75BE8
001B:00F75BDC  83C10C         ADD      ECX,0C
001B:00F75BDF  83C604         ADD      ESI,04
001B:00F75BE2  3939           CMP      [ECX],EDI
001B:00F75BE4  75CF           JNZ      00F75BB5
```

这里面有几个跳转，其中最后一个 `00F75BE4` 是往上跳的。我们看一个就可发现：

`00F75BA4 JZ 00F75C3B`，这个跳正好可跳出这个循环。只要改这个跳，就不会破坏引入表了。

再来，出现注册框后 `OK`，程序再断下来，把 `00F75BA4` 的 `JZ 00F75C3B` 改成 `JMP 00F75C3B`，

清除所有断点，按 `F5` 运行，程序运行后打开 `importREC`，选中程序（注意选择上面那个进程），入口填



输入`0001657E`，点击`IT AutoSearch`，按提示填写好`RVA`、`Size`（例如`RVA 0006F000 Size 0017B85C`）--> 获得输入信息。如果还有无效的指针，大可放心地进行`cut`操作，因为它们本来就不是有用指针了。然后修复脱壳后的文件。OK，大功告成。

脱壳后的程序还存在问题，以后慢慢研究。

总结：这两天连续脱了两个`Armadillo`标准方式加壳的软件，这次脱的`Virtual Personality`很狡猾，在运行时，它有两个进程，一开始还以为是`CopyMem-II`方式加壳，结果按照`CopyMem-II`方式加壳脱壳流程试验始终没有进展，后来以标准方式加壳的脱壳流程居然走下来了。

`4dd466d0e1e0cc123afbe504da2462ca`
