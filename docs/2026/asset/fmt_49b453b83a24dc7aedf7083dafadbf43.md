
# [原创]Windows（x86）页表与虚拟空间之我见

## 运行环境

- 主机系统：Windows 10 x64
- 目标系统：Windows XP sp3 x86

## 工具软件

- 虚拟机：Virtual Box
- 编译器：Visual Studio 2017
- 调试器：Windbg、OllyDBG

## 参考书籍

- 《Windows 内核设计思想》
- 《Windows 核心编程》
- 《软件调试》

## 参考博文

- 《[OS 学习笔记](https://bbs.kanxue.com/elink@42fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4p5I4x3o6l9%4y4K6t1&6z5e0V1I4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5#2x3U0f1K6z5o6f1%4x3b7`.`.)》
- 《[详解Windows内存分页机制](https://bbs.pediy.com/thread-135274.htm)》

本文引用了以上书籍、文章的部分观点，算是一篇学习笔记。在此感谢各位作者！

页表机制准确的说是CPU实现的特性，由OS加以利用。网上有很多陈述页表机制的文章，但以Linux居多，Windows偏少，且很多都是理论层面借图表的阐述，或许对于科班出身的人士来说是小菜一碟，可对我等自学爱好者而言却经常是一脸懵逼的感叹！因此，萌发了动手实践的想法，经过一翻折腾，总算小有心得，就此记录于网际，希望本文能给和我一样的爱好者以帮助。本文可能存在很多歪解，请各位看官多多斧正！

## 本文试图解决的疑问

- 页有多少个？
- 页每个进程都有吗？
- 页多大？
- 页放在哪里？
- 页如何查看？
- 页如何修改？
- 页如何隔离进程？
- 页和虚拟地址的关系？
- 页在何时被使用？

## 从`test.exe`开始

```
// test.exe 源码

#include <iostream>



int main()

{

    std::cin.get ();			// 等待做手脚



    int* p = NULL;

    *p = 0x89abcdef	;		// 向0指针写入数据

    std::cout << "Hello nullptr!\n";	// 改编自经典:)



    return 0;

}
```

代码很简单不出大问题的话，编译运行敲回车直接崩溃QAQ。在默认设置的Win10（17134）`cmd`下可正常运行和退出，但没有打印“Hello nullptr”，在XP下报错终止，错误代码`0xC0000005`（访问违规）。可见Win10在异常处理上有所调整（记得《Windows核心编程》有提过在Vista之后即如此）。

有C/C++基础的码友一定记得一条原则，`0x00000000`为空指针，设计思想很简单，例如调用`malloc`返回0表示失败，如果返回0指针是可用的，那错误用什么表示呢？

我的运行结果是正常的，我承认我搞事情了，嘿嘿！

![](https://bbs.kanxue.com/upload/attach/201809/709052_4MREJRMXSUGF7YM.png)

在动手完成空指针写入数据之前先大致了解一些基本情况，在Windbg枚举进程。

```
kd> !process 0 0

**** NT ACTIVE PROCESS DUMP ****

PROCESS 8a0e59c8  SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000

    DirBase: 00039000  ObjectTable: e1000d10  HandleCount: 240.

    Image: System



PROCESS 89be5020  SessionId: 0  Cid: 05d8    Peb: 7ffd6000  ParentCid: 05e4

    DirBase: 19f3c000  ObjectTable: e1e27cc8  HandleCount:  35.

    Image: cmd.exe



PROCESS 89ba8318  SessionId: 0  Cid: 0578    Peb: 7ffde000  ParentCid: 0268

    DirBase: 19f97000  ObjectTable: e1b8ef08  HandleCount:  82.

    Image: taskmgr.exe



PROCESS 89f52c10  SessionId: 0  Cid: 07c0    Peb: 7ffde000  ParentCid: 05e4

    DirBase: 1d42f000  ObjectTable: e1ba5700  HandleCount:  77.

    Image: OllyICE.exe



PROCESS 89bfc470  SessionId: 0  Cid: 01b8    Peb: 7ffde000  ParentCid: 05d8

    DirBase: 24231000  ObjectTable: e1d764b8  HandleCount:   7.

    Image: test.exe

...
```

信息经过删减，留下了几个比较熟悉的进程，文中类似信息也会将无关清除，清除部分会用“...”表示，不再单独注明。

这些进程分别是：`taskmgr.exe`任务管理器、`cmd.exe`控制台、`System`系统进程、`OllyICE.exe`调试器、`test.exe`测试程序。

每个进程的`DirBase`项的16进制数可以发现以下特征：

1. 各进程不同；
2. 可被4096整除；
3. 以000结尾。

如果你的显示结果与特征不符，那是另外一种模式PAE，本文不做解释。从特征上分析，可被4096整除说明是4K对齐；各进程不相同说明每个进程的起始页不同,因此得以隔离进程；000结尾说明PDE没有属性，PDE为何物见下文。

了解了进程信息之后，再来看看进程的地址、占用空间大小等信息，着重观察`test.exe`。

![](https://bbs.kanxue.com/upload/attach/201809/709052_2NQNJEWQVDBADP7.png)

在任务管理器窗口红色框部分，是各进程使用物理内存占用大小，单位Kb，这些数都可以被4整除，所得商即该程序使用的内存页数。

当`test.exe`在前台时（最大化）使用了193（772K/4）页内存，在后台时（最小化）使用9（36/4）页，由此可见当程序非活动状态或需求非常少时，物理内存会归还给OS。从`test.exe`代码可知，程序阻塞在`cin.get ()`调用，基本上没什么运行需求。（切换的窗口是`cmd`，因test运行于此）。

OD窗口红色框部分，是`test.exe`的逻辑地址和空间大小信息，此空间大小是程序自身可能用到的大小，粗略合计约1000（4M/4K）页，通过OD的简要信息，可获知地址处都存放了什么，如主栈、PE头、`.text`代码段等。这里的空间大小基本是固定的。

经过以上分析得到一个结果，程序虚拟空间远大于物理占用，实际上程序在运行时，不会将所有内容都放入物理内存，仅将当前运行所需要的代码、数据放入物理页。

有了这些线索，我们通过实验来进一步印证。利用PROCESS项的16进制数将`test.exe`切换至当前环境。默认当前环境是`System`进程，所以查询显示信息将是`System`进程的。

```
kd> !process 0 0

...

PROCESS 89bfc470  SessionId: 0  Cid: 01b8    Peb: 7ffde000  ParentCid: 05d8

    DirBase: 24231000  ObjectTable: e1d764b8  HandleCount:   7.

    Image: test.exe

...

kd> .process /i /p 89bfc470

kd> g

kd> r cr3

cr3=24231000
```

使用`r`指令查看了`cr3`寄存器，显示的信息与`DirBase`项一致，实际上OS在后台不断的自动切换`cr3`，让每个进程都得到一点运行时间。

### 实验所需的知识点简单介绍一下（以下仅是小页面，大页面请翻阅书籍）

每个进程都有一页（4K）存放一个页目录（Page Directory Entry，PDE），PDE占4K空间，分成1024项，每项4B，每项描述一个页表（Page Table Entr，PTE），PTE的高20位是基址域，低12位是属性域；PTE也占4K，同样可分成1024项，每项4B，每项描述一页（Page），Page的高20位是基址域，低12位是属性域。

根据上述可得：

1024（PTE 个数）\*1024（Page 个数）\*4096（Page 大小）=4G（总空间）；

1024（PTE 个数）\*1024（Page个数）\*4（描述项大小）=4M（总占用）；

1024（PTE个数）\*1024（Page个数）=1M（总页数）；

实际总占用仅是理论值，一般程序不可能达到4G，下面实验会得到证实。

查看`test.exe`的PDE，我将PDE分成两部分，前512项和后512项分别进行分析，因PDE管理4G虚拟空间，但其高2G（后512项）是内核态使用，剩余低2G（前512项）由用户态使用（PAE模式用户态拥有更多的空间）。

```
kd> !process 0 0

...

PROCESS 89bfc470  SessionId: 0  Cid: 01b8    Peb: 7ffde000  ParentCid: 05d8

    DirBase: 24231000  ObjectTable: e1d764b8  HandleCount:   7.

    Image: test.exe

...

kd> !dd 24231000 l200

#24231000 24766067 245e0067 00000000 00000000

...

#242317c0 00000000 00000000 247cf067 00000000

...

#242317f0 00000000 2459a067 00000000 2451e067

...
```

PDE前512项中只有5个非空PTE，前面说过1个PTE占用4K物理内存，含1024个Page描述，也就是说1个PTE维护4M（1024\*4096）空间，因此可认为OS给`test.exe`分配了20M（4\*5）虚拟空间。

我把以上数据和OD的逻辑地址整理成表进行对比。

| 物理地址（Windbg） | 逻辑地址（OD） | 范围（4M） |
| --- | --- | --- |
| `#24231000` | `00010000 ... 003AF000` | `0x00000000~0x003FFFFF` |
| `#24231004` | `00400000 ... 0042D000` | `0x00400000~0x007FFFFF` |
| `#242317c8` | `7C800000 ... 7C9B0000` | `0x7C800000~0x7CBFFFFF` |
| `#242317f4` | `7F6F0000` | `0x7F400000~0x7F7FFFFF` |
| `#242317fc` | `7FFA0000 ... 7FFE0000` | `0x7FC00000~0x7FFFFFFF` |

我相信你已经找到了相似特征，逻辑地址与PTE是映射关系。更准确的说逻辑地址值是一个三段式数据结构，关于该结构我用一段代码来说明。

```
// demo.exe 源码

#include <iostream>



int main()

{

    struct {

        ULONG offset : 12;

        ULONG pteIndex : 10;

        ULONG pdeIndex : 10;

    } va { 0 };



    using namespace std;



    while (true) {

        cin >> hex >> *(ULONG*)&va;

        cout << hex << "逻辑地址：0x" << *(ULONG*)&va << endl

             << "PDE索引：0x" << va.pdeIndex

             << " PDE物理偏移：0x" << va.pdeIndex * 4 << endl

             << "PTE索引：0x" << va.pteIndex

             << " PTE物理偏移：0x" << va.pteIndex * 4 << endl

             << "页内偏移：0x" << va.offset << endl;

    }



    return 0;

}
```

![](https://bbs.kanxue.com/upload/attach/201809/709052_VWWKMJBRPNXXFV2.png)

PDE可通过`cr3`得到，PDE+偏移即可找到PTE描述，PTE+偏移又可找到Page描述，Page+页内偏移可指向具体数据，这就是用逻辑地址搜索页表的过程。

### 进一步查看PTE索引1（第二个PTE）的内容

```
kd> !process 0 0

...

PROCESS 89bfc470  SessionId: 0  Cid: 01b8    Peb: 7ffde000  ParentCid: 05d8

    DirBase: 24231000  ObjectTable: e1d764b8  HandleCount:   7.

    Image: test.exe

...

kd> !dd 24231000

#24231000 24766067 245e0067 00000000 00000000

...

kd> !dd 245e0000 l400

#245e0000 24430025 2456c025 2436d025 243ee025

#245e0010 242af025 24470025 24271025 244f2025

#245e0020 242b3025 242f4025 24375025 242b6025

#245e0030 23fb7025 242b8025 240f9025 2423a025

#245e0040 243fb025 241bc025 241bd025 242be025

#245e0050 242bf025 24540025 243c1025 24502025

#245e0060 24543025 248c4025 24705025 27da8025

#245e0070 2459e025 24647025 00000000 24708025

#245e0080 24349025 00000000 2444a025 2454b025

#245e0090 2450c025 00000000 00000000 00000000

#245e00a0 24362025 24523025 24664025 2465f067

#245e00b0 24660067 24625025 00000000 00000000

...
```

截取部分中共41项非空Page描述（省略的都是空Page），第二个PTE管理的4M空间范围是`0x00400000~0x007FFFFF`，回顾OD截图在这4M范围内的大小合计是46页，这里相差5个页，原因是什么我不清楚，猜测可能是软件之间的误差，有知道的请告知！

PTE和Page的低12位是属性域，数据上对应的如`025`、`067`等，属性域其中一位描述了数据是在物理内存还是在硬盘的虚拟内存，OS通过属性域控制内存，PTE可控制4M，Page可控制4K。更多属性的内容请翻阅书籍。

在OD截图可知`0x00401000`是`.text`段的起始，从`demo.exe`程序的显示结果获知该逻辑地址PTE的索引1偏移4，现在来看该Page内容：

```
kd> !process 0 0

...

PROCESS 89bfc470  SessionId: 0  Cid: 01b8    Peb: 7ffde000  ParentCid: 05d8

    DirBase: 24231000  ObjectTable: e1d764b8  HandleCount:   7.

    Image: test.exe

...

kd> !dd 24231000

#24231000 24766067 245e0067 00000000 00000000

...

kd> !dd 245e0000

#245e0000 24430025 2456c025 2436d025 243ee025

...

kd> !db 2456c000

#2456c000 b9 a0 c1 42 00 e8 24 2b-00 00 68 29 b5 41 00 e8

#2456c010 d2 25 00 00 59 c3 68 33-b5 41 00 e8 c6 25 00 00

#2456c020 59 c3 68 3d b5 41 00 e8-ba 25 00 00 59 c3 6a 01

#2456c030 6a 00 68 a8 c2 42 00 b9-00 c3 42 00 e8 82 33 00

#2456c040 00 68 47 b5 41 00 e8 9b-25 00 00 59 c3 56 57 6a

#2456c050 00 e8 ee b9 00 00 59 bf-a8 c2 42 00 8b f0 8b cf

#2456c060 e8 ca 33 00 00 6a 00 56-8b cf c7 05 a8 c2 42 00

#2456c070 18 ce 41 00 e8 98 37 00-00 68 51 b5 41 00 e8 63
```

或许有人会问这些数据是什么？我们来看一张截图，看完之后自然豁然开朗……！
