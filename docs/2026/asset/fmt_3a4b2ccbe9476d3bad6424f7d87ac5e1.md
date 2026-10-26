
# [原创]分析某游戏驱动保护的学习历程

## 一、前言

一直都对游戏保护感兴趣，最近想看一看游戏驱动是怎么写的。于是便尝试逆向分析一下。在这个过程中学到很多。

## 二、驱动调试环境的搭建

由于驱动运行在系统内核层，所以对驱动的调试一般采用双机调试。物理机对物理机，或者物理机对虚拟机。因为手上设备限制，就一个笔记本。所以首先想到的是开虚拟机（VMware Workstation Pro）进行双机调试。

### 2.1 虚拟机中游戏运行环境的准备

一般情况下，游戏都会检测虚拟机运行环境。网上收集一波资料，找到hzqst的一个项目。

<a href="elink@d60K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5P5Y4q4K6N6q4)9J5c8W2k6E0N6$3q4J5k6f1S2S2M7X3c8W2L8X3g2V1e0r3!0S2k6r3g2J5">hzqst/VmwareHardenedLoader: Vmware Hardened VM detection mitigation loader (anti anti-vm) (github.com)</a>

按照使用说明，卸载vmtools，修改虚拟机配置文件，安装驱动，就能正常运行游戏。

但是卸载vmtools后，虚拟机很卡，而且屏幕显示也很别扭。所以还是希望能在安装了vmtools的情况下运行游戏。经过测试发现是可行的。以下是操作步骤：

1. 修改虚拟机配置文件

   ```c++
   hypervisor.cpuid.v0 = "FALSE"
   board-id.reflectHost = "TRUE"
   hw.model.reflectHost = "TRUE"
   serialNumber.reflectHost = "TRUE"
   smbios.reflectHost = "TRUE"
   SMBIOS.noOEMStrings = "TRUE"
   ethernet0.address = "00:11:22:33:44:55" // 这里随意
   ```

2. 安装vmloader驱动

3. 修改vmtools显卡驱动名字（可选）

   安装驱动精灵或者驱动人生，备份显卡驱动。用7z或者winrar打开备份的压缩包（不解压），编辑`inf`文件。

   ```c++
   // 将inf文件以下三行任意修改
   DiskID = "VMware Tools"
   CompanyName = "VMware, Inc."
   SVGA = "VMware SVGA 3D"
   ```

   保存文件后。用驱动精灵或者驱动人生还原显卡驱动即可。

完成修改后，成功运行游戏。

![游戏运行截图](upload/attach/202212/958869_CZPVKTJFEYPFUZ9.jpg)

### 2.2 双机调试方式的选取

在看雪论坛看了一圈发现，游戏驱动基本都对Windbg双机调试进行了检测和处理。目前我并不清楚是怎么检测的，所以放弃VMWare+kdstub+windbg的双机调试方式，另寻他法。经过搜索，发现VMWare+gdbstub+IDA-gdb是个很好的方式。

#### 2.2.1 IDA+VMWare调试配置

1. 修改虚拟机配置文件

   ```yaml
   debugStub.listen.guest64 = "TRUE"
   debugStub.hideBreakpoints = "TRUE"
   monitor.debugOnStartGuest64 = "TRUE"
   ```

2. IDA启动gdb调试器

   ![启动调试器](upload/attach/202212/958869_BM5GX2R9ESDCSZY.jpg)

   ![gdb](upload/attach/202212/958869_FE333D8JCBEHKUF.jpg)

   虚拟机在本地运行，则`hostname`填`localhost`，目标系统是64位，则`port`填`8864`（32位是`8832`）在这个窗口等待备用。

   去VMware启动虚拟机后，立刻回到IDA单击OK，那就成功附加上了。

   ![idaok](upload/attach/202212/958869_FCK3BGHTDBN2V5D.jpg)

#### 2.2.2 ntoskrnl.exe调试符号加载

IDA-gdb成功附加之后，默认是没有符号。没有符号的情况下，调试内核并不方便。所以得想办法加载上微软的巨硬PDB。内核`ntoskrnl.exe`的`pdb`符号文件先下载好备用。IDA在调试时是可以加载`pdb`文件。

![loadpdb](upload/attach/202212/958869_KVVZMWFX89TVBMU.jpg)

![pdb](upload/attach/202212/958869_C8ZUD63PY8E5F2U.jpg)

所以只需要得到内核模块`ntoskrnl.exe`的符号文件和内存加载地址就可以了。而IDA附加之后，断下的位置还在系统启动阶段，并没有加载`ntoskrnl.exe`。那么可以寻找到载入内核模块，并调用内核入口函数的时机。就能获取到`KiSystemStartup`函数地址减去函数偏移就能得到`ntoskrnl`的基址。简单逆向`winload.exe`（有符号）找到一个关键函数`OslArchTransferToKernel`。

![entrypoint2](upload/attach/202212/958869_NXANJQ7WX39FU8R.jpg)

![entrypoint3](upload/attach/202212/958869_M6MS2K85X65FE92.jpg)

那么只需要在这个位置下断点，即可跟踪到内核入口。IDA搜索一下`OslArchTransferToKernel`函数末尾指令对应的字节码：

`49 8B CC 56 6A 10 41 55 48 CB`

就能确定`OslArchTransferToKernel`函数位置。

![retfq](upload/attach/202212/958869_FM8JFUT7SXT43XR.jpg)

在单步执行跳过去之前需要先映射一下内存区域。

![memory](upload/attach/202212/958869_4CU3FVUJPKASZ22.jpg)

然后单步执行跳到`KiSystemStartup`函数。

![startup1](upload/attach/202212/958869_QSG8TE52GYVPW7X.jpg)

那么当前的`KiSystemStartup`函数地址为`0xFFFFF80006F90010`，`KiSystemStartup`的偏移是`0x990010`

```text
ntoskrnl.base=0xFFFFF80006F90010-0x990010=0xFFFFF80006600000
```

现在终于获得了基址，配和之前下载备用的`pdb`，IDA再加载一下，就可以有符号了。

![startup2](upload/attach/202212/958869_94K35EG4XX3GAEK.jpg)

## 三、驱动分析过程

目前为止，我们已经有了可以运行游戏的虚拟机环境和能带微软符号调试内核的调试环境。那么就可以做一些感兴趣且有意思的事情了。前人栽树，后人乘凉。先找一下有没有分析过该游戏保护的，拿过来看一看，找一找思路。hzqst大佬！

[[原创\]某南极动物厂新版XX分析——系统线程部分（新瓶旧酒）-软件逆向-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-260331.htm)

看完他的分析，我也想定位一下游戏驱动创建的隐藏了入口的系统线程。那就在启动游戏之前，IDA里在`PsCreateSystemThread`函数处下断点，看第六个参数`StartRoutine`是不是`SeSetAuditParameter`函数内部`jmp rcx`所在地址。

![thread1](upload/attach/202212/958869_ZZ658U4JPJPD7NT.jpg)

一共确定了五个线程的地址，再减去游戏驱动的加载地址（`IopLoadDriver`下断获得）得到函数偏移，就能在IDA里面静态分析了。

```c++
1.FFFFF8001C0B5B60
2.FFFFF8001C0***70
3.FFFFF8001C0***80
4.FFFFF8001C0***80
5.FFFFF8001C0***F0
FFFFE20CF9B6ABE0 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6ABE8 FFFFE20CF9B6ACA9 MEMORY:FFFFE20CF9B6ACA9
FFFFE20CF9B6ABF0 FFFFF8001C1965A1 MEMORY:FFFFF8001C1965A1
FFFFE20CF9B6ABF8 FFFFE20C00000000 MEMORY:FFFFE20C00000000
FFFFE20CF9B6AC00 FFFFAD86F81E36F8 MEMORY:FFFFAD86F81E36F8
FFFFE20CF9B6AC08 FFFFF8000693CC31 SeSetAuditParameter+41
FFFFE20CF9B6AC10 FFFFF8001C0B5B60 MEMORY:FFFFF8001C0B5B60
FFFFE20CF9B6AC90 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC98 0000000000000002 MEMORY:word_2
FFFFE20CF9B6ACA0 FFFFF8001C14B419 MEMORY:FFFFF8001C14B419
FFFFE20CF9B6ACA8 FFFFF80000000000 MEMORY:FFFFF80000000000
FFFFE20CF9B6ACB0 FFFFAD86F81E3238 MEMORY:FFFFAD86F81E3238
FFFFE20CF9B6ACB8 FFFFF8000693CC31 SeSetAuditParameter+41
FFFFE20CF9B6ACC0 FFFFF8001C0***70 MEMORY:FFFFF8001C0***70
FFFFE20CF9B6AC10 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC18 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC20 FFFFF8001C1962F0 MEMORY:FFFFF8001C1962F0
FFFFE20CF9B6AC28 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC30 FFFFAD86F8168578 MEMORY:FFFFAD86F8168578
FFFFE20CF9B6AC38 FFFFF8000693CC31 SeSetAuditParameter+41
FFFFE20CF9B6AC40 FFFFF8001C0***80 MEMORY:FFFFF8001C0***80
FFFFE20CF9B6AC50 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC58 0000000000000002 MEMORY:word_2
FFFFE20CF9B6AC60 FFFFF8001C180C6D MEMORY:FFFFF8001C180C6D
FFFFE20CF9B6AC68 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC70 FFFFAD86F8168338 MEMORY:FFFFAD86F8168338
FFFFE20CF9B6AC78 FFFFF8000693CC31 SeSetAuditParameter+41
FFFFE20CF9B6AC80 FFFFF8001C0***80 MEMORY:FFFFF8001C0***80
FFFFE20CF9B6AC90 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6AC98 0000000000000002 MEMORY:word_2
FFFFE20CF9B6ACA0 FFFFF8001C1835B0 MEMORY:FFFFF8001C1835B0
FFFFE20CF9B6ACA8 0000000000000000 MEMORY:unk_0
FFFFE20CF9B6ACB0 FFFFAD86F8168378 MEMORY:FFFFAD86F8168378
FFFFE20CF9B6ACB8 FFFFF8000693CC31 SeSetAuditParameter+41
FFFFE20CF9B6ACC0 FFFFF8001C0***F0 MEMORY:FFFFF8001C0***F0
```

当我打开另一个IDA载入游戏驱动文件，跳到线程地址，发现直接就能F5（意外之喜）。

![ace_base1](upload/attach/202212/958869_5J5UKPPSJGD4MRE.jpg)

具体分析参考hzqst的帖子，不再多说。看完系统线程部分，自然还想看看其他好功能。再找一波，发现杰克王的帖子。杰克王大佬！

[[原创\]某企鹅xxx-base分析-软件逆向-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-270934.htm)

他的帖子里面对回调部分说得比较少，但是我对游戏驱动回调怎么写的确实感兴趣，开始尝试分析找到回调。

首先找一个能用的ark，看看游戏驱动创建的回调函数地址。同样的减去模块基址得到偏移，再去IDA看一看。

![winark1](upload/attach/202212/958869_VQB52WRYPUZRTB8.jpg)

`LoadImage`回调函数地址`0xFFFFF8001C0B4190`去IDA看看。

![imageroutine1](upload/attach/202212/958869_C9HJ3PNBWSEU7SV.jpg)

好家伙，暗藏只因，趁着双机调试，去那个地址看一看实际的回调函数地址。

![imageroutine2](upload/attach/202212/958869_2FCS89HW8B8Z6TM.jpg)

![imageroutine3](upload/attach/202212/958869_QRHZ884ZMZEWFV9.jpg)

同样能直接F5，其他回调函数都是差不多的模式，不再重复。具体功能我也不再分析细说。

分析完回调，我也想看一看游戏驱动的通信函数，学习一下。跳过去一看，不能F5，直呼坏事。那么下一节浅说代码保护。

![vm1](upload/attach/202212/958869_2YGMRPDDWWWVG26.jpg)

## 四、游戏驱动代码保护浅析

我对代码保护没什么了解，也没有基础。我只是根据双机动态调试，跟踪一下代码运行过程，做出一些判断和操作。

### 4.1 混淆

![vm2](upload/attach/202212/958869_G6ZZN8QH5YS224G.jpg)

在调试过程中发现游戏驱动代码具有一些特别的跳转结构。

```assembly
push r8
mov r8, 0FFFFF8001C2C86C1h
pushfq
add r8, 0FFFFFFFFFFF63D07h
popfq
jmp r8
pop r8
```

其中最奇怪的就是`jmp r8`会跳转到他的下一句指令`pop r8`，那么其实这一段代码什么也没做，就是干扰一下。简单对抗一下IDA-F5小子。发现了关键点，那么我们可以写一下去混淆脚本。同样的，网上找一波资料。找到了 逍遥m 发的帖子。逍遥m大佬！

[[原创\]IDA 驱动vmp变异 去花指令 IDC脚本-软件逆向-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-274287.htm)

那我也是抄一下代码，针对性的改一改。写好一个IDC脚本开干。脚本这里我就不发了，混淆的原理和如何写idc去混淆脚本都贴出来了。相信大家认真点分析都会写去混淆脚本的。毕竟自己动手丰衣足食。贴一下去混淆之后成功F5的截图。

![vm3](upload/attach/202212/958869_KRZBRZ29V8DTHMQ.jpg)

### 4.2 vm

去混淆之后，大部分功能都能看了。但是还有些是被v了的，跑在虚拟机里面。我能力有限，只是简单分析一下。我拿驱动入口里调用的一个函数举例。
