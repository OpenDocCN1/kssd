
# Armadillo 双进程标准壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6446.htm`](https://www.pediy.com/kssd/pediy06/pediy6446.htm)

壹次脱壳法——Armadillo 双进程标准壳 快速脱壳

下载页面： `http://www.skycn.com/soft/11437.html`

软件大小： 1686 KB

软件语言： 英文

软件类别： 国外软件 / 共享版 / 图像捕捉

应用平台： `Win9x/NT/2000/XP`

加入时间： 2003-11-19 10:36:07

下载次数： 1332

推荐等级： *****

【软件简介】：`Mr.Captor` 是一个非常好用的屏幕截图程序!他可以帮助你把看到的任何图片忠实的保留下来!并可以保存为 `BMP`, `PCX`, `GIF`, `JPEG`, `PNG`, `TIFF`, `TGA`, `CUR`, `ICO`, `AVI` 等格式!支持自定义热键!非常容易使用呀!

【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC 1.6`

—————————————————————————————————

【脱壳过程】：

这次试验的目标是 `Mr.Captor` V2.7 版。这个版本是 N 久以前下载的，小猫上网就不去找新版了。看到坛子上各位兄弟脱 Armadillo 脱的如火如荼，呵呵，索性也滥竽充数来一篇啦，除了添柴加火外，没有什么价值啦。

`MrCaptor.exe` 用 `PEiD` 看：Armadillo 3.00a-3.61，运行看是双进程，其实还是标准壳。

以前脱 Armadillo 标准壳一般有两种流程的操作方法：

1、先走到 OEP，DUMP 后第 2 次调试，直接修改 Magic Jump 处为 JMP，得到输入表。

2、在 Magic Jump 处下硬件断点，每次改变标志位使其跳转。这样也是一次就脱壳、修复输入表。但是很慢，如果碰到加密函数多的程序就会吐血的。

OK，请看一下我的方法，壹次脱壳法——Armadillo 双进程标准壳 快速脱壳 :-)

设置 `Ollydbg` 忽略所有的异常选项。老规矩：用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

————————————————————————

## 一、使程序把自己当成子进程运行

```asm
004BC000     60                   pushad//进入 OD 后停在这！
004BC001     E8 00000000         call MrCaptor.004BC006
004BC006     5D                   pop ebp
004BC007     50                   push eax
004BC008     51                   push ecx
004BC009     EB 0F                jmp short MrCaptor.004BC01A
```

下断：`BP OpenMutexA` 当然是用 mysqladm 大虾的方法啦

```asm
77E6074A     55                   push ebp//断在这，看看堆栈
77E6074B     8BEC                 mov ebp,esp
77E6074D     51                   push ecx
77E6074E     51                   push ecx
77E6074F     837D 10 00           cmp dword ptr ss:[ebp+10],0
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`BP OpenMutexA` 时的堆栈：

```text
0012F574   0049D7D2  /CALL 到 OpenMutexA 来自 MrCaptor.0049D7CC
0012F578   001F0001  |Access = 1F0001
0012F57C   00000000  |Inheritable = FALSE
0012F580   0012FBB4  MutexName = "50C::DAB0194A96"  ★注意 0012FBB4
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`Ctrl+G`：`401000` 键入以下代码

```asm
00401000     60                  pushad
00401001     9C                  pushfd
00401002     68 B4FB1200         push 12FBB4 ★ 堆栈里看到的值
00401007     33C0                xor eax,eax
00401009     50                  push eax
0040100A     50                  push eax
0040100B     E8 B4B2A577         call kernel32.CreateMutexA
00401010     9D                  popfd
00401011     61                  popad
00401012     E9 33F7A577         jmp kernel32.OpenMutexA
```

在 `401000` 处新建起源，`F9` 运行，再次中断在 `OpenMutexA` 处。

呵呵，可以把这段代码保存起来，下次用时直接粘贴，修改 `00401002` 处 `push` 的值就 OK 了。

—————————————————————————————————

## 二、Magic Jump，避开 IAT 加密

取消以前断点，下断：`BP GetModuleHandleA+5`

```asm
77E59F93     837C24 04 00        cmp dword ptr ss:[esp+4],0
77E59F98     0F84 23060000       je kernel32.77E5A5C1//断在这，注意看堆栈
77E59F9E     FF7424 04           push dword ptr ss:[esp+4]
77E59FA2     E8 55080000         call kernel32.77E5A7FC
77E59FA7     85C0                test eax,eax
77E59FA9     74 08              je short kernel32.77E59FB3
77E59FAB     FF70 04            push dword ptr ds:[eax+4]
77E59FAE     E8 B0060000         call kernel32.GetModuleHandleW
77E59FB3     C2 0400             retn 4
```

在这里中断十几次，然后 `Alt+F9` 返回程序。其实很好判断返回程序的时机。

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

注意看 `BP GetModuleHandleA+5` 时的堆栈变化：

```text
0012D3AC   00C86622  返回到 00C86622 来自 kernel32.GetModuleHandleA
0012D3B0   0012D4E8  ASCII "kernel32.dll"
0012D3AC   00C86622  返回到 00C86622 来自 kernel32.GetModuleHandleA
0012D3B0   0012D4E8  ASCII "user32.dll"
0012D3AC   00C86622  返回到 00C86622 来自 kernel32.GetModuleHandleA
0012D3B0   0012D4E8  ASCII "MSVBVM60.DLL"
0012D3AC   00C86622  返回到 00C86622 来自 kernel32.GetModuleHandleA
0012D3B0   0012D4E8  ASCII "advapi32.dll"
0012D638   00C9C236  返回到 00C9C236 来自 kernel32.GetModuleHandleA ★OK
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

当堆栈如上变化后，就可以 `Alt+F9` 返回程序代码了。

```asm
00C9C230     FF15 D4A0CA00       call dword ptr ds:[CAA0D4] ; kernel32.GetModuleHandleA
00C9C236     3945 E4             cmp dword ptr ss:[ebp-1C],eax//返回这里
00C9C239     75 09               jnz short 00C9C244
00C9C23B     C745 D0 40E2CA00    mov dword ptr ss:[ebp-30],0CAE240
00C9C242     EB 51               jmp short 00C9C295
00C9C244     391D 38E8CA00       cmp dword ptr ds:[CAE838],ebx
00C9C24A     BF 38E8CA00         mov edi,0CAE838
00C9C24F     74 44               je short 00C9C295//Magic Jump ★
00C9C251     8B35 5855CB00       mov esi,dword ptr ds:[CB5558]
00C9C257     A1 1C96CB00         mov eax,dword ptr ds:[CB961C]
00C9C25C     F647 08 01          test byte ptr ds:[edi+8],1
00C9C260     74 0E               je short 00C9C270
00C9C262     8B48 74             mov ecx,dword ptr ds:[eax+74]
00C9C265     3348 70             xor ecx,dword ptr ds:[eax+70]
00C9C268     3348 60             xor ecx,dword ptr ds:[eax+60]
00C9C26B     F6C1 80             test cl,80
00C9C26E     75 13               jnz short 00C9C283
```

以前脱 Armadillo 3.0 以上的壳时总是在 Magic Jump 处下 硬件执行 断点，每次断下后改标志 Z=1,使其 JMP。如果直接改为 JMP，程序会异常出错！Armadillo 旧版可以直接改的。今天晚上（或者说是凌晨啦）为了这个问题，我细心跟踪了数个小时，终于搞定了这个问题！

1、直接改 `00C9C24F` `jmp 00C9C295`，这样就不必下硬件断点每次改标志位了！ :-)

2、`Ctrl+F` 在当前位置查找命令： `salc` 在 `00C9C5A8` 处，当看到 `jmp`、`salc`、`salc` 代码连在一起时，呵呵，恭喜，找到地方了，在 `salc` 上面的 `jmp` 处下断！ 8)

```asm
00C9C594     FF75 F4             push dword ptr ss:[ebp-C]
00C9C597     E8 A7540000         call 00CA1A43
00C9C59C     FF75 BC             push dword ptr ss:[ebp-44]
00C9C59F     E8 8E430000         call 00CA0932
00C9C5A4     59                  pop ecx
00C9C5A5     59                  pop ecx
00C9C5A6     EB 03               jmp short 00C9C5AB//此处下断
00C9C5A8     D6                  salc
00C9C5A9     D6                  salc
```

`F9` 运行，断在 `00C9C5A6` 处。当然，由于是动态解码，每次调试看到的地址可能是不同的，但代码是相同的！其实断在 `00C9C5A6` 处时，IAT 解密已经完成了。现在我们返回 Magic Jump 处，改回原先修改的代码，在 `00C9C24F` 处点右键－>“撤销选择”即可。

```asm
00C9C24F     EB 44               jmp short 00C9C295//修改的 Magic Jump，撤销选择
00C9C24F     74 44               je short 00C9C295//改回原先的代码
```

为何要这样做？我发现程序在下面会依据原先的代码进行解码，以前下 硬件断点 操作没有修改原代码，所以解码正确。而直接修改 Magic Jump 后改变了原先的代码，导致解码不正确而异常出错！现在我们在解码以前恢复原先的代码，因此就不会再出错了！ 8)

取消以前的所有断点。现在就可以在 `401000` 段下内存断点了，不用分两步来操作了，爽！

—————————————————————————————————

## 三、401000 段 内存断点 大法，直达 OEP

这个方法 jwh51 兄、fxyang 兄等许多兄弟经常使用，我试了一下，果然挺爽。

`Alt+M` 查看内存，在 `401000` 开始的段上下 内存访问断点，`F9` 运行，直接中断在 OEP 处

```asm
004378EF     55                  push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
004378F0     8BEC                mov ebp,esp
004378F2     6A FF               push -1
004378F4     68 98AF4700         push MrCaptor.0047AF98
004378F9     68 546A4300         push MrCaptor.00436A54
004378FE     64:A1 00000000      mov eax,dword ptr fs:[0]
00437904     50                  push eax
00437905     64:8925 00000000    mov dword ptr fs:[0],esp
0043790C     83EC 58             sub esp,58
0043790F     53                  push ebx
00437910     56                  push esi
00437911     57                  push edi
00437912     8965 E8             mov dword ptr ss:[ebp-18],esp
00437915     FF15 00F44600       call dword ptr ds:[46F400] ; kernel32.GetVersion
```

运行 `ImportREC 1.6`，选择这个进程。把 OEP 改为 `000378EF`，点 `IT AutoSearch`，`CUT` 掉无效函数。`FixDump`，正常运行！

测试了一下，这个快速脱壳发对于 fxyang 兄发的 围棋助手 等标准壳同样有效！

————————————————————————

## 四、破解

程序采用了 Armadillo 的注册模块，脱壳后自然就不必注册了，30 天的时间限制也不存在了。

“You have X days left for evaluation”的 NAG 可以修改以下地方：

```asm
00423BE8     3D E7030000         cmp eax,3E7
00423BED     74 3D               je short 00423C2C//改为 JMP
```

至于“Unregistered copy of Mr. Captor”等等字样就随你改啦。



```
—————————————————————————————————     

         ,      \_/   

        /| \_.-~/            \_     ,        青春都一饷   

       ( /~   /               ~-.\_ |   

       \`\\   \_/                    ~ )          忍把浮名   

   _-~~~-.)  )\_\_/;;,.             \_  //'   

  /\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   

 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`   

  /~~//'   /' \`~         ) /--..\_, )\_  \`~   

  "  \`~"  "      \`"     /~'\`   \`\\~~     


                    "     "   "~'  ""
```

Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]\[NUKE\]\[DCM\]

2004-03-16 03:33

`4dd466d0e1e0cc123afbe504da2462ca`
