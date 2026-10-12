
# 一个压缩壳偷 OEP 的脱壳与优化

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-655.htm`](https://www.pediy.com/kssd/pediy08/pediy8-655.htm)

一个压缩壳偷 OEP 的脱壳与优化

作者：CxLrb Email：cxlrb@yahoo.com.cn

目标：Registry Trash Keys Finder 3.7.2 (10-12-2006)

下载：`http://www.shura.totalcmd.net/download/trashreg.zip`

工具：DIE 0.52，OllyDBG 及脱壳插件，PETools，FixRes，ImportREC v1.6F，ResFix，PE Explorer 1.98

保护方式：偷 OEP，PEid 查不到，DIE 查壳：

编译器：Microsoft Visual Basic，加壳器：`.DotFix FakeSigner`

申明：仅仅是学习和研究目的，转载请保证文章完整性，并注明作者及出处，不足之处请指正！

## 概要

1. 寻找原始入口点(OEP)
2. 修复被偷去的 OEP
3. 转储及输入表修复
4. 优化 PE 文件
5. 结语
6. 感谢

## 寻找原始入口点(OEP)

我们将 `TrashReg.exe` 加载到 `peid`，什么都没找到，加载到 `DIE 0.52`，得到结果为,

编译器：Microsoft Visual Basic，加壳器：`.DotFix FakeSigner`

`OD` 调试选项全部打勾，载入 `TrashReg.exe`，“确定”略过入口点预警，继续分析代码选“否”，停在这里：

```
009750CC > /E9 FF000000     jmp     TrashReg.009751D0          -----入口
009750D1   |60              pushad
009750D2   |8B7424 24       mov     esi, [esp+24]
009750D6   |8B7C24 28       mov     edi, [esp+28]
009750DA   |FC              cld
009750DB   |B2 80           mov     dl, 80
009750DD   |33DB            xor     ebx, ebx
009750DF   |A4              movs    byte ptr es:[edi], byte ptr [esi]
009750E0   |B3 02           mov     bl, 2
009750E2   |E8 6D000000     call    TrashReg.00975154
009750E7  ^|73 F6           jnb     short TrashReg.009750DF
009750E9   |33C9            xor     ecx, ecx
009750EB   |E8 64000000     call    TrashReg.00975154
009750F0   |73 1C           jnb     short TrashReg.0097510E
```

试了一下，ESP 定律不凑效，那就单步跟踪，`F8` 单步大约 36 次，来到系统领空：

```
7C92EAF0    8B1C24         mov     ebx, [esp]
7C92EAF3    51             push    ecx
7C92EAF4    53             push    ebx
7C92EAF5    E8 C78C0200    call    ntdll.7C9577C1
7C92EAFA    0AC0           or      al, al
7C92EAFC    74 0C          je      short ntdll.7C92EB0A
7C92EAFE    5B             pop     ebx
7C92EAFF    59             pop     ecx
7C92EB00    6A 00          push    0
7C92EB02    51             push    ecx
7C92EB03    E8 11EBFFFF    call    ntdll.ZwContinue
7C92EB08    EB 0B          jmp     short ntdll.7C92EB15
7C92EB0A    5B             pop     ebx
7C92EB0B    59             pop     ecx
7C92EB0C    6A 00          push    0
```

现在的出路就只有想办法返回主线程，试了代码段内存断点，不凑效，只有继续亲自跟了，在这里我们按住 `Ctrl + F9` 不放大约 1 分钟，直到地址中出现 `0040XXXX`，当然不是很准确，但确实需要返回很多次才可以到达主线程，大概数了一下，需要 150 次，我们找一找就可以看到一大堆跳转，典型的 VB 入口处，

……

```
00403C18  - FF25 14124000   jmp     near [401214]                  ; MSVBVM50.__vbaStopExe
00403C1E  - FF25 9C124000   jmp     near [40129C]                  ; MSVBVM50.__vbaForEachAry
00403C24  - FF25 34124000   jmp     near [401234]                  ; MSVBVM50.__vbaR8Str
00403C2A  - FF25 80104000   jmp     near [401080]                  ; MSVBVM50.__vbaVarCmpNe
00403C30  - FF25 64114000   jmp     near [401164]                  ; MSVBVM50.__vbaVarLikeVar
00403C36  - FF25 B8114000   jmp     near [4011B8]                  ; MSVBVM50.EVENT_SINK_QueryInterface
00403C3C  - FF25 44114000   jmp     near [401144]                  ; MSVBVM50.EVENT_SINK_AddRef
00403C42  - FF25 A4114000   jmp     near [4011A4]                  ; MSVBVM50.EVENT_SINK_Release
00403C48  - FF25 90124000   jmp     near [401290]                  ; MSVBVM50.ThunRTMain
00403C4E    0000             add     [eax], al
00403C50    AD               lods    dword ptr [esi]
00403C51    FB               sti
00403C52    EC               in      al, dx
00403C53    B1 08            mov     cl, 8
00403C55    CA 1F7D          retf    7D1F
```

但是令人失望的是没有类似 VB 的入口语句，如：

```
Push 0040xxxx              ;  ASCII "VB5!6&*"
Call < jmp.&MSVBVM50.ThunRTMain >
```

我们先在这一句下硬件执行断点：

```
00403C48  - FF25 90124000   jmp     near [401290]                  ; MSVBVM50.ThunRTMain
```

免得下次跑掉了还要找很久才来到这里。

## 修复被偷去的 OEP

然后我们就想办法补回偷去的 OEP，很简单，我们就在 `00403C4E` 地址处开始写入类似上面的 2 行代码便可，`call` 后跟

```
00403C48  - FF25 90124000   jmp     near [401290]                  ; MSVBVM50.ThunRTMain
```

这一句的地址无疑，关键是找 `push` 后应该跟哪个地址，初步判断，应该跟存放类似 `"VB5!6&*"` 这样的 ASCII 码的地址值，我们在 `OD` 中往下找找看，如果没有找到，我们分析一下代码再找，

![](img/3afa1fc60df13c78283d3c72de9e163e.jpg)

找到这里：

```
00403F44     |56             db      56                       ;  CHAR 'V'
00403F45     |42             db      42                       ;  CHAR 'B'
00403F46     |35             db      35                       ;  CHAR '5'
00403F47     |21             db      21                       ;  CHAR '!'
00403F48     |8C             db      8C
00403F49     |0E             db      0E
00403F4A     |2A             db      2A                       ;  CHAR '*'
```

我们找到了，只要就要 `00403F44` 这个地址，回到上面的位置，在 `00403C4E` 处写入 2 行代码：

```
Push 00403F44
Call 00403C48
```

将附近几行代码都 `00` 填充(如果不填 `00`，经测试，运行脱壳后的文件在退出程序时出错)，修改后代码如下：

![](img/ae23354feefce6536ed844fe665206de.jpg)

```
00403C36   .- FF25 B8114000 jmp     near [4011B8]            ;  MSVBVM50.EVENT_SINK_QueryInterface
00403C3C   .- FF25 44114000 jmp     near [401144]            ;  MSVBVM50.EVENT_SINK_AddRef
00403C42   .- FF25 A4114000 jmp     near [4011A4]            ;  MSVBVM50.EVENT_SINK_Release
00403C48   .- FF25 90124000 jmp     near [401290]            ;  MSVBVM50.ThunRTMain
00403C4E     68 443F4000   push    TrashReg.00403F44
00403C53     E8 F0FFFFFF   call    TrashReg.00403C48         ;  jmp 到 MSVBVM50.ThunRTMain
00403C58     0000           add     [eax], al
00403C5A     00             db      00
00403C5B     00             db      00
00403C5C     00             db      00
00403C5D     00             db      00
00403C5E     00             db      00
00403C5F     00             db      00
00403C60     30             db      30                       ;  CHAR '0'
```

## 转储及输入表修复

接下来我们用 `Ollydump` 插件将程序 dump 出来，`OEP` 处填入 `00403c4e`，不要选择重建输入表，保存为 `dumped.exe`

![](img/cb2ebbc4830b9bcf7a4eaa597efd49c3.jpg)

用 `ImportREC v1.6F` 修复输入表，`OEP` 栏填入 `3c48`，获取输入函数全部有效：

![](img/def6b4e08fcea962866a932ade372236.jpg)

转储后保存为 `dumped_.exe`，程序已可以运行了！

## 优化 PE 文件

`dumped_.exe` 太大，我们来优化一下，我们用 `PE explorer` 查看一下，共有 8 个区段，其中有几个可能是垃圾区段，如图所示，

![](img/9dbd6234a97aa0de82ec0f60567d550b.jpg)

你可以删除其中几个试试，修复输入表后如果能够运行，那你就成功了，本人测试，5、6、7 区段删除后无影响。我们用专业的删除工具 `petools`，

删除程序的最后面 4 个区段，删除时选择从文件中删除区段，

![](img/d6d8c2a415514ae3b47cad927e6fbe65.jpg)

删除完了，剩下 4 个区段，然后修复一下输入表（`ImportREC` 可以一直不关），这是程序又可以运行起来了，大小为 444K。

对于非汉化目的的脱壳，在这里我们重建一下 PE 也许就可以达到目的了，但为了更完美，我们将资源区段放到最后，看我操作，用 `FixRes` 转储资源区段到磁盘，`RVA` 填入 `6f000`（注：`6f000` 为 PE 的当前的虚拟大小），文件对齐粒度填入 `200`，

![](img/029d871d15cfb1311104526d0467c403.jpg)

然后将转存的资源区段在载入到 PE 文件的最后一个区段，

![](img/adf632f961b05e262cd7fffb5e776047.jpg)

修改资源目录地址为 `6f000`，

![](img/892e27aaa732e2cd1d2cb2a99d36813f.jpg)

直接删除区段 `rsrc`，修复地址后出错，所以选择另一种办法，我们把原来的 `rsrc` 区段用 `00` 填充

![](img/299b235cc5f78c69eba4bb0a2f81d4fa.jpg)

然后将 `rsrc` 区段的虚拟大小值增加到其之前的一个区段的虚拟大小上，即 `4000+E000＝12000`，然后将 `rsrc` 区段从区段头删除，如下

![](img/f83210d7f29927ad2a1dfcba9cd7c589.jpg)

然后 `petools` 重建 PE，修改一下区段名，收工！重建选项一定不要勾选重建资源选项。

![](img/e08428907ae73e8809d2ed1b3dd2e341.jpg)

最后的文件用 `Resource 2006` 打开看不到资源，可以使用《汉化初学者进阶教程 3》中叙述的方法重建资源，这里不再赘述。

## 结语

只因为用了很多繁杂的操作，是因为 `rsrc` 区段后面那个区段在 PE 文件中的位置很重要，改变了程序就无法运行，如果你有更好的方法，可以拿出来分享，谢谢观赏！

## 感谢

一蓑烟雨，看雪论坛，汉化新世纪论坛！

以及各位成员提供的好工具！

2006.10.18 CxLrb

我也谈谈自己跟踪的结果:

1.`OD` 载入提示非法的 win32 程序

参考 `ohuangkeo` 文章:

`http://bbs.pediy.com/showthread.php?threadid=17592`

搞定.

2.楼主文章提到 VB,找 oep:

`OD` 载入运行程序,`Ctrl+G:401000`,搜索 `FF25`,可以看到:

```
00403C42   - FF25 A4114000    jmp dword ptr ds:[4011A4]    ; MSVBVM50.EVENT_SINK_Release
00403C48   - FF25 90124000    jmp dword ptr ds:[401290]    ; MSVBVM50.ThunRTMain
00403C4E     0000             add byte ptr ds:[eax],al
00403C50     AD               lods dword ptr ds:[esi]       ; here,oep
00403C51     FB               sti
00403C52     EC               in al,dx
```



`00403C53` `B1 08` `mov cl,8`  

`00403C55` `CA 1F7D` `retf 7D1F`  

明显可以看到 `oep=00403C50`  

代码被偷了 2 行.  

### 3.补代码:  

这里下硬件访问断点:  

`00403C48` `FF25 90124000` `jmp dword ptr ds:[401290]` `; MSVBVM50.ThunRTMain`  

断下时看堆栈:  

`0012FBF0` `00974BB4` `TrashReg.00974BB4`  

`0012FBF4` `00403F44` `TrashReg.00403F44` `// 这个就是 push`  

`0012FBF8` `0096E344` `返回到 TrashReg.0096E344 来自 TrashReg.0096E605`  

`0012FBFC` `77FBB272` `返回到 ntdll.77FBB272`  

被偷的代码:  

`push 00403f44`  
`call 403c48`  

至此可以搞定脱壳修复.  

### 4.如果想知道如何跟踪到达 oep,上 Rordbg:  

2 次异常,第二次异常如下:  

`026B0412` `正常调用(指令数 5104753): KERNEL32.DLL!VirtualAlloc`  

`0012FFE0` `Unknown Code:FF FF FF FF 6C` `; 这里异常`  

`0012FFE0` `Unknown Code:FF FF FF FF 6C`  

发生异常!  

`FS:[0]==0012FF98`  

`异常处理程序地址：0042B800` `; 注意这个异常处理地址`  

这个异常被成功捕获!  

`0096E150` `正常调用(指令数 5105123): KERNEL32.DLL!LoadLibraryA`  

`0096E2A5` `正常调用(指令数 5105350): KERNEL32.DLL!GetProcAddress`  

然后程序就运行了.  

### 5.OD 再上,在系统处理异常的地方下断:  

`77FBB270` `FFD1` `call ecx` `; 处理异常处`  

这个地址不同操作系统不一样的.  

`Shift+F9` 一直运行到 `rordbg` 跟踪的异常处理,就是这样结果:  

`77FBB270` `FFD1` `call ecx` `; TrashReg.0042B800`  

好了,没有异常了,可以开始跟踪了.  

先是返回到:  

`0096E000` `E8 0B000000` `call TrashReg.0096E010`  

`0096E005` `75 73` `jnz short TrashReg.0096E07A`  

`0096E007` `65:72 33` `jb short TrashReg.0096E03D`  

然后来到:  

`0096E33D` `/75 05` `jnz short TrashReg.0096E344` `; 跳转未实现`  

`0096E33F` `|E8 C1020000` `call TrashReg.0096E605` `; 跟进`  

`0096E344` `\E8 0D000000` `call TrashReg.0096E356`  

跟进后,里面有 `rdtsc` 检查,可以用工具搞定,我是手工修改跳转的.  

然后来到 `Stolen oep` 处:  

`00974B2D` `90` `nop`  

`00974B2E` `C74424 FC 443F40>` `mov dword ptr ss:[esp-4],TrashReg.00403F44`  

`00974B36` `EB 01` `jmp short TrashReg.00974B39`  

`00974B39` `83EC 04` `sub esp,4`  

`00974B3C` `51` `push ecx` `; =push 403F44`  

`00974B3D` `F9` `stc`  

`00974B3E` `1BC9` `sbb ecx,ecx`  

`00974B40` `EB 0B` `jmp short TrashReg.00974B4D`  

`00974BA3` `68 B44B9700` `push TrashReg.00974BB4`  

`00974BA8` `EB 01` `jmp short TrashReg.00974BAB`  

`00974BAB` `68 483C4000` `push TrashReg.00403C48` `; jmp to MSVBVM50.ThunRTMain,=call 403C48`  

`00974BB0` `EB 01` `jmp short TrashReg.00974BB3`  

`00974BB3` `C3` `retn` `; 返回到 00403C48 (TrashReg.00403C48)`  

返回到伪 `oep`:  

`00403C48` `FF25 90124000` `jmp dword ptr ds:[401290]` `; MSVBVM50.ThunRTMain`  

`00403C4E` `0000` `add byte ptr ds:[eax],al`  

`00403C50` `AD` `lods dword ptr ds:[esi]`  

这个就是整个跟踪过程.  

`4dd466d0e1e0cc123afbe504da2462ca`
