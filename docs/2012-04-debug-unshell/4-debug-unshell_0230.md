
# Armadillo3.60 CopyMem-II + Debug-Blocker

> 原文：[`www.pediy.com/kssd/pediy06/pediy6745.htm`](https://www.pediy.com/kssd/pediy06/pediy6745.htm)

**【目标软件】** WINME 下的记事本

**【加壳方式】** `Armadillo3.60 CopyMem-II + Debug-Blocker`

**【调试环境】**：`WinXP`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC 1.6F`

**【实例下载】**：点击此处下载 或 [`bbs.pediy.com/showthread.php?s=&threadid=4666`](http://bbs.pediy.com/showthread.php?s=&threadid=4666)

## 1. 前言

为了练习脱壳，前一段时间从网上下了一个 `Armadillo3.60`，然后给记事本加壳。当然本人初学破解，参考了很多大虾们的文章，然后自己模仿脱壳。可是当我修改了 `Magic Jump` 后，用 `401000` 段“内存断点”大法想到达那个盼望已久的 `OEP` 的时候，意外出现了：程序终止，退出！是我做法错误？然后我又实验了 N 遍，依然如此，为什么？随后又在论坛不停的搜索着，想找一些有关的知识，终于找到了（抱歉，忘记了作者是谁），而且解决了这个问题。为了使像我一样初学者不要重蹈覆辙，我决定详细的写一写，也许你认为很简单，那么请略过。

## 2. 脱壳过程（分以下几步进行）

设置 `Ollydbg` 忽略所有的异常选项，用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

### （1）查找 OEP

用 `OD` 中载入程序，下 `bp WaitForDebugEvent`，`F9` 运行，中断如下：

```
77E93A07 k> 55 push ebp          <====中断在此处，清除断点
77E93A08 8BEC mov ebp,esp
77E93A0A 83EC 68 sub esp,68
77E93A0D 56 push esi
77E93A0E FF75 0C push dword ptr ss:[ebp+C]
```

查看堆栈窗口：

```
0012DAC0 00423D67 /CALL 到 WaitForDebugEvent 来自 NOTEPAD.00423D61
0012DAC4 0012EB84 |pDebugEvent = 0012EB84 <====注意这里
0012DAC8 000003E8 \Timeout = 1000. ms
```

在 `0012EB84` 所在的行上点击右键选“转存中跟随”。

然后再下 `bp WriteProcessMemory`, `F9` 运行

```
77E41A90 k> 55 push ebp    <====中断在此处
77E41A91 8BEC mov ebp,esp
77E41A93 51 push ecx
77E41A94 51 push ecx
77E41A95 8B45 0C mov eax,dword ptr ss:[ebp+C]
```

看数据转存窗口：

```
0012EB84 01 00 00 00 40 07 00 00 ...@..
0012EB8C 5C 01 00 00 01 00 00 80 \....€
0012EB94 00 00 00 00 00 00 00 00 ........
0012EB9C CC 10 40 00 02 00 00 00 ?@....
0012EBA4 00 00 00 00 CC 10 40 00 ....?@.
0012EBAC CC 10 40 00 00 00 00 00 ?@.....
```

红色字体的 `4010CC` 就是 `OEP`，这样我们接着来第 2 步

### （2）dump 文件

现在我们重新用 `OD` 中载入程序，下硬件断点，`he WaitForDebugEvent`，`F9` 运行，中断如下：

```
77E93A07 k> 55 push ebp   <====中断在此处，删除断点
77E93A08 8BEC mov ebp,esp
77E93A0A 83EC 68 sub esp,68
77E93A0D 56 push esi
77E93A0E FF75 0C push dword ptr ss:[ebp+C]
```

`Alt+F9` 返回，搜索常数 `FFFFFFF8`，来到（`004243D6` 处）：

```
0042438A > \83BD D0F5FFFF>cmp dword ptr ss:[ebp-A30],0 <====参考文章里提到的关键代码
00424391 . 0F8C A9020000 jl NOTEPAD.00424640 <====跳转到 00424640 处，因此在 00424640 处下断点
00424397 . 8B8D D0F5FFFF mov ecx,dword ptr ss:[ebp-A30]
0042439D . 3B0D E4454500 cmp ecx,dword ptr ds:[4545E4]
004243A3 . 0F8D 97020000 jge NOTEPAD.00424640
004243A9 . 8B95 44F6FFFF mov edx,dword ptr ss:[ebp-9BC]
004243AF . 81E2 FF000000 and edx,0FF
004243B5 . 85D2 test edx,edx
004243B7 . 0F84 AD000000 je NOTEPAD.0042446A
004243BD . 6A 00 push 0
004243BF . 8BB5 D0F5FFFF mov esi,dword ptr ss:[ebp-A30]
004243C5 . C1E6 04 shl esi,4
004243C8 . 8B85 D0F5FFFF mov eax,dword ptr ss:[ebp-A30]
004243CE . 25 07000080 and eax,80000007
004243D3 . 79 05 jns short NOTEPAD.004243DA
004243D5 . 48 dec eax
004243D6 . 83C8 F8 or eax,FFFFFFF8 <=====搜索来到这里，向上看
004243D9 . 40 inc eax
004243DA > 33C9 xor ecx,ecx
004243DC . 8A88 802A4500 mov cl,byte ptr ds:[eax+452A80]
004243E2 . 8B95 D0F5FFFF mov edx,dword ptr ss:[ebp-A30]
004243E8 . 81E2 07000080 and edx,80000007
004243EE . 79 05 jns short NOTEPAD.004243F5
004243F0 . 4A dec edx
004243F1 . 83CA F8 or edx,FFFFFFF8
004243F4 . 42 inc edx
004243F5 > 33C0 xor eax,eax
004243F7 . 8A82 812A4500 mov al,byte ptr ds:[edx+452A81]
004243FD . 8B3C8D 60E244>mov edi,dword ptr ds:[ecx*4+44E260]
00424404 . 333C85 60E244>xor edi,dword ptr ds:[eax*4+44E260]
0042440B . 8B8D D0F5FFFF mov ecx,dword ptr ss:[ebp-A30]
00424411 . 81E1 07000080 and ecx,80000007
00424417 . 79 05 jns short NOTEPAD.0042441E
00424419 . 49 dec ecx
0042441A . 83C9 F8 or ecx,FFFFFFF8
0042441D . 41 inc ecx
0042441E > 33D2 xor edx,edx
00424420 . 8A91 822A4500 mov dl,byte ptr ds:[ecx+452A82]
00424426 . 333C95 60E244>xor edi,dword ptr ds:[edx*4+44E260]
0042442D . 8B85 D0F5FFFF mov eax,dword ptr ss:[ebp-A30]
00424433 . 99 cdq
00424434 . B9 1C000000 mov ecx,1C
00424439 . F7F9 idiv ecx
0042443B . 8BCA mov ecx,edx
0042443D . D3EF shr edi,cl
0042443F . 83E7 0F and edi,0F
00424442 . 03F7 add esi,edi
00424444 . 8B15 D4454500 mov edx,dword ptr ds:[4545D4]
0042444A . 8D04B2 lea eax,dword ptr ds:[edx+esi*4]
0042444D . 50 push eax
0042444E . 8B8D D0F5FFFF mov ecx,dword ptr ss:[ebp-A30]
00424454 . 51 push ecx
00424455 . E8 FF1F0000 call NOTEPAD.00426459
0042445A . 83C4 0C add esp,0C
0042445D . 25 FF000000 and eax,0FF <====从这里开始修改
00424462 . 85C0 test eax,eax
00424464 . 0F84 D6010000 je NOTEPAD.00424640
```

在 `0042438A` 处下 硬件执行 断点，`F9` 运行到 `0042438A` 处，得到 `ebp-A30=12EB70`，这样上述的修改地方修改为：

```
0042445D FF05 70EB1200 inc dword ptr ds:[12EB70]
00424463 90 nop
00424464 ^ E9 21FFFFFF jmp NOTEPAD.0042438A
```

把 `12EB70` 处置 0，去掉所有断点，并在 `00424640` 处下断，`F9` 运行，断住。好了，所有代码都强制解压完成。

运行 `LordPE`,有 2 个关于 `NOTEPAD` 的进程，选择第 2 个，即可完全 dump 出来了。

### （3）修复 IAT 表

因为 `Armadillo` 的壳使得 `ImportREC` 的"IAT AutoSearch"失效，所以必须确定 IAT 的 `RAV` 以及大小。为此我们用 `OD` 载入刚刚 DUMP 出来的文件，来确定 IAT 的 `RAV` 以及大小。载入之前先修改入口点为 `10CC`，然后载入：

```
004010CC d> 55 push ebp
004010CD 8BEC mov ebp,esp
004010CF 83EC 44 sub esp,44
004010D2 56 push esi
004010D3 FF15 F4734000 call dword ptr ds:[4073F4] <====注意这里
004010D9 8BF0 mov esi,eax
```

我们可以随便找一个 CALL 入手确定 IAT 的 `RAV` 以及大小（不妨找看到的第一个 CALL）。光标定位到数据转存窗口中，然后 `Ctrl+G,4073F4`,可以看到：

```
004072E4 BE 7D 00 00 00 00 00 00 緘......
004072EC D7 23 DA 77 46 BA 3C 00 ?趙 F?.
004072F4 F0 59 DA 77 CC 5E DA 77 餣趙蘜趙
..........(省略若干)
004073DC 7D 15 F5 77 90 9B 3C 00 }鮳悰<.
004073E4 03 38 E5 77 67 31 E5 77 8 鍂 g1 鍂
004073EC 3C 51 E5 77 97 2D E5 77 <Q 鍂?鍂
004073F4 C7 C3 3C 00 45 9A E5 77 敲<.E 氬 w <====到达这里，由此处向上和向下看
004073FC 91 B2 3C 00 8D B8 3C 00 懖<.嵏<.
00407404 50 88 3C 00 CE 8A 47 77 P?.螉 Gw
..........(省略若干)
00407504 EC 72 D1 77 11 9A D1 77 靣褀氀 w
0040750C 06 81 D1 77 D3 3D D1 77 佈 w?褀
00407514 37 89 3C 00 B3 22 32 76 7?.?2v
0040751C 39 DC 33 76 3B C2 32 76 9?v;?v
00407524 10 1D 34 76 D6 8B 32 76 4v 謰 2v
0040752C B7 44 33 76 70 1B 32 76 稤 3vp2v
00407534 8B 88 3C 00 00 00 00 00 媹<.....
```

这样 IAT 的大小是终点-始点=`00407530-004072EC=244`，`RVA=72EC`。为什么是这样呢，我们看到很多 `77XXXXXX` 之类的，而到了 `00407534` 处是 `8B 88 3C 00`。为了确定其是否在 IAT 内，我们可以在 `OD` 的 CPU 窗口内定位光标到 `401000` 处，然后 `Ctrl+B`,填入 `34 75 40`，结果搜索不到，而 `30 75 40` 能够搜索到，是 `JMP [407530]`，因此确定 IAT 的终点点是 `407530`，用同样的方法可以确定起点点是 `4072EC`。

好了，IAT 的 `RAV` 以及大小确定了，现在就要修复 IAT 了。用 `OD` 再一次载入未脱壳的程序，`bp DebugActiveProcess`，`F9` 断下。看堆栈窗口：

```
0012DAC4 00423BDB /CALL 到 DebugActiveProcess 来自 NOTEPAD.00423BD5
0012DAC8 00000324 \ProcessId = 324 <=====子进程句柄
0012DACC 0012FF2C
```

打开另一个 `OD` 附加 `324` 这个子进程。然后 `ALT+F9` 返回程序:

```
00433999 N> - EB FE jmp short NOTEPAD.<ModuleEntryPoint> <====中断在此处，还原代码
0043399B EC in al,dx
0043399C |. 6A FF push -1
0043399E |. 68 503C4500 push NOTEPAD.00453C50
```

还原 `00433999` 处代码代码，`EB FE` 改为 `55 8B`，接下来就要按照 `FLY` 等大虾们的方法了。在接下来的步骤中前一部分可以参考 `Fly` 的《壹次脱壳法——Armadillo 双进程标准壳 快速脱壳》，我们从 `Magic Jump`，避开 IAT 加密开始，取消以前断点，下断：`BP GetModuleHandleA+5`，

```
77E59F98 /0F84 23060000 je kernel32.77E5A5C1 <====中断在此处，注意看堆栈
77E59F9E |FF7424 04 push dword ptr ss:[esp+4]
77E59FA2 |E8 55080000 call kernel32.77E5A7FC
77E59FA7 |85C0 test eax,eax
```

在这里中断若干次，注意堆栈窗口的变化，当出现：

```
......
0012BF20 00AAC807 返回到 00AAC807 来自 kernel32.GetModuleHandleA
0012BF24 00ABD6C8 ASCII "kernel32.dll"
0012BF28 00ABE67C ASCII "VirtualAlloc"
0012BF20 00AAC824 返回到 00AAC824 来自 kernel32.GetModuleHandleA
0012BF24 00ABD6C8 ASCII "kernel32.dll"
0012BF28 00ABE670 ASCII "VirtualFree"
0012BC98 00A9799B 返回到 00A9799B 来自 kernel32.GetModuleHandleA
0012BC9C 0012BDD4 ASCII "kernel32.dll"
```

然后 `Alt+F9` 返回，

```
00A97995 FF15 C480AB00 call dword ptr ds:[AB80C4] ; kernel32.GetModuleHandleA
00A9799B 8B0D E011AC00 mov ecx,dword ptr ds:[AC11E0] <====返回到这里
00A979A1 89040E mov dword ptr ds:[esi+ecx],eax
00A979A4 A1 E011AC00 mov eax,dword ptr ds:[AC11E0]
00A979A9 393C06 cmp dword ptr ds:[esi+eax],edi
00A979AC 75 16 jnz short 00A979C4
```



### 汇编代码片段

```
00A979AE 8D85 B4FEFFFF lea eax,dword ptr ss:[ebp-14C]  

00A979B4 50 push eax  

00A979B5 FF15 CC80AB00 call dword ptr ds:[AB80CC] ; kernel32.LoadLibraryA  

00A979BB 8B0D E011AC00 mov ecx,dword ptr ds:[AC11E0]  

00A979C1 89040E mov dword ptr ds:[esi+ecx],eax  

00A979C4 A1 E011AC00 mov eax,dword ptr ds:[AC11E0]  

00A979C9 393C06 cmp dword ptr ds:[esi+eax],edi  

00A979CC 0F84 AD000000 je 00A97A7F<====Magic Jump，修改为 JMP  

00A979D2 33C9 xor ecx,ecx  

00A979D4 8B03 mov eax,dword ptr ds:[ebx]  
```

### 修复步骤

修改了上面的 `Magic Jump` 后，清除所有断点，`Alt+M` ,在 `401000` 上下 内存访问断点，`F9` 运行，这次中断到了 `OEP`，然后运行 `ImportREC`，选择 `324` 这个进程，`OEP` 填：`10CC`，`RVA`：`72EC`，`SIZE`：`244`，然后“`Get Imports`”,再按“`Show Invalid`”地址，剪掉修复抓取文件,程序能够正常运行,至此脱壳完毕 !第一次用 `OD` 脱壳,感觉就是爽,和 `Softice` 比起来,感觉 `OD` 脱壳很方便.

`4dd466d0e1e0cc123afbe504da2462ca`
