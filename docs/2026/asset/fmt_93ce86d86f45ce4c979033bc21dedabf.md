
```
035349D0 FF15 E8645403 call dword ptr [35464E8] ; USER32.GetDlgItem
035349D6 50 push eax
035349D7 FF15 E0645403 call dword ptr [35464E0] ; USER32.GetWindowTextA
035349DD 8D85 00FFFFFF lea eax, dword ptr [ebp-100]
035349E3 50 push eax
035349E4 E8 35060100 call 0354501E ; jmp 到 msvcrt.strlen
035349E9 85C0 test eax, eax
035349EB 59 pop ecx
035349EC 74 1A je short 03534A08
035349EE 8B0D E01E5503 mov ecx, dword ptr [3551EE0]
035349F4 8D85 00FFFFFF lea eax, dword ptr [ebp-100]
035349FA 50 push eax
035349FB E8 7D77FEFF call 0351C17D ; 比较原始密码
03534A00 84C0 test al, al
03534A02 74 04 je short 03534A08 ; 不是原码，则跳转(不跳)
03534A04 6A 01 push 1
03534A06 EB 4C jmp short 03534A54 ; 回到正常流程
03534A08 33F6 xor esi, esi
```

接着修改魔法跳，可以使用`bp GetModueHandle`或者`HE GetModuleHandle`。这里发现了`kernel32.dll`就可以执行到返回查看了。

然后就是找OEP，这里还是可以使用2.2中对`GetProcessId`下断。这里介绍个新方法。可以在内存窗口`.text`段按F2下断点。因为壳执行完肯定会执行代码段的内容。也就是说代码段是由外壳到源程序的一扇门。所以在此处下断必然成立。

```
004010CC 55 push ebp
004010CD 8BEC mov ebp, esp
004010CF 83EC 44 sub esp, 44
004010D2 56 push esi
004010D3 FF15 E4634000 call dword ptr [4063E4] ; kernel32.GetCommandLineA
```

总结一下，现在有两个方法可以找OEP。第一是对`GetProcessId`下断，第二个就是在`.text`下断。

### 0x2.4 包含Code Splicing和Import Table Elimination的修复

Armadillo使用Code Splicing和Import Table Elimination两项技术使得程序修复变得更加困难。幸好有大佬开发了ArmInline工具可以使得修复变得简单一些。注意：本节只将修复，不讲程序优化。

当我们寻找到OEP之后，就可以着手修复Code Splicing和Import Table Elimination了。

```
0040C434 55 push ebp
0040C435 8BEC mov ebp,esp
0040C437 6A FF push -0x1
0040C439 68 28334100 push fraps.00413328
0040C43E 68 30E94000 push fraps.0040E930
0040C443 64:A1 00000000 mov eax,dword ptr fs:[0]
0040C449 50 push eax
0040C44A 64:8925 0000000>mov dword ptr fs:[0],esp
```

首先祭上大杀器ArmInline，欲要善其事，必先利其器。需要我们填写的就是上述三个区域，不过我这个版本可以自动填写修复的数据，只需要知道我们需要修复的进程，如图，目标进程ID为`FC4`，选中后依次删除拼接代码和巡回IAT基址。

![](upload/attach/202001/739734\_U4QJJEWGS794E6U.jpg)

然后按照常规的方法`dump`和修复IAT就可以了。注意的是使用PELord一定要勾选**从磁盘粘贴文件头(一般默认勾选上了)**

如果你的ArmInline不能自己修复(反正牛逼的师傅都是自己修复的，我不牛逼所以都是软件自动修复的)，关于**Code Splicing**的修复可以这样，Alt+M到内存窗口，在fraps模块之后有一段内存没有被其他模块映射(不知道这样说对不对，反正对于`Kernel32`这样的dll来说肯定是对的。大家理解就好)。**在最后一块内存处**，就是拼接代码起点，这个值不是一个定值。(这个只是经验之谈，需要大佬解释一波的)

![](upload/attach/202001/739734\_NDA6DJDCFX8K535.jpg)

接着修正IAT乱序，首先随便找个函数调用，在信息窗口点`数据窗口跟随地址`，然后向上拖动窗口(你最好改成**显示地址**)。找到IAT起始地址，然后找到结束地址，两者相减。计算大小即可。关于填充地址。可以考虑在一块没有读写的空白区域就好。不过大佬给的建议是在程序加壳前原来IAT的相近地方。可以这样寻找。Alt+M到内存窗口，因为IAT早`rdata`区域，又因为IAT肯定保存了一些IID成员，其中有个`Name`成员，也就是`DllName`。我们通过全局搜索确定

![](upload/attach/202001/739734\_JUET3EJQAC42CNX.jpg)

## 0x3 Armadillo双进程脱壳

这一章节主要讲穿山甲的双进程保护手段。所以双进程保护，简单的来说就是创建两个进程，一个进程是另外一个进程的调试进程，又由于在R3下面一个进程只能被一个调试器附加。这样可以有效避免程序被调试。

### 0x3.1 标准保护

接下来简单讲解一下关于双进程保护的原理，主要可以利用互斥体来判断进程列表是否存在相同的进程(即多开)。首先是利用`CreateMutex`创建一个互斥体。然后在利用`OpenMutex`打开那个互斥体，如果`OpenMutex`成功返回互斥体句柄，说明已经存在一个进程。如果不存在则在`CreateProcess`一个进程。而对于穿山甲壳双转单也是如此。**如果提前创建了一个即将被打开的互斥体。那么程序就不会去创建新的进程。**如下的脱壳方法就是基于这点考虑。

首先对`openMutex`下断点(`HE`,`bp`皆可)。`HE openMutexA`,然后shift+F9。观察堆栈

```
0012F798 00434DB8 /CALL 到 OpenMutexA 来自 NOTEPAD\_.00434DB2
0012F79C 001F0001 |Access = 1F0001
0012F7A0 00000000 |Inheritable = FALSE
0012F7A4 0012FDD8 \MutexName = "8A4::DABDC997F2" ；这是是重点，标记了互斥体名称。记住堆栈地址0012FDD8，以后要用
```

接着需要创建互斥体。转到`401000`处编写汇编代码，为什么需要`401000`，因为这是`.text`段，但是理论上在哪里修改都可以。然后将EIP修改到`401000`处，就可以在这里执行了，然后shitf+F9.

```
00401000 60 pushad ;保存所有寄存器
00401001 9C pushfd ;保存标志寄存器
00401002 68 F8FB1200 push 0012FDD8 ;堆栈里看到的值MutexName
00401007 33C0 xor eax,eax
00401009 50 push eax ;参数2
0040100A 50 push eax ;参数1
0040100B E8 B5A6A577 call kernel32.CreateMutexA ；创建互斥体
00401010 9D popfd
00401011 61 popad ;恢复
00401012 - E9 7A13A677 jmp kernel32.OpenMutexA ;打开互斥体
```

接着就是处理加密IAT和跳转OEP,DUMP的问题了。最后到达OEP如下：

```
004010CC 55 push ebp
004010CD 8BEC mov ebp,esp
004010CF 83EC 44 sub esp,0x44
004010D2 56 push esi
004010D3 FF15 E4634000 call dword ptr ds:[0x4063E4] ; kernel32.GetCommandLineA
004010D9 8BF0 mov esi,eax
```

### 0x3.2 CopyMem-II

去除CopyMem-ll 保护通常有两个方法。

方法1：首先寻找OEP，然后对`WaitForDebugEvent`下断点`bp WaitForDebugEvent`,接着运行程序，看堆栈，当出现`pDebugEvent`字符的时候，选择在数据窗口跟随，然后对`WriteProcessMemory`下断`bp WriteProcessMemory`，中断后，在数据窗口发现OEP。

这里重点讲一下第二个方法：

* `bp WaitForDebugEvent`，shift+F9运行起来，删除断点，然后执行到程序领空,大概停在`0060F8BA`

```
0060F8BA ? 15 E0406400 adc eax,<&KERNEL32.WaitForDebugEvent>
0060F8BF . 85C0 test eax,eax
0060F8C1 . 0F84 2B270000 je MAGCT.00611FF2
0060F8C7 . 8B85 FCFDFFFF mov eax,dword ptr ss:[ebp-0x204]
0060F8CD . 25 FF000000 and eax,0xFF
0060F8D2 . 85C0 test eax,eax
0060F8D4 . 74 13 je XMAGCT.0060F8E9
0060F8D6 . 8B0D 44AF6400 mov ecx,dword ptr ds:[0x64AF44]
0060F8DC . 8379 20 00 cmp dword ptr ds:[ecx+0x20],0x0
0060F8E0 . 74 07 je XMAGCT.0060F8E9
0060F8E2 . C685 FCFDFFFF>mov byte ptr ss:[ebp-0x204],0x0
0060F8E9 > 68 38AE6400 push MAGCT.0064AE38 ; /pCriticalSection = MAGCT.0064AE38
0060F8EE . FF15 A4416400 call dword ptr ds:[<&KERNEL32.EnterCriti>; \EnterCriticalSection
```

* 然后Ctrl+F搜索命令：`or eax，0FFFFFFF8`,想上看有两个比较，一个是`cmp dword ptr ss:[ebp-0xA34]`,另外一个是`cmp ecx,dword ptr ds:[0x64AF48]`，然后对第一个`cmp`下断点，F9运行。这一步你需要记住以下内容，等下patch的时候需要用到内容，第一:第一个`cmp`的地址`0060FE43`,第二：第一个`cmp`【】内的值`ebp-0xA34`,第三：第二个`cmp`【】的值：`0x64AF48`.除此以外，需要将次一个`cmp`栈里面的数据清0

```
0060FE43 > \83BD CCF5FFFF 00 cmp dword ptr ss:[ebp-0xA34],0x0 ; 3
0060FE4A . 0F8C A8020000 jl MAGCT.006100F8 ;在目的地址006100F8下断点
0060FE50 . 8B8D CCF5FFFF mov ecx,dword ptr ss:[ebp-0xA34]
0060FE56 . 3B0D 48AF6400 cmp ecx,dword ptr ds:[0x64AF48] ; 2
0060FE5C . 0F8D 96020000 jge MAGCT.006100F8
0060FE62 . 8B95 40F6FFFF mov edx,dword ptr ss:[ebp-0x9C0]
0060FE68 . 81E2 FF000000 and edx,0xFF
0060FE6E . 85D2 test edx,edx
0060FE70 . 0F84 AD000000 je MAGCT.0060FF23
0060FE76 . 6A 00 push 0x0
0060FE78 . 8BB5 CCF5FFFF mov esi,dword ptr ss:[ebp-0xA34]
0060FE7E . C1E6 04 shl esi,0x4
0060FE81 . 8B85 CCF5FFFF mov eax,dword ptr ss:[ebp-0xA34]
0060FE87 . 25 07000080 and eax,0x80000007
0060FE8C . 79 05 jns XMAGCT.0060FE93
0060FE8E . 48 dec eax
0060FE8F . 83C8 F8 or eax,0xFFFFFFF8 ; 1
0060FE92 . 40 inc eax
```

* 接下来patch数据，我们向下看，找到`add eax,0xff`语句，在这里就可以patch了

```
//原始数据
0060FF16 25 FF000000 and eax,0xFF ; patch
0060FF1B 85C0 test eax,eax
0060FF1D 0F84 D5010000 je MAGCT.006100F8
0060FF23 837D D8 00 cmp dword ptr ss:[ebp-0x28],0x0
0060FF27 75 27 jnz XMAGCT.0060FF50
0060FF29 8B15 D0436400 mov edx,dword ptr ds:[0x6443D0]
//
//patch模块
inc dword ptr ds：[] //第一个CMP内的值
mov dword ptr ds:[XXXX+4],1 //XXXX为第二个CMP[]内的值
jmp XXXX //第一个CMP前的地址
//
//patch后的数据
0060FF16 FF85 CCF5FFFF inc dword ptr ss:[ebp-0xA34] ; patch
0060FF1C C705 4CAF6400 010000>mov dword ptr ds:[0x64AF4C],0x1
0060FF26 ^ E9 18FFFFFF jmp MAGCT.0060FE43
0060FF2B 90 nop
0060FF2C 90 nop
0060FF2D 90 nop
```

* 接着shift+F9,中断在`006100F8`处,就可以dump处子进程了

```
006100EE > \C785 D8F5FFFF 020001>mov dword ptr ss:[ebp-0xA28],0x10002 ; UNICODE "::=::\"
006100F8 > E9 D4100000 jmp MAGCT.006111D1
006100FD > 8B0D B0436400 mov ecx,dword ptr ds:[0x6443B0]
00610103 . 81F1 050000C0 xor ecx,0xC0000005
00610109 . 398D D4F5FFFF cmp dword ptr ss:[ebp-0xA2C],ecx
0061010F . 0F85 92040000 jnz MAGCT.006105A7
00610115 . 70 07 jo XMAGCT.0061011E
00610117 . 7C 03 jl XMAGCT.0061011C
00610119 > EB 05 jmp XMAGCT.00610120
```
