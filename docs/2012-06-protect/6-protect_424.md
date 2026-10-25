
# 堆栈初始数据简单分析及其 anti od, anti unpack 应用 (3 楼补充相关汇编代码)

> 原文：[www.pediy.com/kssd/pediy08/pediy8-414.htm](https://www.pediy.com/kssd/pediy08/pediy8-414.htm)

- **文章标题**: 【原创】堆栈初始数据简单分析及其 anti od, anti unpack 应用
- **文章作者**: 笨笨雄
- **作者邮箱**: nemo314@gmail.com
- **工具名称**: `WINDBG`, `OLLYDBG`
- **作者声明**: 本人见识比较少，不知道这是不是我第一个发现的。如果是老东西，大家可以无视。。。

偶然机会之下，我用 `windbg` 跟 `ollydbg` 打开同一个程序，发现堆栈的初始数据如下：

### windbg

```text
0006ffc4  03 79 e6 77 20 48 08 00-00 00 00 00 00 f0 fd 7f  .y.w H..........
0006ffd4  bc f7 13 01 c8 ff 06 00-bc f7 13 01 ff ff ff ff  ................
0006ffe4  fd 13 e8 77 08 79 e6 77-00 00 00 00 00 00 00 00  ...w.y.w........
0006fff4  00 00 00 00 20 64 00 01-00 00 00 00 c8 00 00 00  .... d..........
00070004  00 01 00 00 ff ee ff ee-62 00 00 50 60 00 00 40  ........b..P`..@
00070014  00 fe 00 00 00 00 10 00-00 20 00 00 00 02 00 00  ......... ......
00070024  00 20 00 00 d6 02 00 00-ff ef fd 7f 01 00 08 06  . ..............
00070034  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
```

### ollydbg

```text
0006FFC4   77E67903  返回到 KERNEL32.77E67903
0006FFC8   00000000
0006FFCC   00000000
0006FFD0   7FFDF000
0006FFD4   00000000
0006FFD8   0006FFC8
0006FFDC   00000000
0006FFE0   FFFFFFFF  SEH 链尾部
0006FFE4   77E813FD  SE 句柄
0006FFE8   77E67908  KERNEL32.77E67908
0006FFEC   00000000
0006FFF0   00000000
0006FFF4   00000000
0006FFF8   01006420  test.<ModuleEntryPoint>
0006FFFC   00000000
```

对比之下，不同的地方有 `0006FFC8`、`0006FFD4`、`0006FFDC`。现在尝试用 `OD` 打开某 `keygenme`，得到下面堆栈的初始数据：

```text
0012FFC4   77E67903  返回到 KERNEL32.77E67903
0012FFC8   00000000
0012FFCC   00000000
0012FFD0   7FFDF000
0012FFD4   00000000
0012FFD8   0012FFC8
0012FFDC   00000000
0012FFE0   FFFFFFFF  SEH 链尾部
0012FFE4   77E813FD  SE 句柄
0012FFE8   77E67908  KERNEL32.77E67908
0012FFEC   00000000
0012FFF0   00000000
0012FFF4   00000000
0012FFF8   004414A0  keygenme.<ModuleEntryPoint>
0012FFFC   00000000
```

显然，目标程序被 `OD` 载入之后，某些位置是不变的。现在来看看 `WINDBG`，载入 `EXPLORER.EXE` 得到：

```text
0006ffc4  03 79 e6 77 98 97 09 00-00 00 00 00 00 f0 fd 7f  .y.w............
0006ffd4  bc f7 17 01 c8 ff 06 00-bc f7 17 01 ff ff ff ff  ................
0006ffe4  fd 13 e8 77 08 79 e6 77-00 00 00 00 00 00 00 00  ...w.y.w........
0006fff4  00 00 00 00 a2 c6 40 00-00 00 00 00 c8 00 00 00  ......@.........
00070004  00 01 00 00 ff ee ff ee-62 00 00 50 60 00 00 40  ........b..P`..@
00070014  00 fe 00 00 00 00 10 00-00 20 00 00 00 02 00 00  ......... ......
00070024  00 20 00 00 73 01 00 00-ff ef fd 7f 01 00 08 06  . ..s...........
00070034  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
```

跟 `OLLYDBG` 一样，某些位置是不变的。

### 总结

1. 经由 `OLLYDBG` 与 `WINDBG` 加载的同一个程序，堆栈中初始化的数据是不同。
2. 不同程序在 `OLLYDBG` 中加载，比较堆栈中的初始数据，并不是完全不相同的。相同的初数数据分布在有特殊意义堆栈地址和未知意义的堆栈地址。特殊意义的地址将会被系统写入数据，那么未知意义的地址呢？例如 `0012FFC8`，或许我们可以通过在 `EP` 使用 `ESP+4` 来访问，假如 `[esp+4]` 为 `0`，那么便可以确定程序被 `OLLYDBG` 调试。
3. 假如 `OLLYDBG` 堆栈初始数据是正确的，那么 `[esp+4]` 不为 `0` 程序便是被 `WINDBG` 调试了。

也就是说，`WINDBG` 或者 `OLLYDBG` 肯定有一个可以通过堆栈的初始数据来判断是否被 `DEBUG` 了。现在我们来看看由系统加载的程序，初始堆栈是怎么样的。

用 `OD` 随便改一个程序，入口处一开始便是 `MESSAGEBOX` 的调用。先用 `WINDBG` 加载，记下 `ESP` 的值，然后再次运行程序，用 `WINDBG` `ATTACH TO A PROCESS`，在数据窗口中跟随到刚才记下的 `ESP`。现在我们得到下面堆栈：

```text
0006ffc4  03 79 e6 77 00 00 00 00-f8 a3 0f 00 00 f0 fd 7f  .y.w............
0006ffd4  90 dc 1a 04 c8 ff 06 00-90 dc 1a 04 ff ff ff ff  ................
0006ffe4  fd 13 e8 77 08 79 e6 77-00 00 00 00 00 00 00 00  ...w.y.w........
0006fff4  00 00 00 00 20 64 00 01-00 00 00 00 c8 00 00 00  .... d..........
00070004  00 01 00 00 ff ee ff ee-02 00 00 00 00 00 00 00  ................
00070014  00 fe 00 00 00 00 10 00-00 20 00 00 00 02 00 00  ......... ......
00070024  00 20 00 00 85 00 00 00-ff ef fd 7f 01 00 08 06  . ..............
00070034  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
```

`EP` 处的 `[ESP+8]` 不为 `0`，而 `OD` 加载的程序，在该处是 `0`。发现这个规律之后，我便立刻改了一个程序的入口用以检查 `OD`，能正确检查出 `OD`。但是当我从 `WIN2K`（最初的版本）转到 `WIN2K SP4` 的时候，便失效了。我意识到，这种特性与系统是相关的，于是也拿到 `XP SP2` 下试验，用 `OD` 加载程序后得出下面两组堆栈初始值：

#### WIN2K SP4

**regedit.exe**

```text
0006FFC4   77E687F5  返回到 KERNEL32.77E687F5
0006FFC8   005916A8
0006FFCC   00000056
0006FFD0   7FFDF000
0006FFD4   00000200
0006FFD8   0006FFC8
0006FFDC   00000200
0006FFE0   FFFFFFFF  SEH 链尾部
0006FFE4   77E7F0B4  SE 句柄
0006FFE8   77E68EC8  KERNEL32.77E68EC8
0006FFEC   00000000
0006FFF0   00000000
0006FFF4   00000000
0006FFF8   01007345  regedit.<ModuleEntryPoint>
0006FFFC   00000000
```

**explorer.exe**

```text
0006FFC4   77E687F5  返回到 KERNEL32.77E687F5
0006FFC8   005916A8
0006FFCC   00000056
0006FFD0   7FFDF000
0006FFD4   00000200
0006FFD8   0006FFC8
0006FFDC   00000200
0006FFE0   FFFFFFFF  SEH 链尾部
0006FFE4   77E7F0B4  SE 句柄
0006FFE8   77E68EC8  KERNEL32.77E68EC8
0006FFEC   00000000
0006FFF0   00000000
0006FFF4   00000000
0006FFF8   00408188  explorer.<ModuleEntryPoint>
0006FFFC   00000000
```

#### WINXP SP2

**notepad.exe**

```text
0007FFC4   7C816D4F  返回到 kernel32.7C816D4F
0007FFC8   7C930738  ntdll.7C930738
0007FFCC   FFFFFFFF
0007FFD0   7FFD7000
0007FFD4   8054C038
0007FFD8   0007FFC8
0007FFDC   81813D40
0007FFE0   FFFFFFFF  SEH 链尾部
0007FFE4   7C8399F3  SE 句柄
0007FFE8   7C816D58  kernel32.7C816D58
0007FFEC   00000000
0007FFF0   00000000
0007FFF4   00000000
0007FFF8   0100739D  NOTEPAD.<ModuleEntryPoint>
0007FFFC   00000000
```

**test.exe**

```text
0006FFC4   7C816D4F  返回到 kernel32.7C816D4F
0006FFC8   7C930738  ntdll.7C930738
0006FFCC   FFFFFFFF
0006FFD0   7FFD4000
0006FFD4   8054C038
0006FFD8   0006FFC8
0006FFDC   816E0B30
0006FFE0   FFFFFFFF  SEH 链尾部
0006FFE4   7C8399F3  SE 句柄
0006FFE8   7C816D58  kernel32.7C816D58
0006FFEC   00000000
0006FFF0   00000000
0006FFF4   00000000
0006FFF8   01006420  test.<ModuleEntryPoint>
0006FFFC   00000000
```

为了看看是否存在硬件相关性，我在另外一台安装了 `XP SP2` 的电脑上做了相同的试验，结果如下：

**esp.exe**

```text
0006FFC4   7C816D4F  返回到 kernel32.7C816D4F
0006FFC8   7C930738  ntdll.7C930738
0006FFCC   FFFFFFFF
0006FFD0   7FFD7000
0006FFD4   8054B938
0006FFD8   0006FFC8
0006FFDC   81AA38A0
0006FFE0   FFFFFFFF  SEH 链尾部
0006FFE4   7C8399F3  SE 句柄
0006FFE8   7C816D58  kernel32.7C816D58
0006FFEC   00000000
0006FFF0   00000000
0006FFF4   00000000
0006FFF8   01006420  esp.<ModuleEntryPoint>
0006FFFC   00000000
```

**explorer.exe**

```text
0007FFC4   7C816D4F  返回到 kernel32.7C816D4F
0007FFC8   7C930738  ntdll.7C930738
0007FFCC   FFFFFFFF
0007FFD0   7FFDD000
0007FFD4   8054B938
0007FFD8   0007FFC8
0007FFDC   81CB6DA8
0007FFE0   FFFFFFFF  SEH 链尾部
0007FFE4   7C8399F3  SE 句柄
0007FFE8   7C816D58  kernel32.7C816D58
0007FFEC   00000000
0007FFF0   00000000
0007FFF4   00000000
0007FFF8   0101E24E  explorer.<ModuleEntryPoint>
0007FFFC   00000000
```

### 结论

我们可以通过判断 `EP` 中的 `ESP+8` 是否为 `FFFFFFFF` 或者 `00000056` 判断程序是否被 `OD` 加载，这里要说明的是 `[ESP+8]=0` 的情况。在 `WIN2K SP4` 中，由系统加载的程序初始堆栈将会是下面那样：

```text
0006ffc4  f5 87 e6 77 00 00 00 00-00 00 00 00 00 f0 fd 7f  ...w............
0006ffd4  00 00 00 00 c8 ff 06 00-00 00 00 00 ff ff ff ff  ................
0006ffe4  b4 f0 e7 77 c8 8e e6 77-00 00 00 00 00 00 00 00  ...w...w........
0006fff4  00 00 00 00 20 64 00 01-00 00 00 00 c8 00 00 00  .... d..........
00070004  00 01 00 00 ff ee ff ee-02 00 00 00 00 00 00 00  ................
00070014  00 fe 00 00 00 00 10 00-00 20 00 00 00 02 00 00  ......... ......
00070024  00 20 00 00 df 00 00 00-ff ef fd 7f 01 00 08 06  . ..............
00070034  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
```

`[ESP+8]` 同样为 `0`，所以这个是不能用的，否则会产生误报。要避免程序在 `WIN2K` 最初的版本被调试，可以检查系统版本，再检查 `[ESP+8]` 是否为 `0`。由于我的同学基本都是用 `XP SP2` 的，所以我无法给出除这 3 个系统之外更多的信息，大家可以用这个方法找出其他系统的特征。

由 `WINDBG` 加载的程序，跟系统的还是有差别的，只是无论系统跟 `WINDBG`，在不同的系统，不同的硬件下，没有特别意义的地址一直都在“无规则”的改变。

我估计会有这样的情况出现是因为一般我们释放堆栈的时候只是对 `ESP` 进行操作，而不会管仍然存在于堆栈中的数据，我本来想通过分析代码找出那些数据究竟是什么时候被填上去的，可惜我想用 `WINDBG` 进行内核调试的时候才发觉我完全不知道应该怎么做，又得看英文了。。。当我熟练掌握 `WINDBG` 的时候，我想我可以同时给出 `WINDBG` 帮助文件的中文翻译了。。。



回到正题，注意到 `[esp+34]` 里面的是 `EP`，我想或者我们可以通过特性来判断在脱壳过程中是否到达 `OEP`。下面是我用 `NSPACK` 做了个实验，`OEP` 处的堆栈：

```
0012FFC4   77E687F5  返回到 KERNEL32.77E687F5
0012FFC8   005916A8
0012FFCC   00000056
0012FFD0   7FFDF000
0012FFD4   00000200
0012FFD8   0012FFC8
0012FFDC   00000200
0012FFE0   FFFFFFFF  SEH 链尾部
0012FFE4   77E7F0B4  SE 句柄
0012FFE8   77E68EC8  KERNEL32.77E68EC8
0012FFEC   00000000
0012FFF0   00000000
0012FFF4   00000000
0012FFF8   1320536B  offset servre.<ModuleEntryPoint>
0012FFFC   00000000
```

显然这个特性不能在脱壳时应用，反而能成为程序校验自己是否被脱壳，而且重要的是，这种校验，不需要使用 `API`。幸好我们仍然能使用内存断点或硬件断点在入口处 `ESP+34` 的地方下断。

最后就是 `[ESP]` 了，`OD` 显示返回到 `KERNEL32.77E687F5`，实际上我们可以通过返回这个地址来达到退出程序的目的。

在做第一个测试的时候，我发现那些返回地址分别对应不同的操作系统：

```
7C816D4F  XP SP2
77E67903  WIN2K 最初的版本
77E687F5  WIN2K SP4 的
```

另一个不需要 `API` 就能判断当前操作系统的方法。

谢谢楼上的鼓励![](img/de82a00003a7d4a29fabed7b826fae7a.jpg)

无论新旧，补上实现的汇编代码吧，不会的可以学习一下。

首先要说的是因为是基于堆栈初始数据的 `ANTI`，只要在堆栈初始数据下内存或硬件访问断点就可以定位该 `ANTI` 了。如果你先 `ANTI` 内存断点和硬件断点，那么这个检查将会非常隐蔽。

现在说正题，下面是根据我的结论得出的简单 `ANTI UNPACK` 的代码，假如程序被 `UNPACK` 了，那么将产生一个 `0x00000000` 访问错误：

```
start proc:

call @@1

@@1:  pop eax       //病毒中常见的获得当前 EIP 的方法
sub eax,5           //减去第一条代码的机器码长度，即为 EP
sub eax,[esp + 34]  //校验 EP 是否等于 OEP，如果程序加壳了，EAX 不为 0（不过看过 FLY 其中一篇脱文，里面介绍的壳是 EP=OEP 的）
je eax              //这里改变 EIP 到处理 UNPACK 的流程或正常流程，随你喜欢了。
ret

end start
```

把 `ANTI` 代码安排在 `OEP`，那么这个 `ANTI` 不使用 `API` 的隐藏性也就变得意义不大了。而且在程序开始使用了病毒技术，很可能会被杀毒软件误报。下面我将提供一个更加灵活的方法，并把 `ANTI OD` 的代码给出。

```
_start proc:

push ebp
mov ebp,esp                //一般程序入口都有的语句，EBP=EP 处 ESP-4
call save_ebp              //这里可以是_START 中的任意 CALL,但是得保证在这之前 EBP 不会被改变。
pop ebp
ret

end _start

save_ebp proc:

push ebp
mov ebp,esp                //这同样会在一般 CALL 中出现
mov dword ptr ds:[00ff0000],ebp    //在函数的一开始插入汇编代码，将 EBP 保存在任意全局变量中，好了，现在你可以在任意深度的 CALL 中访问 EP 的 ESP 了，不过要注意访问必须在此处运行之后。
pop ebp
ret

end save_ebp

chk_od proc:              //检查 OD 是否存在
pushad
mov ebp,dword ptr ds:[00ff0000]    //EBP=EP 处 ESP-4
mov edx,7C816D4F
cmp edx,dword ptr ss:[ebp + 4]
je _IsXPsp2               //用结论中提到的方法判断操作系统,下同
mov edx,77E67903
cmp edx,dword ptr ss:[ebp + 4]
je _IsWin2K
mov edx,77E687F5
cmp edx,dword ptr ss:[ebp + 4]
je _IsWin2Ksp4
jmp chk_end

_IsXPsp2:
mov edx,FFFFFFFF
cmp edx,dword ptr ss:[ebp+0c]      //对比 OD 在 XPSP2 的特征码
je Is_debug               //调用你自己处理被 OD 调试的例程
jmp chk_end

_IsWin2K:
xor edx,edx               //在这个版本中 OD 在该位置的特征码是 00000000
cmp edx,dword ptr ss:[ebp+0c]
je Is_debug
jmp chk_end

_IsWin2Ksp4:
mov edx,00000056          //在这个版本中 OD 在该位置的特征码是 00000056
cmp edx,dword ptr ss:[ebp+0c]
je Is_debug

chk_end:
popad
ret

chk_od end
```

必须要说明的是此方法有很大局限性，经测试，在不同硬件环境相同系统下可用，但是你必须找出所有系统所有 `SP` 版本下的特征码。但是我没有测试在不同版本中的 `OD` 是否仍然可用，我使用的是 `flyodbg---ollydbg v1.10` 中文修改版。

注意到，在相同系统下，由 `OD` 加载程序堆栈的初始数据在 `EP ESP+8` 处是固定不变的，而由系统加载，此处第一次加载跟第二次加载是不同的，此后加载的初始堆栈都跟第二次一样，直到注销或重启系统。假如你能实现判断是否第一次被系统运行，然后每次启动都将 `EP ESP+8` 保存在文件或注册表中，那么你将能写出兼容多个系统的 `ANTI flyodbg---ollydbg v1.10` 中文修改版函数。由于实现超过了我的水平，我无法给出实例，不过为了 `ANTI` 一个版本的 `OD` 是否值得也是一个问题。

下面来看看如何 `ANTI UNPACK`。

```
chk_unpack proc:

pushad
mov ebp,dword ptr ds:[00ff0000]    //EBP=EP 处 ESP-4
mov eax,dword ptr ss:[ebp + 38]    //EAX=EP
call @@1
@@1: pop ebx               //得到当前 EIP
mov edx,eax                //下面将计算 EP 与当前 EIP 的差值，由于是 32 位的无符号加减。我需要处理的是两者之间的绝对值，如果你有更好的方法，可以修改这几行代码。
sub eax,ebx
ja @@2
sub ebx,edx
mov eax,ebx
@@2:
cmp eax,10000h        //留到最后解释
ja Is_Unpack          //调用你的自定义处理函数
popad
ret

end chk_unpack
```

为什么是 `10000h`? 此处跟系统内存的页面分配有关，我没看过相关介绍，我是从看雪精华六里面翻译的 `WIN32` 病毒教程中，关于如何通过内存搜索获得 `KERNEL32` 基址部分得到可以这么做的启发。或者你可以用 `OD` 打开任意一个程序，然后打开内存窗口查看模块间 `CODE` 段的距离，你会发现，这远大于这个值。当然更好的做法是将此常量替换成未加壳程序的 `CODE` 段大小。
