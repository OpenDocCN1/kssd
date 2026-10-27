
# 带狗杀Tdmd狗

【软件名称】商业，为了避免麻烦不写了

【软件限制】加密狗

【破解人】emb & bug.soft

Email：`yzz2000@sina.com` QQ:`50540415`

【破解工具】`Trw2000`,`ws2dasm`,`UltraEdit`,`FI3.01`,`hiew`,大老的加密狗类型检测工具2.0

【软件简介】行业软件（CAD），小弟干这一行。无狗可安装，运行则无任何提示。彩虹天地的`Tdmd.vxd`狗,狗里面有大量花指令。

==================================================================================

## 【分析过程】

用`FI3.01`检测,提示`Borland C++`，无壳。

用“大老的加密狗类型检测工具2.0”检测软件，它竟然说没找到任何狗。大老称此版本支持检测的狗有“rockey4
sense3 tdsd(rc-dl) gs-mh rcokey5 深思4 彩虹SENTINEL hasp系列 hardlock系列”，竟不认识微狗，看来有待改进。

这次杀狗用带狗杀狗方法，破解狗的常用断点：`bpio 378`，`bpx`
`deviceiocontrol`,`bpx createfilea`，今天我们用`bpx createfilea do "d *esp+4)"`
这个断点来破它。从跟踪中可以发现，用 `CreateFileA` / `DeviceIoControl` 从`Tdmd.VXD`中获取数据。

运行软件，被`Trw2000`断下，观察数据窗口，不是`Tdmd`按`F5`继续，显业`Tdmd`停下，说明马上要读狗狗了。按`F12`
返回.

```
:00405F23 6A00
             push 00000000
:00405F25 6800000004              push
04000000
:00405F2A 6A00
 push 00000000
:00405F2C 6A00
 push 00000000
:00405F2E 6A00
 push 00000000
:00405F30 6A00
 push 00000000
:00405F32 6874F34A00              push
004AF374
* Reference To: KERNEL32.CreateFileA,
Ord:0000h
           |
:00405F37 E8B4790A00              Call
004AD8F0
:00405F3C EB01
 jmp 00405F3F-------返回到这里
:00405F3E 81
   BYTE 81-------花指令，说明与狗笼共舞
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00405F3C(U)
|
:00405F3F 83F8FF                  cmp
eax, FFFFFFFF
:00405F42 750A
 jne 00405F4E
:00405F44 EB01
 jmp 00405F47
:00405F46 81
   BYTE 81
```

连续按`F10`,来到这里。看见程序用`DeviceIoControl`读取狗中数据。

```
:00406423 68CC000000
         push 000000CC
:00406428 57
   push edi
:00406429 68F0000000              push
000000F0
:0040642E 55
   push ebp
:0040642F 53
   push ebx
:00406430 FF74241C                push
[esp+1C]
* Reference To: KERNEL32.DeviceIoControl,
Ord:0000h
           |
:00406434 E8AB740A00              Call
004AD8E4
:00406439 8BD8
 mov ebx, eax
:0040643B EB01
 jmp 0040643E
:0040643D 81
   BYTE 81
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:0040643B(U)
|
:0040643E 85DB
 test ebx, ebx
:00406440 741B
 je 0040645D
:00406442 EB01
 jmp 00406445
:00406444 81
   BYTE 81
```

返回到`42AE25`（`call 00407001`）,发现没有花指令，说明已跳出狗笼，这就是花指令巨大作用。经过数天连续12小时工作，发现`call`
`00407001`负责从狗里读数据，几乎每个操作都有涉及到这个子程序，分析`call 00407001`，发现它需要两个参数，一个是`offset`，大概是狗里数据的偏移，二个是`buffer`，是读数据的缓冲区，每次读4个字节，成功的话返回值为0。

```
* Possible StringData Ref from Data Obj ->"?I@%d"
           |
:0042ADE6 C7053CD34C00444B4C00    mov dword ptr [004CD33C], 004C4B44-------缓
冲区buffer
* Possible StringData Ref from
Data Obj ->"noname"
           |
:0042ADF0 68657C4B00              push
004B7C65
:0042ADF5 68A4074D00              push
004D07A4
* Reference To: cw3220.\_strcpy,
Ord:0000h
           |
:0042ADFA E8D32A0800              Call
004AD8D2
:0042ADFF 83C408                  add
esp, 00000008
:0042AE02 C70534D34C00C40C0000    mov dword ptr [004CD334], 00000CC4
* Possible Reference to Dialog:
DialogID\_0004
           |
:0042AE0C C70530D34C0004000000    mov dword ptr [004CD330], 00000004----------读取字节数
:0042AE16 C7052CD34C003F000000    mov dword ptr [004CD32C], 0000003F]-------偏移Offset
:0042AE20 E8DCC1FDFF              call
00407001
:0042AE25 85C0
 test eax, eax   ----------判断返回值有狗为0
:0042AE27 750A
 jne 0042AE33    ----------跳则死
:0042AE29 C70540D34C002B010000    mov dword ptr [004CD340], 0000012B
```

拉出`ws2dasm`，有时候静态分析，视野比较开阔。总共有19处调用狗。

```
* Referenced by a CALL at Addresses:
|:0042AE20   , :0042B099   , :0042B0BB   , :0042B0DC
, :0042B0FC
|:0042B11D   , :0042B19B   , :0042B1EE   , :0042B2D1
, :0043D4AA
|:0043D4FF   , :0044ADDC   , :0044DA4C   , :0044E34E
, :0044E399
|:0044E5EB   , :0045DFC8   , :0045E382   , :0049EE9B
|
:00407001 51
   push ecx
:00407002 EB00
 jmp 00407004
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00407002(U)
|
:00407004 EB0C
 jmp 00407012
:00407006 8DA94B464090            lea ebp, dword
ptr [ecx+9040464B]
:0040700C 871B
 xchg dword ptr [ebx], ebx
:0040700E 8E10
 mov ss, [eax]
:00407010 149F
 adc al, 9F
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00407004(U)
|
:00407012 EB01
 jmp 00407015
:00407014 81
   BYTE 81
```

我们随意取一处看看。

```
:0044E5CE C7052CD34C0054000000
   mov dword ptr [004CD32C], 00000054]-------偏移Offset
* Possible Reference to Dialog:
DialogID\_0004
           |
:0044E5D8 C70530D34C0004000000    mov dword ptr [004CD330], 00000004----------读取字节数
:0044E5E2 8D4DFC                  lea
ecx, dword ptr [ebp-04]
:0044E5E5 890D3CD34C00            mov dword
ptr [004CD33C], ecx-------缓冲区buffer
:0044E5EB E8118AFBFF              call
00407001
:0044E5F0 817DFC6B5EF904          cmp dword ptr [ebp-04],
04F95E6B---------判断读出来的数据
:0044E5F7 7407
 je 0044E600----------正确则跳，狗里读出的数据作为校验使用，因此存在爆破的可能
:0044E5F9 33C0
 xor eax, eax
:0044E5FB A308074D00              mov dword
ptr [004D0708], eax
```

因为有狗，带狗运行，进行所有的操作，以参数`offset`索引,把参数`buffer`中4个字节记录成表。给程序打补丁时，以参数`offset`索引,根据表里索引填入`buffer`中数据。追进`call`
`00407001`，以`call 00406358`为打补丁突破口。

```
|:0040703C(U)
|
:0040703F E814F3FFFF              call
00406358
* Referenced by a (U)nconditional
or (C)onditional Jump at Addresses:
|:0040702C(U), :0040703A(U)
|
:00407044 59
   pop ecx
:00407045 C3
   ret
```

改成`call 004063BD`。

```
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:0040703C(U)
|
:0040703F E879F3FFFF              call
004063BD
* Referenced by a (U)nconditional
or (C)onditional Jump at Addresses:
|:0040702C(U), :0040703A(U)
|
:00407044 59
   pop ecx
:00407045 C3
   ret
```

把19处所得数据放在`406358~4063bb`中。以参数`offset`索引取数据。补丁代码如下。

```
:004063BD 60
               pushad----------保存环境
:004063BE BE58634000              mov esi,
00406358----------表基址
:004063C3 8B053CD34C00            mov eax, dword
ptr [004CD33C]-------缓
冲区buffer
:004063C9 8BF8
 mov edi, eax
:004063CB 33C9
 xor ecx, ecx
:004063CD 8A0D30D34C00            mov cl, byte
ptr [004CD330]----------读取字节数
:004063D3 33C0
 xor eax, eax
:004063D5 8A052CD34C00            mov al, byte
ptr [004CD32C]----------表索引
:004063DB 03F0
 add esi, eax
:004063DD 90
   nop
:004063DE F3
   repz
:004063DF A4
   movsb----------取数据
:004063E0 61
   popad----------恢复环境
:004063E1 90
   nop
:004063E2 33C0
 xor eax, eax----------有狗标志
:004063E4 C3
   ret
```

## 【后记】

好了，到这里`TDMD`狗就被我们破掉了。前几天我放在看雪初学者论坛上“深思洛克3和飞天诚信狗加壳`NOTEPAD.EXE`”，不知大家研究了吗？可来信与我交流。第一次写文章好累！如觉我写得可以，支持一下，我会接着写飞天诚信狗打狗篇。
