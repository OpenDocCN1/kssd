
```c
1:025> kb
ChildEBP RetAddr Args to Child
0452dab4 67db5b8e 00414114 0452ddf8 00000001 mshtml!CTableColCalc::AdjustForCol+0x15
0452db64 67c20713 00000001 0452ddf8 000003e8 mshtml!CTableLayout::CalculateMinMax+0x52f
0452dd80 67c0af19 0452ddf8 0452ddc4 00000001 mshtml!CTableLayout::CalculateLayout+0x276
0452df2c 67cfcc48 0452f5a0 0452e158 00000000 mshtml!CTableLayout::CalcSizeVirtual+0x720
0452e064 67cef5d0 06e19ea8 00000000 00000000 mshtml!CLayout::CalcSize+0x2b8
0452e128 67cef31d 06e19ea8 0001769c 0001769c mshtml!CFlowLayout::MeasureSite+0x312
0452e170 67cef664 0779bf00 00000061 0452f5a0 mshtml!CFlowLayout::GetSiteWidth+0x156
0452e1b0 67cefb40 07bfafb0 06e19ea8 00000001 mshtml!CLSMeasurer::GetSiteWidth+0xce
0452e234 6e2c665d 07862ff8 0452e254 0452e318 mshtml!CEmbeddedILSObj::Fmt+0x150
0452e2c4 6e2c6399 07c12efc 00000000 07025d20 msls31!ProcessOneRun+0x3e9
0452e320 6e2c6252 07c12f18 00018258 00000000 msls31!FetchAppendEscCore+0x18e
0452e374 6e2c61c3 00000000 00000000 00000014 msls31!LsDestroyLine+0x47f
0452e3fc 6e2c293f 00000007 00003832 00000000 msls31!LsDestroyLine+0x9ff
0452e438 67cedd81 00000001 00000007 00003832 msls31!LsCreateLine+0xcb
0452e588 67d017cc 0452f5a0 00000007 07bfafc0 mshtml!CLSMeasurer::LSDoCreateLine+0x127
0452e62c 67d01ef5 0452ee90 0001769c 00000000 mshtml!CLSMeasurer::LSMeasure+0x34
0452e674 67d01db1 00000000 00017e6c 00000083 mshtml!CLSMeasurer::Measure+0x1e6
0452e698 67d011a2 00017e6c 00000083 0779bf40 mshtml!CLSMeasurer::MeasureLine+0x1c
0452e748 67d2a8f6 0452ec68 07470fd8 00000083 mshtml!CRecalcLinePtr::MeasureLine+0x46d
0452ef50 67d2b304 0452f5a0 00000007 0000000e mshtml!CDisplay::RecalcLines+0x8bb
0452f0a0 67d28c5c 0452f5a0 00000007 0000000e mshtml!CDisplay::UpdateView+0x208
0452f154 67d29ee3 0452f5a0 0452f6d8 0873cf10 mshtml!CFlowLayout::CommitChanges+0x9c
0452f264 67c0eb06 0452f5a0 0452f6d8 00000000 mshtml!CFlowLayout::CalcTextSize+0x30f
0452f4ec 67d002ee 0779bf00 0452f6d8 00000000 mshtml!CFlowLayout::CalcSizeCoreCompat+0x1045
0452f508 67d00367 0452f5a0 0452f6d8 00000000 mshtml!CFlowLayout::CalcSizeCore+0x49
0452f544 67d0029c 0452f5a0 0452f6d8 00000000 mshtml!CBodyLayout::CalcSizeCore+0xd8
0452f57c 67cfcc48 0452f5a0 0452f6d8 00000000 mshtml!CFlowLayout::CalcSizeVirtual+0x1af
0452f6b4 67c84121 0779bf00 00000001 00000000 mshtml!CLayout::CalcSize+0x2b8
0452f7a4 67d290f9 00100000 00000007 059ebeb4 mshtml!CFlowLayout::DoLayout+0x543
0452f7e0 67cec8ca 059eb870 00100000 0452f840 mshtml!CView::ExecuteLayoutTasks+0x3b
0452f824 67d2336d 00000000 0452f870 0000008d mshtml!CView::EnsureView+0x355
0452f848 67ce94b2 059eb870 00000000 06d24d58 mshtml!CView::EnsureViewCallback+0xd3
0452f87c 67cd37f7 0452f918 00008002 00000000 mshtml!GlobalWndOnMethodCall+0xff
0452f89c 75ce86ef 000f0402 00000012 00000000 mshtml!GlobalWndProc+0x10c
0452f8c8 75ce8876 67cc1de3 000f0402 00008002 USER32!InternalCallWinProc+0x23
0452f940 75ce89b5 00000000 67cc1de3 000f0402 USER32!UserCallWinProcCheckWow+0x14b
0452f9a0 75ce8e9c 67cc1de3 00000000 0452fa28 USER32!DispatchMessageWorker+0x35e
0452f9b0 6cde04a6 0452f9c8 00000000 00752f58 USER32!DispatchMessageW+0xf
0452fa28 6cdf0446 05688808 00000000 006ccff0 IEFRAME!CTabWindow::\_TabWindowThreadProc+0x452
0452fae0 75f549bd 00752f58 00000000 0452fafc IEFRAME!LCIETab\_ThreadProc+0x2c1
\*\*\* ERROR: Symbol file could not be found. Defaulted to export symbols for C:\Windows\system32\kernel32.dll -
0452faf0 76361174 006ccff0 0452fb3c 7741b3f5 iertutil!CIsoScope::RegisterThread+0xab
WARNING: Stack unwind information not available. Following frames may be wrong.
0452fafc 7741b3f5 006ccff0 73d26994 00000000 kernel32!BaseThreadInitThunk+0x12
0452fb3c 7741b3c8 75f549af 006ccff0 00000000 ntdll!\_\_RtlUserThreadStart+0x70
0452fb54 00000000 75f549af 006ccff0 00000000 ntdll!\_RtlUserThreadStart+0x1b
```

这里可能会出现没有符号的问题，解决方法如下：

在windbg的窗口里输入

![](upload/attach/202208/721335\_2NMKZGDR9GVCYR7.jpg)

```
.sympath SRV\*c:\localsymbols\*http://msdl.microsoft.com/download/symbols
```

然后可以看见

![](upload/attach/202208/721335\_2XYYBW4KX5JUA97.jpg)

分析一下

首先导致崩溃的（分析内容在下述代码注释了）

```
1:025> uf mshtml!CTableColCalc::AdjustForCol
mshtml!CTableColCalc::AdjustForCol:
67f3f152 8bff mov edi,edi
67f3f154 55 push ebp
67f3f155 8bec mov ebp,esp
67f3f157 8b08 mov ecx,dword ptr [eax]
67f3f159 53 push ebx
67f3f15a 8b5d08 mov ebx,dword ptr [ebp+8]
67f3f15d 57 push edi
67f3f15e 8bc1 mov eax,ecx
67f3f160 83e00f and eax,0Fh
67f3f163 8d7e18 lea edi,[esi+18h]；可以看到edi来源于esi，但是esi的处理代码并不在这个函数里，所以继续向上回溯。
67f3f166 50 push eax
67f3f167 890f mov dword ptr [edi],ecx；向edi指向的内存里拷贝值导致crash
67f3f169 e89eacdbff call mshtml!CUnitValue::IsScalerUnit (67cf9e0c)
67f3f16e 85c0 test eax,eax
67f3f170 7411 je mshtml!CTableColCalc::AdjustForCol+0x31 (67f3f183)
...
```

这样就清楚了，我们要在上一个函数下断。

重启一下windbg，重新attach

```
0:021> .childdbg 1
Processes created by the current process will be debugged
0:021> lmm mshtml
start end module name
0:021> sxe ld:mshtml
0:021> g
```

这个时候把poc拖进去（注意到没有提示允许activeX运行）

```
\*\*\*\*\*\*\*\*\*\*\*\*\*\* Symbol Path validation summary \*\*\*\*\*\*\*\*\*\*\*\*\*\*
Response Time (ms) Location
Deferred srv\*
\*\*\*\*\*\*\*\*\*\*\*\*\*\* Symbol Path validation summary \*\*\*\*\*\*\*\*\*\*\*\*\*\*
Response Time (ms) Location
Deferred srv\*
Symbol search path is: srv\*
Executable search path is: srv\*
Page heap: pid 0xDC4: page heap enabled with flags 0x3.
(dc4.e34): Break instruction exception - code 80000003 (first chance)
eax=00000000 ebx=00000000 ecx=0016f5d8 edx=774064f4 esi=7ffda000 edi=0467b384
eip=7745e60e esp=0016f5f4 ebp=0016f620 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
ntdll!LdrpDoDebuggerBreak+0x2c:
7745e60e cc int 3
1:014> g
ModLoad: 691c0000 69772000 C:\Windows\System32\mshtml.dll
eax=07237000 ebx=00000000 ecx=00171000 edx=00000000 esi=7ffda000 edi=0467b384
eip=774064f4 esp=0467b29c ebp=0467b2f0 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
ntdll!KiFastSystemCallRet:
774064f4 c3 ret
1:025> lm m mshtml
start end module name
691c0000 69772000 mshtml (deferred)
```

- `lm` (List Loaded Modules)

`lm`命令显示指定的已加载模块。输出中包含模块状态和路径。

- `m Pattern`

指定模块名必须匹配的模板。`Pattern`可以包含各种通配符和修饰符。关于语法的更多信息，查看字符串通配符语法。

- `sx` 命令显示当前进程的异常列表和所有非异常的事件列表，并且显示调试器遇到每个异常和事件时的行为。

- `sxe Break`

当发生该异常时，在任何错误处理器被激活之前目标立即中断到调试器中。这种处理类型称为第一次处理机会。

- `ld` (Load Symbols)

`ld` 命令加载指定模块的符号并刷新所有模块信息。

这样组合起来，就是`ld`制定`mshtml`加载，然后`sxe`强制在加载这个模块后断下。

现在我们就可以对这个函数下断了。

```
1:025> bp mshtml!CTableLayout::CalculateMinMax
1:025> bl
0 e 692d018a 0001 (0001) 1:\*\*\*\* mshtml!CTableLayout::CalculateMinMax
1:025> g
(c84.798): Unknown exception - code 80010108 (first chance)
(c84.8e8): Unknown exception - code 80010108 (first chance)
Breakpoint 0 hit
eax=ffffffff ebx=0492aea8 ecx=00412802 edx=ffffffff esi=00000000 edi=0467e70c
eip=692d018a esp=0467e4b0 ebp=0467e6c8 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
mshtml!CTableLayout::CalculateMinMax:
692d018a 8bff mov edi,edi
```

- `bp`, `bu`, `bm` (Set Breakpoint)

`bp`、`bu`和`bm`命令设置一个或多个软断点(software breakpoints)。可以组合位置、条件和选项来设置各种不同类型的软断点。

- `bl` (Breakpoint List)

`bl` 命令列出已存在的断点的信息。

- `g` 命令开始指定进程或线程的执行。这种执行将会在程序结束、遇到BreakAddress 或者其他造成调试器停止的事件发生时停止。

我在调试的时候辅助了一下IDA，其实是可以不用的。

直接静态分析找到`CalculateMinMax`

![](upload/attach/202208/721335\_TTZ2TYFWVJPDT76.jpg)

另外这里也需要导入符号。

![](upload/attach/202208/721335\_BD25NKSTAG52RS8.jpg)

![](upload/attach/202208/721335\_7F5PXTFAUVNKNP4.jpg)

![](upload/attach/202208/721335\_H59NB8FVZXE35QY.jpg)

单步继续跟随调试，按`p`就可以单步执行（ 不进入函数那种），不过其实按回车也可以。
