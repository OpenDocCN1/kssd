
```
kd> dq 0xfffff880`03b82b88
fffff880`03b82b88  fffff800`03ee0205  fffff960`00134164  fffff880`03b82b98  00000000`00000000  00000000`0000002a  fffff880`03b82ba8  fffff960`00133fd6  00000000`00000003  fffff880`03b82bb8  00000000`00000000  00000000`00000000  00000000`0192efb0  fffff880`03b82bc8  00000000`00000005  fffff960`00134164  fffff880`03b82bd8  fffff800`03edff93  fffffa80`31d69660  fffff880`03b82be8  fffffa80`31d63910  00000000`00000000  fffff880`03b82bf8  fffffa80`31d63910  00000000`00000000
```

`StackCurrent`为``0xfffff880`03b82b88``，`dq`查看存的是 `fffff800 03ee0205`。

```
kd> u fffff800`03ee0205
nt!KiSystemServiceExit+0x26a:
fffff800`03ee0205 488b4c2420      mov     rcx,qword ptr [rsp+20h]
fffff800`03ee020a 488b542428      mov     rdx,qword ptr [rsp+28h]
fffff800`03ee020f 4c8b442430      mov     r8,qword ptr [rsp+30h]
fffff800`03ee0214 4c8b4c2438      mov     r9,qword ptr [rsp+38h]
fffff800`03ee0219 4c8b542440      mov     r10,qword ptr [rsp+40h]
fffff800`03ee021e 4883c450        add     rsp,50h
fffff800`03ee0222 41ffd2          call    r10
fffff800`03ee0225 488945b0        mov     qword ptr [rbp-50h],rax
```

`u`看下汇编，正好是`KiSystemServiceExit`，正如前面猜想的寻找`KiSystemServiceExit`。

```
kd> dq 0xfffff880`03b82b88
fffff880`03b82b88  fffff800`03ee0205  fffff960`00134164  fffff880`03b82b98  00000000`00000000  00000000`0000002a  fffff880`03b82ba8  fffff960`00133fd6  00000000`00000003  fffff880`03b82bb8  00000000`00000000  00000000`00000000  00000000`0192efb0  fffff880`03b82bc8  00000000`00000005  fffff960`00134164  fffff880`03b82bd8  fffff800`03edff93  fffffa80`31d69660  fffff880`03b82be8  fffffa80`31d63910  00000000`00000000  fffff880`03b82bf8  fffffa80`31d63910  00000000`00000000
```

以这个基准偏移9个（不是9个字节，偏移一个指8字节，也就是偏移72的地方）的地方，为地址``ffff880`03b82bd0``，这地址里的值为 `fffff960 00134164`，也就是要替换的函数指针。

```
kd> u fffff960`00134164
win32k!NtUserCallNoParam:
fffff960`00134164 48895c2408      mov     qword ptr [rsp+8],rbx
fffff960`00134169 57              push    rdi
fffff960`0013416a 4883ec20        sub     rsp,20h
fffff960`0013416e 8bf9            mov     edi,ecx
fffff960`00134170 488b0da1a72000  mov     rcx,qword ptr [win32k!gpresUser (fffff960`0033e918)]
fffff960`00134177 ff15f3d21c00    call    qword ptr [win32k!_imp_ExEnterPriorityRegionAndAcquireResourceExclusive (fffff960`00301470)]
fffff960`0013417d c605249e200001  mov     byte ptr [win32k!gbValidateHandleForIL (fffff960`0033dfa8)],1
fffff960`00134184 48890565c32000  mov     qword ptr [win32k!gptiCurrent (fffff960`003404f0)],rax
```

`u`看一下是什么函数，为`win32k!NtUserCallNoParam:`，这个地址为`UserCallNoParam`的开头。

```
MyDriver2!IfhpInternalGetCpuClock+0x15e [c:\users\hasee\documents\visual studio 2013\projects\mydriver1\mydriver1\infinityhook.cpp @ 622]
nt!EtwpReserveTraceBuffer+0xe2
nt!EtwpLogKernelEvent+0x24d
nt!EtwpTraceKernelEvent+0xa6
nt!PerfInfoLogSysCallEntry+0x47
nt!KiSystemServiceExit+0x26a
user32!NtUserCallNoParam+0xa
```

调用栈的发起者为`user32!NtUserCallNoParam+0xa`，我们都知道调用一个函数，都是push 参数1，push参数 2，push 参数3，然后call ，，这时候堆栈里上面是返回地址，往下是参数，他以返回地址为基准向下加偏移寻找要替换的函数指针，那会不会他寻找的就是参数，

```
: Args to Child        : Call Site
fffff880`03b82b40 : fffff960`00134164 : nt!PerfInfoLogSysCallEntry+0x47
```

windbg查看调用栈显示参数，为了美观省略了后面的参数，参数为``fffff960`00134164``，这个不正好和寻找的那个一样

```
kd> dq fffff880`03b82b40
fffff880`03b82b40  fffffa80`31d69660  fffff880`03b82c60  fffff880`03b82b50  00000000`0192f250  00000000`00000001  fffff880`03b82b60  00000000`00000000  fffff800`00501802  fffff880`03b82b70  fffff880`03b82b90  00000000`00000008  fffff880`03b82b80  fffff880`03b82b88  fffff800`03ee0205  fffff880`03b82b90  fffff960`00134164  00000000`00000000
```

查看这个函数的堆栈，在``fffff880`03b82b88``是我们寻找的基准系统调用的位置

```
kd> dq 0xfffff880`03b82b88
fffff880`03b82b88  fffff800`03ee0205  fffff960`00134164  fffff880`03b82b98  00000000`00000000  00000000`0000002a  fffff880`03b82ba8  fffff960`00133fd6  00000000`00000003  fffff880`03b82bb8  00000000`00000000  00000000`00000000  00000000`0192efb0  fffff880`03b82bc8  00000000`00000005  fffff960`00134164  fffff880`03b82bd8  fffff800`03edff93  fffffa80`31d69660  fffff880`03b82be8  fffffa80`31d63910  00000000`00000000  fffff880`03b82bf8  fffffa80`31d63910  00000000`00000000
```

回过头来再以这个基准系统调用的位置，要替换的函数指针值为``fffff960`00134164``，一处为偏移1的（偏移1不是指偏移一个字节，是8个字节）``fffff880`03b82b90``，宁一处为偏移9的``fffff880`03b82bd0``,原作者替换的是偏移9处的函数指针。

总结一下：

作者通过寻找`EtwPerfInfoLogSysCallEntry`放入堆栈的特定值来寻找，`EtwPerfInfoLogSysCallEntry`函数的堆栈位置，再向下寻找系统调用找到`KiSystemServiceExit`函数返回地址在堆栈中位置，再通过以`KiSystemServiceExit`为基准，替换偏移9处的参数指针，达到hook的目的。

# 0x3 逆向其他事件

## 1 特定值的寻找

```
EVENT_TRACE_FLAG_PROCESS
0x00000001
Property->EnableFlags = 0x00000001;
```

通过官网查询PROCESS事件的falg为1，修改这个值就行了，

```
nt!EtwpReserveTraceBuffer+0xe2
nt!EtwpLogKernelEvent+0x122
nt!EtwpTraceKernelEvent+0xa6
nt! ?? ::NNGAKEGL::`string'+0x219f0
nt!PspExitProcess+0x4e
nt!PspExitThread+0x4e9
nt!NtTerminateProcess+0x138
nt!KiSystemServiceCopyEnd+0x13
ntdll!NtTerminateProcess+0xa
```

函数虽然变了，但在调用栈的大概位置没变，查看``nt! ?? ::NNGAKEGL::`string'+0x21a03``

```
fffff800`0412de2b 8b5504          mov     edx,dword ptr [rbp+4]
fffff800`0412de2e 488d8d90000000  lea     rcx,[rbp+90h]
fffff800`0412de35 450fb7cd        movzx   r9d,r13w
fffff800`0412de39 41b801000000    mov     r8d,1
fffff800`0412de3f c744242803195000 mov     dword ptr [rsp+28h],501903h
fffff800`0412de47 488364242000    and     qword ptr [rsp+20h],0
fffff800`0412de4d e8bedae8ff      call    nt!EtwpTraceKernelEvent (fffff800`03fbb910)
fffff800`0412de52 488d4d20        lea     rcx,[rbp+20h]
```

可以在``fffff800`0412de3f``看见`501903h`，宁一个特定值不好从代码看，我们在`mov dword ptr [rsp+28h],501903h`在这段系统代码下个断点，断下后记下rsp+28的值，等断在我们的代码后再`dq`查看内存偏移1处的后三位，经过多次测试，第二个值为`8d0` 。

## 2 寻找地址猜想

```
nt!EtwpReserveTraceBuffer+0xe2
nt!EtwpLogKernelEvent+0x122
nt!EtwpTraceKernelEvent+0xa6
nt! ?? ::NNGAKEGL::`string'+0x219f0
nt!PspExitProcess+0x4e
nt!PspExitThread+0x4e9
nt!NtTerminateProcess+0x138
nt!KiSystemServiceCopyEnd+0x13
ntdll!NtTerminateProcess+0xa
```

从调用栈看，函数指针应该是`PspExitProcess`的地址，但我找了半天，`u`查看没找到`PspExitProcess`函数。
