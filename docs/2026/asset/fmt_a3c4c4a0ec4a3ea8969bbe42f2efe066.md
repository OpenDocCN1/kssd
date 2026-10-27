
# SDprotector v1.1加壳的某软件脱壳过程

日期：2005年7月16日　　脱壳人：csjwaman[DFCG]

―――――――――――――――――――――――――――――――――――――――――――

**软件名称**：某软件

**下载地址**：本站找

**软件限制**：SDprotector v1.1加壳

**破解声明**：初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**操作系统**：WINXP

**脱壳工具**：OD等传统工具

―――――――――――――――――――――――――――――――――――――――――――

## 脱壳过程

SDprotector v1.1壳，softworm大侠曾写过有关文章。本人认真拜读过。本文就是在softworm大侠文章指引下完成的。在此谢过softworm大侠:)

本软件用PEiD检测显示为Microsoft Visual C++，显然被伪装了。用十六进制工具打开文件，在文件头可以看到如下字符：

```
000003D0   00 00 00 00 00 00 00 00  00 00 00 00 31 2E 31 00   ............1.1.
000003E0   53 44 50 21 0C 09 FF FF  45 32 1E 7F 9A AA 27 A8   SDP!..?E2.?'
```

可见是SDP1.1的壳。

### 一、查找入口

先侦察一下，设置OD忽略所有异常载入程序，F9运行，程序发现调试器。看来程序有反调试手段。下面利用异常来跟踪。跟踪时总的原则：一是多用脚本（这是从softworm处学到的，能省去好多麻烦），因为程序对运行时间检测非常严格；二是不要修改代码（或修改后及时恢复），因为程序有代码完整性检查；三是不要对API下普通断点（硬件断点会被壳删除），因为程序会检查CC。

1、设置OD忽略除INT3中断、单步中断、内存访问异常外的其他所有异常（注意“同时忽略以下定义异常或异常范围”处不能打钩）

2、运行 `UnhandleExceptionFilter` 插件（每次运行脚本前都须先运行这个插件。原因不赘述，见softworm大侠的相关文章）。

3、写个简单的脚本记录一下程序出现警告提示时的中断和异常次数。

我的脚本（挺烂，但能用就行）：

```
var seh

eoe aman1

eob aman1

run

aman1:

add seh,1

log seh

esto
```

运行脚本直到出现警告提示，查看LOG：

```
004EF000   ID 00000908 的主要线程已经创建
00400000   模块 C:\Documents and Settings\csjwaman\桌面\Project2\Project2.exe
             固定表格已经被封装或者已经损坏！
77E40000   模块 C:\WINDOWS\system32\kernel32.dll
77F50000   模块 C:\WINDOWS\System32\ntdll.dll
004EF000   程序入口
004EF07A   单步事件位于 Project2.004EF07A
           seh = 00000001
004F44FA   INT3 命令在 Project2.004F44FA
           seh = 00000002
004F48E2   单步事件位于 Project2.004F48E2
           seh = 00000003
77D10000   模块 C:\WINDOWS\system32\user32.dll
7E190000   模块 C:\WINDOWS\system32\GDI32.dll
77DA0000   模块 C:\WINDOWS\system32\ADVAPI32.dll
78000000   模块 C:\WINDOWS\system32\RPCRT4.dll
76300000   模块 C:\WINDOWS\System32\IMM32.DLL
62C20000   模块 C:\WINDOWS\System32\LPK.DLL
004F48E2   单步事件位于 Project2.004F48E2
           seh = 00000004
004F48E2   单步事件位于 Project2.004F48E2
           seh = 00000005
004F48E2   单步事件位于 Project2.004F48E2
           seh = 00000006
004EF8F8   INT3 命令在 Project2.004EF8F8
           seh = 00000007
004EFB3A   INT3 命令在 Project2.004EFB3A
           seh = 00000008
004EFD54   单步事件位于 Project2.004EFD54
           seh = 00000009
004EFEFA   单步事件位于 Project2.004EFEFA
           seh = 0000000A
72F10000   模块 C:\WINDOWS\System32\USP10.dll
5ADC0000   模块 C:\WINDOWS\System32\uxtheme.dll
77BE0000   模块 C:\WINDOWS\system32\msvcrt.dll
74680000   模块 C:\WINDOWS\System32\MSCTF.dll
53000000   模块 C:\PROGRA~1\3721\helper.dll
53001000     Code size in header is 00000000, extending to size of section '.rdata'
70A70000   模块 C:\WINDOWS\system32\SHLWAPI.dll
77310000   模块 C:\WINDOWS\system32\COMCTL32.dll
37210000   模块 C:\WINDOWS\DOWNLO~1\CnsMin.dll
71BA0000   模块 C:\WINDOWS\System32\NETAPI32.dll
77BD0000   模块 C:\WINDOWS\system32\VERSION.dll
10000000   模块 C:\Herosoft\HeroV8\VCvtShell.dll
773A0000   模块 C:\WINDOWS\system32\SHELL32.dll
00B90000   模块 C:\WINDOWS\System32\msctfime.ime
71950000   模块 C:\WINDOWS\WinSxS\x86\_Microsoft.Windows.Common-Controls\_6595b64144ccf1df\_6.0.2600.1515\_x-ww\_7bb98b8a\comctl32.dll
00BC0000   模块 C:\WINDOWS\System32\wnwbio.ime
7CAB0000   模块 C:\WINDOWS\System32\ole32.dll
```

从LOG可以发现当第10次中断后，程序提示错误。那么我们就在第10次中断时开始跟踪：

修改一下上面的脚本：

```
var seh

eoe aman1

eob aman1

run

aman1:

add seh,1

log seh

cmp seh,a

je stop1

esto

stop1:

coe

cob

ret
```

运行脚本后中断在

```
004EFEFA     64:8F00              pop dword ptr fs:[eax] ///中断在此。
004EFEFD     5B                   pop ebx
004EFEFE     E8 01000000          call 004EFF04                       ; 004EFF04
004EFF03     FF58 05              call far fword ptr ds:[eax+5]
004EFF06     6BFF FF              imul edi,edi,-1
004EFF09     FF80 38E97587        inc dword ptr ds:[eax+8775E938]
```

堆栈：

```
0012FF98    0012FFE0   指针到下一个 SEH 记录
0012FF9C    004EFED3   SE 句柄
```

在`004EFED3`处下断，然后`SHIFT+F9`通过，来到：

```
004EFED3     E8 01000000          call 004EFED9///中断在此。取消断点。
004EFED8     FF58 05              call far fword ptr ds:[eax+5]
004EFEDB     96                   xchg eax,esi
```

这里有花指令，按一下F7代码变成：

```
004EFED9     58                   pop eax                             ; Project2.004EFED8
004EFEDA     05 96FFFFFF          add eax,-6A
004EFEDF     8038 E8              cmp byte ptr ds:[eax],0E8
004EFEE2   ^ 75 B2                jnz short 004EFE96                  ; 004EFE96
004EFEE4     C600 E9              mov byte ptr ds:[eax],0E9///为后面的检测单步跟踪设标志。
004EFEE7     2BC0                 sub eax,eax
004EFEE9     C3                   retn///到系统DLL中。
```

在`4EFEFD`处下断，F9后从系统DLL回到：

```
004EFEFD     5B                   pop ebx ///中断在此。取消断点。
004EFEFE     E8 01000000          call 004EFF04                       ; 004EFF04
004EFF03     FF58 05              call far fword ptr ds:[eax+5]
```

这里有花指令，按二下F7代码变成：

```
004EFF04     58                   pop eax                             ; Project2.004EFF03
004EFF05     05 6BFFFFFF          add eax,-95
004EFF0A     8038 E9              cmp byte ptr ds:[eax],0E9///检测前面设置的检测单步跟踪的标志。
004EFF0D   ^ 75 87                jnz short 004EFE96 ///不能跳。
004EFF0F     C600 E8              mov byte ptr ds:[eax],0E8
004EFF12     9D                   popfd
004EFF13     61                   popad
004EFF14     3D 00000080          cmp eax,80000000
004EFF19     7C 06                jl short 004EFF21                   ; 004EFF21
004EFF1B     EB 06                jmp short 004EFF23                  ; 004EFF23
004EFF1D     0010                 add byte ptr ds:[eax],dl
```

用F7走到：

```
004EFF49     58                   pop eax
004EFF4A     58                   pop eax
004EFF4B     9D                   popfd
004EFF4C     74 31                je short 004EFF7F ///这里必须跳，不跳则提示错误。修改标志位强行跳转。
004EFF4E     74 03                je short 004EFF53                   ; 004EFF53
004EFF50     75 01                jnz short 004EFF53                  ; 004EFF53
004EFF52     E8 E8010000          call 004F013F                       ; 004F013F
004EFF57     00FF                 add bh,bh
004EFF59     58                   pop eax
004FBBBA     E8 F784FFFF          call 004F40B6                       ; 004F40B6
004FBBBF     05 08010000          add eax,108
004FBBC4     50                   push eax
004FBBC5     E8 73FFFFFF          call 004FBB3D                       ; 004FBB3D
004FBBCA     35 47F2EA87          xor eax,87EAF247///这个固定值可能是用于加密的。
004FBBCF     C3                   retn
004F0191    /EB 01                jmp short 004F0194                  ; 004F0194
004F0193    |90                   nop///花指令。
004F0194    \0F84 0E010000        je 004F02A8  ///此处是子进程和父进程的分支处。跳则以子进程身份运行。修改标志位强行跳转。
004F019A     E8 01000000          call 004F01A0                       ; 004F01A0
004F019F     FF58 05              call far fword ptr ds:[eax+5]
```

在`004F0194`处如果不跳，则程序后面调用`CreateProcessA`创建新的进程，父进程则退出。

下面再用脚本来看看程序强行以子进程身份运行后的中断和异常次数。

脚本如下：

```
var seh
var mess
//#log
eoe aman1
eob aman1
run
aman1:
add seh,1
log seh
cmp seh,a //第10次中断时跳转。
je stop1
esto
stop1:
coe
cob
mov seh,0
bp 4eff4c
esto
bc eip
mov !zf,1
sti
run
bp 4f0194 //子进程与父进程分流处。
esto
bc eip
mov eip,4f02a8 //直接跳过创建子进程，强行让父进程以子进程运行。
eoe aman2
eob aman2
run
aman2:
add seh,1
log seh
esto
ret
```

运行脚本后，程序直接运行了。看LOG记录：

```
004EF000   ID 00000E60 的主要线程已经创建
00400000   模块 C:\Documents and Settings\csjwaman\桌面\Project2\Project2.exe
             固定表格已经被封装或者已经损坏！
77E40000   模块 C:\WINDOWS\system32\kernel32.dll
77F50000   模块 C:\WINDOWS\System32\ntdll.dll
004EF000   程序入口
             IsDebugPresent hidden
             IsDebugPresent hidden
004EF07A   单步事件位于 Project2.004EF07A
           seh = 00000001
004F44FA   INT3 命令在 Project2.004F44FA
           seh = 00000002
004F48E2   单步事件位于 Project2.004F48E2
           seh = 00000003
77D10000   模块 C:\WINDOWS\system32\user32.dll
7E190000   模块 C:\WINDOWS\system32\GDI32.dll
77DA0000   模块 C:\WINDOWS\system32\ADVAPI32.dll
78000000   模块 C:\WINDOWS\system32\RPCRT4.dll
76300000   模块 C:\WINDOWS\System32\IMM32.DLL
62C20000   模块 C:\WINDOWS\System32\LPK.DLL
```
