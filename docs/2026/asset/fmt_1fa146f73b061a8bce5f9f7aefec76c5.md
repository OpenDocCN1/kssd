
## POC分析

一个简单的LPC通信，有以下几个步骤：

1. Server指定一个端口名，并创建端口
2. Server监听创建的端口
3. Client连接Server创建的端口
4. Server接受Client连接请求并完成连接
5. Client发送请求报文，并等待Server响应
6. Server接受请求报文并响应

在LPC通信过程中，如果报文较大，通信双方就会采用共享内存区的方式交换数据，但会通过报文进行协调同步。

LPC通信流程如下图所示（图片源自 @iamelli0t 师傅在看雪SDC的演讲PPT）

<img src="https://gitee.com/tianxuan-securitylab/source/raw/master/img/1623401452747.png" alt="1623401452747" style="zoom: 50%;" />

更多LPC相关内容，请自行查阅，本文不做详述。

目前已知的是通过`NtRequestWaitReplyPort`发送LPC消息到`splwow64.exe`进程后，由`splwow64!TLPCMgr::ProcessRequest`对LPC消息进行处理，所以对`splwow64!TLPCMgr::ProcessRequest`下断。

```
0:009> bu splwow64!TLPCMgr::ProcessRequest
0:009> bu gdi32full!GdiPrinterThunk
0:009> g
Breakpoint 0 hit
splwow64!TLPCMgr::ProcessRequest:
00007ff7`0bf176ac 48895c2418      mov     qword ptr [rsp+18h],rbx ss:00000000`0279f3e0=0000000000d7ca90
0:007> r
rax=0000000000000000 rbx=0000000000d7ca90 rcx=0000000000d756f0
rdx=0000000000d7cac0 rsi=0000000000d7cac0 rdi=0000000000d786a0
rip=00007ff70bf176ac rsp=000000000279f3c8 rbp=0000000000b6a478
r8=000000000279f328  r9=0000000000b6a478 r10=0000000000000000
r11=0000000000000244 r12=000000007ffe03b0 r13=000000000000022c
r14=0000000000d78778 r15=0000000000000000
...
splwow64!TLPCMgr::ProcessRequest:
00007ff7`0bf176ac 48895c2418      mov     qword ptr [rsp+18h],rbx ss:00000000`0279f3e0=0000000000d7ca90
```

`rdx`=0000000000d7cac0 即为`LpcRequest`

![1623812660207](https://bbs.kanxue.com/upload/attach/202107/929902_TUR3E7W6FH5D4MV.jpg)

IDA反汇编 `TLPCMgr::ProcessRequest`

![1622713289115](https://bbs.kanxue.com/upload/attach/202107/929902_ZMAUESXN78B63ZW.jpg)

windbg调试上图所示代码

```
0:007> 
splwow64!TLPCMgr::ProcessRequest+0x6e:
00007ff7`0bf1771a 66833f20        cmp     word ptr [rdi],20h ds:00000000`00d7cac0=0020
0:007> 
splwow64!TLPCMgr::ProcessRequest+0x72:
00007ff7`0bf1771e 418bef          mov     ebp,r15d
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x75:
00007ff7`0bf17721 418bdf          mov     ebx,r15d
0:007> 
splwow64!TLPCMgr::ProcessRequest+0x78:
00007ff7`0bf17724 41be57000000    mov     r14d,57h
0:007> 
splwow64!TLPCMgr::ProcessRequest+0x7e:
00007ff7`0bf1772a 7523            jne     splwow64!TLPCMgr::ProcessRequest+0xa3 (00007ff7`0bf1774f) [br=0]
0:007> 
splwow64!TLPCMgr::ProcessRequest+0x80:
00007ff7`0bf1772c 4d397d48        cmp     qword ptr [r13+48h],r15 ds:00000000`00d75738={GDI32!GdiPrinterThunk (00007ffa`c8e48eb0)} //判断GDI32!GdiPrinterThunk指针
0:007> 
splwow64!TLPCMgr::ProcessRequest+0x84:
00007ff7`0bf17730 741d            je      splwow64!TLPCMgr::ProcessRequest+0xa3 (00007ff7`0bf1774f) [br=0]
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x86:
00007ff7`0bf17732 8b5f28          mov     ebx,dword ptr [rdi+28h] ds:00000000`00d7cae8=00000088
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x89:
00007ff7`0bf17735 8d43f0          lea     eax,[rbx-10h]
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x8c:
00007ff7`0bf17738 3defff0f00      cmp     eax,0FFFEFh
0:007> r eax
eax=78
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x91:
00007ff7`0bf1773d 0f8737030000    ja      splwow64!TLPCMgr::ProcessRequest+0x3ce (00007ff7`0bf17a7a) [br=0]
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x97:
00007ff7`0bf17743 8bcb            mov     ecx,ebx
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x99:
00007ff7`0bf17745 e8de430000      call    splwow64!operator new[] (00007ff7`0bf1bb28)
0:007> p
splwow64!TLPCMgr::ProcessRequest+0x9e:
00007ff7`0bf1774a 488bf0          mov     rsi,rax
0:007> r
rax=0000000000d785e0 rbx=0000000000000088 rcx=000000007ffe0380
rdx=0000000000000001 rsi=0000000000000000 rdi=0000000000d7cac0
rip=00007ff70bf1774a rsp=000000000279f2e0 rbp=0000000000000000
r8=0000000000000000  r9=0000000000000001 r10=0000000000d70000
r11=000000000279f250 r12=00007ff70bf22048 r13=0000000000d756f0
r14=0000000000000057 r15=0000000000000000
...
splwow64!TLPCMgr::ProcessRequest+0x9e:
00007ff7`0bf1774a 488bf0          mov     rsi,rax
```

上面的代码主要做了三件事，先判断`LpcRequest.MessageHeader.DataSize`是否为`0x20`，接着判断`GDI32!GdiPrinterThunk`函数指针是否存在，如果都存在，取`LpcRequest.MsgSendLen`的值`0x88`给`EBX`，然后调用`splwow64!operator new`在`splwow64.exe`进程空间内分配了一块`0x88`大小的内存空间，接下来我们称这块空间为`InputBuffer`。

继续看IDA的反汇编代码

![1622719910927](https://bbs.kanxue.com/upload/attach/202107/929902_FM8ZDHCA2SUQWUF.jpg)

首先进行了复制操作，从LPC通信使用的共享内存复制数据到`InPutBuffer`中，然后取出`LpcRequest.PtrMsgReply`的值给`v9`，接着取出`LpcRequest.MsgReplyLen`的值给`v10`，最后取出 `LpcRequest.MessageHeader.MessageType`的值给 `v11`。接下来判断`v11`、`v12`的值，这里对`v11`、`v12`值的判断结果会影响程序流程是否进入存在漏洞的函数。因为`v11`和`v12`的值都是从`LpcRequest`中得到的，所以我们可以通过控制`LpcRequest`，让程序按照我们预期的流程走，也就是进入`gdi32!GdiPrinterThunk`函数，在`gdi32!GdiPrinterThunk`中又调了`gdi32full!GdiPrinterThunk`函数。

windbg调试上面这块代码
