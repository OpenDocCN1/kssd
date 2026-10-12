
# 用 WinDbg 动态脱 Reflector

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-57.htm`](https://www.pediy.com/kssd/pediy08/pediy8-57.htm)

用 WinDbg 动态脱 Reflector

Bi11[CCG]

4 Feb 2006

我才知道 WinDbg+SOS 能调托管程序。。。无敌。。。

举个例子吧，脱 Reflector 玩玩，最新的 4.2.0.0。

精华 7 里 henryouly 的《研读 Reflector 的保护原理心得》文中已经说了原理。简单说就是先解压，然后用 `Assembly.Load(byte[])` 加载。这次我们的任务是动态将这个要加载的 `byte[]` 给 dump 出来。

## 用 WinDbg 加载 Reflector

`Microsoft (R) Windows Debugger  Version 6.6.0003.5`

...

在加载 `mscorjit.dll` 时设异常，执行

```
0:000> sxe ld:mscorjit.dll
0:000> g
```

...

`ModLoad: 79430000 7947d000   C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\MSCORJIT.DLL`

`eax=00000000 ebx=00000000 ecx=00f60000 edx=7c92eb94 esi=00000000 edi=00000000`

`eip=7c92eb94 esp=0012e99c ebp=0012ea90 iopl=0 nv up ei ng nz ac po nc`

`cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000     efl=00000296`

`ntdll!KiFastSystemCallRet:`

`7c92eb94 c3 ret`

载入 `sos.dll` (WinDbg 的插件，就在 `%windir%\Microsoft.NET\Framework\v1.1.4322\` 下。如果不能加载，请先设 `PATH` 环境变量)

```
0:000> .load sos
```

## 找 Assembly.Load 的 MethodDesc

```
0:000> !name2ee mscorlib.dll System.Reflection.Assembly.Load
```

`Loaded Son of Strike data table version 5 from "C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\mscorwks.dll"`

...

`\-----------------------`

`MethodDesc: 79ba35a8`

`Name: [DEFAULT] Class System.Reflection.Assembly System.Reflection.Assembly.Load(SZArray UI1)`

`\-----------------------`

给这个 `MethodDesc` 的 `m_CodeOrIL` 上加个断点(等 RV 添入)

```
0:000> ba w4 79ba35a8+4
0:000> g
```

`Breakpoint 0 hit`

...

`RV` 应该添入了，设个断点

```
0:000> bp poi(79ba35a8+4)
0:000> g
```

`Breakpoint 1 hit`

`eax=79a3bea8 ebx=01283de0 ecx=02447a68 edx=00b4568c esi=01285f54 edi=01285f30`

`eip=79a3bea8 esp=0012f644 ebp=0012f674 iopl=0 nv up ei pl nz na po nc`

`cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000     efl=00000206`

`mscorlib_79990000+0xabea8:`

`79a3bea8 50 push eax`

看看堆栈，没问题～

```
0:000> !clrstack
```

```
Thread 0
ESP       EIP
0012f644  79a3bea8 [DEFAULT] Class System.Reflection.Assembly System.Reflection.Assembly.Load(SZArray UI1)
0012f648  00f702d7 [DEFAULT] [hasThis] Void Reflector.Application..ctor(Class Reflector.IWindowManager)
0012f67c  00f70090 [DEFAULT] Void Reflector.Application.ᐁ()
0012f9b0  791d94bc [FRAME: GCFrame]
0012fa94  791d94bc [FRAME: GCFrame]
```

看看堆栈里的 objects

```
0:000> !dumpstackobjects
```

```
ESP/REG  Object   Name
ebx      01283de0 Reflector.Application
ecx      02447a68 System.Byte[]
esi      01285f54 ᐄ
edi      01285f30 System.IO.MemoryStream
0012f64c 01283de0 Reflector.Application
0012f654 01284dec System.IO.__UnmanagedMemoryStream
0012f660 01283de0 Reflector.Application
```

我们找到了她的地址：`2447a68`，看看：

```
0:000> d 02447a68
```

```
02447a68  3c 2c b6 00 00 00 0e 00-4d 5a 00 00 05 00 00 00  <,......MZ......
02447a78  04 00 00 00 ff ff 00 00-80 00 00 00 00 00 00 00  ................
02447a88  40 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  @...............
02447a98  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
02447aa8  00 00 00 00 80 00 00 00-0e 1f e8 00 00 5a 83 c2  .............Z..
02447ab8  0d b4 09 cd 21 b8 01 4c-cd 21 54 68 69 73 20 70  ....!..L.!This p
02447ac8  72 6f 67 72 61 6d 20 63-61 6e 6e 6f 74 20 62 65  rogram cannot be
02447ad8  20 72 75 6e 20 69 6e 20-44 4f 53 20 6d 6f 64 65   run in DOS mode
```

换只眼看看

```
0:000> dd 02447a68
```

```
02447a68  00b62c3c 000e0000 00005a4d 00000005
02447a78  00000004 0000ffff 00000080 00000000
02447a88  00000040 00000000 00000000 00000000
02447a98  00000000 00000000 00000000 00000000
02447aa8  00000000 00000080 00e81f0e c2835a00
02447ab8  cd09b40d 4c01b821 685421cd 70207369
02447ac8  72676f72 63206d61 6f6e6e61 65622074
02447ad8  6e757220 206e6920 20534f44 65646f6d
```

用 `LordPE` 把 `0x2447a70` 开始，`0xe0000` 大小的内存 Dump 到文件。改个 `exe`，`peid` 查看入口，`_CorDllMain`，是 `dll` 文件。那就再把文件名改成 `dll`。

拖到 `Reflector`，正常，收工。初次玩 `WinDbg`，有出丑的地方还请指正～

顺便提一下，4.2 的 `Reflector` 混淆用的都是不可显示的 `Unicode`，出来一个个框框，太有创意了。

## 主要参考资料

在托管代码中设置断点(WINDBG)

[`blog.joycode.com/gangp/articles/20417.aspx`](http://blog.joycode.com/gangp/articles/20417.aspx)

用 WinDbg 探索 CLR 世界 [3] 跟踪方法的 JIT 过程

[`www.blogcn.com/User8/flier_lu/blog/1678453.html`](http://www.blogcn.com/User8/flier_lu/blog/1678453.html)

`SOS - Son of Strike`

`%???%\SDK\v1.1\Tool Developers Guide\Samples\sos\SOS.htm`

`4dd466d0e1e0cc123afbe504da2462ca`
