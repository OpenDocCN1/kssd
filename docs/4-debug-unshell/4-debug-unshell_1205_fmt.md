
# 一个菜鸟看 ZPROTECT 普通函数加密

> 原文：[`www.pediy.com/kssd/pediy10/66250.html`](https://www.pediy.com/kssd/pediy10/66250.html)

已经有脚本出来了 汗

大家好，我是 `lunglungyu`

用 `DEMO` 版加的 强度比较低 只是说说自己的思路 `_________________________________`

搞了下 `ZPROTECT` 脱壳 找回 己加密输入表 （低强度）

其他去掉 方便我菜鸟搞

只 `GETMODULEHANDLEA` 加密

文章写的不好 别见怪

```text
0040DACB  yh_pac>^\E9 4BFBFFFF          jmp 0040D61B                            ; yh_pac.0040D61B

0040DAD0        ^ 0F85 76FBFFFF         jnz 0040D64C                            ; yh_pac.0040D64C

0040DAD6          E9 BA010000           jmp 0040DC95                            ; yh_pac.0040DC95

0040DADB          8BF7                  mov esi,edi                             ; ntdll.7C930738

0040DADD        ^ E9 A7FDFFFF           jmp 0040D889                            ; yh_pac.0040D889

0040DAE2          C9                    leave

0040DAE3          06                    push es

0040DAE4        ^ 0F84 A2FEFFFF         je 0040D98C                             ; yh_pac.0040D98C

0040DAEA          E9 D7000000           jmp 0040DBC6                            ; yh_pac.0040DBC6

0040DAEF          C2 0400               retn 4

0040DAF2        ^ E9 0FFDFFFF           jmp 0040D806                            ; yh_pac.0040D806

0040DAF7          894424 10             mov dword ptr ss:[esp+10],eax

0040DAFB        ^ E9 20FDFFFF           jmp 0040D820                            ; yh_pac.0040D820

0040DB00          4E                    dec esi
```

开始的 `EP` 是如上这样的

`F8` 下

```text
0040D61B          E8 01000000           call 0040D621                           ; yh_pac.0040D621
```

看见上面的 `CALL`，跟（`F7`）

`F8` `4` 下

看见 `PUSHAD`

用 `ESP` 定律

`hr` `esp`

`F9` 运行

断下后去掉硬件断点

`F8` 到 `RETN`

便是 `OEP`

这个是其他文件载的 `@@`

```text
\------------------------------------

sto

sti

sto

sto

sto

sto

sto

BPHWS esp,"r"

run

bc esp

sto

sto

msg "reach oep"

ret

\-----------------------------
```

就是这角本 到 `oep`

偶写的不好 也加不上 `IAT` 修复

```text
00402317          6A 00                 push 0
00402319          E8 F2000000           call 00402410                           ; yh_pac.00402410
0040231E          A3 24404000           mov dword ptr ds:[404024],eax
00402323          6A 00                 push 0
00402325          68 75224000           push 402275
0040232A          6A 00                 push 0
0040232C          68 E8030000           push 3E8
00402331          FF35 24404000         push dword ptr ds:[404024]
00402337          E8 26000000           call 00402362                           ; jmp 到 USER32.DialogBoxParamA
0040233C          50                    push eax
0040233D          E8 B6000000           call 004023F8                           ; jmp 到 kernel32.ExitProcess
00402342          C3                    retn
00402343          CC                    int3
00402344        - FF25 EC304000         jmp dword ptr ds:[4030EC]               ; USER32.wsprintfA
0040234A        - FF25 E8304000         jmp dword ptr ds:[4030E8]               ; USER32.AppendMenuA
00402350        - FF25 E4304000         jmp dword ptr ds:[4030E4]               ; USER32.CallWindowProcA
00402356        - FF25 E0304000         jmp dword ptr ds:[4030E0]               ; USER32.CloseClipboard
0040235C        - FF25 DC304000         jmp dword ptr ds:[4030DC]               ; USER32.CreatePopupMenu
00402362        - FF25 D8304000         jmp dword ptr ds:[4030D8]               ; USER32.DialogBoxParamA
00402368        - FF25 D4304000         jmp dword ptr ds:[4030D4]               ; USER32.EnableWindow
0040236E        - FF25 D0304000         jmp dword ptr ds:[4030D0]               ; USER32.EndDialog
00402374        - FF25 CC304000         jmp dword ptr ds:[4030CC]               ; USER32.EnumChildWindows
0040237A        - FF25 C8304000         jmp dword ptr ds:[4030C8]               ; USER32.GetAsyncKeyState
00402380        - FF25 C4304000         jmp dword ptr ds:[4030C4]               ; USER32.GetClassNameA
00402386        - FF25 C0304000         jmp dword ptr ds:[4030C0]               ; USER32.GetClipboardData
0040238C        - FF25 BC304000         jmp dword ptr ds:[4030BC]               ; USER32.GetDlgItem
00402392        - FF25 B8304000         jmp dword ptr ds:[4030B8]               ; USER32.GetWindowLongA
00402398        - FF25 B4304000         jmp dword ptr ds:[4030B4]               ; USER32.GetWindowRect
0040239E        - FF25 B0304000         jmp dword ptr ds:[4030B0]               ; USER32.GetWindowTextA
004023A4        - FF25 AC304000         jmp dword ptr ds:[4030AC]               ; USER32.IsClipboardFormatAvailable
004023AA        - FF25 A8304000         jmp dword ptr ds:[4030A8]               ; USER32.LoadIconA
004023B0        - FF25 A4304000         jmp dword ptr ds:[4030A4]               ; USER32.MessageBoxA
004023B6        - FF25 A0304000         jmp dword ptr ds:[4030A0]               ; USER32.MoveWindow
004023BC        - FF25 9C304000         jmp dword ptr ds:[40309C]               ; USER32.OpenClipboard
004023C2        - FF25 98304000         jmp dword ptr ds:[403098]               ; USER32.ScreenToClient
004023C8        - FF25 94304000         jmp dword ptr ds:[403094]               ; USER32.SendDlgItemMessageA
004023CE        - FF25 90304000         jmp dword ptr ds:[403090]               ; USER32.SendMessageA
004023D4        - FF25 8C304000         jmp dword ptr ds:[40308C]               ; USER32.SetWindowLongA
004023DA        - FF25 88304000         jmp dword ptr ds:[403088]               ; USER32.SetWindowTextA
004023E0        - FF25 84304000         jmp dword ptr ds:[403084]               ; USER32.TrackPopupMenu
004023E6        - FF25 6C304000         jmp dword ptr ds:[40306C]               ; kernel32.CloseHandle
004023EC        - FF25 70304000         jmp dword ptr ds:[403070]               ; kernel32.CreateFileA
004023F2        - FF25 28304000         jmp dword ptr ds:[403028]               ; kernel32.DeviceIoControl
004023F8        - FF25 2C304000         jmp dword ptr ds:[40302C]               ; kernel32.ExitProcess
004023FE        - FF25 74304000         jmp dword ptr ds:[403074]               ; kernel32.FormatMessageA
00402404        - FF25 68304000         jmp dword ptr ds:[403068]               ; kernel32.GetCurrentDirectoryA
0040240A        - FF25 64304000         jmp dword ptr ds:[403064]               ; ntdll.RtlGetLastWin32Error
00402410        - FF25 60304000         jmp dword ptr ds:[403060]
00402416        - FF25 5C304000         jmp dword ptr ds:[40305C]               ; kernel32.GetProcAddress
0040241C        - FF25 58304000         jmp dword ptr ds:[403058]               ; kernel32.GetProcessHeap
00402422        - FF25 54304000         jmp dword ptr ds:[403054]               ; kernel32.GetVersion
00402428        - FF25 50304000         jmp dword ptr ds:[403050]               ; kernel32.GlobalLock
0040242E        - FF25 4C304000         jmp dword ptr ds:[40304C]               ; kernel32.GlobalUnlock
00402434        - FF25 48304000         jmp dword ptr ds:[403048]               ; ntdll.RtlAllocateHeap
0040243A        - FF25 44304000         jmp dword ptr ds:[403044]               ; ntdll.RtlFreeHeap
00402440        - FF25 40304000         jmp dword ptr ds:[403040]               ; ntdll.RtlZeroMemory
00402446        - FF25 3C304000         jmp dword ptr ds:[40303C]               ; kernel32.SetCurrentDirectoryA
0040244C        - FF25 38304000         jmp dword ptr ds:[403038]               ; kernel32.lstrcmpiA
00402452        - FF25 34304000         jmp dword ptr ds:[403034]               ; kernel32.lstrcpyA
```



```
00402458        - FF25 30304000        jmp dword ptr ds:[403030]                  ; kernel32.lstrlenA
0040245E        - FF25 14304000        jmp dword ptr ds:[403014]                  ; ADVAPI32.CloseServiceHandle
00402464        - FF25 04304000        jmp dword ptr ds:[403004]                  ; ADVAPI32.ControlService
0040246A        - FF25 08304000        jmp dword ptr ds:[403008]                  ; ADVAPI32.CreateServiceA
00402470        - FF25 0C304000        jmp dword ptr ds:[40300C]                  ; ADVAPI32.DeleteService
00402476        - FF25 18304000        jmp dword ptr ds:[403018]                  ; ADVAPI32.OpenSCManagerA
0040247C        - FF25 00304000        jmp dword ptr ds:[403000]                  ; ADVAPI32.OpenServiceA
00402482        - FF25 10304000        jmp dword ptr ds:[403010]                  ; ADVAPI32.StartServiceA
00402488        - FF25 20304000        jmp dword ptr ds:[403020]                  ; comdlg32.GetOpenFileNameA
0040248E        - FF25 7C304000        jmp dword ptr ds:[40307C]                  ; SHELL32.DragQueryFileA
00402494          0000                add byte ptr ds:[eax],al
00402496          0000                add byte ptr ds:[eax],al
```

好。看见了

这里注意下

我给加密的部分

```
00402410        - FF25 60304000        jmp dword ptr ds:[403060]
```

因为上面有调用

要`PROGRAM`运行一定要修复好，所以用`GETMODULEHANDLEA`这个，而不是那些运行到特定时候才调用的

跟进

`call 00402410`

就是给加密的`GETMODULEHANDLEA`

```
00EC0000          50                    push eax
00EC0001          60                    pushad
00EC0002          68 92D455AA           push AA55D492
00EC0007          E8 9404ECFF           call 00D804A0
00EC000C          61                    popad
00EC000D          C3                    retn
```

`call 00D804A0`这句是重点，我们进去

里面一堆`JMP`混淆

先探口风

```
00D9E8AB          8B0CB1                mov ecx,dword ptr ds:[ecx+esi*4]          ; kernel32.GetModuleHandleA
```

这句由`OD`的分析看见了是`getmodulehandleA`

`ds:[00DA42AC]=7C80B6A1 (kernel32.GetModuleHandleA)`

`ecx=00DA4250`

其实这里可以直接把它的位址放到`ptr ds:[403060]`

便弄成修复。。但是这是最简单的

我们看看他是如何搞出这地址的

`mov ecx,dword ptr ds:[ecx+esi*4]`

换言之，我们要知道`D9E8AB`时（在我的电脑上）的`ECX`和`ESI`哪里来的

`Ctrl + F2`重载在重复`STEPS`到`00EC0007 E8 9404ECFF call 00D804A0`这`CALL`

留意堆的变化

我略了当中那些混淆的`JMP`

因此代码不是跟次序的

程序行的先後

`mov eax,dword ptr ss:[esp+4]`当时的`ESP 12FF90`

`Eax`变成`AA55D492`

`push esi`

`FFFFFFFF`

`push eax`

刚才的`AA55D492`了

后来有一个`CALL`

`call dword ptr ds:[D8C514]`

我们先跳过

但注意堆化

前

`EAX AA55D492`

`ECX 0012FFB0`

`EDX 7C92EB94 ntdll.KiFastSystemCallRet`

`EBX 7FFD5000`

`ESP 0012FF88`

`EBP 0012FFF0`

`ESI FFFFFFFF`

`EDI 7C930738 ntdll.7C930738`

`EIP 00D9B19F`

后

`EAX 00000017`

`ECX 0012FFB0`

`EDX 7C92EB94 ntdll.KiFastSystemCallRet`

`EBX 7FFD5000`

`ESP 0012FF8C`

`EBP 0012FFF0`

`ESI FFFFFFFF`

`EDI 7C930738 ntdll.7C930738`

`EIP 00D9B1A5`

`EIP` `ESP`变化是正常的

经历这`CALL`

`eax`变了

看回头是中间

```
mov eax,dword ptr ss:[esp+24]
sub eax,17B415D1
xor eax,24DB7550
xchg edx,esi
lea eax,dword ptr ds:[eax+F792273]
```

等等作的

这里我们不理

我们看见

```
00D94C12          8B0D 28CCD800         mov ecx,dword ptr ds:[D8CC28]
```

是`ECX`！！！！

`ecx`变了`00DA4250`

`test ecx,ecx`

`mov esi,eax`原来这里`ESI`跟`EAX`有关

`EAX`经过运算的值搬到了`ESI`

```
mov eax,dword ptr ds:[D8CC2C]
sub eax,ecx
sar eax,2
cmp esi,eax
```

最后我们又到了`mov ecx,dword ptr ds:[ecx+esi*4]`

`ds:[00DA42AC]=7C80B6A1 (kernel32.GetModuleHandleA)`

因为偶`ASSEMBLY`不好

所以中间不能说个明白

就是`zprotect`加壳是把`API`的位址拆解，然后用算法合回

当中主要用到`EAX`运算

然后最后再把不同的需要值放到`ECX`和`ESI`算回`API`位址

在来

```
mov dword ptr ss:[esp+2C],ecx
```

这里是吧`GETMODULEHANDLEA`这个的地址记录

`POP ESI`恢复`FFFFFFFF`原的值

`return`出`CALL`

在`return`

就到了`GETMODULEHANDLE`的地址。

`API`行後回到程序代段大概就是这样

就是由`EP`附近的

```
mov dword ptr ds:[404024],eax
```

行下去

每都是这样.

这就是`zprotect`导入函数加密中普通加密的部分

下面我们在来写清楚一下

其实

当我们到达`OEP`时

```
00402317          6A 00                 push 0
00402319          E8 F2000000           call 00402410
          ; yh\_packe.00402410
```

这个就是跳往加密`API`的

即

```
00402410        - FF25 60304000        jmp dword ptr ds:[403060]
```

因为我只加密这个，所以很容易改回

跟进去

```
00EC0000          50                    push eax
00EC0001          60                    pushad
00EC0002          68 92D455AA           push AA55D492
00EC0007          E8 9404ECFF           call 00D804A0
00EC000C          61                    popad
00EC000D          C3                    retn
00EC000E          0000                add byte ptr ds:[eax],al
```

可以知道`call 00D804A0`后`API`会给弹进堆

/在`FPU`中

`F4`到`00EC000C 61 popad`

其实已经可以看见给加密的函数

`EAX 00000037`

`ECX 7C80B6A1 kernel32.GetModuleHandleA`就是这里

`EDX 7C92EB94 ntdll.KiFastSystemCallRet`

`EBX 7FFD9000`

`ESP 0012FF98`

`EBP 0012FFF0`

`ESI FFFFFFFF`

`EDI 7C930738 ntdll.7C930738`

`EIP 00EC000C`

或是。在

```
0012FF98   7C930738  8..|  ntdll.7C930738
0012FF9C   FFFFFFFF  ....
0012FFA0   0012FFF0  ....
0012FFA4   0012FFB8  ....
0012FFA8   7FFD9000  ....
0012FFAC   7C92EB94  ...|  ntdll.KiFastSystemCallRet
0012FFB0   0012FFB0  ....
0012FFB4   00000000  ....
0012FFB8   7C80B6A1  ...|  kernel32.GetModuleHandleA
0012FFBC   0040231E  .#@.  返回到 yh\_packe.0040231E 来自
yh\_packe.00402410
0012FFC0   00000000  ....
0012FFC4   7C816FD7  .o.|  返回到 kernel32.7C816FD7
0012FFC8   7C930738  8..|  ntdll.7C930738
0012FFCC   FFFFFFFF  ....
0012FFD0   7FFD9000  ....
0012FFD4   80873B38  8;..
0012FFD8   0012FFC8  ....
0012FFDC   82793528  (5y.
0012FFE0   FFFFFFFF  ....  SEH 链尾部
0012FFE4   7C839AA8  ...|  SE 处理器
```

也可以看见

但我不会写角本 恢复`IAT`..汗

我们再`F8`一下

就会由`OD`的分析看见`RETN`会到`GETMODULEHANDLEA`了

行後 就会返回程序的原本流程

这次。我们来看看`ZPROTECT`的加密调用代码 一

我们先在加密导入表置中去掉所有加密（就是把所有函数加密改为不加密）

选上加密调用代码

再次加壳

看看是怎么回事

对比下两者

未加壳

```
00402317 yh.<模块>  6A 00                 push 0
00402319          E8 F2000000           call 00402410
          ; <jmp.&kernel32.GetModuleHandleA>
0040231E          A3 24404000           mov dword ptr ds:[404024],eax
00402323          6A 00                 push 0
00402325          68 75224000           push 402275
0040232A          6A 00                 push 0
0040232C          68 E8030000           push 3E8
00402331          FF35 24404000         push dword ptr ds:[404024]
00402337          E8 26000000           call 00402362
          ; <jmp.&user32.DialogBoxParamA>
0040233C          50                    push eax
0040233D          E8 B6000000           call 004023F8
          ; <jmp.&kernel32.ExitProcess>
00402342          C3                    retn
00402343          CC                    int3
00402344        - FF25 EC304000         jmp dword ptr ds:[4030EC]
          ; USER32.wsprintfA
0040234A        - FF25 E8304000         jmp dword ptr ds:[4030E8]
          ; USER32.AppendMenuA
00402350        - FF25 E4304000         jmp dword ptr ds:[4030E4]
          ; USER32.CallWindowProcA
00402356        - FF25 E0304000         jmp dword ptr ds:[4030E0]
          ; USER32.CloseClipboard
0040235C        - FF25 DC304000         jmp dword ptr ds:[4030DC]
          ; USER32.CreatePopupMenu
```

加了壳的

```
00402317          6A 00                 push 0
00402319          E8 F2000000           call 00402410
          ; yh\_packe.00402410
0040231E          A3 24404000           mov dword ptr ds:[404024],eax
00402323          6A 00                 push 0
00402325          68 75224000           push 402275
0040232A          6A 00                 push 0
0040232C          68 E8030000           push 3E8
00402331          FF35 24404000         push dword ptr ds:[404024]
00402337          E8 26000000           call 00402362
          ; yh\_packe.00402362
```



## 1. 初始代码分析与修改

在 `0040233C` 至 `00402361` 处，主要操作是将 `FF25` 指令替换为 `jmp` 指令，并将多余位置填充为 `nop` 指令。

```
0040233C        50                 push eax  
0040233D        E8 B6000000        call 004023F8             ; yh\_packe.004023F8  
00402342        C3                 retn  
00402343        CC                 int3  
00402344    -   E9 4BBF0000        jmp 0040E294              ; yh\_packe.0040E294  
00402349        90                 nop  
0040234A    -   E9 C9BC0000        jmp 0040E018              ; yh\_packe.0040E018  
0040234F        90                 nop  
00402350    -   E9 A3BE0000        jmp 0040E1F8              ; yh\_packe.0040E1F8  
00402355        90                 nop  
00402356    -   E9 41BD0000        jmp 0040E09C              ; yh\_packe.0040E09C  
0040235B        90                 nop  
0040235C    -   E9 A3BE0000        jmp 0040E204              ; yh\_packe.0040E204  
00402361        90                 nop  
```

修改逻辑：将 `FF25` 改为 `jmp`，多余部分 `nop`。

## 2. 进入 CALL 目标地址分析

进入 `CALL` 后，观察到 `0040E258` 处的代码结构：

```
0040E258        68 A7B93C15        push 153CB9A7  
0040E25D    ^   E9 52FEFFFF        jmp 0040E0B4              ; yh\_packe.0040E0B4  
0040E262        D310             rcl dword ptr ds:\[eax\],cl  
0040E264        68 8BB93C15        push 153CB98B  
0040E269    ^   E9 46FEFFFF        jmp 0040E0B4              ; yh\_packe.0040E0B4  
0040E26E        E6 57            out 57,al  
0040E270        68 B4B93C15        push 153CB9B4  
0040E275    ^   E9 3AFEFFFF        jmp 0040E0B4              ; yh\_packe.0040E0B4  
0040E27A        1968 68            sbb dword ptr ds:\[eax+68\],ebp  
0040E27D        8EB9 3C15E92E      mov seg?,word ptr ds:\[ecx+2EE9153C\] ; 未定义的段寄存器  
0040E283        FE                 ???                        ; 未知命令  
0040E284        FFFF               ???                        ; 未知命令  
```

## 3. 追踪 CALL 跳转

使用 `F8` 单步发现跳转至 `00D80BA0`：

```
00D80BA0      /E9 98FC0000       jmp 00D9083D  
00D80BA5      |CC                int3  
00D80BA6      |CC                int3  
00D80BA7      |CC                int3  
```

解密逻辑位于壳内部区段。此处的结构与前一篇类似，深入跟踪至 `00D9083D`：

```
00D9083D        60                 pushad  
00D9083E        E9 9D3B0000        jmp 00D943E0  
00D90843        238B 1D28C6D8      and ecx,dword ptr ds:\[ebx+D8C6281D\]  
00D90849        00E9               add cl,ch  
00D9084B        19B7 0000F205      sbb dword ptr ds:\[edi+5F20000\],esi  
00D90851        6B8F 5DE9CFC6 00   imul ecx,dword ptr ds:\[edi+C6CFE95D\],0  
```

该函数以 `pushad` 开头。根据 `ESP` 定律，可以恢复栈状态并定位到函数入口。

使用 `F8` 调试，查看 `hr esp`。

`F9` 执行至 `popad` 和后续 `jmp`。

发现 `retn` 返回的目标是加密的 `GetModuleHandleA`。壳本身已处理好代码逻辑。此时可以返回到加密出口处。

## 4. 恢复原始指令

回到 `00402410` 处进行修复：

原始状态：
```
00402410    -   E9 43BE0000        jmp 0040E258              ; yh\_packe.0040E258  
00402415        90                 nop  
```

修改目标：恢复为 `FF25` 指令格式，即 `FF25` + 地址。需要找到 `ptr` 的地址。

已知函数地址为 `7C80B6A1`。
记忆要点：低位到高位为 `A1 B6 80 7C`。
在反汇编中搜索该字节串。
在代码段 `00401000` 处搜索。
找到偏移 `403060` 处。

修改 `00402410` 为：
```
00402410    -   FF25 60304000      jmp dword ptr ds:\[403060\]     ; kernel32.GetModuleHandleA  
```

## 5. 脚本化与自动化

理论上可以使用脚本自动完成上述过程。
去壳和解密逻辑相似，通过跟踪即可理解。
利用特征码进行搜索即可定位关键位置。

目前已有相关脚本可用。主要采用手动调试与 `exec` + `end` 的方式编写脚本。

整理了一份脚本，用于恢复被混合 `jmp` 指令破坏的 API 地址及堆栈状态。调试过程中涉及许多混淆手法，大部分为体力工作。

## 6. 进入 CALL 前的寄存器状态

在执行 `CALL` 前，寄存器状态如下：

```
EAX  00000000
ECX  0012FFB0
EDX  7C92EB94 ntdll.KiFastSystemCallRet
EBX  7FFDE000
ESP  0012FFBC
EBP  0012FFF0
ESI  FFFFFFFF
EDI  7C930738 ntdll.7C930738
EIP  0040E258 yh\_packe.0040E258
```

执行流程：
```
push 153CB9A7
pushad
push dword ptr ss:\[esp+20\] ; ZPROTECT 利用 ESP 辅助加密
call 00D94483                ; 以下为 CALL 内部执行
```

## 7. CALL 内部解密逻辑

`CALL` 内部代码片段：

```
xchg dword ptr ss:\[esp\],edx
lea edx,dword ptr ds:\[edx+4138\]
xchg dword ptr ss:\[esp\],edx
mov eax,dword ptr ds:\[D8C628\]
cmp byte ptr ds:\[eax+17E\],0
je 00D8F8D2
```

后续代码通过动态生成指令计算 API 地址：

```
mov eax,dword ptr ss:\[esp+4\]
push esi
push eax
call dword ptr ds:\[D8C514\]

pushad
mov edx,dword ptr ss:\[esp+24\]
xchg ebp,esi
mov esi,esi
xchg esi,esi
neg edx
neg edx
xchg eax,ebp
xor edx,56AB3560
mov eax,6EA50155
push ecx
pop ebp
mov ebp,26841A9
mov ebp,ecx
push edx
pop edi
add edi,427D094C
neg edi
xor edi,79EB69E2
xchg edx,eax
mov esi,edi
mov ebx,esi
mov edi,115E6AAB
mov esi,ecx
mov dword ptr ss:\[esp+1C\],ebx
popad
retn 4

mov ecx,dword ptr ds:\[D8CC28\]
test ecx,ecx
mov esi,eax
je 00D992C1
jmp 00D91DA4

mov eax,dword ptr ds:\[D8CC2C\]
sub eax,ecx
sar eax,2
cmp esi,eax
jb 00D94F31
```

## 8. 解密完成后的状态

当执行到 `jb 00D94F31` 时，寄存器状态更新为：

```
EAX  00000037
ECX  00DA4258
EDX  7C92EB94 ntdll.KiFastSystemCallRet
EBX  7FFDE000
ESP  0012FF8C
EBP  0012FFF0
ESI  0000000F
EDI  7C930738 ntdll.7C930738
EIP  00D94F31
```

执行后续指令：

```
mov ecx,dword ptr ds:\[ecx+esi\*4\]
```

此时 `ds:\[00DA4294\]` 的值为 `7C80B6A1`，即 `kernel32.GetModuleHandleA` 的地址。`ecx` 寄存器存储该 API 地址。

## 9. 尾部处理与返回

尾部处理代码（用于过渡到 API 调用）：

```
mov dword ptr ss:\[esp+2C\],ecx
pop esi
retn 4
popad
retn
```

`popad` 恢复堆栈平衡。
`retn` 返回至 API。
后续程序将进入正常执行流程。
