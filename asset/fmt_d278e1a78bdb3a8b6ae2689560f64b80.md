
# 请问 SSDT 中的索引 ID1000 以上的函数（NtUserValidateHandleSecure）

> 原文：[`www.pediy.com/kssd/pediy10/59883.html`](https://www.pediy.com/kssd/pediy10/59883.html)

索引 id 没有错，我跟过的，是 `mov eax, 1248` `callxxxxxx` `-->` `mov edx, esp` `sysenter`

`dd 8080d8b0+1248*4` 应该是没有错的，但是之后 `dd 21ae830f` 却不对了（内容空）

过程如下：

引用: `lkd> dd KeServiceDescriptorTable`

```
8088a500  8080d8b0 00000000 0000011c 80840fc4

8088a510  00000000 00000000 00000000 00000000

8088a520  00000000 00000000 00000000 00000000

8088a530  00000000 00000000 00000000 00000000

8088a540  00000002 00002710 bf80c227 00000000

8088a550  f8cbca80 f86f74a0 83adfa90 80a38040

8088a560  00000000 00000000 ffea8ad6 ffffffff

8088a570  094c9316 01c872f6 00000000 00000000
```

`lkd> dd 8080d8b0+1248*4`

```
808121d0  21ae830f 558b0001 8153e80c c25dffff

808121e0  90900008 8b909090 ec8b55ff 08758b56

808121f0  8b0c46ff 8113e8ce c085ffff e25e840f

80812200  5d5e0000 900004c2 ffffffff 80930288

80812210  80930291 ffffffff 00000000 808a3594

80812220  ffffffff 00000000 80930168 90909090

80812230  ffffffff 8092ee28 8092ee3b 8bf3b70f

80812240  0fc72bc6 d233c2af 0007d0bb 83f3f700
```

`lkd> dd 21ae830f`

```
21ae830f  ???????? ???????? ???????? ????????

21ae831f  ???????? ???????? ???????? ????????

21ae832f  ???????? ???????? ???????? ????????

21ae833f  ???????? ???????? ???????? ????????

21ae834f  ???????? ???????? ???????? ????????

21ae835f  ???????? ???????? ???????? ????????

21ae836f  ???????? ???????? ???????? ????????

21ae837f  ???????? ???????? ???????? ????????
```

所以我疑惑了，感觉是不是不是 `1248*4` 不应该是乘以四？我感觉这也太大了。。。

翻一下资料 `Windows XP Build 2600 System Services` 中索引编号是这样排列的：

`0000~00FF`

`0100~011B`

`1000~1299`

索引号并不是连续的，所以我感觉直接用 `1248*4` 十分不妥。

接着我直接数了 `NtUserValidateHandleSecure` 的顺位（第几个）

然后用 `dd 8080d8b0+顺位数*4`，感觉也不对。

那就最后请问各位,如何找到索引编号为 1248 的函数 `NtUserValidateHandleSecure` 的地址？

`KeServiceDescriptorTableShadow+248*4`

`KeServiceDescriptorTableShadow` 其实是个数组,`Index` 的高四位表示使用的是这个数组的下标,比如 `0x1248`,就应该是 `KeServiceDescriptorTableShadow[1].Base + 0x248 * sizeof(ULONG)`,一般来说使用 1 下标的函数都在 `Win32K.sys` 实现.不知道我说清楚了没

引用: *最初由 **老 Y**发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg) `symbols` 加载不正确

上 `MSoft` 下载了 `WindowsXP-KB835935-SP2-slp-Symbols`

安装。

打开 `Windbg`,`Ctrl+s`,输入了 `C:\\WINDOWS\\Symbols`,点 `ok`

重新进入后出现：

引用: `Microsoft (R) Windows Debugger Version 6.8.0004.0 X86`

`Copyright (c) Microsoft Corporation. All rights reserved.`

`Unable to read head of debugger data list`

`Connected to Windows XP 2600 x86 compatible target, ptr64 FALSE`

`Symbol search path is: C:\\WINDOWS\\Symbols`

`Executable search path is:`

`*** ERROR: Module load completed but symbols could not be loaded for ntoskrnl.exe`

`KdDebuggerDataBlock not available!`

`*********************************************************************`

`WARNING: Local kernel debugging requires booting with kernel`

`debugging support (/debug or bcdedit -debug on) to work optimally.`

`*********************************************************************`

`Windows XP Kernel Version 2600 UP Free x86 compatible`

`Product: WinNt, suite: TerminalServer SingleUserTS`

`Kernel base = 0x80800000 PsLoadedModuleList = 0x80883620`

`Debug session time: Wed Feb 20 14:47:31.875 2008 (GMT+8)`

`System Uptime: 0 days 3:54:28.467`

`lkd> dd KdDebuggerDataBlock`

`Couldn't resolve error at 'KdDebuggerDataBlock'`

`lkd> dd KeServiceDescriptorTable`

`Couldn't resolve error at 'KeServiceDescriptorTable'`

想把这个问题解决因为以后可能要到网吧调程序了.

引用: *最初由 **vrowang123**发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg)上 `MSoft` 下载了 `WindowsXP-KB835935-SP2-slp-Symbols`

打开 `Windbg`,`Ctrl+s`,输入了 `C:\\WINDOWS\\Symbols`,点 `ok`

不必装集合的符号包。直接指定环境变量

`set _NT_SYMBOL_PATH=X:\\<symbols path>;srv*X:\\<symbols path>*http://msdl.microsoft.com/download/symbols`

或者在 `kd` 中 `.sympath` 手工指定 `=` 后的内容。

还可以一次性同步一次，效果相当于装集合的 "最新" 符号包

`symchk /op /r x:\\windows\\system32`
