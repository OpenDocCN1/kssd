
# 反逆向工程揭密（试炼篇）-全文完

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-707.htm`](https://www.pediy.com/kssd/pediy07/pediy7-707.htm)

## Abstract

Rather than doing another complete analysis of the binary, i will rather present the techniques i have used in the challenge, and how i have implemented them. The Scan of the Month 33 was released by the Honeynet Project in November 2004. I invite everyone to read the excellent submissions we received this month once they have read my paper. I am presenting the binary from the protection author point of view, while they presented it from the analyst point of view. You will learn the methods and techniques used to Protect / Unprotect a binary with this month's challenge. A lot of weaknesses were left on purpose in this binary and they will be presented here.

**Keywords**: `Software Protection`; `Reverse Code Engineering`; `Linux`; `Anti-Debugging`; `Anti-Anti-Debugging`

**摘要**：

有人偏爱详细的分析过程，而我却喜欢从技术和实现方法的层面上去探讨。2004 年的 11 月，The Honeynet Project 发布了 The scan of the month 33。我推荐大家好好的研读一下。他们研究的就是前者。我除了对技术和实现方法进行研究外，大量的漏洞在我的文中也有披露。

**关键词**：软件保护；逆向工程；Linux; 反调试；调试。

## 1. Introduction

This month's challenge is to analyze an unknown binary, in an effort to reinforce the value of reverse engineering, and improve (by learning from the security community) the methods, tools and procedures used to do it. This challenge is similar to SotM 32. However, this binary has mechanisms implemented to make the binary much harder to analyze, to protect against reverse engineering.

**1. 简介**：

最新的 month 33 挑战了一段未知的二进制代码，这对于我们加深逆向印象、改进方法、了解工具的使用及其整个过程具有积极的作用。这与先前发布的 The scan of the month 32 是相同的。唯一不同的是，此二进制代码更加难于分析。

**Skill Level**: `Advanced`/`Expert`

All we are going to tell you about the binary is that it was 'found' on a WinXP system and has now be sent to you for analysis. You will have to analyse it in-depth and get as much information as possible about its inner working, and what is the goal of the binary. The main goal of this challenge is to teach people how to analyse heavily armored binaries. Such techniques could be used in the future, and its time to get used to them.

**技术含量**：中/高级

你将自己独立分析这些基于 `WinXP` 环境下的二进制代码。想完成挑战，你必须深刻的分析问题，搜集更多的信息去了解内部原理以及体会代码中包含的目的。最终使你能够独立的分析一层又一层的代码。

## 2. Identify and explain any techniques in the binary that protect it from being analyzed or reverse engineered

**2．鉴别并解释阻碍分析和逆向的技术。**

Many techniques have been used in order to slow down analysis and break reverse engineers tools:

*   **PE Header Modifications**

    Many fields of the PE header were modified in order to disturb analysing tools, and thus, the Reverse Engineer. I will quickly cover the most important changes:

    `->Optional Header`

    ```text
    Magic: 0x010B (HDR32_MAGIC)
    MajorLinkerVersion: 0x02
    MinorLinkerVersion: 0x19 -> 2.25
    SizeOfCode: 0x00000200
    SizeOfInitializedData: 0x00045400
    SizeOfUninitializedData: 0x00000000
    AddressOfEntryPoint: 0x00002000
    BaseOfCode: 0x00001000
    BaseOfData: 0x00002000
    ImageBase: 0x00DE0000 <--- "Non Standard" ImageBase
    SectionAlignment: 0x00001000
    FileAlignment: 0x00001000
    MajorOperatingSystemVersion: 0x0001
    MinorOperatingSystemVersion: 0x0000 -> 1.00
    MajorImageVersion: 0x0000
    MinorImageVersion: 0x0000 -> 0.00
    MajorSubsystemVersion: 0x0004
    MinorSubsystemVersion: 0x0000 -> 4.00
    Win32VersionValue: 0x00000000
    SizeOfImage: 0x00049000
    SizeOfHeaders: 0x00001000
    CheckSum: 0x00000000
    Subsystem: 0x0003 (WINDOWS_CUI)
    DllCharacteristics: 0x0000
    SizeOfStackReserve: 0x00100000
    SizeOfStackCommit: 0x00002000
    SizeOfHeapReserve: 0x00100000
    SizeOfHeapCommit: 0x00001000
    LoaderFlags: 0xABDBFFDE <--- Bogus Value
    NumberOfRvaAndSizes: 0xDFFFDDDE <--- Bogus Value
    ```

    The "standard" `ImageBase` usually is `400000` for `Win32` applications and Reverse Engineers are used to analyse programs with such an `ImageBase`. While it isn't a protection by itself, this simple modification will confuse some Reverse Engineers, because they aren't used to such memory addresses.

    **包括以下这些：**

    （1）修改 PE 文件头：

    目的：干扰工具的分析。附上重要的修改：

    `->Optional Header`

    ```text
    Magic: 0x010B (HDR32_MAGIC)
    MajorLinkerVersion: 0x02
    MinorLinkerVersion: 0x19 -> 2.25
    SizeOfCode: 0x00000200
    SizeOfInitializedData: 0x00045400
    SizeOfUninitializedData: 0x00000000
    AddressOfEntryPoint: 0x00002000
    BaseOfCode: 0x00001000
    BaseOfData: 0x00002000
    ImageBase: 0x00DE0000 <--- 非一般性基址
    SectionAlignment: 0x00001000
    FileAlignment: 0x00001000
    MajorOperatingSystemVersion: 0x0001
    MinorOperatingSystemVersion: 0x0000 -> 1.00
    MajorImageVersion: 0x0000
    MinorImageVersion: 0x0000 -> 0.00
    MajorSubsystemVersion: 0x0004
    MinorSubsystemVersion: 0x0000 -> 4.00
    Win32VersionValue: 0x00000000
    SizeOfImage: 0x00049000
    SizeOfHeaders: 0x00001000
    CheckSum: 0x00000000
    Subsystem: 0x0003 (WINDOWS_CUI)
    DllCharacteristics: 0x0000
    SizeOfStackReserve: 0x00100000
    SizeOfStackCommit: 0x00002000
    SizeOfHeapReserve: 0x00100000
    SizeOfHeapCommit: 0x00001000
    LoaderFlags: 0xABDBFFDE <--- 伪值
    NumberOfRvaAndSizes: 0xDFFFDDDE <--- 伪值
    ```

    `400000` 作为 `Win32` 应用程序的一般基址，常常被逆向者在分析程序时使用。当程序不能保护自身时，这种简单的修改就可以对逆向工程造成影响，因为程序并不使用这样的内存地址。

*   **"Anti" `OllyDbg`**

    `LoaderFlags` and `NumberOfRvaAndSizes` were modified.. I have Reverse Engineered `OllyDBG` and `Soft ICE` to find a few tricks that could slow down the analysis of a binary. With those two modifications, `Olly` will pretend that the binary isn't a good image and will eventually run the application without breaking at its entry point. This could be a bad thing if you wanted to debug a mal ware on your computer, because you would get infected.

    **反 `OllyDbg`:**

    通过修改 `LoaderFlags` 和 `NumberOfRvaAndSizes` 实现。逆向 `OllyDbg` 和 `SoftICE` 后，我们会发现这种方法可以阻碍二进制代码的分析，因为程序会认为此二进制不是正确的基址，从而不在入口点处执行。导致直接的后果是：如果你调试一个病毒程序，你的计算机将会被感染。

*   **Anti `Soft ICE` : Blue Screen of Death and no Chocolate**

    The `NumberOfRvaAndSizes` field has been modified in order to reboot any computer running a recent version of `Soft ICE`. While Disassembling the `PE Loader` of `Soft ICE`, i found a very critical vulnerability in `Soft ICE` that allows one binary to crash any computer running `Soft ICE` without any code execution. This vulnerability (bug) has been reported to `Compuware` and should be fixed in the next version. Apparently it didn't happen on some of the authors of the submissions for some reasons. Oh well.

    Here is the disassembly of `Soft ICE PE loader` to find out why it reboots your computer:

    ```asm
    .text:000A79FE
    .text:000A79FE loc_A79FE: ; CODE XREF: sub_A79B9+31j
    .text:000A79FE ; sub_A79B9+3Cj
    .text:000A79FE ; DATA XREF: .text:00012F9Bo
    .text:000A79FE sti
    .text:000A79FF mov esi, ecx
    .text:000A7A01 mov ax, [esi]
    .text:000A7A04 cmp ax, 'ZM'
    .text:000A7A08 jnz not_PE_file
    .text:000A7A08
    .text:000A7A0E mov edi, [esi+_IMAGE_DOS_HEADER.e_lfanew]
    .text:000A7A11 add edi, esi
    .text:000A7A13 mov ax, [edi]
    .text:000A7A16 cmp ax, 'EP'
    .text:000A7A1A jnz not_PE_file
    .text:000A7A1A
    .text:000A7A20 movzx ecx, [edi+IMAGE_NT_HEADERS.FileHeader.NumberOfSections]
    .text:000A7A24 or ecx, ecx
    .text:000A7A26 jz not_PE_file
    .text:000A7A26
    .text:000A7A2C mov eax, [edi+IMAGE_NT_HEADERS.OptionalHeader.NumberOfRvaAndSizes]
    .text:000A7A2F lea edi, [edi+eax*8+IMAGE_NT_HEADERS.OptionalHeader.DataDirectory]
    .text:000A7A33 mov eax, ecx
    .text:000A7A35 imul eax, 28h
    .text:000A7A38 mov al, [eax+edi] ; CRITICAL BUG! One can force EAX+EDI to be equal to zero. Reading at [0] in ring 0 isn't nice eh ;-
    .text:000A7A3B
    .text:000A7A3B loc_A7A3B: ; DATA XREF: .text:00012FA5o
    .text:000A7A3B cli
    .text:000A7A3C call sub_15C08
    .text:000A7A3C
    .text:000A7A41 mov byte_FA259, 0
    .text:000A7A48 push eax ; Save EAX
    ```



# 反 SoftIce

## 蓝屏死机花屏

修改 `NumberOfRvaAndSizes` 会令含 SoftIce 的系统重起。通过反汇编 SoftIce 装载器，我发现：无需代码执行，一个二进位的变动就可使一台含 SoftIce 的电脑立刻崩溃。不过此漏洞已上报 COMPUWARE 并将在下一版本中改进。但是，由于某种原因，一些系统却运行正常。让我们来看一下反汇编后的 SoftICE 装载器里到底有什么：

```assembly
.text:000A79FE
.text:000A79FE loc_A79FE: ; CODE XREF: sub_A79B9+31j
.text:000A79FE ; sub_A79B9+3Cj
.text:000A79FE ; DATA XREF: .text:00012F9Bo
.text:000A79FE sti
.text:000A79FF mov esi, ecx
.text:000A7A01 mov ax, [esi]
.text:000A7A04 cmp ax, 'ZM'
.text:000A7A08 jnz not_PE_file
.text:000A7A08

.text:000A7A0E mov edi, [esi+_IMAGE_DOS_HEADER.e_lfanew]
.text:000A7A11 add edi, esi
.text:000A7A13 mov ax, [edi]
.text:000A7A16 cmp ax, 'EP'
.text:000A7A1A jnz not_PE_file
.text:000A7A1A

.text:000A7A20 movzx ecx, [edi+IMAGE_NT_HEADERS.FileHeader.NumberOfSections]
.text:000A7A24 or ecx, ecx
.text:000A7A26 jz not_PE_file
.text:000A7A26

.text:000A7A2C mov eax, [edi+IMAGE_NT_HEADERS.OptionalHeader.NumberOfRvaAndSizes]
.text:000A7A2F lea edi, [edi+eax*8+IMAGE_NT_HEADERS.OptionalHeader.DataDirectory]
.text:000A7A33 mov eax, ecx
.text:000A7A35 imul eax, 28h
.text:000A7A38 mov al, [eax+edi] ; EAX+EDI=0 则崩溃。权限 0 中读取[0]地址可不妙。
.text:000A7A3B

.text:000A7A3B loc_A7A3B: ; DATA XREF: .text:00012FA5o
.text:000A7A3B cli
.text:000A7A3C call sub_15C08
.text:000A7A3C

.text:000A7A41 mov byte_FA259, 0
.text:000A7A48 push eax ; Save EAX
.text:000A7A49 mov eax, dword_16B56F ; 用双字修改 EAX 中值
.text:000A7A4E mov dr7, eax ; 寄存器 7 从 EAX 中取值
.text:000A7A51 pop eax ; 恢复 EAX
.text:000A7A52 mov dword_FC6CC, esp
.text:000A7A58 mov esp, offset unk_FBABC
.text:000A7A5D and esp, 0FFFFFFFCh
.text:000A7A60 xor al, al ; AL 取 0?为什么要 mov al, [eax+edi]?
.text:000A7A60 ; 不明白.原始代码?
.text:000A7A62 call sub_4D2EB
.text:000A7A62

.text:000A7A67 call sub_36AC1
.text:000A7A67

.text:000A7A6C xor edx, edx
.text:000A7A6E

.text:000A7A6E loc_A7A6E: ; CODE XREF: sub_A79B9+124j
.text:000A7A6E call sub_74916
.text:000A7A6E
```

As you can see from the code above, we can force Soft ICE to read at memory location `[0]` or something similar using a special value inside the PE header. For this binary I didn't bother calculating the exact value to read at address `[0]`, that's may explain why it didn't crash for some people. I won't explain how to calculate this special value because it is trivial and I don't want Dark lords to use that trick without a little brainstorming.

To fix this problems, one needs to patch the value in the PE Header. The standard value for `NumberOfRvaAndSizes` is `0x10`. Just patch this value in the PE Header and the Soft ICE wrecking will be gone. The OllyDBG problem as well, because it is based on BOTH fields modifications. You can also nullify the other field if you want.

如你所见，我们可以令 SoftIce 读取 `[0]` 地址或其它特殊值。但我并不知道这时确切的数值，这可能与系统是否崩溃有关。我不想讨论怎样计算这一特殊值，因为它与主题无关，另外也给大家留下讨论的空间。

你可以通过值的修改来修正这一问题。`NumberOfRvaAndSizes` 的一般值为 `0x10`。修改它。另外因为 Olly 和 SoftIce 都基于这种技术，因此你都可以通过改变 `NumberOfRvaAndSizes` 的值或归 0 来搞定。

注：未完，有空会继续翻译，不知翻译的如何，欢迎大家提出宝贵意见，不胜感激。

## 节的修改：工具杀手

> **Section Modification: Or how to kill many tools.**
>
> `->Section Header Table`
>
> **1. item:**
>
> *   `Name`: `CODE`
> *   `VirtualSize`: `0x00001000`
> *   `VirtualAddress`: `0x00001000`
> *   `SizeOfRawData`: `0x00001000`
> *   `PointerToRawData`: `0x00001000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xE0000020` (`CODE`, `EXECUTE`, `READ`, `WRITE`)
>
> **2. item:**
>
> *   `Name`: `DATA`
> *   `VirtualSize`: `0x00045000`
> *   `VirtualAddress`: `0x00002000`
> *   `SizeOfRawData`: `0x00045000`
> *   `PointerToRawData`: `0x00002000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xC0000040` (`INITIALIZED_DATA`, `READ`, `WRITE`)
>
> **3. item:**
>
> *   `Name`: `NicolasB`
> *   `VirtualSize`: `0x00001000`
> *   `VirtualAddress`: `0x000047000`
> *   `SizeOfRawData`: `0xEFEFADFF` `<--- BIG Size of section on the disk.`
> *   `PointerToRawData`: `0x000047000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xC0000040` (`INITIALIZED_DATA`, `READ`, `WRITE`)
>
> **4. item:**
>
> *   `Name`: `.idata`
> *   `VirtualSize`: `0x00001000`
> *   `VirtualAddress`: `0x000048000`
> *   `SizeOfRawData`: `0x00001000`
> *   `PointerToRawData`: `0x000047000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xC0000040` (`INITIALIZED_DATA`, `READ`, `WRITE`)

> **节的修改：工具杀手。**
>
> `->Section Header Table`（节头表）
>
> **条目 1：**
>
> *   `Name`: `CODE`
> *   `VirtualSize`: `0x00001000`
> *   `VirtualAddress`: `0x00001000`
> *   `SizeOfRawData`: `0x00001000`
> *   `PointerToRawData`: `0x00001000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xE0000020` (`CODE`, `EXECUTE`, `READ`, `WRITE`)
>
> **条目 2：**
>
> *   `Name`: `DATA`
> *   `VirtualSize`: `0x00045000`
> *   `VirtualAddress`: `0x00002000`
> *   `SizeOfRawData`: `0x00045000`
> *   `PointerToRawData`: `0x00002000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xC0000040` (`INITIALIZED_DATA`, `READ`, `WRITE`)
>
> **条目 3：**
>
> *   `Name`: `NicolasB`
> *   `VirtualSize`: `0x00001000`
> *   `VirtualAddress`: `0x000047000`
> *   `SizeOfRawData`: `0xEFEFADFF` `<--- 大容量的节`
> *   `PointerToRawData`: `0x000047000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xC0000040` (`INITIALIZED_DATA`, `READ`, `WRITE`)
>
> **条目 4：**
>
> *   `Name`: `.idata`
> *   `VirtualSize`: `0x00001000`
> *   `VirtualAddress`: `0x000048000`
> *   `SizeOfRawData`: `0x00001000`
> *   `PointerToRawData`: `0x000047000`
> *   `PointerToRelocations`: `0x00000000`
> *   `PointerToLinenumbers`: `0x00000000`
> *   `NumberOfRelocations`: `0x0000`
> *   `NumberOfLinenumbers`: `0x0000`
> *   `Characteristics`: `0xC0000040` (`INITIALIZED_DATA`, `READ`, `WRITE`)

> From those informations, we can conclude a few things. First, the binary doesn't seem to be compressed, because the Virtual Address and Size matche the Raw Offset and Size at one exception, the `NicolasB` section. This section has an extremly big size of raw data, which will crash a few tools and make a few others very very slow.
>
> 通过以上信息，我们可以推断出以下一些结论。1。二进制代码未被压缩，因为除了 `NicolasB`，其它节的 `Virtual Address = the Raw Offset`，`Virtual Size = Raw Size`。`NicolasB` 节包含了巨大的 raw 数值，这将使进行逆向的工具崩溃或受到阻碍。



## 内存分配问题与修复

> IDA will try to allocate a LOT of memory because it thinks that the section is THAT big, turning your computer into a very slow turtle ;-)。Eventually, it will load the file, or run out of memory, depending of the computer you are using to do the analysis.

当节包含巨大数值时，`IDA` 会分配大量的内存空间，这将使你的计算机运行缓慢。事实上，是装载文件或是内存溢出，都取决于正在用来分析的电脑性能。

> This modification will also create havoc with many tools such as `Objdump`, `PE editor`, some memory `dumpers` etc. It is very easy to fix this problem,

另外，这种修改还会对许多工具如 `Objdump`、`Peeditor`、一些内存 `Dumper` 等产生严重破坏。不过修正这个问题很简单，只要修改一下 `Raw Size` 即可。

> you need to correct the Raw Size. If you look at the section following this special one, you will find that it starts at the very same Raw Offset.

发现某个节与其它不同，但是它们仍然是具有相同的 `Raw Offset`。

> This means that the other section is actually null on the disk. You can therefore, safely replace the big value by zero.

这就意味着，如果其它节是空的，处于安全考虑，你可以把特殊的那个节也取空。

## 保护漏洞：PE 头

> Protection Weakness:

### 英文说明

> While writing this binary, i knew people were going to patch the PE header but i didn't do any integrity checks on purpose. Originally i wanted to use the value in the PE Header as keys to decrypt a few layers of the protection, and the result would have been an unworking binary if this one had been changed.

> I have also changed a few other things in the PE header, but nothing of real interest here. (who said Cosmetic?)

### 中文说明

保护漏洞：

当我贴出这些代码时，许多人都会去对 `PE` 头进行修正，可我不是这样做的。最初，我想利用 `PE Header` 中的数值作为打开保护层的钥匙，但是一旦改变数值，它就变成了一段无效的二进制代码。另外我还试着修改其它地方，可都一无所获。

## 垃圾代码 (Junk Code)

> • Junk Code

> All along the binary, i have added junk code between real instructions, in order to make the analysis a little harder. The junk code are long blocks of code that does nothing but fancy operations to disturb the analyst, especially when he choose to do a static analysis of the binary. Each block of Junk Code is different and have been generated by a personal tool. A Thrash generator which creates macros to be inserted in the code source around real instructions.

> Here is how it looks inside a disassembler:

> The junk code starts with a `pushad` (save all registers states onto the stack) and finish with a `popad` (restore register states). Here is the end of a block of junk:

### 中文说明

垃圾代码

在真正的代码中，我添加了一些垃圾代码，这是为了增加分析的难度。它们冗长无意义，但是可以阻止分析的进行，特别是静态分析时。而每一个又各不相同，由独立的工具产生。一个好的程序会把这种代码大量的运用于真正的代码之中。

以下是我们在反汇编器中所看到的：

垃圾代码以 `pushad` 开头（在堆栈中保存所有寄存器状态）并以 `popad`（恢复寄存器状态）。这是它们的尾部：

## 保护漏洞：垃圾代码

> Protection Weakness:

> The thrash generator isn't perfect (at least with the options i have used here ;)) and it is easy to find the start and the end of a block of junk code. The junk code is bounded by `pushad`/`popad`. When i wrote this binary i was aware of this problem, but this is a perfect real life example of protection weakness. It allows Reverse Engineers to practice `IDA`/`Ollydbg` scripting. Very interesting scripts were found in the submissions.

> I invite you to have a look at them if you didn't know how to write one. When i wrote the binary, i already had a better version of my Thrash generator that doesn't use any `pushad`/`popad` around the blocks of useless code, but we will keep it for another challenge, if any.

### 中文说明

保护漏洞：

这个程序不是完美的，你很容易找到垃圾代码的头和尾。即 `pushad`/`popad`。当我写这段代码时想过这个问题，但这确实是最典型的保护漏洞。

逆向者用它来练习 `IDA`/`Ollydbg` 脚本会非常有趣。如果你不会，建议你去了解一下。当我完成这个程序后，我又写了更好的非典型漏洞的版本，这是后话了。

## SEH - 结构化异常处理

> • SEH - Structured Exception Handling

> Windows SEH were used extensively in this binary. It allows one to access the context structure of the current application, and therefore, access privileged registers such as Debug Registers. Those registers are used by Hardware Breakpoints (BPM). If you can access them, you can also erase the hardware breakpoints.

> SEH-结构异常链

> SEH 被广泛地运用在二进制代码中。它允许你访问当前应用程序的 `context structure` 和优先级寄存器（调试器）。这些寄存器被硬件断点（`BPM`）所使用。因为可存取，因此这些硬件断点也可被擦除。

## 通过 SEH 侦测时间

> • Timing Detection Through SEH

> Here is a little detection i invented to detect debuggers. If we merge SEH (And access to context structure) with the known Timing Detection Technique, we can detect a lot of Ring 3 debuggers and Tracers. The idea is to read the Time Stamp Counter using `RDTSC` (number of cycles executed by the CPU basically) and then generating an Exception.

> 通过 SEH 侦测时间

> 这是一个侦测调试器的程序。当我在 `SEH`（存取 `context structure`）中加入时间侦测技术后，可以发现：许多 `Ring 3` 级别的调试器和追踪器都可以被侦测到。原理是通过 `RDTSC`（`CPU` 执行数字循环）读取了 `Time Stamp` 指针最终产生异常。

> In the exception handler, we can access the `EAX` register (previously modified by `RDTSC`) in the Context Structure, which contains the TSC. In the Exception Handler, we use `RDTSC` one more time, to get the current TSC value. Now, we can compare both TSC to see whether the program has been debugged/traced or not. If such an action has occured, the difference of cycles will be huges, thus triggering the Payload.

> In this binary, i just modified `EIP` through the context structure. The application resumes at a different location skipping mandatory instructions. The application crashes eventually. It seems that on some version on `Windows`, it doesn't work as expected because of the utilisation of the `CPUID` instruction, that will modify the `ECX` register.

在异常链中，我们可以访问 `Context Structure` 中的 `EAX` 寄存器（先前被 `RDTSC` 修改）。同时，我们更多的使用 `RDTSC` 得到当前 `TSC` 的数值。现在，我们比较这两个 `TSC`，看是否程序被调试/跟踪。如果调试/跟踪，两者值会有巨大差距，导致最终被发现。虽然我修改了 `context structure` 中 `EIP` 的数值，使得程序跳过强制指令定位到新的位置。可是最终程序仍然崩溃。可以看出 `Windows` 中的一些版本，因为 `CPUID` 指令修改了 `ECX` 寄存器而并不象我们想象中的那样运行。

> The detection became less stealth because of this "bug", but it would still have been a matter of time until someone discovered it anyway.

> Many people wondered why i used `CPUID` in the program before `RDTSC`. The reason is that on recent `CPU` such as `P4`, there is a feature called: Out of Order Execution. The `CPUID` is a synchronization instruction which tells to the CPU not to use Out Of Order execution, avoiding False Positives in the debugger detection. If you don't tell to the CPU not to use OOO execution, you don't know in which order the CPU is going to execute your code. It can be different from your source code. Sometimes, it will create a false positive and your program will crash for no reason.

> Here is the code of this detection:

> `E0000h` is the maximum cycles difference accepted by this detection. If the number is bigger, then a debugger is most likely running and debugging our application.

尽管此侦测的原理有了一些进展，但仍然有很多需要去发觉。许多人奇怪为什么 `CPUID` 使用在 `RDTSC` 之前。因为现在的 `CPU`（如 `P4`）具有被称之为“Out of Order Execution”的功能。`CPUID` 是一个同步指令，为了避免调试器侦测中的错误行为，它会禁止 `CPU` 使用 Out of Order 功能。即使你打开了这项功能，你也不知道在什么地方执行了你的代码，并可能与你的原代码有出入、执行错误的命令甚至使系统莫名其妙的崩溃。

下面是侦测的代码：

```
E0000h
```

`E0000H` 是侦测差距的上限。如果超过这个上限，调试器可能会正常运行。

## 保护漏洞：时间侦测

> Protection Weakness:

> I have used a fixed value for the number of cycles: `E0000h`. I could have (Actually i can do it with my layer generator) used a random value rather than a constant and therefore, making the scan for this constant useless. I could also have used different instructions for each SEH to make the creation of a generic pattern difficult. The biggest weakness of this detection is the constant and the usage of the same instructions for every checks. It is also possible to write a Kernel Module Driver to catch every execution of `RDTSC` (See Intel documentation for further informations) and return very similar values, thus bypassing the detection completely.

### 中文说明

保护漏洞：

我使用了上限 `E0000H`。然而我可以使用任意数值代替这个常数。另外我还可以对每个 `SEH` 使用不同的指令使其产生不同的效果。最大的漏洞就是常数的设置和每个校验使用相同的指令。因此，我们可以写出一段 `Kernel Module Driver`，去获取 `RDTSC` 的执行过程和返回的数值，以此击败这种保护方式。

## BPX 侦测

> • BPX Detection:

> As we are going to use API functions, We have to protect them from beeing `BPX`'ed by an attacker. Rather than Using `GetProcAddress` to get



## BPX 侦测

> 为了保护二进制文件免受反汇编攻击，我编写了一个加密层生成器，可以生成任意数量的层。在此二进制文件中，我使用了 175 层。层生成器包含许多选项。以下是从配置文件中提取的选项（`0` 表示禁用）：

```text
SEH=1 
RANDOM_LAYER_SIZE=0 
RANDOM_REGISTERS=1 
RANDOM_ENCRYPTION=0 
ENCRYPTED_RETURN_ADDRESS=1 
TIMING_DETECTION=1 
RANDOM_CONSTANT=0 
JUNKS=0 
PUSHAD_POPAD=1 
RANDOM_ORDER=0 
USE_DIFFERENT_LOOP_CODE=0 
RANDOM_FIRST_BLOCK=0 
NUMBER=175 
```

下面逐一解释这些选项：

### `SEH`

配置器指示层生成器在层内部使用（或不使用）`SEH`。

### `RANDOM_LAYER_SIZE`

配置器指示层生成器为每一层使用不同的尺寸。此选项未启用，旨在简化分析。

### `RANDOM_REGISTERS`

如果启用此选项，所有层将使用不同的寄存器。这体现了一种“多态性”。此选项已启用。

### `RANDOM_ENCRYPTION`

当启用此选项时，每一层将具有不同的加密算法。我未启用此选项，因此所有层都具有静态加密代码（默认层）。

### `ENCRYPTED_RETURN_ADDRESS`

此选项会对层内部的返回地址进行加密。这避免了通过简单补丁跳过 `SEH` 的风险。此选项已启用。

## 保护漏洞

首先，导入表未受保护，因此任何人都可以从二进制文件中读取导入的函数。从导入表中，我们可以发现 `printf`、`GetCommandLineA` 和 `ExitProcess` 的使用。这是一个漏洞。逆向工程师可以在这些函数上设置断点，或者至少猜测它们将在某时某刻被使用。以我们的二进制文件为例，可以猜测应用程序正在等待一个特殊的命令行。解决方案是手动加载导入表。

为此，我们可以使用自制的一个 `GetProcAddress` 函数来浏览我们想要从中导入函数的 DLL 的导出表，然后从中获取 API 函数的地址。当一个二进制文件启动时，`Kernel32` 的地址始终在栈上，因此我们可以使用这个值来获取 DLL 的镜像基址（`ImageBase`）（或者使用 `PEB`、`SEH` 链等）。我们将拥有所需的一切来得到 `LoadLibrary` 的地址，这允许我们加载任何 DLL，从而获取任何 API 函数的地址。使用这种方法，我们完全不需要导入表。

好吧，实际上，这是不正确的。为了与所有版本的 Windows 保持兼容，有一件强制要做的事情。我们需要创建一个非常小的导入表，至少包含一个来自 `Kernel32` 的导入，否则二进制文件将无法在 `Windows 2000` 上运行。`Windows 2000` 的 PE 加载器与 `Windows XP` 的不同。`XP` 不在乎是否存在导入表。

这个小导入表仅仅是为了解决兼容性问题，真正的导入表是加密的，并且将在运行时由保护措施解密。然后，只需要模拟操作系统加载导入的过程即可。我们需要手动将 API 地址放入导入地址表（`IAT`，即解密后的导入表）中。逆向工程师在到达解密和加载导入的代码部分之前，对二进制文件使用的 API 函数毫无头绪。

`BPX` 保护有几个弱点。我只检查 API 入口点的四个字节，如果 API 包含许多指令，这很容易绕过。可以在四个字节边界后的第一条指令上设置断点。更好的检查方法会使用长度反汇编引擎（`LDE`），它告诉我们指令的大小。有了它，我们可以安全地扫描大量指令而不触发任何误报。

一条真正的指令可以包含字节 `0xCC` 但仍然不是一个断点。例如：`Mov eax, 0x4010CC`。检测会在此指令上触发误报，因为它内部包含 `0xCC`。另一方面，`LDE` 会告诉我们这条指令的大小（5 字节）。`INT 3`（断点）要么是 1 个字节（`0xCC`），要么是 2 个字节（`0xCD 0x03`）。因此，我们会跳过当前指令并检查下一个。

此外，`BPX` 检查只在二进制文件的给定位置对每个 API 执行一次。一旦我们步过这些检查，我们就可以在任何 API 函数上设置断点而不触发任何错误。这个弱点是故意未修复的，因为这是保护系统中常见的错误。下一节将描述另一种 `BPX` 检测。



## 配置选项说明

### `RANDOM ENCRYPTION`
当此选项打开时，每个 `Layer` 都具有独立的加密运算法则。我禁止了它。因此，所有的 `Layers` 都有同一个加密代码。

### `ENCRYPTED RETURN ADDRESS`
此选项将对 `Layer` 中返回的地址进行加密，避免了仅仅简单的修改就绕过 `SEH` 情况的发生。此选项为打开。

### `TIMING_DETECTION`
Tells whether the layers must use Timing Detection or not. I enabled this option.

**是否 `Layers` 使用时间侦测技术。我打开了这个选项。**

### `RANDOM_CONSTANT`
The Random constant is to tell whether we want to use a static value for the timing detection or not. This option wasn't enable.

All layers were using the default value: `E0000h`. Enabling this option will also modify the code that checks for the Difference between both `TSC`.

**此选项表明是否在时间侦测中使用静态数值。此选项禁止，所有 `Layer` 使用默认值 `E0000H`。打开它就是修改了两个 `TSC` 的差距值的上限。**

### `JUNKS`
Enable of Disable Junks in the Layers. I disabled this option because the layers are WAY biggers when it is enabled. The resulting binary is too huge and slow if you use a big number of layers.

**禁止 `Layer` 中的垃圾代码。因为选项打开后 `Layer` 将变的冗长，代码也随之变的巨大，运行将变的缓慢。**

### `PUSHAD_POPAD`
This option tells the Layer Generator to use (or not) `Pushad/Popad` around the Junk Code. The layer generator directly use the Thrash Generator (external tool) i have programmed. I was using `pushad popad` in the junk code, that's why it is enabled. This option does nothing if the Junks option is disabled.

**此选项决定加密器是否在垃圾代码中使用 `PUSHAD/POPAD` 格式。我的加密器是直接使用 `Thrash Generator`（外国的工具）。在代码中我使用了上面的格式，因此选项是打开的。但如果 `JUNKS` 选项被禁止，此选项即被屏蔽。**

### `RANDOM_ORDER`
Each layer use a table to access part of its code. If this option is enabled, Each layer has a random order of execution. I didn't enable this one on purpose.

**每个 `Layer` 使用表访问自身的代码。当此选项打开时，每个 `Layer` 执行的顺序是任意的。我禁止了这个选项。**

### `USE_DIFFERENT_LOOP_CODE`
Each layer loops a given number of time. With this option, one can use different code to test the end of the loop. It makes it harder for the reverse engineer to find removal pattern. This option wasn't enabled. A default checking code was used.

**每个 `Layer` 循环特定的时间。通过此选项，我们可以使用不同的代码测试循环的结束。这使得逆向者更加难以找到切入点。此选项禁止。使用默认代码校验。**

### `RANDOM_FIRST_BLOCK`
This option allows one to use random value inside the first elements of the layers tables. You will see in some submissions that the static value were used to bypass the layers. I didn't enable this option to see whether someone was going to use it or not.

**此选项可以把任意值放入 `Layer` 表的第一个元素中。一些文章利用静态数值绕过 `Layer`。我禁止了此选项就是看看是否有人可以做到。**

### `NUMBER`
This is the number of the layer, the generator must use. I used 175 layers in this challenge. I can generate 65000 layers in a few seconds because the generator engine is programmed in Assembly Language.

**这是加密器必须设置的 `Layer` 数值。我设置了 175 个。甚至我可以在几秒之内就产生 65000 个 `Layer`，因为我的加密器核心是由汇编语言编制的。**

## 加密层结构展示

### Layer Selector

```assembly
xor esi,esi ; ESI = 0 

mad_loop175_1: ; Loop label 

inc esi ; ESI++ 

mov edi,dword ptr [ebp+(esi*4)+EIPtable175_1] ; Grab block address 

mov ebx,dword ptr [ebp+(esi*4)+RETable175_1] ; Grab "Encrypted" Return address 

Add ebx, [ebp+_startloader] ; Add Base. 

push ebx ; Save Return Address from the stack 

Call tricky_call175_1 ; Fake call 

db 0EBh,01,0E8h ; Some junk crap 

fake_ret175_1: ; fake return address label. 

Add edi, [ebp+_startloader] ; Add EDI Base. EDI now contains address of a block ; inside the layer. 

jmp edi ; Execute that block. 

return_addy175_1: 

cmp esi, 4 ; When we get back from the ;block, we check whether we ;have done every blocks. 

jnz mad_loop175_1 ; if we didn't, loop! 

bpxcheck175_1: ; Label used for BPX check. 

jmp @layer175_1 

tricky_call175_1: 

pop ebx ; Ret address is in EBX 

jmp fake_ret175_1 ; Jmp to fake return address. 

@layer175_1: ; end of the layer.
```

### Layer Selector (中文注释版)

```assembly
xor esi,esi ; ESI = 0 

mad_loop175_1: ; 循环标签 

inc esi ; ESI++ 

mov edi,dword ptr [ebp+(esi*4)+EIPtable175_1] ; 获取 block 地址 

mov ebx,dword ptr [ebp+(esi*4)+RETable175_1] ; 获取"Encrypted" 返回地址 

Add ebx, [ebp+_startloader] ; 添加基址 

push ebx ; 从堆栈中保存返回地址 

Call tricky_call175_1 ; 伪 call 

db 0EBh,01,0E8h ; 垃圾代码 

fake_ret175_1: ; 伪返回地址标签 

Add edi, [ebp+_startloader] ; 添加 EDI 基址.EDI ; 包含 Layer 中的 block 的地址 

jmp edi ; 执行 block. 

return_addy175_1: 

cmp esi, 4 ; 当我们从 block 返回时，校验是否已经完成了全部的 block 

jnz mad_loop175_1 ; 没有则循环 

bpxcheck175_1: ; 用于 BPX 校验的标签. 

jmp @layer175_1 

tricky_call175_1: 

pop ebx ; EBX 中包含 ret 地址 

jmp fake_ret175_1 ; 跳向伪返回地址. 

@layer175_1: ; layer 结束.
```

上述代码是 `Layer` 的主体部分。通过某种方式在 `Layer` 中循环执行 `block`。同时还包括返回地址和伪 `CALL`。如果调试时你不过这个 `CALL`，代码不会被执行而程序将会运行。假使是一个病毒程序，你的系统将会被感染。如果你不幸的在分析代码时终止，你将不得不重新来过。

## Layer Blocks

```assembly
include obfuscation/junk198.inc ; I have added a few junks macro ; manually in order to add a ; little fun :) 

dec_loader175_1: ; Decrypt label 

xor byte ptr [edx],cl ; Defaut options were used. ; Very simple encryption. 

inc edx ; Code to decrypt++ 

dec ecx ; Loop index-- 

test ecx, ecx ; is ECX = 0 ? 

jnz dec_loader175_1 ; no :() therefore we continue ; to decrypt. 

; This encryption can be different for each layer if you enable the option in the layer Generator. 

lea edx , [ebp+bpxcheck175_1] ; Grab address of BPX check. 

cmp byte ptr [edx],0CCh ; Any break point ? 

jnz return175_1 ; no. Good boy. 

rdtsc ; Ah.. he did put a bpx.. ; EAX = random value 

push eax ; push eax on stack 

ret ; Return to it :) Crash the ; poor guy. 

return175_1: ; on return block 

include obfuscation/junk199.inc ; a few junk 

include obfuscation/junk19A.inc ; ditto. 

SEHBLOCK 66137317 28513829 ; SEH block macro with ; keys in parameters. 

ret ; return 

inst175_2_1: ; another block of code 

add dword ptr [esp], 41952561 ; fix return address and return. 

ret 

inst175_3_1: ; Another block. 

mov ecx, (offset _end174_1- @layer175_1) ; Get Size of layer 

add dword ptr [esp], 13007360 ; Fix return address and return 

ret 

inst175_1_1: ; Another block 

lea edx, [ebp+@layer175_1] ; Get Layer address 

add dword ptr [esp], 30560857 ; Fix the return address ; and return. 

ret 

EIPtable175_1 dd 000DEADh, (offset inst175_1_1 - offset startloader), (offset inst175_2_1 - offset startloader), (offset inst175_3_1 - offset startloader), (offset _end174_1 - offset startloader) 

; This is a table of offset used to redirect the code. 

RETable175_1 dd 0031000h, (offset return_addy175_1 - offset startloader - 30560857) , (offset return_addy175_1 - offset startloader - 41952561),(offset return_addy175_1 - offset startloader - 13007360),(offset return_addy175_1 - offset startloader - 37623488) 

; This is a table of return address with a little "encryption". 

; You can notice the first member of the tables : DEADh and 31000h. Those values are constants and can be random using the RANDOM_FIRST_BLOCK  
```




## 注释

- `; option in the layer generator.`
- 上述 Layer 由我编写的简易 Layer 生成器 `Layer generator Engine` 生成。已为读者添加了注释。

## Layers Blocks

除了 `obfuscation/junk198.inc`，我还添加了一些 `junks` 宏。

```asm
dec_loader175_1: ; 解密标签
xor byte ptr [edx],cl ; 使用默认选项
; 简单的加密
inc edx ; 代码解密++
dec ecx ; 循环索引
test ecx, ecx ; 校验 ECX 是否等于 0。
jnz dec_loader175_1 ; 非也:( 继续解密).
```

如果打开加密器中的选项，每个 Layer 的加密将会不同。

```asm
lea edx , [ebp+bpxcheck175_1] ; 获取 BPX 校验地址.
cmp byte ptr [edx],0CCh ; 有断点吗?
jnz return175_1 ; 不错。好孩子。
rdtsc ; 它放了一个 bpx..
; EAX =任意值
push eax ; eax 入栈
ret ; 返回 :)崩了你：-）
return175_1: ; on return block
include obfuscation/junk199.inc ; 垃圾代码
include obfuscation/junk19A.inc ; 同上.
SEHBLOCK 66137317 28513829 ; 以 key 作为参数的 SEH block 宏
ret ; 返回
inst175_2_1: ; 又一个代码 block
add dword ptr [esp], 41952561 ; 修正返回地址并返回
inst175_3_1: ; .
mov ecx, (offset _end174_1- @layer175_1) ; 大小
add dword ptr [esp], 13007360 ;
inst175_1_1: ; 另一个 block
lea edx, [ebp+@layer175_1] ; 得到 layer 地址
add dword ptr [esp], 30560857 ; 修正返回地址并返回
ret
```

`EIPtable175_1 dd 000DEADh, (offset inst175_1_1 - offset startloader), (offset inst175_2_1 - offset startloader), (offset inst175_3_1 - offset startloader), (offset _end174_1 - offset startloader)`

这是一张用来重定向代码的偏移表。

`RETable175_1 dd 0031000h, (offset return_addy175_1 - offset startloader - 30560857) , (offset return_addy175_1 - offset startloader - 41952561), (offset return_addy175_1 - offset startloader - 13007360),(offset return_addy175_1 - offset startloader - 37623488)`

这是一张经过一定“加密”的返回地址表。

注意表中的第一个成员：`DDH` 和 `31000H`。它们是常数并可以在加密器的 `RANDOM_FIRST_BLOCK` 选项中任意取值。

### Protection Weakness

Those layers have a few weaknesses. You can use `BPM` (Hardware Break Point) on the next layer once you have passed the `SEH` that is going to clear the debug registers. Another weakness is the static size of the layer. Using this information, one can pass the layers rather quickly with a few `Soft ICE` macros for instance. I didn't turn the random size option on, on purpose to allow such attacks.

此 Layer 包含了一些漏洞。其中一个是：只要通过了清空寄存器的 `SEH`，你就可以在下个 Layer 上设置硬件断点。另一个是由于 Layer 的固定大小。只要运用一些如 `SoftICE` 宏的插件就可以很快地通过 Layer。我禁止了 `RANDOM SIZE` 选项，以允许这种攻击。

Those layers always use the same encryption algo, which can allow one to write scripts to decrypt the binary. And as you can read in a few submissions, some people did it. I did put this weakness on purpose as well. In a challenge i had done in the past, i had used random encryption for each layers, this time i choose not to use it. It is possible to bypass the 175 layers in a few seconds easily as well using a live approach. As we know wich API functions are going to be used, we can set a break point after the `BPX` checks have occured. Another possibility is to create a little utility that will `PATCH` the system dll in memory (each application has a copy of the dll) and to redirect them to a place that you contol. This way you can put breakpoints without triggering any Detection code.

另外 Layer 相同加密方式的使用，使得有人可以写出解密所需的脚本文件。一些人就做到了。我也故意放过了这个漏洞。以前，我任意加密每个 Layer，这次我没有。你可以利用现成的脚本文件很容易的通过这 175 个 Layer。你可以在 `BPX` 校验后对 API 函数设置断点。另一种可能就是内存中的系统 DLL（每个应用程序都用到的）进行修改，使它们重定向到你所控制的范围。这种方法可以使设置的断点不触发任何侦测代码而获得成功。

Talking of patching the Windows dll files, it is possible to patch `ntdll` to avoid the Debug Registers access in the context structure, by hooking the Exception Handling Mechanism of Windows. This allows one to put Hardware Breakpoints anywhere without ever having problems, never seeing his debug breakpoints beeing erased etc. The cool thing is you don't even need a Kernel Mode Driver to do that. I leave this as an exercice for interested people.

说到这里，`NTDLL` 的修改可以避免寄存器通过挂起系统的异常机制（hooking the exception handling mechanism of windows）访问 `CONTEXT STRUCTURE`。因此设置断点将会很顺利，也不用担心被擦除。你甚至连 `Kernel Mode Driver` 都可弃之一边。留下这个供大家练习。

### Virtual Machine

The final protection of the binary is a complete Virtual Machine i wrote for the challenge. I have designed a Virtual CPU that will interpret my own Assembly language. The Virtual Machine is quite simple to understand and isn't very complex. Virtual Machines seem to be a new trend in protection systems, so i thought it could be a good thing to write one for such a challenge. The instruction encoding is very trivial, and could have been a lot harder to understand. The first Version i had in mind was a lot more complex. I wanted not only to have a pseudo language, but also to program the instructions handlers emulating real x86 instructions. Each handler would be a few hundred instructions long and a lot harder to analyse. A small program has been written with this Virtual Machine Assembly language, and it was used to authenticate the user running the binary.

Read next part for further informations

虚拟设备

最后一种代码保护是我为这次挑战而写的虚拟设备。我设计了一个用来解释自身汇编语言的虚拟 CPU。这种保护很容易理解，并不复杂。

虚拟设备在各种保护方式中是新兴事物，因此我觉得很有必要在我的挑战中写出来。指令编码既微不足道又难于理解。因此我的首个版本是十分复杂的。不仅是伪语言，我还需要写出指令的操作过程。

如果仿效 x86 指令，每个操作将包含上百的指令且很难分析。

有一种用虚拟设备汇编语言写成的小程序，可以被用来鉴别作者运行的代码。

阅读下面的部分获取进一步的信息

### 3. Something uncommon has been used to protect the code from beeing reverse engineered, can you identificate what it is and how it works?

Even though, a few protection systems are using some kind of Virtual Machines, those aren't very common. Especially in Malwares and other exploits.

3.一些反逆向的特殊保护方式，你能认出它们并知道工作原理吗？

虽然，一些程序使用虚拟设备保护自身，但是这种方式仍然不普遍。特别是在病毒和其它开发中。

### Virtual CPU description and Inner working

#### Registers

```asm
REGISTERS STRUC
R0_ dd ? ; 000
R1_ dd ? ; 001
R2_ dd ? ; 002
COUNTER_ dd ? ; 003
EIP_ dd ? ; 004 -> reserved
STATE_ dd ? ; 005
REGISTERS ENDS
```

这是我源代码的原始结构。每个寄存器都是双字。有些寄存器没有使用，因为它们是为将来新的虚拟设备保留的。就是“`EIP_`”。我打算每个指令中添加其它的信息，但是我没有这样做，因为我不想程序过于复杂。但我会修改所有指令的指示器。这使得代码流程变的十分疯狂。文件中指令顺序将不会影响真正流程的执行。

`STATE` 寄存器是最小的 `E Flags`。此寄存器的修改依赖与其它的指令。

`COUNTER` 寄存器在循环指令中使用。循环中常用的是 `ECX` 寄存器。

```asm
regs REGISTERS <>
R0 equ 000b
R1 equ 001b
R2 equ 010b
COUNTER equ 011b
EIP equ 100b
STATE equ 101b
```

Here are a few other definitions used in my program. I started to represent the registers in binary because i wanted to do complex opcode decoding. I will do that for another version ;)

Registers Initialization



## 寄存器初始化

```
mov dword ptr [regs.R0],"livE" ; Registers are initialized with a
; Slayer Song Title.
..
mov dword ptr [regs.R1],"saH " ; Evil Has No Boundaries!
..
mov dword ptr [regs.R2]," oN "
..
mov dword ptr [regs.COUNTER],"nuoB"
..
mov dword ptr [regs.EIP],"irad"
..
mov dword ptr [regs.STATE],"! se"
```

在 VM 的开头，我首先使用一个金属打击乐队的曲名初始化我的寄存器。选择这个曲名是因为我计划在虚拟机中进行一些邪恶的操作。虽然它不如初始版本复杂，但足以保留那个有趣的字符串 ;)。

虚拟机包含约 34 条指令。（我根据指令不同的用途进行计数）

我将展示几个指令处理器来解释虚拟机的内部工作机制，但不会在此展示所有指令。

我赋予了程序一些新的定义。我在二进制代码中描述寄存器，这是因为我想对操作码进行完全的解码。这将在下个版本中实现。

寄存器初始化：

```
mov dword ptr [regs.R0],"livE" ;寄存器以曲名初始化
..
mov dword ptr [regs.R1],"saH " ; 恶到极点了
..
mov dword ptr [regs.R2]," oN "
..
mov dword ptr [regs.COUNTER],"nuoB"
..
mov dword ptr [regs.EIP],"irad"
..
mov dword ptr [regs.STATE],"! se"
```

在 VM 的开头，我使用一个金属打击乐队的曲名对寄存器进行初始化。选择它是因为我将会用虚拟设备“作恶”。

尽管没有第一个版本复杂，但是仍然差不了多少。

虚拟设备包含 34 种指令。（根据各个指令独特的作用区分）

我会通过一些指令操作来解释虚拟设备的工作原理，但不是全部（指令）。

## Pcode Fetcher

虚拟机在寄存器初始化后的第一件事就是获取 Pcode 的入口点并跳转到第一个 Pcode 处理器。

```
movzx eax, byte ptr [esi] ; ESI is Pcode Entry Point. This code
; gets the first instruction Prefix.
mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset table to
; find the Pcode family it has to
; execute.
movzx eax, byte ptr [esi+1] ; get second byte, use it as an
; index into last table.
; The VM now knows what instruction it
; has to emulate and goes to it.
JMPNEXT ; Emulate a jmp dword ptr [eax*4+edi]
; with Exception Handling and
; Context Manipulation.
; Jmp to the next Pcode
; instruction handler
```

获取 Pcode:

虚拟设备的寄存器初始化后的第一件事就是获取 Pcode 的入口点并来到首个 Pcode 处理中。

```
movzx eax, byte ptr [esi] ; ESI 是 Pcode 入口点.此代码得到首个指令名称
mov edi, dword ptr [eax*4+poffset] ; 使用偏移表找到 Pcode 执行什么
movzx eax, byte ptr [esi+1] ;得到第二个字节，作为最后表的索引
VM 得到仿效的指令并执行
跳向下个; 使用异常操作和 Context 仿效 jmp dword ptr [eax*4+edi]
;跳向下个 Pcode 指令操作
```

## Examples of Instructions implemented inside the Virtual Machine

在开始这些示例之前，我想说明一下虚拟机中存在的一些指令未被使用，而是被留作陷阱。其中三条使用了自修改代码（Self-modifying code）。有人报告说它们不起作用，但它们应该能工作。这种一位差异（off by one）是因为操作码是从其他指令处理器中被调用的。

虚拟设备指令实现的实例：

在我开始实例之前，我想讲述一下一些虚拟设备中未被使用，并被留下作为一种陷阱的指令。其中的三个使用了自校验代码。

人们认为它们是不起作用的，但事实正好相反。这是因为操作码是从其它指令操作中被调用的。

有两条指令在需要执行特定代码段时会在线修改一条指令，随后恢复该指令的状态。我太懒了，没有检查这些指令是否真的有漏洞，或者是否没有使用正确的参数。其中一条`unused`指令确实有一个漏洞，我很高兴有人注意到了它。由于该指令未被使用，因此它只是一个陷阱指令。

该指令本应是一个虚拟`BSWAP`，但它没有保存交换的结果。另一条未使用的指令是`INT 3`。这条指令允许用户在 Pcode 程序中设置断点，并使用调试器从该指令开始跟踪。我在最终的虚拟机中保留了这条指令，很高兴看到有人发现并滥用了它！

### STOPVM

这里展示的第一条指令非常简单。它告诉虚拟机停止，程序将返回到普通的 `x86` 汇编程序。

```
@STOPVM:
pop dword ptr fs:[0] ; Im using SEH to jmp from handlers to
;handlers in the VM. 在 VM 中，我使用 SEH 从一个操作跳到另一个操作。
add esp,4 ; Therefore i need to remove the handler
; installed before i do anything. 因此我需要在开始之前删除安装操作
popad ; i restore the registers.. 恢复寄存器
..
push dword ptr [Pret] ; Put the Return Address (to get out of
; the VM) on the stack. 把返回地址放入堆栈中（从 VM 中退出）
..
xor [esp],'HAX0 '; Decrypt it with a funny string: （用有趣的字符串进行解密）
;HAXO(R)
..
ret ; Get out of the VM. 退出 VM。
```

此指令没有使用任何字节码获取器，因为它不需要跳转到另一个处理器。现在我将展示一条真正的指令：一个虚拟的入栈指令。

### LOAD

```
@Load:
pop dword ptr fs:[0]
add esp,4
popad
; Same as every handler, remove SEH and restore registers. 与每个操作一样，删除 SHE 恢复寄存器。
mov eax,dword ptr [esi+2] ; Get into EAX the first operand
; of the instruction. 进入 EAX，指令的首个操作数
xor eax,37195411h ; Decrypt it. 解码
push eax ; Push it onto the stack. 压入堆栈
mov eax,0FFFFFF3Fh ; EAX = FFFFFF3Fh
not eax ; EAX = not(EAX) = C0h
shr eax,5 ; EAX = EAX shr 5 = 6 :
; This is the instruction length 这是指令长度
lea esi, [esi+eax] ; ESI = Instruction Pointer.指令指针
; Displace the Instruction Pointer 显示指令指针
; 6 bytes further. 6+字节
movzx eax, byte ptr [esi] ; ESI now points to the new
; instruction to be executed. ESI 指向将执行的新的指令
mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset
; table to find the Pcode family 使用偏移表找到需要执行的 Pcode 系
; it has to execute.
movzx eax, byte ptr [esi+1] ; get second byte, use it as an
; index into last table. 得到第二个字节，作为最后表的索引
; The VM now knows what instruction it

; has to emulate and goes to it. 现在 VM 得到仿效什么指令并执行
JMPNEXT ; Emulate a jmp dword ptr [eax*4+edi]; with Exception Handling and
; Context Manipulation. 跳向下个。使用异常操作和 context 仿效 jmp dword ptr [eax*4+edi]

; Jmp to the next Pcode 跳向下个 Pcode 指令操作
; instruction handler
```

从这个小小的处理器可以看出，该指令长度为 6 个字节。它只接受一个参数，该参数位于指令起始处向后 2 字节的位置（`ESI+2`）。该参数使用 `37195411h` 进行加密。解密后的参数被压入堆栈，然后虚拟机调用下一条指令。

因此，我们可以说这条指令是一个 push。由于 `push` 已经是 `x86` 指令，因此我将我的虚拟入栈指令命名为 `LOAD`。

可以这样使用它：`LOAD number`

### VMXOR

```
@VMXORDISPATCHER:
pop dword ptr fs:[0]
add esp,4
popad
; Same as every handler, remove SEH and restore registers. 与每个操作一样，删除 SEH 并恢复寄存器
movzx eax, byte ptr [esi+2] ; Get the Index Register to acces
; the Virtual CPU registers. 得到索引寄存器访问虚拟 CPU 寄存器
mov eax, dword ptr [regs+eax*4] ; edi = Register value to know
; which register is going to be EDI=寄存器值以得到与哪个寄存器有关
; concerned (RO, R1 , R2)
; EAX = value used by the XOR. EAX=XOR 使用的值
```



`movzx ecx, byte ptr [esi+3] ; ECX = type of XOR. ECX=XOR 的类型`

`; Byte ptr ? Word Ptr ? or`

`; Dword Ptr..`

`jmp dword ptr [xortable+ecx*4] ; Jmp to the good handler 跳向正确操作`

`; accordingly.`

`@VMXORBPTR:`

`movzx ecx, byte ptr [esi+4] ; Get Index Register for the`

`; destination. 得到目标索引寄存器`

`mov ecx, dword ptr [regs+ecx*4] ; edi = Register value to know`

`; which register is going to be EDI=寄存器值以得到哪个寄存器被使用`

`; used (RO, R1 , R2)`

`xor byte ptr [ecx],al ; XOR BYTE PTR`

`add esi,5 ; Instruction Length is 5 指令长度 5`

`movzx eax, byte ptr [esi] ; ESI now points to the new`

`; instruction to be executed. ESI 指向执行的新指令`

`mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset`

`; table to find the Pcode family 使用偏移表找到执行的 Pcode 系`

`; it has to execute.`

`movzx eax, byte ptr [esi+1] ; get second byte, use it as an`

`; index into last table. 得到第二个字节，使用它作为最后表的索引`

`; The VM now knows what`

`; instruction it has to emulate`

`; and goes to it. VM 得到仿效哪个指令并执行`

`JMPNEXT ; Emulate a jmp dword ptr`

`; [eax*4+edi] with Exception`

`; Handling and Context`

`; Manipulation. 使用异常造作和 CONTEXT 仿效 jmp dword ptr`

`; [eax*4+edi]`

`; Jmp to the next Pcode 跳向下个 Pcode 指令操作`

`; instruction handler`

`@VMXORWPTR:`

`movzx ecx, byte ptr [esi+4] ; Get Index Register for the`

`; destination 得到目标索引寄存器`

`mov ecx, dword ptr [regs+ecx*4] ; edi = Register value to know`

`; which register is going to be EDI=寄存器值以得到使用哪个寄存器`

`; used (RO, R1 , R2)`

`xor word ptr [ecx],ax ; XOR WORD PTR`

`add esi,5 ; Instruction Length is 5 指令长度 5`

`movzx eax, byte ptr [esi] ; ESI now points to the new instruction to be executed. ESI 指向执行的新指令`

`mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset`

`; table to find the Pcode family`

`; it has to execute. 使用偏移表找到执行的 Pcode 系`

`movzx eax, byte ptr [esi+1] ; get second byte, use it as an`

`; index into last table. 得到第二个字节，使用它作为最后表的索引`

`; The VM now knows what VM 得到仿效什么指令并执行`

`; instruction it has to emulate`

`; and goes to it.`

`JMPNEXT ; Emulate a jmp dword ptr`

`; [eax*4+edi] with Exception`

`; Handling and Context`

`; Manipulation. 使用异常操作和 CONTEXT 仿效 jmp dword ptr`

`; [eax*4+edi]`

`; Jmp to the next Pcode 跳向 Pcode 指令操作`

`; instruction handler`

`@VMXORDPTR:`

`movzx ecx, byte ptr [esi+4] ; Get Index Register for the`

`; destination 得到目标索引寄存器`

`mov ecx, dword ptr [regs+ecx*4] ; edi = Register value to know`

`; wich register is going to be`

`; used (RO, R1 , R2) EDI=寄存器值以得到哪个寄存器被使用`

`xor dword ptr [ecx],eax ; XOR DWORD PTR`

`add esi,5 ; Instruction Length is 5 指令长度 5`

`movzx eax, byte ptr [esi] ; ESI now points to the new`

`; instruction to be executed. ESI 指向执行的新指令`

`mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset`

`; table to find the Pcode family`

`; it has to execute. 使用偏移表找到执行的 Pcode 系`

`movzx eax, byte ptr [esi+1] ; get second byte, use it as an`

`; index into last table. 得到第二个字节，使用它作为最后表的索引`

`; The VM now knows what VM 得到仿效哪个指令并执行`

`; instruction it has to emulate`

`; and goes to it.`

`JMPNEXT ; Emulate a jmp dword ptr`

`; [eax*4+edi] with Exception`

`; Handling and Context`

`; Manipulation. 使用异常操作和 CONTEXT 仿效 jmp dword ptr`

`; [eax*4+edi]`

`; Jmp to the next Pcode 跳向 Pcode 指令操作`

`; instruction handler`

From this piece of code we can learn many things. The XOR instructions are coded with 5 Bytes. It has two parameters.
One register has the value used to do the XOR and One Register has a pointer to the location to be xored.It also have
a byte saying whether it is a XOR BYTE PTR, a XOR WORD PTR or a XOR DWORD PTR.

上述代码中，我们可以学到许多东西。XOR 指令长度 5 个字节。含有 2 个参数。一个寄存器做 XOR BYTE PTR，另一个指向 XOR
的地址。另外还有一个检验是 XOR BYTE PTR，XOR WORD PTR 还是 XOR DWORD PTR 的字节。

This instruction handler is therefore handling Virtual XOR instruction.
这个指令操作是处理虚拟 XOR 指令的。

How were the virtual instruction used to create a program ?
I will now show how i did to create virtual instruction, because the x86 assembler doesn't know them and will never
compile a LOAD or a VMXOR.To do so, i used a very simple way: MACRO. For each instruction, a corresponding macro has
been created, and is used to encode the instruction for me. This way i can write a program with my Assembly mnemonics
without caring of the opcodes representation.I will now show the 3 Macros used for the examples Virtual Instruction
descrived above

虚拟指令怎样创建一个程序？
现在我要解释我是怎样创建虚拟指令的，因为 x86 汇编程序不知道它们，也不能编译 LOAD 或 VMXOR，因此我运用简单的宏就
做到了。对于每个指令，创建相应的宏，并使用它来对指令进行加密。通过这种方式，我不必在乎操作码的参与，只要使
用汇编助记忆码就写成了程序。下面我帖出用于下面虚拟指令实例的 3 个宏：

```
STOPVM macro
db 02,00
endm
```

This is the macro for the STOPVM instruction. Usage: `STOPVM` 用于 STOPVM 的宏

```
Load macro x
db 00,00
dd x xor 37195411h
endm
```

This is the macro for the LOAD instruction. Usage: `LOAD x` 用于 LOAD 指令的宏

```
VMXOR macro reg0,kind,reg1
db 01,03,reg0,kind,reg1
endm
```

This is the macro for the VMXOR instruction. Usage: `VMXOR Rx xPTR Rx` 用于 VMXOR 指令的宏

P-code Program used in this challenge:
The P-code is my own assembly language, thus IDA doesn't know anything about it. Here is how it looks under a disassembler:
Ok, it doesn't look so good. So now, here is the complete program (copy pasted from my source) i have written with
my OWN assembly language. Cool isn't it ? :-)

挑战中使用的 Pcode 程序：
Pcode 是使用我自己的汇编语言，因此 IDA 不能获取任何信息。下面是反汇编器中的样子。
哦，它看起来不是很舒服。下面的是完全用我的汇编语言编写的程序。是不是很酷？

```
CODE@:

Pcode1:
MOVE pcrypt COUNTER
LOADPTR startpcodecrypted
RestoreREG R0
MOVE 'S' R2

decryptpcode:
VMXOR R2 BPTR R0
INCR R0
DECR COUNTER
BNZ decryptpcode

startpcodecrypted:
MOVE 05CC80E31h R1
APICALL GetCommandLineA
SCANB " " 255h
BZ youwishdude
DLOAD R0 R2
ADDREG R2 01D9BDC45h
ADDREG R1 74519745h
SUBREG R2 0AD45DFE2h
ADDREG R1 0DEADBEEFh
ADDREG R2 "hell"
SUBREG R1 17854165h
SUBREG R2 "Awai"
ADDREG R1 "show"
ADDREG R2 "its "
SUBREG R1 " no "
ADDREG R2 "driv"
ADDREG R1 "merc"
SUBREG R2 "nuts"
SUBREG R1 "y!!!"
SUBREG R2 "eh?!"
ANDREG R2 0DFFFFFFFh
LOADREG R2
LOADREG R1
CMPQ firstcheckdone
CLEAR COUNTER
BZ youwishdude

firstcheckdone:
INCR R0
ADDREG R0 2
INCR R0
WLOAD R0 R1
LOADREG R1
RESTOREREG R2
LOADREG R0
LOADPTR tricky-98547h
RESTOREREG R0
ADDREG R0 98548h
DECR R0
VMXOR R2 WPTR R0
RESTOREREG R0
BLOAD R0 R2
ADDREG R0 2
BLOAD R0 R1
RADD R2 R1
VMCALL sub_check_routine

tricky:
ENCRYPTED CLEAR COUNTER ; This one get patched at run time! 加密清除指针在运行时修改

cracked:
LOADREG COUNTER
LOADPTR congrats
APICALL printf
CleanStack 8
BR outout

youwishdude:
LOAD 0
LOADPTR notgood
APICALL printf
CleanStack 8

outout:
STOPVM

sub_check_routine:
MOVE 'L' R1
INCR R1
INCR R1
ADDREG R1 5
DECR R1
SUBREG R1 4
SUBREG R2 'Z'
CMPREG R1 R2
BNZ youwishdude
DECR R0
BLOAD R0 R2
ADDREG R0 2
BLOAD R0 R1
RADD R2 R1
INCR R2
SUBREG R2 4Eh
LOADPTR retdecrypt-0DEADh ; push ptr to patch 把 PTR 压入栈修改
RESTOREREG R0
ADDREG R0 0DEACh
INCR R0
VMXOR R2 BPTR R0
MOVE msgcrypt COUNTER
LOADPTR goodboy
RestoreREG R0
INCR R2

decryptmsg:
VMXOR R2 BPTR R0
INCR R0
DECR COUNTER
BNZ decryptmsg
```



# 汇编代码片段

```asm
retdecrypt:  

VMRETCRYPTED   

DATA@: 

notgood db "Please Authenticate!",10,13,0 

goodboy: 

congrats db "Welcome...",10,13 

db "Exploit for it doesn't matter 1.x Courtesy of Nicolas Brulez",0 

goodboyend: 
```

# 密码保护机制分析

> 此小程序在代码中使用密码保护。如你所见，密码保护非常短。我可以使用上百的命令行以使程序难于分析。同时此代码是明显的垃圾代码。我也可以在程序中放置 `Pcode` 垃圾指令。此密码校验非常差非常简单，因此关注此密码的程序远远大于虚拟设备的程度。下次，我会使它更加复杂，以使大家只能分析虚拟设备和指令的设置。

> 你可以比较原始的 `Pcode` 程序，你会发现教程做的很棒。

## 关于密码保护的一些注意

> 密码使用非常简单的运算法则校验，但是它也被使用在解密 `Pcode` 的其它部分。这里有个漏洞可以无强制会操作码分析而找到正确的值。

### 加密字符串分析

下面是加密字符串：

```c
0x14, 0x26, 0x2F, 0x20, 0x2C, 0x2E, 0x26, 0x6D,  
0x6D, 0x6D, 0x49, 0x4E, 0x06, 0x3B, 0x33, 0x2F,  
0x2C, 0x2A, 0x37, 0x63, 0x25, 0x2C, 0x31, 0x63,  
0x2A, 0x37, 0x63, 0x27, 0x2C, 0x26, 0x30, 0x2D,  
0x64, 0x37, 0x63, 0x2E, 0x22, 0x37, 0x37, 0x26,  
0x31, 0x63, 0x72, 0x6D, 0x3B, 0x63, 0x00, 0x2C,  
0x36, 0x31, 0x37, 0x26, 0x30, 0x3A, 0x63, 0x2C,  
0x25, 0x63, 0x0D, 0x2A, 0x20, 0x2C, 0x2F, 0x22,  
0x30, 0x63, 0x01, 0x31, 0x36, 0x2F, 0x26, 0x39,  
0x43  
```

> 大家都知道 `C` 字符串以空结尾。因此，使用于加密此字符串的值是 `0x43`。关键在于加密字符串的最后字节。`X XOR 0 = X`。:-)

> 另一种可能的方式是找到查看代码结构的正确值。我们做一个调用程序，因此我们需要指令完成返回。此指令在虚拟设备中以虚拟 `RET` 实现。由此看来，我们只要找到计算这个密码的指令操作码就可以了。

# 问题 4：快速分析非一般特点

## 提供快速分析非一般特点的方式

> 围绕这个问题，我需要反汇编我的虚拟设备。有人给了我完整工作的反汇编器，因此我没有自己写一个。我再次邀请你看看他们的教程。一个作者在到期日后 E-MAIL 我一个包括源代码的工作的 `IDA` 处理模块。此模块我未使用，因为是在到期日后给我的，但它有值得学习的地方。这个模块将在结果公布后上传到 `HONEYNET` 上。

# 问题 5：工具选择

## 哪个工具最适合分析这种代码，为什么？

> 在我看来是交互式反汇编器或 `CPU EMULATORS`。反汇编器可以用来分析静态代码，打消困惑，加密代码等。如果它提供写处理模块，你甚至可以写出虚拟设备的反汇编器并分析整个静态代码。`CPU EMULATOR` 可以快速的加密代码，`LAYERS` 等等。如果它可以消除困惑，你就拥有了一个强劲的工具。我不喜欢调试器，因为他们不可信赖。我可以很容易的写出驱动挂起调试以加密代码。调试器毫无用处并且可能使系统重起。

# 问题 6：二进制文件用途

## 鉴别代码用途

> 代码在等待使用者鉴别通过命令行传达到程序中的密码。一旦确认，代码将产生一些消息。看起来象伪使用。在实际中，它是迷惑眼睛的真实保护。

# 问题 7：用户交互

## 等待使用者的代码是什么？请详述你发现的

> 代码等待命令行中的密码。密码用来访问真正的程序。为了找到密码，你需要逆向代码。解密每个访问虚拟设备的 `LAYERS`。此虚拟设备包括校验已输密码的虚拟程序。你需要逆向虚拟设备（或盲目的跟踪）以了解指令设置。不过它非常简单，容易逆向。我推荐你看以下教程了解详细信息。

# 问题 8：增加逆向难度

## 你认为什么代码可以是逆向难于分析？

> 此代码留下许多安全缺陷，需要很多改进。

- **无 `PUSH/POPAD` 的垃圾代码**：`Junk Code without pushad/popad`
- **跟踪 `80%` 的 `Pcode` 垃圾代码**：`P-code Junk code to really drive people tracing the protection nuts`
- **静态加密缺陷**：文档中提及加密缺陷。主要是静态加密和每个 `LAYERS` 的固定大小。
- **地址表随机常数**：`LAYER` 在地址表的第一个值中使用任意常数。
- **更好的 `BPX` 侦测**：更好的 `BPX` 侦测将会改进。
- **`SEH` 用于初始化/解密代码**：防止 `NOP`。
- **时间侦测随机常数**：时间侦测的任意常数将使脚本绕过侦测更加困难。
- **保护工具生成代码**：保护由我自己的工具产生。种类的增加会自动的增加找到切入点的难度。
- **虚拟设备操作**：虚拟设备操作很简单。需要更多的迷惑性的代码。
- **虚拟设备指令**：虚拟设备中需要使用更多指令延长你的分析时间。
- **复杂操作码解码**：复杂操作码的解码对逆向是一种挑战。



# 9. 结论

## 利用密码术代替简单方式校验序列号
*   **Utilization of Cryptography rather than a simple algo to check for the serial number**
    *   利用密码术代替简单方式校验序列号。

## 更多不同的 LAYERS
*   **More Layers and different ones. There are a lot of ways to stop ring 3 debuggers that could have been used to stop anyone trying to debug it.**
    *   更多不同的`LAYERS`。有许多方式都可终止`RING3`级别的调试器进行调试。

## 输入表保护防止得到 API 函数
*   **Imports Protection to make sure noone knows the API function used until he meets them in the code.**
    *   输入表保护防止得到`API`函数，而只能从代码中得到它们。

## 利用宏仿效 x86 指令
*   **Emulation Macros to emulate simple x86 instructions. With those macros remplacing simple instruction, it would be a lot harder to analyse. one instruction would have a 20 instructions equivalent block of code for instance.**
    *   利用宏仿效`x86`指令。通过替换简单指令，使得分析更加困难。一条指令将包含同等的`20`条代码`block`。

## 结论内容
反逆向技术的使用可以减缓代码的分析。病毒可以在将来使用此技术，我们需要了解它。虽然大多数病毒是一些毫无技巧可言的笨蛋编写的，但仍然会有少数编写复杂代码的高手。将来，我们会在隐藏弱点进行反逆向的系统中发现需要的代码。间谍程序也会使用此技术隐藏它的活动。虽然具有挑战性甚至包含`password`保护运算，此代码仍有许多缺陷。此保护一周编写完成，比较难。最后，我指出逆向工程不是盗版技术而是用于安全社会的日常中。

(全文完)

4dd466d0e1e0cc123afbe504da2462ca
