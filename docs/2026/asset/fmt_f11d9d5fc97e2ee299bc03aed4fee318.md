
```
001F255D push eax
001F255E call dword ptr ss:[ebp+0xD2723EA]; ntdll.ZwOpenSection
001F2564 add esp,0x20
001F2567 xor edx,edx
001F2569 test eax,eax
001F256B setns dl
001F256E neg edx
001F2570 pop eax ; 001F0863
001F2571 and eax,edx
001F2573 retn ; 这里直接跳转到下一个函数
001F2574 push edi
……….
001F2574 push edi
001F2575 xor edi,edi
001F2577 call 001F252F
001F257C je 001F2661
001F2582 push eax
001F2583 push 0xB7C4
001F2588 mov edx,esp
001F258A push 0x0
001F258C mov ecx,esp
001F258E push 0x40 ; Win32Protect
001F2590 push 0x100000; AllocationType
001F2595 push 0x2 ; InheritDisposition
001F2597 push edx ; ViewSize
001F2598 push 0x0 ; SectionOffset
001F259A push 0xB7C4 ; CommitSize
001F259F push 0x0 ; ZeroBits
001F25A1 push ecx ; \*BaseAddress
001F25A2 push ebx ; ProcessHandle
001F25A3 push eax ; SectionHandle
001F25A4 call dword ptr ss:[ebp+0xD2723DE]; ntdll.ZwMapViewOfSection
001F25AA pop edi
001F25AB pop ecx
001F25AC call dword ptr ss:[ebp+0xD2722C6]; kernel32.CloseHandle
001F25B2 test edi,edi
001F25B4 je 001F2661
001F25BA mov ecx,dword ptr ss:[ebp+0xD271891]
001F25C0 jecxz short 001F25CE
001F25C2 lea edx,dword ptr ss:[ebp+0xD271000]
001F25C8 add edx,ecx
001F25CA push edi
001F25CB push ebx
001F25CC call edx ; ntdll.KiFastSystemCallRet
001F25CE>test dword ptr ss:[ebp+0xD275B83],0x
001F25D8 je short 001F25EF
001F25DA mov eax,dword ptr ss:[ebp+0xD2723DA]; ntdll.ZwDeviceIoControlFile
001F25E0 test eax,eax
001F25E2 je short 001F25EF
001F25E4 lea ecx,dword ptr ds:[edi+0x68B1]
001F25EA call 001F2477
001F25EF mov eax,dword ptr ss:[ebp+0xD2723C6]; ntdll.ZwCreateFile
001F25F5 lea ecx,dword ptr ds:[edi+0x6591]
001F25FB call 001F2477
001F2600 mov eax,dword ptr ss:[ebp+0xD2723E2]; ntdll.ZwOpenFile
001F2606 lea ecx,dword ptr ds:[edi+0x6616]
001F260C call 001F2477
001F2611 mov eax,dword ptr ss:[ebp+0xD2723CA]; ntdll.ZwCreateProcess
001F2617 lea ecx,dword ptr ds:[edi+0x6620]
001F261D call 001F2477
001F2622 mov eax,dword ptr ss:[ebp+0xD2723CE]; ntdll.ZwCreateProcessEx
001F2628 test eax,eax
001F262A je short 001F2637
001F262C lea ecx,dword ptr ds:[edi+0x662D]
001F2632 call 001F2477
001F2637 mov eax,dword ptr ss:[ebp+0xD2723D6]; ntdll.ZwCreateUserProcess
001F263D test eax,eax
001F263F je short 001F264C
001F2641 lea ecx,dword ptr ds:[edi+0x663A]
001F2647 call 001F2477
001F264C mov eax,dword ptr ss:[ebp+0xD2723F2]; ntdll.ZwQueryInformationProcess
001F2652 test eax,eax
001F2654 je short 001F2661
001F2656 lea ecx,dword ptr ds:[edi+0x666E]
001F265C call 001F2477
001F2661 mov eax,edi
001F2663 pop edi
001F2664 retn
```

![](upload/attach/202002/214974\_3WFD25UBGN82BEB.jpg)

图：OD-挂钩底层API函数

#### 2.3.3.9 发送网络数据包

在远程线程注入执行后，在`winlogon.exe`进程出现了联网行为。再此之前的单步跟踪则是`dwm.exe`进行联网访问，推测病毒是将联网执行的恶意代码注入到某个进程后，将联网操作交给了被注入的进程中。且联网的进程是存在随机性的。

![](upload/attach/202002/214974\_9KXMZ95M8GTA683.jpg)

图：联网行为监控

运行后会访问国外IP：148.81.111.121，通过VT查询该IP与多个恶意代码HASH绑定，Wireshark抓包后的数据包：

![](upload/attach/202002/214974\_5ZHZ8NHDCU8MV2P.jpg)

图：访问IP

通过Windows系统自带的DNS记录跟踪大量域名后，用系统命令确定病毒感染后访问的国外IP绑定的域名为`ant.trenz.pl`。

![](upload/attach/202002/214974\_K2RNU7GZW7BZ6NY.jpg)

图：Windwos自带DNS记录访问的域名

![](upload/attach/202002/214974\_WXFC2W4KHXSYFWX.jpg)

图：系统命令关联域名的IP地址

PChunter查看`winlogon`已经被注入了3个线程。

![](upload/attach/202002/214974\_KRWU43VX9HNHXG3.jpg)

#### 2.3.3.10 恢复执行原始文件

跟到最后的几个`call`，已经是运行原先程序的代码。因为我用母体感染的是我自己写的程序。`printf("11111111")`就是。。。

# 3、思考小结

这个病毒分析起来还是有意思的，没有办法像某些常见病毒一样可以IDA F5大法轻松查看伪C代码。C和汇编的基础需要提升的前提下，也引发了一些思考。

### 1、病毒分析方面

- 1）使用OD动态调试对于病毒运行的流程架构不像IDA一般清晰，如何使用IDAPython辅助将`esi`等寄存器传值的函数在IDA中标识出来让自己通过静态分析，提升分析速度。

- 2）对于挂钩底层API函数这里，是在同事指点下修正过了。网上的文章分析中提到使用了`zwXXXXX`这类函数就是挂钩了内核函数是不严谨的。只能相当于调用了系统DLL中比较底层的函数，距离内核还差一段距离。

### 2、编程技术方面

- 1）如果是我自己写这类程序，还有什么思路可以隐藏进程。

- 2）注入到系统进程内存中的数据，采用什么手段可以排查。

# 4、致谢参考文章

virut详细分析 - 米哈伊尔 - 博客园

<mark class="encrypted">447K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3E0Z5j5h3W2D9i4K6u0r3M7q4)9J5c8U0f1$3x3e0f1J5z5o6k6Q4x3X3g2Z5N6r3#2D9</mark>

附件密码：pediy

# 1.样本概况

| 病毒名称 | Virus.Win32.Virut.ce |

| --- | --- |

| MD5 | 6A500B42FC27CC5546079138370C492F |

| 文件大小 | 131 KB (134,144 字节) |

| 壳信息 | 无壳 |

| 文件名 | 9-29\_vir.exe（感染程序） |

| 数字签名 | NO |

| 时间戳 | 2000年1月10日, 12:00:00 |

## 1.1 样本行为

　恶意代码为感染型病毒，母体病毒通过将正常的PE文件进行加密，然后每个受感染的文件又可以进行全盘传播，通过连接国外IP服务器发送上线报文。

## 1.2 测试环境及工具

| 操作系统 | Windows7（x86） |

| --- | --- |

| 行为检测 | Wireshark、Api Monitor、Malware Defender |

| 文件检测 | 010 Editor、PEID |

| 虚拟机平台 | VMware10 |

| 调试工具 | OD、IDA |

## 1.3 分析目标

　1）样本的网络行为、文件行为

　2）感染传播的方式

# 2、具体行为分析

## 2.1 主要行为概述

![](https://bbs.kanxue.com/upload/attach/202002/214974_MDGCUWM2H268CG4.jpg)

## 2.2 恶意程序对用户造成的危害

### 2.2.1 感染途径

- 通过母体文件，或被母体文件感染过的文件感染。

- 运行后会将恶意代码以及模块注入`explorer.exe\Conhost.exe\winlogo.exe\dwm.exe`等进程。

### 2.2.2 恶意行为

- 感染已经打开的进程和磁盘可执行文件，重写程序入口点的代码并指向病毒代码。

- 与国外IP发送数据包进行通讯连接。

## 2.3 恶意代码分析

### 2.3.1 恶意程序加固方式

　1）加密处理：病毒代码处理经过多层的解密指令，运行过程中对后续代码进行解密，解密代码会变形。例如原反汇编指令代码如下：

```
00424772 imul edx,eax,0xCFA924B3

00424778 mov ecx,0x1A7DDAE2

0042477D sbb byte ptr ss:[ebp-0x223],dh; 带借位减法指令,这里会修改42479D位置的ret变成call

00424783 pop edi

00424784 lea edi,dword ptr ds:[edi-0x69B4>

0042478A test dl,bh ; dl = 4D('M');查找文件头

0042478C mov dl,ah

0042478E adc al,bl

00424790 adc al,cl

00424792 push dword ptr ss:[ebp-0x17D]

00424798 add edi,ebp ; 9-29_vir.004249C0

0042479A xchg cl,cl

0042479C pop edx

0042479D retn    ; import - NoTips ---- By Jmp2 -

0042479E add al,0x3

004247A0 add byte ptr ds:[eax],al

004247A2 mov edi,edi
```

　执行完`0042477D sbb byte ptr ss:[ebp-0x223],dh`这条语句后，机器码为`C3`的改成了`E8`，`ret`就变成了`call`。修改后的代码如下：

![](https://bbs.kanxue.com/upload/attach/202002/214974_QDAWXQD6FV48YZP.jpg)

图：指令被修改

　2）花指令：代码会在指令中增加`00`，在鼠标滑动时指令发生变化。

![](https://bbs.kanxue.com/upload/attach/202002/214974_5WY45UW489JXUZ7.jpg)

图：花指令

### 2.3.2 恶意行为过程截获

- 恶意行为

  （1）修改其他进程的内存：

![](https://bbs.kanxue.com/upload/attach/202002/214974_43TV6BF5BY6TSZN.jpg)

图：修改其程序内存

（2）API行为监控：

![](https://bbs.kanxue.com/upload/attach/202002/214974_Y9BR5DBEYDWNFXN.jpg)

图：API行为监控

（3）感染文件变化：

![](https://bbs.kanxue.com/upload/attach/202002/214974_5DDRYE9TVY64398.jpg)

图：源文件与感染文件对比

- 网络行为

　通过Malware Defender对被感染的文件进行监控，发现每个被感染的文件自成母体病毒文件向当前的系统进程`dwm.exe`进行了修改。通过`dwm.exe`文件访问国外IP。

![](https://bbs.kanxue.com/upload/attach/202002/214974_GGQ3AMMBJFAKCYR.jpg)

图：注入进程，进程访问网络

　病毒运行后会访问国外IP：148.81.111.121，通过VT查询该IP与多个恶意代码HASH绑定，Wireshark抓包后的数据包：

![](https://bbs.kanxue.com/upload/attach/202002/214974_HXD8RSCJUJHKU76.jpg)

图：访问IP

　通过Windows系统自带的DNS记录跟踪大量域名后，通过系统命令确定病毒感染后访问的国外IP绑定的域名为`ant.trenz.pl`。

![](https://bbs.kanxue.com/upload/attach/202002/214974_TA4SZPAXPKWZRM9.jpg)

图：Windwos自带DNS记录访问的域名

![](https://bbs.kanxue.com/upload/attach/202002/214974_5948XWDKKHHT5CC.jpg)

图：系统命令关联域名的IP地址

### 2.3.3 恶意程序的代码分析记录

#### 2.3.3.1 调用病毒代码

　病毒代码执行前，首先获取`kernel32.dll`入口循环遍历代码所需函数，通过IDA静态可查看代码运行的结构是在不断进行循环。

![](https://bbs.kanxue.com/upload/attach/202002/214974_MNDFD2KNZXZ6HRA.jpg)

图：OD-kernel32入口地址

![](https://bbs.kanxue.com/upload/attach/202002/214974_QQNR5QQ6DGTGM7D.jpg)

图：IDA-kernel32函数地址遍历

　当面对这类感染类病毒，很多使用寄存器进行传值的指令是无法用IDA轻轻松松F5查看源码的。

　所以分析过程中我采用通过行为监控+动态调试的思路分析！打开Malware Defender，借助吾爱破解OD进行F8单层步过直到程序结束。当Malware Defender出现关键行为时，对走过的`call`进行记录，然后深入这个`call`再逐层分析。

　跟踪到第6个函数时，程序出现了修改其他进程内存、其他进程中创建线程的行为。

![](https://bbs.kanxue.com/upload/attach/202002/214974_TAZPFX7WWXRTGD2.jpg)

图：第6个call为重点分析函数

　通过反复还原虚拟机快照对比，发现该病毒在感染后 `cmp al,bl` 执行的分支是不一样的，如果没有被感染的系统`JNZ`不跳转程序继续向下执行，否则`JNZ`则跳转到下一个分支处。对比值是`eax`与`ebx`的寄存器的低位值。此时的`AL`与`BL`的值分别为：`AL=B7`，`BL=00`。

　为了验证这个`B7`的值是否为随机值，切换到另外一台虚拟机再次单步跟踪有操作`EAX`的的位置，发现在`00424A38`地址处执行完`mov eax,dword ptr ds:[eax+0x34];`，`B7`的值就没有发生过改变，由此得出`B7`这个值就是病毒判断系统有没有被感染的标识，对比反汇编代码如下：
