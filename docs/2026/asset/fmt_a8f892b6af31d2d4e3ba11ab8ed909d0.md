
# [原创] Easy RM to MP3 Converter v2.7.3（CVE-2009-1330）个人漏洞分析报告

```
//------------------------------------------------------------
//
//    个人学习二进制漏洞挖掘时写的漏洞报告总结，算作是项目总结
//    研究的类型包括：缓冲区溢出、UAF漏洞、类型混淆
//    包括内容如下：
//    基本的漏洞成因
//    调试工具的使用
//    shellcode(弹框、bindshell)的编写以及改进
//    SEH常识
//    exploit的编写
//    堆喷射
//    ASLR/DEP的基本常识
//    ROP链的构造等
//
//    本来是发到MottoIn上的一些投稿，因为错过了时间，所以发到看雪上接收批评
//    最近在研究渗透以及安卓，驱动也没研究完。。。个人时间比较紧，有空再整理成单独的教程
//------------------------------------------------------------
```

这个漏洞是 0day 书上的标准缓冲区漏洞，`PoC` 实现也非常简单，很多人也都研究过了，我简单的整理一下个人的思路，以及本文所涵盖的内容。

![](http://bbs.pediy.com/upload/attach/201702/734388_b1ga7h6wh9t70jy.png)

这篇包含了弹窗 `shellcode` 的编写以及 `exploit` 的实现，超出了一篇漏洞报告所涵盖的内容，写的时候没考虑太多，有空再整理。

很早之前写的东西，有些概念理解错误，之后更正。

**可直接下载附件**

---

| 软件名称 | Easy RM to MP3 Converter |
| --- | --- |
| 软件版本 | 2.7.3 |
| 漏洞模块 | `RM2MP3Converter.exe` |
| 模块版本 |  |
| 编译日期 | 2006-09-29 |
| 操作系统 | Windows XP/2003/7/8.1/10 |
| 漏洞编号 | CVE-2009-1330 |
| 危害等级 | 中危 |
| 漏洞类型 | 缓冲区溢出 |
| 威胁类型 | 本地 |

**分析人：Red_0range**

2016年12月25日

## 1. 软件简介

Easy RM to MP3 Converter 为 05-06 年流行的一款体积较小的轻量级音频转换软件。

支持常用音频格式文件 `MP3`，`WMA`，`WAV`，`OGG` 等之间的转换。

![](https://bbs.kanxue.com/upload/attach/201702/734388_shq6kwe802d8f6j.png)

*软件界面*

## 2. 漏洞成因

`Easy RM to MP3 Converter`（version 2.7.3.700）加载畸形过长的 `.m3u` 文件时触发缓冲区溢出。

## 3. 利用过程

### 3.0. 概览

构造畸形过长的 `.m3u` 文件，软件加载文件后跳转到指定代码，执行弹出对话框操作（根据 `shellcode` 决定）。

![](https://bbs.kanxue.com/upload/attach/201702/734388_js8p75q9keroklo.png)

*软件漏洞利用示例*

### 3.1. 相关知识说明

#### 3.1.1. 缓冲区溢出

缓冲区溢出可以分为堆溢出和栈溢出，堆跟栈是两片不同架构的内存区域。

- **堆**：程序运行时动态申请而分配的空间，大小不固定，可动态扩展。特点是需要主动申请：例如 C 语言中的 `malloc` 函数，申请的内存添加到堆上，堆被扩大。主动释放：例如 `free` 函数，假如不主动释放，就会造成内存泄漏。
- **栈**：栈在程序运行时自动产生，负责保存进程的运行上下文，当函数调用时，逻辑上会在栈中开辟一块新区域，我们称之为栈帧（stack frame），栈帧内保存调用的参数、返回值，包括上一级函数的返回地址。例如：

```
void test(){
    …
}
main(){
    test();
    …
}
```

例如 `main` 函数中调用 `test` 函数，假如有参数，逻辑上新建的栈帧中会保存参数，并且会保存 `main` 函数的地址，`test` 函数执行完成后用来返回到 `main` 函数，回到原先的栈帧，然后继续执行下面的代码。

逻辑上新开辟的栈帧已经消失，但物理保存的数据并不会消失。栈的溢出原理就是参数的值超出了缓冲区的大小，覆盖了返回地址，函数调用完成后，返回到了攻击者指定的地址去执行代码。

简单的来说，如果我们输入的数据长度超过了开发人员定义的缓冲区，那么这个数据就可以覆盖掉 `EIP`，`EIP` 是指令寄存器，它存放当前指令的下一条指令的地址。如果它被来自用户输入的垃圾数据覆盖了，程序通常会崩溃，因为它跳转到的地址并尝试指向，但执行的并不是有效的指令。

![](https://bbs.kanxue.com/upload/attach/201702/734388_alwdo5vdeuvys81.png)

*典型的内存布局*

![](https://bbs.kanxue.com/upload/attach/201702/734388_ou5p9rvozuf2l1s.png)

*输入超过用户缓冲区*

![](https://bbs.kanxue.com/upload/attach/201702/734388_3dql9wc7n2yqvu6.png)

*开发人员错误处理 - 输入覆盖缓冲区和 EIP，导致它跳转到无效的内存地址，程序崩溃*

#### 3.1.2. SEH

SEH 是 Windows 提供的异常处理机制之一。

Windows 对 R3 环境下应用异常处理流程为：

1. 交给调试器（进程必须被调试）
2. 执行 VEH
3. 执行 SEH
4. `TopLevelEH`（进程被调试时不会被执行）
5. 交给调试器（上面的异常处理都说处理不了，就再次交给调试器）
6. 调用异常端口通知 `csrss.exe`

![](http://bbs.pediy.com/upload/attach/201702/734388_xakum8stk9gl3k6.png)

*Windows 异常处理流程*

具体在程序中表示为，当程序发生异常崩溃时，系统会弹出对话框并调用调试器，其中 `EIP` 指向错误的返回地址，因此触发缓冲区异常操作的常用手段之一即为返回地址淹没。

用程序演示得到下图的结果：

![](https://bbs.kanxue.com/upload/attach/201702/734388_yb9ccwplb3n07vq.png)

用 `x32dbg` 接管调试程序，可以看到程序的 `EIP` 被用户输入的数据淹没。

#### 3.1.3. 相关术语

- **Fuzz**：模糊性检测，即构造一系列无规则的“坏”数据插入应用程序，判断程序是否出现异常，以发现潜在的 bug。
- **Shellcode**：`Shellcode` 是一填充数据，针对特定漏洞填充触发相应操作。
- **Payload**：`Payload` 是指除了触发异常的数据之外汇编代码的机器码二进制数据。
- **Exploit**：触发漏洞并完成攻击的整体流程。
- **POC**：Proof of Concept 概念证明，软件漏洞触发的原因和利用，以及个人 `shellcode` 编写。

## 4. POC

### 4.1. Fuzz

已知软件加载超过 45kb 的 `.m3u` 文件会触发异常。

构造 >45kb 相应的 `.m3u` 文件，并使用软件加载，观察触发异常的位置，并使用二分法逐渐缩减范围。

![](https://bbs.kanxue.com/upload/attach/201702/734388_21ldovu04ba1ftq.png)

*不断缩小范围，最终确定程序在 `F111` 处触发*

### 4.2. Shellcode 编写

相应的攻击代码使用了 VS2015 的内联汇编，并用了 Release 版进行编译。

`Shellcode` 编写流程：

- `GetPC` 硬编码字符串
- 加载通用模块 `kernel32.dll` / `kernelbase.dll`
- 在模块导出表获取 `LoadLibrary` 函数
- 在模块导出表获取 `GetProcAddress` 函数
- 加载 `user32.dll` 模块
- 通过 `GetProcAddress` 获取 `MessageBoxA` 函数地址
- 通过 `GetProcAddress` 获取 `ExitProcess` 函数地址
- 调用 `MessageBoxA`
- 调用 `ExitProcess`

#### 4.2.1. GetPC

`GetPC`，也即 Get Program Counter，取得程序计数器的值，在 x86 下就是 `GetEIP` 了，其作用是在进程的内存空间中得到当前的 `EIP` 的值，通常用于需要对代码自身进行操作的场合下，比如自解码和自修改代码（更一般的来说常见于病毒、溢出攻击代码等，也称之为代码重定位技术）。

代码示例：

```
//GetProcAddress\0
        /*47 65 74 50 72 6F 63 41 64 64 72 65 73 73 */
        //-0x14
        _asm _emit(0x47)     _asm _emit(0x65)     _asm _emit(0x74)     _asm _emit(0x50)
        _asm _emit(0x72)     _asm _emit(0x6F)     _asm _emit(0x63)     _asm _emit(0x41)
        _asm _emit(0x64)     _asm _emit(0x64)     _asm _emit(0x72)     _asm _emit(0x65)
        _asm _emit(0x73)     _asm _emit(0x73)    _asm _emit(0x00)
        __ShellCode:
                   //GetPC
                   call GETPC
                       GETPC :
                pop edx                      // edx==储存当前eip
```

#### 4.2.2. 获取 kernel32.dll / kernelbase.dll 的地址

由于我们需要动态获取 `LoadLibraryA()` 以及 `ExitProcess()` 这两个函数的地址，而这两个函数又是存在于 `kernel32.dll` 中的，因此这里需要先找到 `kernel32.dll` 的地址，然后通过对其进行解析，从而查找那两个函数。

所有的 Win32 程序都会自动加载 `ntdll.dll` 以及 `kernel32.dll` 这两个最基础的动态链接库。（在 64 位系统中，`kernel32.dll` 的地址由 `kernelbase.dll` 所代替）

因此如果想要在 64 位平台下定位 `kernelbase.dll` 中的 API 地址，可以使用如下方法（以下结合 WinDbg 演示，以 64 位系统为例）：

![](https://bbs.kanxue.com/upload/attach/201702/734388_pm30hklo90nvqkv.png)

![](https://bbs.kanxue.com/upload/attach/201702/734388_n6mxsuxiwuno2k8.png)

// `PEB_LDR_DATA+0x1C` 是 `InInitializationOrderModuleList` 的位置

![](https://bbs.kanxue.com/upload/attach/201702/734388_rhp13p5yphbzfeq.png)

// 进入链表第一个就是 `ntdll.dll`

![](https://bbs.kanxue.com/upload/attach/201702/734388_pbiq821b67za9xx.png)

// 第二个节点保存的是 `kernelbase.dll` 的基地址

![](https://bbs.kanxue.com/upload/attach/201702/734388_bungce9kbc68tql.png)

相应的代码示例：

```
mov     ebx, fs : [0x30]                  // [TEB+0x30]是PEB的位置 
mov     ecx, [ebx + 0xC]                  // [PEB+0xC]是PEB_LDR_DATA的位置 
mov     ecx, [ecx + 0x1C]                 // [PEB_LDR_DATA+0x1C]是InInitializationOrderModuleList的位置 
mov     ecx, [ecx]                               // 进入链表第一个就是ntdll.dll 
mov     ebp, [ecx + 0x8]                  // ebp保存的是kernelbase.dll的基地址
```

#### 4.2.3. 解析 kernel32.dll / kernelbase.dll 导出表

找到了 `kernel32.dll`，由于它也是属于 PE 文件，那么我们可以根据 PE 文件的结构特征，对其导出表进行解析，不断遍历搜索，从而找到我们所需要的 API 函数。其步骤如下：

1. 从 `kernel32.dll` 加载基址算起，偏移 0x3c 的地方就是其 PE 头。
2. PE 头偏移 0x78 的地方存放着指向函数导出表的指针。
3. 至此，可以按如下方式在函数导出表中算出所需函数的入口地址：
   - 导出表偏移 0x1c 处的指针指向存储导出函数偏移地址（RVA）的列表。
   - 导出表偏移 0x20 处的指针指向存储导出函数函数名的列表。
   - 函数的 RVA 地址和名字按照顺序存放在上述两个列表中，我们可以在名称列表中定位到所需的函数是第几个，然后在地址列表中找到对应的 RVA。
   - 获得 RVA 后，再加上前边已经得到的动态链接库的加载地址，就获得了所需 API 此刻在内存中的虚拟地址，这个地址就是我们最终在 ShellCode 中调用时需要的地址。

按照这个方法，就可以获得 `kernel32.dll` 中的任意函数。

需要注意的是，`kernelbase.dll` 中没有 `LoadLibrary` 这个函数，取而代之的是 `LoadLibraryExA` 这个函数。`Kernelbase.dll` 与 `kernel32.dll` 之间存在一些映射关系，函数地址可能会不一样。同时 `Kernelbase.dll` 对比 `kernel32.dll` 缺少一部分函数，因此对于 64 位系统，一般在加载 `kernelbase.dll` 之后再度使用 `LoadLibraryExA` 加载 `kernel32.dll`，以增强代码的健壮性。（下面代码中未做这方面的处理）

相应代码示例：
