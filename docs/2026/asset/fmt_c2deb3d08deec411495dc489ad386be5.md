
# [原创]解决只能通过鼠标双击触发的文档类漏洞样本调试问题

不知道大家碰没碰到过这种问题：漏洞只能通过双击打开样本文件的方式才能触发，其他方式均不能触发。比如，先打开程序，然后使用文件菜单打开；再比如，先打开程序，然后拖拽文档文件进入程序方式打开；再比如使用命令行打开文档；都不行，使用调试器载入程序后就更不行了。

怎么办？使用双击内核调试，调试用户端的文档程序，然后触发溢出。

本文以大家都熟悉的`CVE-2010-3333`为例，具体过程如下：

## CVE-2010-3333双机调试过程

### 1.调试环境

`winxp sp3 cn`

`office2003 sp3 cn`

`mso.dll` `11.0.8172.0`

![](https://bbs.kanxue.com/upload/attach/202002/92999_QTS88ND69H95XWF.jpg)

### 2.调试过程

#### 0.侵入`winword.exe`进程

```
kd> !process 0 0
**** NT ACTIVE PROCESS DUMP ****
Failed to get VadRoot
PROCESS 821b9830  SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 00b40000  ObjectTable: e1000c90  HandleCount: 262.
    Image: System
Failed to get VadRoot
PROCESS 82000da0  SessionId: none  Cid: 017c    Peb: 7ffd9000  ParentCid: 0004
    DirBase: 06b80020  ObjectTable: e13c1308  HandleCount:  19.
Image: smss.exe
...
PROCESS 82072648  SessionId: 0  Cid: 06f8    Peb: 7ffde000  ParentCid: 067c
    DirBase: 06b802e0  ObjectTable: e1ac6cd0  HandleCount: 270.
    Image: WINWORD.EXE
kd> .process 82072648
Implicit process is now 82072648
WARNING: .cache forcedecodeuser is not enabled
```

可能需要`reload`，`ld`之类的命令，自己看着办。

#### 1.内核调试，中断到数据copy点

```
Microsoft (R) Windows Debugger Version 10.0.17763.1 X86
Copyright (c) Microsoft Corporation. All rights reserved.
Opened \\.\pipe\kd\_winxp
Waiting to reconnect...
Connected to Windows XP 2600 x86 compatible target at (Thu Sep  5 11:33:08.840 2019 (UTC + 8:00)), ptr64 FALSE
Kernel Debugger connection established.
**** Path validation summary ****
...
Break instruction exception - code 80000003 (first chance)
30ed442c f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

#### 2.查看seh链

```
kd> !teb
TEB at 7ffdd000
    ExceptionList:        0012ffb0
    StackBase:            00130000
    StackLimit:           00119000
    SubSystemTib:         00000000
    FiberData:            00001e00
    ArbitraryUserPointer: 00000000
    Self:                 7ffdd000
    EnvironmentPointer:   00000000
    ClientId:             000000dc . 000000ec
    RpcHandle:            00000000
    Tls Storage:          001432c0
    PEB Address:          7ffde000
    LastErrorValue:       0
    LastStatusValue:      c0000034
    Count Owned Locks:    0
    HardErrorMode:        0
kd>  dt ntdll!_EXCEPTION_REGISTRATION_RECORD -l next poi(7ffdd000)
next at 0x0012ffb0
---------------------------------------------
   +0x000 Next             : 0x0012ffe0 _EXCEPTION_REGISTRATION_RECORD
   +0x004 Handler          : 0x30aa1abc     _EXCEPTION_DISPOSITION  +30aa1abc
next at 0x0012ffe0
---------------------------------------------
   +0x000 Next             : 0xffffffff _EXCEPTION_REGISTRATION_RECORD
   +0x004 Handler          : 0x7c839ac0     _EXCEPTION_DISPOSITION  +7c839ac0
```

#### 3.下内存写入断点，等待修改seh

```
kd> ba w 4 0x0012ffe4
kd> g
Single step exception - code 80000004 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
30ed442c f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

#### 4.再次查看seh链

```
kd>  dt ntdll!_EXCEPTION_REGISTRATION_RECORD -l next poi(7ffdd000)
next at 0x0012ffb0
---------------------------------------------
   +0x000 Next             : 0x8b2b06eb _EXCEPTION_REGISTRATION_RECORD
   +0x004 Handler          : 0x30001bdd     _EXCEPTION_DISPOSITION  +30001bdd
next at 0x8b2b06eb
---------------------------------------------
   +0x000 Next             : ????
   +0x004 Handler          : ????
Memory read error 8b2b06ef
```

#### 5.下断seh处理函数

```
kd> bu 0x30001bdd
```

#### 6.执行到`ppret`

```
kd> g
Breakpoint 1 hit
30001bdd 59              pop     ecx
```

![](https://bbs.kanxue.com/upload/attach/201909/92999_6RHSAKPS7PGFU9B.jpg)

#### 7.shellcode执行

a.下断解密指令，以免单步异常影响解密

![](https://bbs.kanxue.com/upload/attach/201909/92999_4SXGHDQ5UP4CP7W.jpg)

b.按几次`F5`，解密部分指令

c.下断解密完成指令，按`F5`解密完成

```
kd> bu 001237f7
```

![](https://bbs.kanxue.com/upload/attach/201909/92999_EGVDUM5ZF3E8QHS.jpg)

d.完成`shellcode`的运行

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
