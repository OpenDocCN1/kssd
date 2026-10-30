
# [原创]CVE-2017-7269：IIS6.0远程代码执行漏洞逆向分析记录

## 0x0 写在前面

该漏洞可针对IIS 6.0的WebDAV服务发起攻击，通过缓冲区溢出造成远程代码执行。漏洞提交者已在GitHub上公布Poc代码以及漏洞描述，我们可以方便得到Poc以及漏洞信息进行调试

作为一个初学者，看完别人的分析文章需要做的是上手调试，于是形成了这篇调试笔记。行文略显烦琐，错误含糊之处希望各位指(pao)出(hong)

本文的分析思路是跟踪执行脚本时大规模拷贝的数据，弄清数据的含义以及拷贝之后的流程走向，因为没有对相关函数进行反汇编阅读，理解上难免狭隘肤浅

## 0x1 调试环境

- [操作系统]: Windows Server 2003 R2 Enterprise Edition SP2
- [调试器]: WinDbg 6.11.0001.402 X86
- [Python]: Python 2.7

## 0x2 搭建调试环境

### 0x2.1 安装IIS 6.0

在Add or Remove Programs中选择 Add/Remove Windows Components

![](https://bbs.kanxue.com/upload/attach/201704/727775_vkbseh1bllgf5ei.png)

对Application Server选项的描述中提到了IIS

点击Details全部选中(只选择IIS应该也可以)，然后点击Next

在安装组件的过程中需要插入安装光盘

安装完成之后打开IIS服务管理器

![](https://bbs.kanxue.com/upload/attach/201704/727775_z5osk2df86htp87.png)

鼠标右键将WebDAV设置为Allow

打开服务，查看WebClient，需要将其开启

![](https://bbs.kanxue.com/upload/attach/201704/727775_ms06zui00pqtu4j.png)

目前处于停止状态且无法开启，将该服务设为自启动，然后重启

此时服务正常开启

### 0x2.2 运行样本文件

将python路径设为环境变量，使用控制台窗口运行样本

![](https://bbs.kanxue.com/upload/attach/201704/727775_o2zn2jfpeb8nlos.png)

查看任务管理器，多出了隐藏窗口的calc.exe进程

![](https://bbs.kanxue.com/upload/attach/201704/727775_arhsw6rw029wtbm.png)

使用Process Explorer查看进程依赖关系

![](https://bbs.kanxue.com/upload/attach/201704/727775_4sn5mvh6xhtf7kd.png)

可以看到calc.exe的父进程是w3wp.exe

查看calc.exe进程详细信息

![](https://bbs.kanxue.com/upload/attach/201704/727775_8y1u0hm9oa70pkx.png)

## 0x3 调试样本

由于w3wp.exe并不是开机启动的进程，而样本只是简单启动计算器，干脆先执行一次脚本，然后使用Windbg附加调试

既然会创建新进程，那就对几个关键API下断等它来

```
0:022> bu CreateProcessA

0:022> bu CreateProcessW

0:022> bu WinExec

0:022> bl

 0 e 77e424a9     0001 (0001)  0:**** kernel32!CreateProcessA

 1 e 77e42474     0001 (0001)  0:**** kernel32!CreateProcessW

 2 e 77ea411e     0001 (0001)  0:**** kernel32!WinExec
```

执行起来，中断到kernel32!WinExec函数中

```
0:025> g

Breakpoint 2 hit

eax=77ea411e ebx=7ffe0300 ecx=68031614 edx=876f8b31 esi=68031460 edi=680124e3

eip=77ea411e esp=680313f8 ebp=68031581 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

kernel32!WinExec:

77ea411e 8bff            mov     edi,edi
```

查看调用栈

```
0:006> k

ChildEBP RetAddr  

68031581 00000000 kernel32!WinExec
```

没有调用关系可以参考，好了，老招数，对esp下断

```
0:022> bu WinExec

0:022> ba w4 680313f8 "kb; r; gc"

0:022> bl

 0 e 77ea411e     0001 (0001)  0:**** kernel32!WinExec

 1 e 680313f8 w 4 0001 (0001)  0:**** rsaenh!g_pfnFree+0x13c "kb; r; gc"
```

凭感觉收获真是太多了

```
0:022> g

ChildEBP RetAddr  Args to Child              

0114f798 67119469 680312c0 0114f800 00000000 httpext!ScStoragePathFromUrl+0x360

0114f7ac 6712544a 020f2878 680312c0 0114f800 httpext!CMethUtil::ScStoragePathFromUrl+0x18

0114fc34 6712561e 020f01e8 01ecee36 0114fc78 httpext!HrCheckIfHeader+0x124

0114fc44 6711f659 020f01e8 01ecee36 00000001 httpext!HrCheckStateHeaders+0x10

0114fc78 6711f7c5 020f0238 0114fcd4 671404e2 httpext!CPropFindRequest::Execute+0xf0

0114fc90 671296f2 020f0238 00000004 011d5b88 httpext!DAVPropFind+0x47

0114fce0 67117bc6 01ecffa0 020f01e8 011d5b88 httpext!CDAVExt::DwMain+0x12e

0114fe04 5a322991 011d5b88 011d48c8 011d5518 httpext!DwDavFSExtensionProc+0x3f

0114fe24 5a3968ff 011d5af8 67117b87 0114fe50 w3isapi!ProcessIsapiRequest+0x214

...(lines have been omitted)...

eax=000005fc ebx=000002fd ecx=00000137 edx=680312c0 esi=020f29b8 edi=68031404

eip=67126fdb esp=0114f330 ebp=0114f798 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010202

httpext!ScStoragePathFromUrl+0x360:

67126fdb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]

...(lines have been omitted)...

ChildEBP RetAddr  Args to Child              

68031581 00000000 0c528b30 8b14528b b70f2872 rsaenh!g_pfnFree+0x356

eax=68031633 ebx=7ffe0300 ecx=00000000 edx=6803163e esi=68031460 edi=680124e3

eip=68031612 esp=680313f8 ebp=68031581 iopl=0         nv up ei pl nz ac po cy

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000213

rsaenh!g_pfnFree+0x356:

68031612 ffd5            call    ebp {rsaenh!g_pfnFree+0x2c5 (68031581)}

ChildEBP RetAddr  Args to Child              

68031581 00000000 0c528b30 8b14528b b70f2872 rsaenh!g_pfnFree+0x33e

eax=77ea411e ebx=7ffe0300 ecx=68031614 edx=876f8b31 esi=68031460 edi=680124e3

eip=680315fa esp=680313f8 ebp=68031581 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

rsaenh!g_pfnFree+0x33e:

680315fa ffe0            jmp     eax {kernel32!WinExec (77ea411e)}

...(lines have been omitted)...
```

重新加载样本，对如下rep movsd指令的地址下断

```
httpext!ScStoragePathFromUrl+0x360:

67126fdb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

设置断点

```
0:022> bu 67126fdb

0:022> bl

 0 e 77ea411e     0001 (0001)  0:**** kernel32!WinExec

 1 e 67126fdb     0001 (0001)  0:**** httpext!ScStoragePathFromUrl+0x360
```

抢先剧透，这个漏洞的产生的确来自这里，这里的数据拷贝会来三次

通常俺喜欢从上层函数开始往下看

```
0:006> k

ChildEBP RetAddr  

00fef798 67119469 httpext!ScStoragePathFromUrl+0x360

00fef7ac 67125484 httpext!CMethUtil::ScStoragePathFromUrl+0x18

00fefc34 6712561e httpext!HrCheckIfHeader+0x15e

00fefc44 6711f659 httpext!HrCheckStateHeaders+0x10

00fefc78 6711f7c5 httpext!CPropFindRequest::Execute+0xf0



0:006> ub 67119469

httpext!CMethUtil::ScStoragePathFromUrl+0x2:

67119453 55              push    ebp

67119454 8bec            mov     ebp,esp

67119456 8b4910          mov     ecx,dword ptr [ecx+10h]

67119459 8b5508          mov     edx,dword ptr [ebp+8]

6711945c 6a00            push    0

6711945e ff7510          push    dword ptr [ebp+10h]

67119461 ff750c          push    dword ptr [ebp+0Ch]

67119464 e812d80000      call    httpext!ScStoragePathFromUrl (67126c7b)
```

好了重新加载样本，新增断点，对地址0x67119464下断

```
0:022> bu 680313f8

0:022> bu 67119464

0:022> bl

 0 e 77ea411e     0001 (0001)  0:**** kernel32!WinExec

 1 e 67119464     0001 (0001)  0:**** httpext!CMethUtil::ScStoragePathFromUrl+0x13
```

运行样本，中断在httpext!ScStoragePathFromUrl函数调用前

```
0:022> g

Breakpoint 1 hit

eax=0118f800 ebx=020f01e8 ecx=01ecfa80 edx=020f08f0 esi=00000000 edi=77bd8ef2

eip=67119464 esp=0118f7a0 ebp=0118f7ac iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

httpext!CMethUtil::ScStoragePathFromUrl+0x13:

67119464 e812d80000      call    httpext!ScStoragePathFromUrl (67126c7b)
```

当前函数是这样的

```
httpext!CMethUtil::ScStoragePathFromUrl:

67119451 8bff            mov     edi,edi

67119453 55              push    ebp

67119454 8bec            mov     ebp,esp

67119456 8b4910          mov     ecx,dword ptr [ecx+10h]

67119459 8b5508          mov     edx,dword ptr [ebp+8]

6711945c 6a00            push    0

6711945e ff7510          push    dword ptr [ebp+10h]

67119461 ff750c          push    dword ptr [ebp+0Ch]

67119464 e812d80000      call    httpext!ScStoragePathFromUrl (67126c7b)

67119469 5d              pop     ebp

6711946a c20c00          ret     0Ch
```

跟进httpext!ScStoragePathFromUrl，单步几句

```
httpext!ScStoragePathFromUrl:

67126c7b b8150d1467      mov     eax,offset httpext!swscanf+0x14b5 (67140d15)

67126c80 e803100000      call    httpext!_EH_prolog (67127c88)

67126c85 81ec50040000    sub     esp,450h

67126c8b a1c0701467      mov     eax,dword ptr [httpext!__security_cookie (671470c0)]

67126c90 8945f0          mov     dword ptr [ebp-10h],eax
```

此时内存布局为

![](https://bbs.kanxue.com/upload/attach/201704/727775_f4ldfyleon7yrxt.png)

G一下，到拷贝数据的位置

```
0:006> g

Breakpoint 2 hit

eax=00000130 ebx=00000097 ecx=0000004c edx=00fef804 esi=020f0910 edi=00fef828

eip=67126fdb esp=00fef330 ebp=00fef798 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

httpext!ScStoragePathFromUrl+0x360:

67126fdb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
```

先不急，观察一下上下文，附近有四句rep movsd指令
