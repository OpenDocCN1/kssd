
经过一系列的判断后，程序最终进入了`gdi32full!GdiPrinterThunk`，且传入的三个参数为：`InputBuffer`、`PtrMsgReply`和`MsgReplyLen`。

```
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x1ec:
00007ff7 0bf17898 458bc6          mov     r8d,r14d
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x1ef:
00007ff7 0bf1789b 498bd4          mov     rdx,r12
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x1f2:
00007ff7 0bf1789e 488bce          mov     rcx,rsi
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x1f5:
00007ff7 0bf178a1 ff15e9720000    call    qword ptr [splwow64!_guard_dispatch_icall_fptr (00007ff7 0bf1eb90)] ds:00007ff7 0bf1eb90={ntdll!LdrpDispatchUserCallTarget (00007ffa c946c590)}
0:007> r
rax=00007ffac8e48eb0 rbx=0000000000000000 rcx=0000000000d785e0
rdx=0000000000d20088 rsi=0000000000d785e0 rdi=0000000000d7cac0
rip=00007ff70bf178a1 rsp=000000000279f2e0 rbp=0000000000000000
r8=0000000000000010  r9=0000000000000000 r10=0000000000d70000
r11=0000000000d785e0 r12=0000000000d20088 r13=0000000000d756f0
r14=0000000000000010 r15=0000000000000000
iopl=0         nv up ei pl zr na po nc
cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b
efl=00000246
splwow64!TLPCMgr::ProcessRequest+0x1f5:
00007ff7 0bf178a1 ff15e9720000    call    qword ptr [splwow64!_guard_dispatch_icall_fptr (00007ff7 0bf1eb90)] ds:00007ff7 0bf1eb90={ntdll!LdrpDispatchUserCallTarget (00007ffa c946c590)}
..................................................
0:007> p
ntdll!LdrpDispatchUserCallTarget+0x7:
00007ffa c946c597  4c8bd0          mov     r10,rax
0:007> ln rax
Browse module
Clear breakpoint 2    (00007ffa c8e48eb0)   GDI32!GdiPrinterThunk   |  (00007ffa c8e48ebc)   GDI32!_imp_load_GdiProcessSetup
Exact matches:
0:007> p
ntdll!LdrpDispatchUserCallTarget+0xa:
00007ffa c946c59a  49c1ea09        shr     r10,9
0:007>
ntdll!LdrpDispatchUserCallTarget+0xe:
00007ffa c946c59e  4f8b1cd3        mov     r11,qword ptr [r11+r10*8] ds:00007ff5 d81a9238 =8888888888888888
0:007>
ntdll!LdrpDispatchUserCallTarget+0x12:
00007ffa c946c5a2  4c8bd0          mov     r10,rax
0:007> ln rax
Browse module
Clear breakpoint 2    (00007ffa c8e48eb0)   GDI32!GdiPrinterThunk   |  (00007ffa c8e48ebc)   GDI32!_imp_load_GdiProcessSetup
Exact matches:
.............................................
0:007> p
Breakpoint 1 hit
gdi32full!GdiPrinterThunk:
00007ffa c71a59b0  48895c2410      mov     qword ptr [rsp+10h],rbx ss:00000000 0279f2e8=0000000000d785e0
0:007> r
rax=00007ffac8e48eb0 rbx=0000000000000000 rcx=0000000000d785e0
rdx=0000000000d20088 rsi=0000000000d785e0 rdi=0000000000d7cac0
rip=00007ffac71a59b0 rsp=000000000279f2d8 rbp=0000000000000000
r8=0000000000000010  r9=0000000000000000 r10=00000fff591c91d7
r11=8888888888888888 r12=0000000000d20088 r13=0000000000d756f0
r14=0000000000000010 r15=0000000000000000
iopl=0         nv up ei pl nz na po cy
cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b
efl=00000207
gdi32full!GdiPrinterThunk:
00007ffa c71a59b0  48895c2410      mov     qword ptr [rsp+10h],rbx ss:00000000 0279f2e8=0000000000d785e0
```

进入`gdi32full!GdiPrinterThunk`函数后，先获取索引值，因为不同的索引值，会被不同的函数处理。索引值为位于`InputBuffer+0x4`处的`DWORD`。

![1623379603013](https://bbs.kanxue.com/upload/attach/202107/929902_QKYTC62Y7ZVEW6B.jpg)

下面是我们期望进入的处理函数，可以看出，当`Fun_Index`为`0x6D`，就可以进入我们期望的代码块。

![1623380313615](https://bbs.kanxue.com/upload/attach/202107/929902_PF99QV2HAUEJTUQ.jpg)

在进入触发漏洞的代码`Memcpy`前，还要经过`4`个`if`判断和一个`Decode`函数。

![1623380851289](https://bbs.kanxue.com/upload/attach/202107/929902_ZJTEWFD7EB6CCK9.jpg)

这`4`个`if`判断的值，都可以被我们直接或间接控制，所以程序最终会来到漏洞函数`Memcpy`，且三个参数：目的地址、源地址、大小都可以被我们控制，所以这里实现了一个在`splwow64`进程空间内的`Write What Where Primitive`。`Decode`函数的作用是对`Encode`的`DocumentEvent`指针进行解码，也就是对`fpDocumentEvent`指针进行解码，从而得到真实的函数指针。
