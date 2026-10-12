
# 关于《Foxit Phantom 的完美破解》的一点补充打印驱动破解

> 原文：`www.pediy.com/kssd/pediy11/120824.html`

【破解作者】`solo_lxy`

【使用工具】`OD`, `IDA Pro`

【软件名称】`FoxitPhantom 1.0.2.1123`

【软件简介】`FoxitPhantom` 是一个商业级的 `PDF` 应用软件，是一个集创建，阅读和编辑 `PDF` 为一体的新工具。

【下载地址】 `http://www.skycn.com/soft/57351.html`

【破解声明】 只是感兴趣，没有其他目的。失误之处敬请赐教！

论坛上 `TopmenC` 的一篇关于《Foxit Phantom 的完美破解》（`http://bbs.pediy.com/showthread.php?t=102691`）对 `FoxitPhantom 1.0.2.1123` 的破解给我们提供了重要的线索。其中最关键的地方就是在 `0044DB23` 位置处将 `jnz` 修改为如下：

```
0044DB23   . /E9 89020000    jmp     0044DDB1

0044DB28      |90             nop
```

原来的 `jnz` 指令占 6 个字节，改后分解为两个指令，在 `OD` 中这个改变自动完成，我们只需要按空格键，将 `jnz` 该为 `jmp` 即可。这样整个软件基本上绝大部分功能都能用了，真的很好用。不过，我还是在使用的过程中发现了有个小瑕疵。

安装完 `FoxitPhantom 1.0.2.1123` 后，如果你系统中安装了 `Office Word`，它会自动地在 `word` 中添加一个工具栏，通过这个工具栏我们可以直接将 `word` 文档转换成 `pdf` 格式。以前为了使用这个功能，我好象是要下载安装一个忘了叫什么名字的驱动，现在好了只要安装了 `FoxitPhantom`，阅读、编辑、转换 `pdf` 文档一个工具就搞定了。

`[CENTER]![](img/880f9524140309a9f0e5e2cefcbd56ba.jpg)CENTER]`

这时，如果你直接点击转换按钮，还是会出现天数限制对话框，30 天的限制时间后你就不能再用了。

过期后，`word` 中的这个工具栏也就自动卸载了。也就是说仅仅对 `FoxitPhantom` 主程序做如上的修改并不是很完善。

既然提供了文档转换功能，那按经验，肯定是在系统中安装了虚拟的 `pdf` 打印机。我的电脑->控制面板->打印机和传真，果然我的打印机设置页面多了个 `Foxit Phantom Printer` 打印机：

`![](img/f9a2adfe349094505de3f69495117870.jpg)` 为了找出 `Foxit Phantom` 安装的虚拟驱动，我随便新建了个 `test.txt` 文档，用 `notepad` 打开并打印，这时 `Foxit Phantom` 会打开主程序，并提示会提醒你：

`![](img/c8ef58627309288013e319800cfb38d2.jpg)`

这里请注意，`Foxit Phantom` 主程序是可以关闭的，并不影响打印。通过上图状态栏的提示我们可以发现，打印机驱动程序代码直接运行在 `notepad` 的进程空间中。这时，用 `OD` 附加 `notepad`(测试文档:`test.txt`),`ALT+M` 寻找可以模块，很快发现 `notepad` 进程中有个 `fpmvpr_d` 的模块，这个很有可能是 `Foxit Phantom` 的虚拟打印机驱动。搜索 `fpmvpr_d` 文件，发现以下两位置有此文件：`C:\WINDOWS\system32\spool\drivers\w32x86\3`；`C:\WINDOWS\system32\spool\drivers\w32x86\`，文件全称 `fpmvpr_drv.dll`。查看文件属性可发现其描述为：`Foxit Phantom Printer: Virtual Printer Driver`。通过这我们可以断定这个就是我们要找的驱动。不过这里有两个一样名字的文件，只不过储存位置不同。要想知道那个是我们关注的，这里有个最简单快速的办法，在 `notepad` 中点击打印选择打印机确定后，分别删除上面两个文件夹中的 `fpmvpr_drv.dll` 文件，这样很快知道 `C:\WINDOWS\system32\spool\drivers\w32x86\3` 中的 `fpmvpr_drv.dll` 正是我们要找的（因为系统不让我们删）。

用 `Resource Hacker` 打开 `fpmvpr_drv.dll`，查看对话框资源发现 124（16 进制 `7C`）号对话框就是我们上面看到的天数限制对话框。用 `OD` 加载，查找常量 `7C`，很快就定位到如下代码（通过字符串参考“`Sorry,your free trial period for Foxit Phantom has expired!` ”同样可以）：

```
10027704  |.   3BC3           cmp     eax, ebx

10027706  |.   A1 50B10D10    mov     eax, dword ptr [100DB150]

1002770B      /74 31          je      short 1002773E ；改为 jmp

1002770D  |. |3BC3           cmp     eax, ebx

1002770F      |74 12         je      short 10027723  ; 注意这里的跳转：会弹出过期对话框，程序直接退出，要将这里的 je 改为 jmp (可不改，上面已经跳过了)

10027711  |. |53             push    ebx                          ; /Style

10027712  |. |68 B02F0D10    push    100D2FB0                      ; |Title = "Foxit Phantom"

10027717  |. |68 C42E0D10    push    100D2EC4                      ; |Text = "Sorry, your free trial period for Foxit Phantom has expired!.

                            ; Please visit the Foxit Phantom site to order it online."

1002771C  |. |53             push    ebx                          ; |hOwner

1002771D  |. |FF15 84720910  call    dword ptr [<&USER32.MessageBoxW>>; \MessageBoxW

10027723  |> |891D 50B10D10  mov     dword ptr [100DB150], ebx

10027729  |. |33C0          xor     eax, eax

1002772B  |. |8B4D F4       mov     ecx, dword ptr [ebp-C]

1002772E  |. |64:890D 00000>mov     dword ptr fs:[0], ecx

10027735  |. |5F             pop     edi

10027736  |. |5E             pop     esi

10027737  |. |5B             pop     ebx

10027738  |. |8BE5          mov     esp, ebp

1002773A  |. |5D             pop     ebp

1002773B  |. |C2 0C00       retn    0C

1002773E  |> \3BC3          cmp     eax, ebx

10027740      74 17         je      short 10027759            ; 在这里将 je 改为 jmp，跳过下面的对话框

10027742  |.   8B15 40A70D10 mov     edx, dword ptr [100DA740]

10027748  |.   68 10710210   push    10027110

1002774D  |.   53             push    ebx

1002774E  |.   6A 7C         push    7C                         ; 注意这里就是弹出还剩下多少天对话框的 ID=124=0x7C

10027750  |.   52             push    edx

10027751  |.   E8 EA9CFDFF   call    10001440

10027756  |.   83C4 10      add     esp, 10

10027759  |> \891D 50B10D10 mov     dword ptr [100DB150], ebx

1002775F  |.   885D FC      mov     byte ptr [ebp-4], bl

10027762  |.   897D EC      mov     dword ptr [ebp-14], edi

10027765  |>   8B45 08      mov     eax, dword ptr [ebp+8]

10027768  |.   8B70 08      mov     esi, dword ptr [eax+8]

1002776B  |.   8B8E BC070000 mov     ecx, dword ptr [esi+7BC]

10027771  |.   3BCB         cmp     ecx, ebx

10027773  |.   75 5A        jnz     short 100277CF ;这里如果跳转，就直接退出函数，在这里 nop 掉 jnz

10027775  |.   68 58040000  push    458

1002777A  |.   E8 F52E0000  call    1002A674

1002777F  |.   8BC8         mov     ecx, eax

10027781  |.   83C4 04      add     esp, 4

10027784  |.   894D 08      mov     dword ptr [ebp+8], ecx

10027787  |.   3BCB         cmp     ecx, ebx

10027789  |.   C645 FC 02   mov     byte ptr [ebp-4], 2

1002778D  |.   74 09        je      short 10027798

1002778F  |.   E8 9CB0FDFF  call    10002830

10027794  |.   8BC8         mov     ecx, eax

10027796  |.   EB 02        jmp     short 1002779A

10027798  |>   33C9         xor     ecx, ecx

1002779A  |>   56             push    esi                          ; /Arg1

1002779B  |.   885D FC      mov     byte ptr [ebp-4], bl            ; |

1002779E  |.   898E BC070000 mov     dword ptr [esi+7BC], ecx          ; |

100277A4  |.   E8 F7C2FDFF  call    10003AA0                       ; \fpmvpr_d.10003AA0

100277A9  |.   8B4D 10      mov     ecx, dword ptr [ebp+10]

100277AC  |.   8B55 0C      mov     edx, dword ptr [ebp+C]

100277AF  |.   51             push    ecx                          ; /Arg2

100277B0  |.   8B8E BC070000 mov     ecx, dword ptr [esi+7BC]       ; |

100277B6  |.   52             push    edx                          ; |Arg1

100277B7  |.   E8 34C7FDFF  call    10003EF0                       ; \fpmvpr_d.10003EF0  ; 开启一个线程，转换 pdf

100277BC  |.   8B4D F4      mov     ecx, dword ptr [ebp-C]

100277BF  |.   64:890D 00000>mov     dword ptr fs:[0], ecx

100277C6  |.   5F             pop     edi

100277C7  |.   5E             pop     esi

100277C8  |.   5B             pop     ebx

100277C9  |.   8BE5          mov     esp, ebp

100277CB  |.   5D             pop     ebp

100277CC  |.   C2 0C00       retn    0C

100277CF  |>   8B7D 10      mov     edi, dword ptr [ebp+10]

100277D2  |.   3BFB         cmp     edi, ebx

100277D4  |.   74 40        je      short 10027816

100277D6  |.   E8 B5B5FDFF  call    10002D90

100277DB  |.   8B8E BC070000 mov     ecx, dword ptr [esi+7BC]

100277E1  |.   56             push    esi                          ; /Arg1

100277E2  |.   E8 B9C2FDFF  call    10003AA0                       ; \fpmvpr_d.10003AA0

100277E7  |.   8B86 BC070000 mov     eax, dword ptr [esi+7BC]

100277ED  |.   8B4D 0C      mov     ecx, dword ptr [ebp+C]

100277F0  |.   57             push    edi                          ; /Arg2

100277F1  |.   51             push    ecx                          ; |Arg1

100277F2  |.   8998 A0010000 mov     dword ptr [eax+1A0], ebx       ; |

100277F8  |.   8B8E BC070000 mov     ecx, dword ptr [esi+7BC]       ; |

100277FE  |.   E8 EDC6FDFF  call    10003EF0                       ; \fpmvpr_d.10003EF0

10027803  |.   8B4D F4      mov     ecx, dword ptr [ebp-C]

10027806  |.   64:890D 00000>mov     dword ptr fs:[0], ecx

1002780D  |.   5F             pop     edi

1002780E  |.   5E             pop     esi

1002780F  |.   5B             pop     ebx

10027810  |.   8BE5          mov     esp, ebp

10027812  |.   5D             pop     ebp

10027813  |.   C2 0C00       retn    0C
```

上面的反汇编代码中，我已经做了关键注释，破解该驱动可以做如下修改：

1. `1002770B` 将 `je` 改为 `jmp`
2. `10027740` 将 `je` 改为 `jmp`



### 10027773 将 `jnz` 改为 `nop`

通过如上修改，基本上我想要的功能算是全了。

上面的代码分析，其实我在做的时候主要使用的还是 `IDA Pro`，通过 `IDA Pro` 反汇编出来的代码还是比 `OD` 方面看多了。不过使用 `IDA Pro`，不太好查找 `UNICODE` 字符串参考，所以辅助使用了 `OD`。

这里有个疑问，还请知道的认识颠簸。通过如上分析我们知道打印的时候使用了 `fpmvpr\_drv.dll`，也就是说 `fpmvpr\_drv.dll` 存在于你用于打印的程序空间中，那么如何才能用 `OD` 动态调试 `fpmvpr\_drv.dll` 能，我在破解的时候发现，只要一用 `OD` 附加，再执行打印，`OD` 就直接死了。各位有办法的话，请指教一二，谢谢。

这段时间的使用，发现 `Foxit Phantom` 真的是很好用，比 `Adobe Reader` 要小，要快同时可编辑 `pdf` 稳当，支持多种格式到 `pdf` 的转换，作为一般用户，有这个软件，处理阅读 `pdf` 文档的确足够。

我这里有全部修改的破解文件，有需要的可以 `E-mail` 我，其实看了上面的解释，改动也不多，只有两个文件共 4 处补丁。
