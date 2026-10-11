
# Asprotect 1.23RC4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6718.htm`](https://www.pediy.com/kssd/pediy06/pediy6718.htm)

原文链接：`http://bbs.pediy.com/showthread.php?s=&threadid=5876`

昨天终于把 `Asprotect 1.23RC4` 加密的 `Notepad` 脱壳了，太激动了！这是我第一次凭自己的能力脱的！

虽然以前就脱过这个壳，不过以前仅仅只知道 `Trace N` 次后 `Dump` 后开 `AsprDbgr` 修复 `IAT`... 为什么要这样做却完全不懂，现在不仅知道 `HOW` 而且知道 `WHY` 了！

下面是脱壳过程，就算是我的第一篇脱文吧

我脱这个壳用了 8 个小时（网吧上网好贵....），走了 N 条弯路，那些弯路我就不写了，只记下怎样找到正确的方法的

目标：`Asprotect 1.23RC4` 加密的 `Notepad`：点击此处下载或鼠标右键另存为。

操作系统是 `Win98`

对这个壳我已经知道一些情况，`Asprotect` 会用 20 多次 `SEH`，然后把程序入口的一些代码移动别的地方（`StolenCode`），并且 `IAT` 表没有指向正确的 `API`，而是指到了壳 `HookAPI` 的代码

我的思路是找到入口点，然后 `Dump`，再修复 `IAT`，至于 `StolenCode` 到时候再考虑

首先用 `OD` 加载 `Notepady.exe`

在 选项->调试选项 里把"忽略(传递到程序)以下的异常"的 6 个钩去掉

表示当这些异常发生时就中断下来，这样就可以跟踪到壳激活 `SEH` 的地方

这么做的原因是一个没有错误的程序不会激活 `SEH`，假设记事本就是这么个程序

这样 `SEH` 就一般只会在壳的代码里被激活了，所以跟到最后一个 `SEH` 时表示已经接近程序入口点了

（看到很多篇脱文里都只拦截内存异常，而不是所有异常都拦截，一直没有想同这是为什么，谁可以提示一下吗？）

然后按 `F9` 运行程序，会中断下来，因为壳制造异常来激活 `SEH`，按 `Shift+F9` 忽略异常继续执行

在 14 个 `SEH` 时再按一下 `Shift+F9` 就出现对话框提示检测到调试器（现在知道是 `IsDebug` 的 `Hide` 在 `Win98` 下无效），所以要手工避开检测

按几下 `F8` 单步运行，来到下面的代码：

```
010C3EFE      74 09                je short 010C3F09

010C3F00      E8 4BD7FFFF          call 010C1650

010C3F05      8BD8                 mov ebx,eax

010C3F07      EB 07               jmp short 010C3F10

010C3F09      E8 B6D6FFFF          call 010C15C4

010C3F0E      8BD8                 mov ebx,eax

010C3F10      84DB                test bl,bl

010C3F12      75 09               jnz short 010C3F1D

010C3F14      E8 3BD7FFFF          call 010C1654           //可能是检测函数

010C3F19      84C0                test al,al           //返回 0 就跳过 call 010C2678

010C3F1B      74 10               je short 010C3F2D

010C3F1D      A1 A47E0C01         mov eax,dword ptr ds:[10C7EA4]

010C3F22      50                  push eax           //运行到这里发现 eax->"Debugger detected...."

010C3F23      68 6C3F0C01         push 10C3F6C                 ; ASCII "Protection Error"

010C3F28      E8 4BE7FFFF         call 010C2678           //由上面的字符串猜测这里应该是产生对话框并中断程序的 CALL

010C3F2D      E8 7EE6FFFF         call 010C25B0

010C3F32      33C0                xor eax,eax
```

分析一下发现 `call 010C1654` 可能是检测函数，当这个函数返回 `0` 时就跳过 "Protection Error" 的对话框

验证一下上面的猜测：重新运行，到 `test al,al` 这里就把 `eax` 的值改为 `0`，然后狂按 `Shift+F9`，发现记事本正确运行了

这样就避开了检测，以后每次运行 `Notepady.exe` 都要在这里中断下来修改 `eax`，下面就不再重复说明

再次加载程序，按 `Shift+F9`，发现按了 29 次就打开记事本了（所以第 28 次时是最后一个 `SEH`）

然后按 `F12` 暂停程序，这时执行的代码就是程序的代码了，跟踪一下发现是在

```
while(GetMessage(....))
{
    TranslateMessage();
    DispatchMessage();
}
```

这个循环里，代码如下：

```
0040213F      50                  push eax

00402140      FF15 98644000       call dword ptr ds:[406498] //TranslateMessage

00402146      8D45 E4            lea eax,dword ptr ss:[ebp-1C]

00402149      50                  push eax

0040214A      FF15 9C644000       call dword ptr ds:[40649C] //DispatchMessage

00402150      56                  push esi

00402151      8D45 E4            lea eax,dword ptr ss:[ebp-1C]

00402154      56                  push esi

00402155      56                  push esi

00402156      50                  push eax

00402157      FF15 A0644000       call dword ptr ds:[4064A0] //GetMessage

0040215D      85C0                test eax,eax

0040215F    ^ 75 A5               jnz short NOTEPADY.00402106
```

`TranslateMessage`, `DispatchMessage`, `GetMessage` 都被 `OD` 识别出来，说明壳没有加密这 3 个函数

从中还可以看出程序的代码段地址大概是在 `402000` 左右（猜测可能就是 `VC` 生成的 `.text` 段，从 `401000` 开始）

另外，`IAT` 应该在 `406400` 附近

重新加载程序，在最后一个 `SEH` 时按 `F7` 跟踪

如果 `EIP` 从 `010*****` 跳到 `0040****` 就说明壳已经把控制权交给了程序的代码

```
010C39EC      3100                xor dword ptr ds:[eax],eax //最后一个异常,按 Shfit+F8 单步执行

010C39EE      64:8F05 00000000    pop dword ptr fs:[0]

010C39F5      58                  pop eax

010C39F6      833D B07E0C01 00    cmp dword ptr ds:[10C7EB0],0

010C39FD      74 14               je short 010C3A13

010C39FF      6A 0C               push 0C

010C3A01      B9 B07E0C01         mov ecx,10C7EB0

010C3A06      8D45 F8             lea eax,dword ptr ss:[ebp-8]

010C3A09      BA 04000000         mov edx,4

010C3A0E      E8 2DD1FFFF         call 010C0B40 //到这里按 F8 跳过去,因为跳过这个 CALL 并没有显示记事本的窗口,说明程序入口点还在下面

010C3A13      FF75 FC             push dword ptr ss:[ebp-4] //从这里以后按 F7 单步跟踪

010C3A16      FF75 F8             push dword ptr ss:[ebp-8]

010C3A19      8B45 F4             mov eax,dword ptr ss:[ebp-C]

010C3A1C      8338 00             cmp dword ptr ds:[eax],0

010C3A1F      74 02               je short 010C3A23

010C3A21      FF30                push dword ptr ds:[eax]

010C3A23      FF75 F0             push dword ptr ss:[ebp-10]

010C3A26      FF75 EC             push dword ptr ss:[ebp-14]

010C3A29      C3                  retn
```

按下 `F7` 不放，过了一段时间发现似乎是在一个复杂的循环里，几个 `CALL` 相互调用着循环

既然我们已经知道程序代码大概是 `402000` 附近

就可以用 调试->跟踪进入 来让 `OD` 自动跟踪

具体这样操作：

点 调试->设置条件 在"EIP 位于范围内"打上钩，后面填入 `00400000,00403000`（这样当 `EIP` 在这里面就会中断下来）点确定

点 调试->开始或清除运行跟踪 然后点 调试->跟踪进入 （我怎么知道这样操作？ N 次尝试后发现的...）

![](img/8c1a47a0ac57860fe881c180c59cac0f.jpg)

等待几秒钟，`OD` 中断下来：

```
004010C5      0000                add byte ptr ds:[eax],al

004010C7      000D 0A000000       add byte ptr ds:[A],cl

004010CD      0000                add byte ptr ds:[eax],al

004010CF      0000                add byte ptr ds:[eax],al

004010D1      0000                add byte ptr ds:[eax],al

004010D3      FF15 E4634000       call dword ptr ds:[4063E4] //OD 中断在这里,可以看出上面的代码显然没有意义

004010D9      8BF0                mov esi,eax

004010DB      8A00                mov al,byte ptr ds:[eax]

004010DD      3C 22               cmp al,22

004010DF      75 1B               jnz short NOTEPADY.004010FC

004010E1      56                  push esi

004010E2      FF15 F4644000       call dword ptr ds:[4064F4]

004010E8      8BF0                mov esi,eax

004010EA      8A00                mov al,byte ptr ds:[eax]

004010EC      84C0                test al,al

004010EE      74 04               je short NOTEPADY.004010F4

004010F0      3C 22               cmp al,22
```

点 查看->运行跟踪 可以查看刚才 `OD` 记录的全部运行过程，拉到最下面看到壳是在 `010D3A88` `retn` 这里跳到上面的代码（`004010D3`）

![](img/e2770ca4f33f728dc9c2034418c2bfaf.jpg)

点 调试->关闭运行跟踪

这时我们已经找到了程序入口点 `OEP=004010D3`

不过正确的入口点应该是在 `0x401000`（`VC` 默认生成的入口点）

而且应该是 `push ebp;mov ebp,esp` 开头的

所以 `OEP=004010D3` 前面肯定有一部分代码被壳搬到别的地方去执行了

从跟踪运行里可以看到这些被搬掉的代码（如果按上面说的执行，那么已经关闭运行跟踪，没关系，再 `Trace` 一遍...）

![](img/a42a3f467340f33320c045faee61b9cf.jpg)

所以正确的 `OEP` 应该是 `010D3A75`（图中黄的那行），因为下面有很多 `rep stos`，跟踪一个 `VC6` 程序会发现 `VC` 总是在每个函数开头都加上一句 `rep stos` 来初始化栈区，所以猜测 `010D3A75` 是原来的 `OEP`

这时应该可以把 `010D3A75` 这里的代码搬到 `0401000` 来就行了，不过我不是这么做

既然 `Notepad` 是 `VC` 写的，那么应该有一段 `VC` 生成的入口代码来调用 `WinMain`

我的想法是不去修补入口代码，而是自己写一段代码来 `call WinMain`，就象这样：

`WinMain(GetModuleHandle(NULL), NULL, GetCommandLine(), SW_SHOWNORMAL);`

用汇编就是

```
push 0A
call GetCommandLineA
push eax
push 0
push 0
call GetModuleHandle
push eax
call WinMain
```

所以只要在下面找 `WinMain` 就行（不过我最后发现这样实际上绕了弯路）

从 `004010D3` 这个 `OEP` 开始 `F8` 跟踪：

```
004010D3      FF15 E4634000       call dword ptr ds:[4063E4]
004010D9      8BF0                mov esi,eax
```

`0x4010D3` 这个 `CALL` 的地址应该是在 `IAT` 里（想想刚才看到的 `GetMessage` 的地址）

但是 `OD` 没有识别出来，可能是被壳加了密的 `API`，先不跟踪它，等修复 `IAT` 时再研究

不过这个 `call` 返回了 `eax=81D6DB44`, (ASCII `"E:\Crack\NOTEPADy.EXE"`)

由此猜测这个 `API` 可能是 `CommandLineA`，等修复 `IAT` 时再检验是不是

`F8` 几步以后进入一个循环，从寄存器窗口看出这个循环好象在处理命令行参数

过了这个循环就来到这里：

```
00401146      50                  push eax
00401147      56                  push esi
00401148      6A 00               push 0
0040114A      6A 00               push 0
0040114C      FF15 9C634000       call dword ptr ds:[40639C] //从 40639C 来看应该是 API
00401152      50                  push eax
00401153      E8 760F0000         call NOTEPADY.004020CE //可能是 WinMain
00401158      50                  push eax
00401159      8BF0                mov esi,eax
0040115B      FF15 A0634000       call dword ptr ds:[4063A0] //KERNEL32.ExitProcess
```
