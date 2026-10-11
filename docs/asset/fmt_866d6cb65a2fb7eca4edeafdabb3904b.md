
`temp!main+0x3`:

```
00401003 33ff             xor     edi,edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401005 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x5`:

```
00401005 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401006 esp=0012ff40 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x6`:

```
00401006 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401007 esp=0012ff3c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x7`:

```
00401007 6a03            push    0x3
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401009 esp=0012ff38 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x9`:

```
00401009 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=0040100a esp=0012ff34 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0xa`:

```
0040100a 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=0040100b esp=0012ff30 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0xb`:

```
0040100b 6800000080      push    0x80000000
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401010 esp=0012ff2c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x10`:

```
00401010 6810304000      push    0x403010
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401015 esp=0012ff28 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x15`:

```
00401015 ff1504204000    call    dword ptr [temp!__imp__CreateFileA (00402004)]{kernel32!CreateFileA (77e7b476)} ds:0023:00402004=77e7b476
```

`0:000> p`

```
eax=ffffffff ebx=7ffdf000 ecx=77f939e3 edx=00000002 esi=00000000 edi=00000000
eip=0040101b esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei ng nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000286
```

`temp!main+0x1b`:

```
0040101b 8bf0            mov     esi,eax
```

在调用 `CreateFile` 函数之后，`EAX` 中将存储返回值。我们注意到其值为 `ffffffff`，也就是”Invalid Handle Value”，我们还想知道 `GetLastError` 的值，它存储在 `fs:34` 这个位置。`FS` 是 `TEB` 选择子，我们可以把它 dump 出来。

`0:000> dd fs:34`

```
0038:00000034  00000002 00000000 00000000 00000000
0038:00000044  00000000 00000000 00000000 00000000
0038:00000054  00000000 00000000 00000000 00000000
0038:00000064  00000000 00000000 00000000 00000000
0038:00000074  00000000 00000000 00000000 00000000
0038:00000084  00000000 00000000 00000000 00000000
0038:00000094  00000000 00000000 00000000 00000000
0038:000000a4  00000000 00000000 00000000 00000000
```

`CDB` 还有一种更快速的方式能做到这点，`!gle`:

`0:000> !gle`

```
LastErrorValue: (Win32) 0x2 (2) - The system cannot find the file specified.
LastStatusValue: (NTSTATUS) 0xc0000034 - Object Name not found.
```

`0:000>`

找不到对应的文件。那么问题出在哪呢？我们需要进一步调试，我们来看看传递给 `CreateFile` 的参数。

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401010 esp=0012ff2c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x10`:

```
00401010 6810304000      push    0x403010
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401015 esp=0012ff28 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x15`:

```
00401015 ff1504204000    call    dword ptr [temp!__imp__CreateFileA
   (00402004)]{kernel32!CreateFileA (77e7b476)} ds:0023:00402004=77e7b476
```

幸运的是，这是内存中的一个常量，它不太可能会改变，因为我们并没有运行到离 `CreateFile` 太远。

然后，我们可以使用”`da`”，”`dc`”，“`du`”命令。”`da`”命令打印出 ANSI 字符串，”`du`”打印出 Unicode 字符串，”`dc`”和”`dd`”类似，不过它是打印出所有的字符，包括不可显示的。我们知道这是一个 ANSI 字符串，”`da`”命令:

`0:000> da 403010`

```
00403010  "c:MyFile.txt"
```

`0:000>`

我们看到这是错误的字符串，我们应该用 `c:\\MyFile.txt` 来代替它。

因此，我们找到并更正这个错误，但是，我们还是不能写入。我们再进一步调试。

重新载入一遍。

`C:\programs\DirectX\Games\src\Games\temp\bin>cdb temp`

```
Microsoft (R) Windows Debugger  Version 6.3.0005.1
Copyright (c) Microsoft Corporation. All rights reserved.
```

```
CommandLine: temp
Symbol search path is:
  SRV*c:\symbols*http://msdl.microsoft.com/download/symbols
Executable search path is:
ModLoad: 00400000 00404000    temp.exe
ModLoad: 77f50000 77ff7000    ntdll.dll
ModLoad: 77e60000 77f46000    C:\WINDOWS.0\system32\kernel32.dll
ModLoad: 77c10000 77c63000    C:\WINDOWS.0\system32\MSVCRT.dll
(80c.c94): Break instruction exception - code 80000003 (first chance)
eax=00241eb4 ebx=7ffdf000 ecx=00000004 edx=77f51310 esi=00241eb4 edi=00241f48
eip=77f75a58 esp=0012fb38 ebp=0012fc2c iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202
```

`ntdll!DbgBreakPoint`:

```
77f75a58 cc             int     3
```

`0:000> bp temp!main`

`0:000> g`

```
Breakpoint 0 hit
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401000 esp=0012ff50 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main`:

```
00401000 51             push    ecx
```

`0:000> p`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401001 esp=0012ff4c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x1`:

```
00401001 56             push    esi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401002 esp=0012ff48 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x2`:

```
00401002 57             push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401003 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x3`:

```
00401003 33ff            xor     edi,edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401005 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x5`:

```
00401005 57             push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401006 esp=0012ff40 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x6`:

```
00401006 57             push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401007 esp=0012ff3c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
```
