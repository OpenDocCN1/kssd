
# Delphi 研究之驱动开发篇（二）--工具及环境搭建

> 原文：[`www.pediy.com/kssd/pediy10/58070.html`](https://www.pediy.com/kssd/pediy10/58070.html)

上篇教程主要是讲解了用 Delphi 开发 Windows 驱动程序需要解决的一些技术上的问题，虽然嗦了一大堆，也不知道讲清楚了没有^\_^。本篇我们开始讲述用 Delphi 构建驱动开发环境。

用 Delphi 开发驱动程序所必须的工具：

`Dcc32.exe` Delphi 编译器，我用的是 Delphi 2007 的 `dcc32`

`rmcoff` -- 我用 BCB 开发的 Delphi 目标文件符号名修改、OMF 到 coff 格式转换以及删除 obj 文件中无用代码及重复符号的工具

`Link.exe` -- microsoft 链接器，不要使用 7.1xx 版的，似乎有 bug

DDK 相关结构、APIs 的 Delphi 声明文件（我已经完成部分结构、APIs 的声明转换，放在我的 KmdKit4D 工具包里）

有上面的东东就可以开发 Windows 驱动程序了，下面就让我们来写一个最简单的驱动程序：

代码:

```
unit driver;

interface

uses nt\_status, ntoskrnl;

function \_DriverEntry(DriverObject:PDriverObject;RegistryPath:PUnicodeString):NTSTATUS; stdcall;

implementation

procedure DriverUnload(DriverObject:PDriverObject); stdcall;
begin
 DbgPrint('DriverUnload(DriverObject:0x%.8X)',\[DriverObject\]);
 DbgPrint('DriverUnload(-)',\[\]);
end;

function \_DriverEntry(DriverObject:PDriverObject;RegistryPath:PUnicodeString):NTSTATUS; stdcall;
begin
 DbgPrint('DriverEntry(DriverObject:0x%.8X;RegistryPath:0x%.8X)',\[DriverObject,RegistryPath\]);

 DriverObject^.DriverUnload:=@DriverUnload;

 Result:=STATUS\_SUCCESS;
 DbgPrint('DriverEntry(-):0x%.8X',\[Result\]);
end;

end.

```

以上就是一个最简单的驱动程序，就像其他的可执行程序一样，每个驱动程序也有一个入口点，这是当驱动被装载到内存中时首先被调用的，驱动的入口点是 `DriverEntry` 过程（注：过程也就是子程序），`DriverEntry` 这个名称只是一个标记而已，你可以把它命名为其他任何名字--只要它是入口点就行了。`DriverEntry` 过程用来对驱动程序的一些数据结构进行初始化，它的函数原型定义如下：

代码:

```
function \_DriverEntry(DriverObject:PDriverObject;RegistryPath:PUnicodeString):NTSTATUS; stdcall;

```

当然你也可以不用 `DriverEntry` 这个名字，任意的名字都可以，不过前面的下划线是必需的。`nt\_status` 和 `ntoskrnl` 两个单元包含了常用的数据结构和 APIs 的声明。由于我常开发 Unix 下的程序，所以我习惯使用 `make` 编译程序，个人感觉 `make` 比较智能和方便，因此在推荐大家使用 `make` 编译程序。我用的是 borland make 5.2 版。Makefile 的写法可以参考 `http://bbs.pediy.com/showthread.php?t=56912`，以下是编译这个程序的 makefile:

代码:

```
NAME=driver
DCC=dcc32
INCLUDE=d:\\mickeylan\\KmdKit4D\\include
LIB\_PATH=d:\\mickeylan\\KmdKit4D\\lib
DCCFLAGS=-U$(INCLUDE) -B -CG -JP -$A-,C-,D-,G-,H-,I-,L-,P-,V-,W+,Y-
LIBS=ntoskrnl.lib hal.lib win32k.lib ntdll.lib
LINKFLAGS=/NOLOGO /ALIGN:32 /BASE:0x10000 /SUBSYSTEM:NATIVE /DRIVER /LIBPATH:$(LIB\_PATH) /FORCE:UNRESOLVED /FORCE:MULTIPLE /ENTRY:DriverEntry

all : $(NAME).sys

$(NAME).sys : $(NAME).obj   
 omf2d $(NAME).obj /U\_\*
 link $(LINKFLAGS) $(LIBS) /out:$(NAME).sys $(NAME).obj
   

$(NAME).obj : $(NAME).pas
 $(DCC) $(DCCFLAGS) $(NAME).pas

clean :   
 del \*.obj
 del \*.dcu
 del \*.sys

```

在命令行下执行 `make` 即可编译生成驱动文件，是不是很简单^\_^。此程序的源码放在 KmdKit4D 的 `sample\\basic` 目录下，该目录下还有一个 `loaddriver.bat`，执行此批处理文件即可加载驱动，并且可以在 DbgView 的窗口里看见驱动程序输出的调试信息。

到这里，你应该对用 Delphi 开发驱动程序有了个大体的了解了，下面让我们再来写一个很有趣的驱动程序以加深了解。这个程序是从 Four-F 的 KmdKit 的 `giveio` 转换来的（我比较懒，不想写新的^\_^），写个驱动程序让用户模式下的进程能通过读取端口来访问电脑的 CMOS。

大家都知道，端口是被 Windows 保护起来的，正常情况下，用户模式下的程序是无法直接操作端口的，通过我们的驱动程序修改 I/O 许可位图（I/O permission bit map，IOPM），这样用户模式下的相应进程就被允许自由地存取 I/O 端口，这方面详细资料见 `http://www.intel.com/design/intarch/techinfo/pentium/PDF/inout.pdf`。每个进程都有自己的 I/O 许可位图，每个单独的 I/O 端口的访问权限都可以对每个进程进行单独授权，如果相关的位被设置的话，对对应端口的访问就是被禁止的，如果相关的位被清除，那么进程就可以访问对应的端口。既然 I/O 地址空间由 64K 个可单独寻址的 8 位 I/O 端口组成，IOPM 表的最大尺寸就是 2000h 字节（注：每个端口的权限用 1 个 bit 表示，64K 个端口除以 8 得到的就是 IOPM 的字节数，也就是 65536/8＝8192 字节＝2000h 字节）。

TSS 的设计意图是为了在任务切换的时候保存处理器状态，从执行效率的考虑出发，Windows NT 并没有使用这个特征，它只维护一个 TSS 供多个进程共享，这就意味着 IOPM 也是共享的，因此某个进程改变了 IOPM 的话，造成的影响是系统范围的。

`ntoskrnl.exe` 中有些未公开的函数是用来维护 IOPM 的，它们是 `Ke386QueryIoAccessMap` 和 `Ke386SetIoAccessMap` 函数。

代码:

```
function Ke386QueryIoAccessMap(
  dwFlag:DWORD;
  pIopm:PVOID): NTSTATUS; stdcall;

```

`Ke386QueryIoAccessMap` 函数从 TSS 中拷贝 2000h 字节的当前 IOPM 到指定的内存缓冲区中，缓冲区指针由 `pIopm` 参数指定。

各参数描述如下：

◎ `dwFlag`--0 表示将全部缓冲区用 0FFh 填写，也就是所有的位都被设置，所有的端口都被禁止访问；1 表示从 TSS 中将当前 IOPM 拷贝到缓冲区中

◎ `pIopm`--用来接收当前 IOPM 的缓冲区指针，注意缓冲区的大小不能小于 2000h 字节

如果函数执行成功的话会在返回值的低 8 位返回非 0 值；如果执行失败则返回零。

代码:

```
function Ke386SetIoAccessMap(
        dwFlag:DWORD;
        pIopm:PVOID): NTSTATUS; stdcall;

```

`Ke386SetIoAccessMap` 函数刚好相反，它从 `pIopm` 参数指定的缓冲区中拷贝 2000h 字节的 IOPM 到 TSS 中去。

各参数描述如下：

◎ `dwFlag`--这个参数只能是 1，其他任何值函数都会返回失败

◎ `pIopm`--指向包含 IOPM 数据的缓冲区，缓冲区的尺寸不能小于 2000h 字节

如果函数执行成功的话会在返回值的低 8 位返回非 0 值；如果执行失败则返回零

当 IOPM 拷贝到 TSS 后，IOPM 的偏移指针必须被定位到新的数据中去，这可以通过 `Ke386IoSetAccessProcess` 函数来完成，这也是 `ntoskrnl.exe` 中的一个很有用的未公开函数。

代码:

```
function Ke386IoSetAccessProcess(
        pProcess: PKPROCESS;
        dwFlag:DWORD): NTSTATUS; stdcall;

```

`Ke386IoSetAccessProcess` 允许或者禁止对进程使用 IOPM。其参数说明如下：

◎ `pProcess`--指向 KPROCESS 结构

◎ `dwFlag`--0 表示禁止对 I/O 端口进行存取，将 IOPM 的偏移指针指到 TSS 段外面；1 表示允许存取 I/O 端口，将 IOPM 的偏移指针指到 TSS 段的 88h 中

如果函数执行成功的话会在返回值的低 8 位返回非 0 值；如果执行失败则返回零

顺便提一下，`ntoskrnl` 中的所有函数都有前缀，通过这个前缀你就可以辨别该函数属于系统功能中的哪一类。不同的前缀表示不同的功能--如 `i` 前缀表示内部使用（internal）、`p` 表示私有函数（private）、`f` 表示 fastcall。再如，`Ke` 表示内核函数（kernel），`Psp` 表示内部进程支持函数（internal process support），`Mm` 表示内存管理函数（Memory Manager）等等。

`Ke386IoSetAccessProcess` 函数的第一个参数指向进程对象，也就是 KPROCESS 结构（在 `\include\nt\_status.dcu` 中定义），`Ke386IoSetAccessProcess` 会将 KPROCESS 结构中 `IopmOffset` 字段的值设置为合适的值。

代码:



### 驱动程序源代码

```delphi
unit giveio;

interface

uses
  nt\_status, ntoskrnl, ntutils;

const
  IOPM\_SIZE = $2000;   

function \_DriverEntry(DriverObject:PDriverObject;pusRegistryPath:PUnicodeString):NTSTATUS; stdcall;

implementation

function \_DriverEntry(DriverObject:PDriverObject;pusRegistryPath:PUnicodeString):NTSTATUS; stdcall;
var
  status:NTSTATUS;
  oa:OBJECT\_ATTRIBUTES;
  hKey:HANDLE;
  kvpi:KEY\_VALUE\_PARTIAL\_INFORMATION;
  pIopm:PVOID;
  pProcess: PVOID;
  iRet: NTSTATUS;
  resultLen: ULONG;
  KeyValue: TUnicodeString;
begin
  DbgPrint('giveio: Entering DriverEntry',\[\]);
  status := STATUS\_DEVICE\_CONFIGURATION\_ERROR;
  InitializeObjectAttributes(oa, pusRegistryPath, 0, 0, nil);
  iRet := ZwOpenKey(hKey, KEY\_READ, @oa);
  if iRet = STATUS\_SUCCESS then
  begin
    RtlInitUnicodeString(KeyValue, 'ProcessId');
    if (ZwQueryValueKey(hKey, @KeyValue,
      KeyValuePartialInformation, PVOID(@kvpi),
      sizeof(kvpi), resultLen) <> STATUS\_OBJECT\_NAME\_NOT\_FOUND) and
      (resultLen <> 0) then
    begin
      DbgPrint('giveio: Process ID: %X', \[kvpi.dData\]);
      {Allocate a buffer for the I/O permission map}
      pIopm := MmAllocateNonCachedMemory(IOPM\_SIZE);
      if pIopm <> nil then
      begin
        if PsLookupProcessByProcessId(kvpi.dData, pProcess) = STATUS\_SUCCESS then
        begin
          DbgPrint('giveio: PTR KPROCESS: %08X', \[@pProcess\]);
          iRet := Ke386QueryIoAccessMap(0, pIopm);
          if iRet and $ff <> 0 then
          begin
            {I/O access for 70h port}
            asm
              pushad
              mov ecx, pIopm
              add ecx, 70h / 8
              mov eax, \[ecx\]
              btr eax, 70h MOD 8
              mov \[ecx\], eax

              {I/O access for 71h port}
              mov ecx, pIopm
              add ecx, 71h / 8
              mov eax, \[ecx\]
              btr eax, 71h MOD 8
              mov \[ecx\], eax
              popad
            end;

            iRet := Ke386SetIoAccessMap(1, pIopm);
            if iRet and $FF <> 0 then
            begin
              iRet := Ke386IoSetAccessProcess(pProcess, 1);
              if iRet and $FF <> 0 then
              begin
                DbgPrint('giveio: I/O permission is successfully given',\[\]);
              end else
              begin
                DbgPrint('giveio: I/O permission is failed',\[\]);
                status := STATUS\_IO\_PRIVILEGE\_FAILED;
              end;
            end else
            begin
              status := STATUS\_IO\_PRIVILEGE\_FAILED;
            end;
          end else
          begin
            status := STATUS\_IO\_PRIVILEGE\_FAILED;
          end;
          ObfDereferenceObject(pProcess);
        end else
        begin
          status := STATUS\_OBJECT\_TYPE\_MISMATCH;
        end;
        MmFreeNonCachedMemory(pIopm, IOPM\_SIZE);
      end else
      begin
        DbgPrint('giveio: Call to MmAllocateNonCachedMemory failed',\[\]);
        status := STATUS\_INSUFFICIENT\_RESOURCES;
      end;
    end;
    ZwClose(hKey);
  end;
  DbgPrint('giveio: Leaving DriverEntry',\[\]);
  result := status;
end;

end.
```

### Makefile 配置

以下是 makefile:

代码:

```makefile
NAME=giveio
DCC=dcc32
INCLUDE=e:\\mickeylan\\KmdKit4D\\include
LIB\_PATH=e:\\mickeylan\\KmdKit4D\\lib
DCCFLAGS=-U$(INCLUDE) -B -CG -JP -$A-,C-,D-,G-,H-,I-,L-,P-,V-,W+,Y-
LIBS=ntoskrnl.lib hal.lib win32k.lib ntdll.lib
LINKFLAGS=/NOLOGO /ALIGN:32 /BASE:0x10000 /SUBSYSTEM:NATIVE /DRIVER /LIBPATH:$(LIB\_PATH) /FORCE:UNRESOLVED /FORCE:MULTIPLE /ENTRY:DriverEntry

all : $(NAME).sys

$(NAME).sys : $(NAME).obj   
  omf2d $(NAME).obj /U\_\*
  link $(LINKFLAGS) $(LIBS) /out:$(NAME).sys $(NAME).obj ntutils.obj
    
$(NAME).obj : $(NAME).pas
  $(DCC) $(DCCFLAGS) $(NAME).pas

clean :   
  del \*.obj
  del \*.dcu
  del \*.sys
```

### 结语

通过上面的两个例子的学习，相信大家已经能用 Delphi 写些基本的驱动程序了。本教程的第二部分也就到此为止了，后面还会有更精彩的内容。

PS：随本教程我发布了我的 KmdKit4D 0.01 预览版，目前只完成了万里长征的第一步，后面的路还很长，希望有兴趣的朋友能和我一起努力来走完剩下的路^\_^

### 附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | KmdKit4D.rar \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=58070`](http://bbs.pediy.com/showthread.php?t=58070) \] |
|---|---|
