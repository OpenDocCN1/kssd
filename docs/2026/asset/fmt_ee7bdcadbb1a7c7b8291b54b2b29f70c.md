
继续往上翻代码：

```
1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25
411870:  pop    edi
pop    esi
pop    ebx
leave
ret
R411874|402114              <------这里
402114:  ret                    <------这里
R402114|12f350              <------这里
12f350:  mov    eax, 0x1271eb44
mov    edx, 0x12345678
xor    eax, edx
mov    ecx, dword ptr [eax]
mov    ecx, dword ptr [ecx]
mov    ecx, dword ptr [ecx]
add    cx, 0x3c
jmp    ecx
J12f366|6c6974
6c6974:  push   ebp
mov    ebp, esp
and    esp, 0xf8
sub    esp, 0x000003c4
```

这不明显就是溢出覆盖的返回地址嘛~有的Shellcode可能会故意清除已执行代码或者栈结构，有了trace记录则不会受这种情况影响，到了这里如果结合IDA可以很容易定位溢出位置，如果继续查看该日志文件可能信息不是很大了，而且溢出位置之前的部分代码并非第一次执行，所以trace的日志里不再打印实际的汇编指令，后面继续分析，不使用IDA非要找虐的话，也可以使用hutrace的`print_ins_reg`功能打印所有执行指令以及指令状态信息，把上面执行的命令`print_ins_info`替换为`print_ins_reg`再次执行trace过程，这样的trace过程就比较慢了，两分钟生成了约500兆的接近全部运行状态的日志文件（注意还不是全部状态哦，只是指令引用的寄存器及内存状态信息，不过大部分情况已经足够了），虽然运行效率低点，好消息是不用像TTD等工具那样反复重放操作了，如果结合IDA的话可以在对处理流程更了解的基础上，设置只处理trace的起始地址，划分关注的代码这样trace的效率更高。

我们下面演示不用调试器、IDA分析的情况下，利用无脑全流程trace的方式分析该漏洞，通过记录的指令指令状态信息定位溢出点，首先根据溢出覆盖的返回地址信息：

```
1  2
I0x411874|ret                                                 |    esp:0x12f1d0
I0x402114|ret                                                 |    esp:0x12f1d4
```

日志文件中搜索`0x12f1d0`：

![](https://bbs.kanxue.com/upload/attach/202206/543033_2KP98CV2BPXHW84.png)

看到图中左侧框中的数据变化规律可以看出在循环处理栈中的数据，继续往上翻日志：

![](https://bbs.kanxue.com/upload/attach/202206/543033_JJRKBUQ4J4UVQVR.png)

这不就找到溢出点位置了，后面可以再据此根据rtf文件结构、shellcode在rtf中的位置、搜索rtf数据在trace日志文件中的处理流程等等，探究漏洞成因，结合IDA的话其实基本也可以做到不用调试就能澄清漏洞细节了，这里我就不继续了。。。

### 补充说明

使用`drconfig.exe`方式在程序启动时自动注入dynamorio插件默认是追踪`msappid.exe`子进程的，且对`msappid.exe`子进程来说设置的trace运行参数是一致的，如果想对子进程设置不同的trace参数，可以设置“ `-ops "-no_follow_children"`”参数，然后再使用`drconfig.exe`再对`msappid.exe`子进程注册并设置trace参数。

```
1
drconfig.exe -reg EQNEDT32.EXE -syswide_on -ops "-no_follow_children" -c hutracelib.dll -only_from_app -print_ret_addr -print_ins_info -logdir C:\Users\admin\Desktop\bin\ -config C:\Users\admin\Desktop\bin\hutrace.config
```

### 1.2 CVE-2018-4990 adobe pdf 漏洞载荷分析

继续搞个pdf的洞简单分析下，首先关闭Adobe的沙盒保护：

![](https://bbs.kanxue.com/upload/attach/202206/543033_FJFJSJRAHPUPRKB.png)

不会影响漏洞触发，但是开启该机制会影响hutrace的trace操作，关闭之后执行下述命令指定对漏洞模块`EScript.api`的追踪：

```
1
C:\Users\admin\Desktop\bin>hutrace.exe -only_from_app -print_ret_addr -print_ins_info -trace_to_librarys EScript.api -- "C:\Program Files\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe" C:\Users\admin\Desktop\CVE-2018-4990-modify.pdf
```

漏洞的Shellcode为一个简单的MessagexBox弹框，大概三分钟的时间形成了近20个记录的线程日志，其中最大的日志文件约1.9G，这还不包含寄存器等状态信息，首先搜索其API执行情况，从后往前查看或者直接搜索`MessageBox`：

```
1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16
//010edit搜索的部分结果
Line 102653393    9h    ~~3600~~ MSVCR120.dll!strcmp N:0x10533a
Line 102653416    9h    ~~3600~~ MSVCR120.dll!strcmp N:0x10533b
Line 102653435    9h    ~~3600~~ MSVCR120.dll!strcmp N:0x10533c
Line 102653516    9h    ~~3600~~ KERNEL32.dll!VirtualAlloc N:0x10533d
Line 102653522    9h    ~~3600~~ KERNELBASE.dll!VirtualAlloc N:0x10533e
Line 102653597    9h    ~~3600~~ KERNEL32.dll!GetProcAddress N:0x10533f
Line 102653601    9h    ~~3600~~ KERNELBASE.dll!GetProcAddress N:0x105340
Line 102653616    9h    ~~3600~~ KERNEL32.dll!LoadLibraryA N:0x105341
Line 102653631    9h    ~~3600~~ KERNEL32.dll!GetProcAddress N:0x105342
Line 102653635    9h    ~~3600~~ KERNELBASE.dll!GetProcAddress N:0x105343
Line 102653654    9h    ~~3600~~ USER32.dll!MessageBoxA N:0x105344
Line 102653672    9h    ~~3600~~ KERNEL32.dll!GetProcAddress N:0x105345
Line 102653676    9h    ~~3600~~ KERNELBASE.dll!GetProcAddress N:0x105346
Line 102653686    9h    ~~3600~~ KERNEL32.dll!ExitProcess N:0x105347
Line 102653728    9h    ~~3600~~ ntdll.dll!RtlEnterCriticalSection N:0x105348
```

查看`VirtualAlloc`函数附近的代码：

```
1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79
R23802a02|2383af18
R2383af18|23861a4e
R23861a5d|23858051
R2385806e|238c18e9
I238c18f4|23929f00
23929f00:  xchg   esp, eax
ret
R23929f01|238c7e76
238c7e76:  pop    esp
pop    ecx
pop    ecx
pop    ebp
ret 0x0004
R238c7e7a|23882803
23882803:  ret
R23882803|23882802
23882802:  pop    ecx
ret
R23882803|23811784
R23811786|2398af26
2398af26:  xchg   esi, eax
ret
R2398af27|23818000
23818000:  pop    ebp
ret
R23818001|2390f282
2390f282:  pop    ebx
ret
R2390f283|23882802
R23882803|238095c3
238095c3:  pop    edi
ret
R238095c4|2395b084
2395b084:  pop    edx
ret
R2395b085|23978e07
23978e07:  pop    eax
ret
R23978e08|23841ec9
23841ec9:  pushad
ret
R23841eca|238095c4
238095c4:  ret
R238095c4|77e305f4
~~3600~~ KERNEL32.dll!VirtualAlloc N:0x10533d
arg 0: 0x0d1300b0 => 0x00000000 (type=void*, size=0x0)
arg 1: 0x10201 (type=size_t, size=0x4)
arg 2: 0x1000 (type=DWORD, size=0x4)
arg 3: 0x40 (type=DWORD, size=0x4)
and return to module id:54, retraddr: 0x23823aaa,offset:0x23aaa
~~3600~~ KERNELBASE.dll!VirtualAlloc N:0x10533e
arg 0: 0x0d1300b0 => 0x00000000 (type=void*, size=0x0)
arg 1: 0x10201 (type=size_t, size=0x4)
arg 2: 0x1000 (type=DWORD, size=0x4)
arg 3: 0x40 (type=DWORD, size=0x4)
and return to module id:54, retraddr: 0x23823aaa,offset:0x23aaa
23823aaa:  jmp    esp
J23823aaa|d1300ac
and VirtualAlloc return value: 0xd130000
and VirtualAlloc return value: 0xd130000
d1300ac:  add    byte ptr [eax], al
add    byte ptr [eax], al
nop
nop
nop
```

这不就是rop嘛，如果只关注该exp的功能的话可以往后分析其Shellcode功能，分析漏洞成因的话当然到这里还不算完，日志文件里记录的还有流程信息、内存API申请释放等的结果信息可以辅助分析成因，如果继续跟上一个漏洞一样无脑全指令状态记录的话我曾经测试过虚拟机里大概需要五六个小时，日志文件70多个G。。。虽然已经比调试器trace等快得多了，但是trace的日志太大很影响检索，也没有必要，可以考虑在一定的分析基础上再使用hutrace对trace的日志文件进行裁剪或者对漏洞成因进行一定的探究后再指定起始地址开始进行寄存器状态等的记录，也可以用Windbg的TTD、Windbg开页堆等等快速定位，手段比较多，hutrace毕竟只是一个trace工具，不是一个智能全自动化分析工具，纯粹依赖hutrace的话可能有的场景下还不如windbg的几条命令快一点，根据实际情况能辅助到分析过程即可。

## 2. 病毒样本分析
