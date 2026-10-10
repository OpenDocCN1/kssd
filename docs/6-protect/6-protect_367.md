

# 反逆向工程揭密（试炼篇）-全文完

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-707.htm`](https://www.pediy.com/kssd/pediy07/pediy7-707.htm)

Abstract   
Rather than doing another complete analysis of the binary, i will rather present the techniques i have used in the challenge,   
and how i have implemented them. The Scan of the Month 33 was released by the Honeynet Project in November 2004. I invite everyone   
to read the excellent submissions we received this month once they have read my paper. I am presenting the binary from the protection  
 author point of view, while they presented it from the analyst point of view. You will learn the methods and techniques used to Protect  
 / Unprotect a binary with this month's challenge. A lot of weaknesses were left on purpose in this binary and they will be presented here.   
Keywords: Software Protection; Reverse Code Engineering; Linux; Anti-Debugging; Anti-Anti-Debugging    
摘要：  
有人偏爱详细的分析过程，而我却喜欢从技术和实现方法的层面上去探讨。2004 年的 11 月，The Honeynet Project 发布了 The scan of the month 33。  
我推荐大家好好的研读一下。他们研究的就是前者。我除了对技术和实现方法进行研究外，大量的漏洞在我的文中也有披露。  
关键词：软件保护；逆向工程；Linux;反调试；调试。  
  
1. Introduction   
This month's challenge is to analyze an unknown binary, in an effort to reinforce the value of reverse engineering, and improve (by learning   
from the security community) the methods, tools and procedures used to do it. This challenge is similar to SotM 32. However, this binary has   
mechanisms implemented to make the binary much harder to analyze, to protect against reverse engineering.   
1.简介：  
最新的 month 33 挑战了一段未知的二进制代码，这对于我们加深逆向印象、改进方法、了解工具的使用及其整个过程具有积极的作用。这与先前发布的 The scan of the month 32 是相同的。唯一不同的是，此二进制代码更加难于分析。  
  
Skill Level: Advanced/Expert   
All we are going to tell you about the binary is that it was 'found' on a WinXP system and has now be sent to you for analysis. You will have to  
 analyse it in-depth and get as much information as possible about its inner working, and what is the goal of the binary. The main goal of this challenge is  
 to teach people how to analyse heavily armored binaries. Such techniques could be used in the future, and its time to get used to them.   
2. Identify and explain any techniques in the binary that protect it from being analyzed or reverse engineered   
技术含量：中/高级  
你将自己独立分析这些基于 WinXP 环境下的二进制代码。想完成挑战，你必须深刻的分析问题，搜集更多的信息去了解内部原理以及体会代码中包含  
的目的。最终使你能够独立的分析一层又一层的代码。  
2．鉴别并解释阻碍分析和逆向的技术。  
  
Many techniques have been used in order to slow down analysis and break reverse engineers tools:   
• PE Header Modifications   
Many fields of the PE header were modified in order to disturb analysing tools, and thus, the Reverse Engineer. I will quickly cover the most  
 important changes:   
\->Optional Header   
Magic: 0x010B (HDR32\_MAGIC)   
MajorLinkerVersion: 0x02   
MinorLinkerVersion: 0x19 -> 2.25   
SizeOfCode: 0x00000200   
SizeOfInitializedData: 0x00045400   
SizeOfUninitializedData: 0x00000000   
AddressOfEntryPoint: 0x00002000   
BaseOfCode: 0x00001000   
BaseOfData: 0x00002000   
ImageBase: 0x00DE0000 <--- "Non Standard" ImageBase   
SectionAlignment: 0x00001000   
FileAlignment: 0x00001000   
MajorOperatingSystemVersion: 0x0001   
MinorOperatingSystemVersion: 0x0000 -> 1.00   
MajorImageVersion: 0x0000   
MinorImageVersion: 0x0000 -> 0.00   
MajorSubsystemVersion: 0x0004   
MinorSubsystemVersion: 0x0000 -> 4.00   
Win32VersionValue: 0x00000000   
SizeOfImage: 0x00049000   
SizeOfHeaders: 0x00001000   
CheckSum: 0x00000000   
Subsystem: 0x0003 (WINDOWS\_CUI)   
DllCharacteristics: 0x0000   
SizeOfStackReserve: 0x00100000   
SizeOfStackCommit: 0x00002000   
SizeOfHeapReserve: 0x00100000   
SizeOfHeapCommit: 0x00001000   
LoaderFlags: 0xABDBFFDE <--- Bogus Value   
NumberOfRvaAndSizes: 0xDFFFDDDE <--- Bogus Value   
The "standard" ImageBase usually is 400000 for Win32 applications and Reverse Engineers are used to analyse programs with such an ImageBase.  
 While it isn't a protection by itself, this simple modification will confuse some Reverse Engineers, because they aren't used to such memory addresses.   
包括以下这些：  
（1）修改 PE 文件头：  
目的：干扰工具的分析。附上重要的修改：  
\->Optional Header   
Magic: 0x010B (HDR32\_MAGIC)   
MajorLinkerVersion: 0x02   
MinorLinkerVersion: 0x19 -> 2.25   
SizeOfCode: 0x00000200   
SizeOfInitializedData: 0x00045400   
SizeOfUninitializedData: 0x00000000   
AddressOfEntryPoint: 0x00002000   
BaseOfCode: 0x00001000   
BaseOfData: 0x00002000   
ImageBase: 0x00DE0000 <--- 非一般性基址  
SectionAlignment: 0x00001000   
FileAlignment: 0x00001000   
MajorOperatingSystemVersion: 0x0001   
MinorOperatingSystemVersion: 0x0000 -> 1.00   
MajorImageVersion: 0x0000   
MinorImageVersion: 0x0000 -> 0.00   
MajorSubsystemVersion: 0x0004   
MinorSubsystemVersion: 0x0000 -> 4.00   
Win32VersionValue: 0x00000000   
SizeOfImage: 0x00049000   
SizeOfHeaders: 0x00001000   
CheckSum: 0x00000000   
Subsystem: 0x0003 (WINDOWS\_CUI)   
DllCharacteristics: 0x0000   
SizeOfStackReserve: 0x00100000   
SizeOfStackCommit: 0x00002000   
SizeOfHeapReserve: 0x00100000   
SizeOfHeapCommit: 0x00001000   
LoaderFlags: 0xABDBFFDE <--- 伪值  
NumberOfRvaAndSizes: 0xDFFFDDDE <--- 伪值  
400000 作为 Win32 应用程序的一般基址，常常被逆向者在分析程序时使用。当程序不能保护自身时，这种简单的修改就可以对逆向工程造成影响，  
因为程序并不使用这样的内存地址。  
  
"Anti" OllyDbg:   
LoaderFlags and NumberOfRvaAndSizes were modified.. I have Reverse Engineered OllyDBG and Soft ICE to find a few tricks that could slow down  
 the analysis of a binary. With those two modifications, Olly will pretend that the binary isn't a good image and will eventually run the  
 application without breaking at its entry point. This could be a bad thing if you wanted to debug a mal ware on your computer, because you would get infected.   
反 OllyDbg:  
通过修改 LoaderFlags 和 NumberOfRvaAndSizes 实现。逆向 OllyDbg 和 SoftICE 后，我们会发现这种方法可以阻碍二进制代码的分析，因为程序会认为  
此二进制不是正确的基址，从而不在入口点处执行。导致直接的后果是：如果你调试一个病毒程序，你的计算机将会被感染。  
  
Anti Soft ICE : Blue Screen of Death and no Chocolate:   
The NumberOfRvaAndSizes field has been modified in order to reboot any computer running a recent version of Soft ICE. While Disassembling   
the PE Loader of Soft ICE, i found a very critical vulnerability in Soft ICE that allows one binary to crash any computer running Soft ICE without any   
code execution. This vulnerability (bug) has been reported to Compuware and should be fixed in the next version. Apparently it didn't happen on some  
 of the authors of the submissions for some reasons. Oh well.   
Here is the disassembly of Soft ICE PE loader to find out why it reboots your computer:   
.text:000A79FE   
.text:000A79FE loc\_A79FE: ; CODE XREF: sub\_A79B9+31j   
.text:000A79FE ; sub\_A79B9+3Cj   
.text:000A79FE ; DATA XREF: .text:00012F9Bo   
.text:000A79FE sti   
.text:000A79FF mov esi, ecx   
.text:000A7A01 mov ax, \[esi\]   
.text:000A7A04 cmp ax, 'ZM'   
.text:000A7A08 jnz not\_PE\_file   
.text:000A7A08   
.text:000A7A0E mov edi, \[esi+\_IMAGE\_DOS\_HEADER.e\_lfanew\]   
.text:000A7A11 add edi, esi   
.text:000A7A13 mov ax, \[edi\]   
.text:000A7A16 cmp ax, 'EP'   
.text:000A7A1A jnz not\_PE\_file   
.text:000A7A1A   
.text:000A7A20 movzx ecx, \[edi+IMAGE\_NT\_HEADERS.FileHeader.NumberOfSections\]   
.text:000A7A24 or ecx, ecx   
.text:000A7A26 jz not\_PE\_file   
.text:000A7A26   
.text:000A7A2C mov eax, \[edi+IMAGE\_NT\_HEADERS.OptionalHeader.NumberOfRvaAndSizes\]   
.text:000A7A2F lea edi, \[edi+eax\*8+IMAGE\_NT\_HEADERS.OptionalHeader.DataDirectory\]   
.text:000A7A33 mov eax, ecx   
.text:000A7A35 imul eax, 28h   
.text:000A7A38 mov al, \[eax+edi\] ; CRITICAL BUG! One can force EAX+EDI to be equal to zero. Reading at \[0\] in ring 0 isn't nice eh ;-)   
.text:000A7A3B   
.text:000A7A3B loc\_A7A3B: ; DATA XREF: .text:00012FA5o   
.text:000A7A3B cli   
   
  
    
.text:000A7A3C call sub\_15C08   
.text:000A7A3C   
.text:000A7A41 mov byte\_FA259, 0   
.text:000A7A48 push eax ; Save EAX   
.text:000A7A49 mov eax, dword\_16B56F ; EAX is modified by a saved dword   
.text:000A7A4E mov dr7, eax ; Debug Register 7 take the value in EAX   
.text:000A7A51 pop eax ; EAX is restored   
.text:000A7A52 mov dword\_FC6CC, esp   
.text:000A7A58 mov esp, offset unk\_FBABC   
.text:000A7A5D and esp, 0FFFFFFFCh   
.text:000A7A60 xor al, al ; AL is zeroed? Why this mov al, \[eax+edi\] then ?   
.text:000A7A60 ; I don't see the point. old code?   
.text:000A7A62 call sub\_4D2EB   
.text:000A7A62   
.text:000A7A67 call sub\_36AC1   
.text:000A7A67   
.text:000A7A6C xor edx, edx   
.text:000A7A6E   
.text:000A7A6E loc\_A7A6E: ; CODE XREF: sub\_A79B9+124j   
.text:000A7A6E call sub\_74916   
.text:000A7A6E   
反 SoftIce：蓝屏死机花屏：  
修改 NumberOfRvaAndSizes 会令含 SoftIce 的系统重起。通过反汇编 SoftIce 装载器，我发现：无需代码执行，一个二进位的变动就可使一台含 SoftIce 的  
电脑立刻崩溃。不过此漏洞已上报 COMPUWARE 并将在下一版本中改进。但是，由于某种原因，一些系统却运行正常。让我们来看一下反汇编后的 SoftICE  
装载器里到底有什么：  
.text:000A79FE   
.text:000A79FE loc\_A79FE: ; CODE XREF: sub\_A79B9+31j   
.text:000A79FE ; sub\_A79B9+3Cj   
.text:000A79FE ; DATA XREF: .text:00012F9Bo   
.text:000A79FE sti   
.text:000A79FF mov esi, ecx   
.text:000A7A01 mov ax, \[esi\]   
.text:000A7A04 cmp ax, 'ZM'   
.text:000A7A08 jnz not\_PE\_file   
.text:000A7A08   
.text:000A7A0E mov edi, \[esi+\_IMAGE\_DOS\_HEADER.e\_lfanew\]   
.text:000A7A11 add edi, esi   
.text:000A7A13 mov ax, \[edi\]   
.text:000A7A16 cmp ax, 'EP'   
.text:000A7A1A jnz not\_PE\_file   
.text:000A7A1A   
.text:000A7A20 movzx ecx, \[edi+IMAGE\_NT\_HEADERS.FileHeader.NumberOfSections\]   
.text:000A7A24 or ecx, ecx   
.text:000A7A26 jz not\_PE\_file   
.text:000A7A26   
.text:000A7A2C mov eax, \[edi+IMAGE\_NT\_HEADERS.OptionalHeader.NumberOfRvaAndSizes\]   
.text:000A7A2F lea edi, \[edi+eax\*8+IMAGE\_NT\_HEADERS.OptionalHeader.DataDirectory\]   
.text:000A7A33 mov eax, ecx   
.text:000A7A35 imul eax, 28h   
.text:000A7A38 mov al, \[eax+edi\] ; EAX+EDI=0 则崩溃。权限 0 中读取\[0\]地址可不妙。   
.text:000A7A3B   
.text:000A7A3B loc\_A7A3B: ; DATA XREF: .text:00012FA5o   
.text:000A7A3B cli   
.text:000A7A3C call sub\_15C08   
.text:000A7A3C   
.text:000A7A41 mov byte\_FA259, 0   
.text:000A7A48 push eax ; Save EAX   
.text:000A7A49 mov eax, dword\_16B56F ; 用双字修改 EAX 中值   
.text:000A7A4E mov dr7, eax ; 寄存器 7 从 EAX 中取值   
.text:000A7A51 pop eax ; 恢复 EAX   
.text:000A7A52 mov dword\_FC6CC, esp   
.text:000A7A58 mov esp, offset unk\_FBABC   
.text:000A7A5D and esp, 0FFFFFFFCh   
.text:000A7A60 xor al, al ; AL 取 0?为什么要 mov al, \[eax+edi\]?   
.text:000A7A60 ; 不明白.原始代码?   
.text:000A7A62 call sub\_4D2EB   
.text:000A7A62   
.text:000A7A67 call sub\_36AC1   
.text:000A7A67   
.text:000A7A6C xor edx, edx   
.text:000A7A6E   
.text:000A7A6E loc\_A7A6E: ; CODE XREF: sub\_A79B9+124j   
.text:000A7A6E call sub\_74916   
.text:000A7A6E   
  
As you can see from the code above, we can force Soft ICE to read at memory location \[0\] or something similar using a special value inside   
the PE header. For this binary i didn't bother calculating the exact value to read at address \[0\], that's may explain why it didn't crash   
for some people.I won't explain how to calculate this special value because it is trivial and i don't want Dark lords to use that trick  
 without a little brainstorming.   
To fix this problems, one needs to patch the value in the PE Header. The standard value for NumberOfRvaAndSizes is 0x10.Just patch this   
value in the PE Header and the Soft ICE wrecking will be gone. The OllyDBG problem as well, because it is based on BOTH fields modifications.   
You can also nullify the other field if you want.   
如你所见，我们可以令 SoftIce 读取\[0\]地址或其它特殊值。但我并不知道这时确切的数值，这可能与系统是否崩溃有关。我不想讨论怎样计算这  
一特殊值，因为它与主题无关，另外也给大家留下讨论的空间。  
你可以通过值的修改来修正这一问题。NumberOfRvaAndSizes 的一般值为 0x10。修改它。另外因为 Olly 和 SoftIce 都基于这种技术，因此你都可  
以通过改变 NumberOfRvaAndSizes 的值或归 0 来搞定。  
  
  
注：未完，有空会继续翻译，不知翻译的如何，欢迎大家提出宝贵意见，不胜感激。 

> • Section Modification: Or how to kill many tools.   
> \->Section Header Table   
> 1. item:   
> Name: CODE   
> VirtualSize: 0x00001000   
> VirtualAddress: 0x00001000   
> SizeOfRawData: 0x00001000   
> PointerToRawData: 0x00001000   
> PointerToRelocations: 0x00000000   
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xE0000020   
> (CODE, EXECUTE, READ, WRITE)   
> 2. item:   
> Name: DATA   
> VirtualSize: 0x00045000   
> VirtualAddress: 0x00002000   
> SizeOfRawData: 0x00045000   
> PointerToRawData: 0x00002000   
> PointerToRelocations: 0x00000000   
>    
>   
>     
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xC0000040   
> (INITIALIZED\_DATA, READ, WRITE)   
> 3. item:   
> Name: NicolasB   
> VirtualSize: 0x00001000   
> VirtualAddress: 0x00047000   
> SizeOfRawData: 0xEFEFADFF <--- BIG Size of section on the disk.   
> PointerToRawData: 0x00047000   
> PointerToRelocations: 0x00000000   
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xC0000040   
> (INITIALIZED\_DATA, READ, WRITE)   
> 4. item:   
> Name: .idata   
> VirtualSize: 0x00001000   
> VirtualAddress: 0x00048000   
> SizeOfRawData: 0x00001000   
> PointerToRawData: 0x00047000   
> PointerToRelocations: 0x00000000   
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xC0000040   
> (INITIALIZED\_DATA, READ, WRITE)   
> 节的修改：工具杀手。  
> \->Section Header Table （节头表）  
> 条目 1：  
> Name: CODE   
> VirtualSize: 0x00001000   
> VirtualAddress: 0x00001000   
> SizeOfRawData: 0x00001000   
> PointerToRawData: 0x00001000   
> PointerToRelocations: 0x00000000   
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xE0000020   
> (CODE, EXECUTE, READ, WRITE)   
>   
> 条目 2：  
> Name: DATA   
> VirtualSize: 0x00045000   
> VirtualAddress: 0x00002000   
> SizeOfRawData: 0x00045000   
> PointerToRawData: 0x00002000   
> PointerToRelocations: 0x00000000   
>    
>   
>     
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xC0000040   
> (INITIALIZED\_DATA, READ, WRITE)   
>   
> 条目 3：  
> Name: NicolasB   
> VirtualSize: 0x00001000   
> VirtualAddress: 0x00047000   
> SizeOfRawData: 0xEFEFADFF <--- 大容量的节  
> PointerToRawData: 0x00047000   
> PointerToRelocations: 0x00000000   
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xC0000040   
> (INITIALIZED\_DATA, READ, WRITE)   
>   
> 条目 4：  
> Name: .idata   
> VirtualSize: 0x00001000   
> VirtualAddress: 0x00048000   
> SizeOfRawData: 0x00001000   
> PointerToRawData: 0x00047000   
> PointerToRelocations: 0x00000000   
> PointerToLinenumbers: 0x00000000   
> NumberOfRelocations: 0x0000   
> NumberOfLinenumbers: 0x0000   
> Characteristics: 0xC0000040   
> (INITIALIZED\_DATA, READ, WRITE)  
>   
> From those informations, we can conclude a few things. First, the binary doesn't seem to be compressed, because the Virtual Address and Size matche   
> the Raw Offset and Size at one exception, the NicolasB section. This section has an extremly big size of raw data, which will crash a few tools and   
> make a few others very very slow.   
> 通过以上信息，我们可以推断出以下一些结论。1。二进制代码未被压缩，因为除了 NicolasB，其它节的 Virtual Address= the Raw Offset，Virtual Size= Raw Size。NicolasB 节包含了巨大的 raw 数值，这将使进行逆向的工具崩溃或受到阻碍。  
>   
> IDA will try to allocate a LOT of memory because it thinks that the section is THAT big, turning your computer into a very slow turtle ;-). Eventually, it will load the file, or run out of memory, depending of the computer you are using to do the analysis.   
> This modification will also create havoc with many tools such as Objdump, PE editor, some memory dumpers etc. It is very easy to fix this problem,  
>  you need to correct the Raw Size. If you look at the section following this special one, you will find that it starts at the very same Raw Offset.  
>  This means that the other section is actually null on the disk. You can therefore, safely replace the big value by zero.   
> 当节包含巨大数值时，IDA 会分配大量的内存空间，这将使你的计算机运行缓慢。事实上，是装载文件或是内存溢出，都取决于正在用来分析的电脑性能。  
> 另外，这种修改还会对许多工具如 Objdump,Peeditor,一些内存 Dumper 等产生严重破坏。不过修正这个问题很简单，只要修改一下 Raw Size 即可。尽管你发  
> 现某个节与其它不同，但是它们仍然是具有相同的 Raw Offset. 这就意味着，如果其它节是空的，处于安全考虑，你可以把特殊的那个节也取空。  
>   
> Protection Weakness:   
> While writing this binary, i knew people were going to patch the PE header but i didn't do any integrity checks on purpose. Originally i wanted to use  
>  the value in the PE Header as keys to decrypt a few layers of the protection, and the result would have been an unworking binary if this one had been changed.  
> I have also changed a few other things in the PE header, but nothing of real interest here. (who said Cosmetic?)   
> 保护漏洞：  
> 当我贴出这些代码时，许多人都会去对 PE 头进行修正，可我不是这样做的。最初，我想利用 PE Header 中的数值作为打开保护层的钥匙，但是一旦改变数值，  
> 它就变成了一段无效的二进制代码。另外我还试着修改其它地方，可都一无所获。  
>   
> • Junk Code   
> All along the binary, i have added junk code between real instructions, in order to make the analysis a little harder. The junk code are long blocks   
> of code that does nothing but fancy operations to disturb the analyst , especially when he choose to do a static analysis of the binary. Each block of  
>  Junk Code is different and have been generated by a personal tool. A Thrash generator which creates macros to be inserted in the code source around  
>  real instructions.   
> Here is how it looks inside a disassembler:   
> The junk code starts with a pushad (save all registers states onto the stack) and finish with a popad (restore register states).Here is the end of   
> a block of junk:   
> 垃圾代码  
> 在真正的代码中，我添加了一些垃圾代码，这是为了增加分析的难度。它们冗长无意义，但是可以阻止分析的进行，特别是静态分析时。而每一个又各  
> 不相同，由独立的工具产生。一个好的程序会把这种代码大量的运用于真正的代码之中。  
> 以下是我们在反汇编器中所看到的：  
> 垃圾代码以 pushad 开头（在堆栈中保存所有寄存器状态）并以 popad（恢复寄存器状态）。这是它们的尾部：  
>   
> Protection Weakness:   
> The thrash generator isn't perfect (at least with the options i have used here ;) and it is easy to find the start and the end of a block of junk code.   
> The junk code is bounded by pushad/popad. When i wrote this binary i was aware of this problem, but this is a perfect real life example of  
>  protection weakness. It allows Reverse Engineers to practice IDA/Ollydbg scripting. Very interesting scripts were found in the submissions.  
>  I invite you to have a look at them if you didn't know how to write one. When i wrote the binary, i already had a better version of my Thrash  
>  generator that doesn't use any pushad/popad around the blocks of useless code, but we will keep it for another challenge, if any.  
> 保护漏洞：  
> 这个程序不是完美的，你很容易找到垃圾代码的头和尾。即 pushad/popad。当我写这段代码时想过这个问题，但这确实是最典型的保护漏洞。  
> 逆向者用它来练习 IDA/Ollydbg 脚本会非常有趣。如果你不会，建议你去了解一下。当我完成这个程序后，我又写了更好的非典型漏洞的版本，  
> 这是后话了。  
>   
> • SEH - Structured Exception Handling   
> Windows SEH were used extensively in this binary. It allows one to access the context structure of the current application, and therefore,   
> access privileged registers such as Debug Registers. Those registers are used by Hardware Breakpoints (BPM). If you can access them,   
> you can also erase the hardware breakpoints.   
> SEH-结构异常链  
> SEH 被广泛地运用在二进制代码中。它允许你访问当前应用程序的 context structure 和优先级寄存器（调试器）。这些寄存器被硬件断点（BPM）  
> 所使用。因为可存取，因此这些硬件断点也可被擦除。  
>   
> • Timing Detection Through SEH   
> Here is a little detection i invented to detect debuggers. If we merge SEH (And access to context structure) with the known Timing Detection  
>  Technique, we can detect a lot of Ring 3 debuggers and Tracers. The idea is to read the Time Stamp Counter using RDTSC (number of cycles  
>  executed by the CPU basically) and then generating an Exception.  
>  通过 SEH 侦测时间  
> 这是一个侦测调试器的程序。当我在 SEH（存取 context structure）中加入时间侦测技术后，可以发现：许多 Ring 3 级别的调试器和追踪器都  
> 可以被侦测到。原理是通过 RDTSC（CPU 执行数字循环）读取了 Time Stamp 指针最终产生异常。  
>   
> In the exception handler, we can access the EAX register (previously modified by RDTSC) in the Context Structure, which contains the TSC.  
>  In the Exception Handler, we use RDTSC one more time, to get the current TSC value. Now, we can compare both TSC to see whether the program  
>  has been debugged/traced or not. If such an action has occured, the difference of cycles will be huges, thus triggering the Payload.  
>  In this binary, i just modified EIP through the context structure. The application resumes at a different location skipping mandatory   
> instructions.The application crashes eventually. It seems that on some version on Windows, it doesn't work as expected because of the   
> utilisation of the CPUID instruction, that will modify the ECX register.   
> 在异常链中，我们可以访问 Context Structure 中的 EAX 寄存器（先前被 RDTSC 修改）。同时，我们更多的使用 RDTSC 得到当前 TSC 的数值。现在，  
> 我们比较这两个 TSC，看是否程序被调试/跟踪。如果调试/跟踪，两者值会有巨大差距，导致最终被发现。虽然我修改了 context structure 中  
> EIP 的数值，使得程序跳过强制指令定位到新的位置。可是最终程序仍然崩溃。可以看出 Windows 中的一些版本，因为 CPUID 指令修改了 ECX 寄存  
> 器而并不象我们想象中的那样运行。  
>   
> The detection became less stealth because of this "bug", but it would still have been a matter of time until someone discovered it anyway.  
>  Many people wondered why i used CPUID in the program before RDTSC. The reason is that on recent CPU such as P4, there is a feature called:   
> Out of Order Execution. The CPUID is a synchronization instruction which tells to the CPU not to use Out Of Order execution, avoiding False  
>  Positives in the debugger detection. If you don't tell to the CPU not to use OOO execution, you don't know in which order the CPU is going to   
> execute your code. It can be different from your source code. Sometimes, it will create a false positive and your program will crash for no  
>  reason.   
> Here is the code of this detection:   
> E0000h is the maximum cycles difference accepted by this detection. If the number is bigger, then a debugger is most likely running and   
> debugging our application.   
> 尽管此侦测的原理有了一些进展，但仍然有很多需要去发觉。许多人奇怪为什么 CPUID 使用在 RDTSC 之前。因为现在的 CPU（如 P4）具有被称之为  
> “Out of Order Execution”的功能。CPUID 是一个同步指令，为了避免调试器侦测中的错误行为，它会禁止 CPU 使用 Out of Order 功能。即使你打  
> 开了这项功能，你也不知道在什么地方执行了你的代码，并可能与你的原代码有出入、执行错误的命令甚至使系统莫名其妙的崩溃。  
> 下面是侦测的代码：  
> E0000H 是侦测差距的上限。如果超过这个上限，调试器可能会正常运行。  
>   
> Protection Weakness:   
> I have used a fixed value for the number of cycles: E0000h. I could have (Actually i can do it with my layer generator) used a random value  
>  rather than a constant and therefore, making the scan for this constant useless. I could also have used different instructions for each   
> SEH to make the creation of a generic pattern difficult. The biggest weakness of this detection is the constant and the usage of the same  
>  instructions for every checks. It is also possible to write a Kernel Module Driver to catch every execution of RDTSC (See Intel   
> documentation for further informations) and return very similar values, thus bypassing the detection completely.  
> 保护漏洞：  
> 我使用了上限 E0000H。然而我可以使用任意数值代替这个常数。另外我还可以对每个 SEH 使用不同的指令使其产生不同的效果。最大的漏洞就  
> 是常数的设置和每个校验使用相同的指令。因此，我们可以写出一段 Kernel Module Driver，去获取 RDTSC 的执行过程和返回的数值，以此击败  
> 这种保护方式。  
>   
> • BPX Detection:   
> As we are going to use API functions, We have to protect them from beeing BPX'ed by an attacker. Rather than Using GetProcAddress to get  
>  the API address and then to check for an int 3 opcode (0xCC) in the API function code, i have used a different method. I directly access   
> the Import Table , more precisely, the Import Address Table to read the API function address and then start to search for breakpoints.   
> The int 3 opcode is 0xCC and is known by Reverse Engineers. In order to make a little less obvious, i have obfuscated the breakpoint check   
> using a "SHR" (Shift Right) instruction: 0x660 shr 3 = 0xCC ;-). The program will then check four bytes at API function entry point, looking for a breakpoint. If a breakpoint is found, i have used a  
>  funny way to crash the application. Im using RDTSC to generate a pseudo random number and i put this number onto the stack. To modify EIP,   
> i simply use the RET instruction, which will transfer us to random memory address, crashing our application. Each time a detection occurs,  
>  the address is different, thus hard to monitor. The crash occurs far from the detection code and Soft ICE's FAULT ON won't catch it either.   
> BPX 侦测：  
> 当使用 API 函数时，我们需要防止攻击者利用 BPX 作恶。一般的方法是：使用 GetProcAddress 得到 API 地址并在函数代码中检测 INT3 操作码，而我使  
> 用了另一种不同的方法：在输入表读取 API 函数地址时直接访问输入表，查找设置的断点。  
> 大家都知道，INT3 的操作码是 0xCC。为了使其更具迷惑性，我使用了“SHR”这种指令：0x660 shr 3=0xCC.程序因此在 API 函数的入口点中校验这  
> 四个字节，查找断点。一旦发现，程序崩溃。我使用 RDTSC 产生任意一个假的数值，并把此值放入堆栈。修改了 EIP 后，我使用 RET 语句引导我们到  
> 达任意一个内存地址中，从而使程序崩溃。而且每次侦测的地址都不同也无法监测。崩溃来自于侦测的代码，就连 SoftIce 的 FAULT ON 功能也无法实现。  
>   
> Protection Weakness:   
> First, the Imports aren't protected, therefore anyone can read the Imported functions from the binary. From The import table we can see   
> that printf, GetCommandLineA and ExitProcess are used. This is a weakness. A Reverse Engineer can put breakpoints on those functions, or  
>  at least, guess they are going to be used at some point. In the case of our binary, one can guess that the application is waiting for a special  
>  command line. A solution would be to load the Import Table manually.  
> 保护漏洞：  
> 首先，输入表未受保护，因此任何人都可以从代码中读取输入函数。从输入表中，我们可以发现如 printf,GetCommandLineA 和 ExitProess 语句的  
> 使用。这是一个漏洞。逆向者可以在上述地方下断点或预测将会使用的地方。以代码为例，我们可以预测到程序正在等待一项特殊的指令。而对  
> 策就是手动装载输入表。  
>   
> For this we could use a home made GetProcAddress function to browse the Export Table of the dlls we want to import functions from, and then,   
> get the address of the API function from there. A Kernel32 address is always on the stack when a binary is started, so we could have used this  
>  value to get the dll's ImageBase (Or use the PEB, SEH chaining etc..). We would have everything needed to get the address of Loadlibrary which  
>  allows us to Load ANY dll, and thus, to get the address of ANY API function. With this method, we don't need any Import Table at all.  
> 这样说来，我们可以通过 GetProcAddress 浏览 dll 的输出表函数并得到 API 地址。当代码运行时，堆栈中的 Kernel32 地址可以帮助我们得到 dll 的基址  
> （或使用 PEB，SHE 链等）。我们从装载 dll 的 Loadlibrary 的地址处得到所有 API 函数的地址，而无须借助任何输入表。  
>   
> Well actually, this isn't true. There is a mandatory thing to do to keep compatibility with all versions of Windows. We have to create a very  
>  small Import Table, with at least ONE import from Kernel32, else the binary won't run on Windows 2000. The Windows 2000 PE Loader is different   
> from the one in Windows XP. XP doesn't care whether there is any import table or not.   
> 事实上，这是不正确的。为了与各个版本的 Windows 兼容，输入表是必须的。我们需要建立一个输入表，至少要包括一个 Kernel32 中的输入，否则 2000  
> 中将无法运行。因为 2000 与 XP 的 PE 装载器是不同的，后者并不在乎是否有输入表存在。  
>   
> The small Import Table is just for compatibility issue, the real import table is encrypted and will be decrypted at runtime by the protection.  
> Then, it is just a matter of loading the Imports mimicing the Operating System. We need to put the API address in the Import Address Table  
>  (of the decrypted Import Table) manually. The Reverse Engineer has no clue about the API functions used by the binary until he gets to the  
>  part of the code that will decrypt and load the Imports.  
> 这种输入表完全是为了兼容而服务，真正的输入表在运行时被其保护所加解密。  
> 模拟一下系统装载输入表的过程。我们需要手动的把 API 地址放到 IAT 中。逆向者只有在得到已解密和装载的输入后才能了解真正的 API 函数。  
>   
> The BPX protection has a few weaknesses. I only check for four bytes at API entry point, which can be easily bypassed, if the API has many   
> instructions. One could put a breakpoint to the first instruction after the 4 bytes boundary.A Better check would use a Length Disassembler  
>  Engine (LDE) which tells us the size of the instructions. With this, we can safely scan a lot of instructions without triggering any false  
>  positive.  
> BPX 保护有一个缺点。由于指令的单一，仅仅校验 API 入口点的 4 个字节很容易被解密，只要在这个陷阱后的第一个指令处下断点即可。更好的校验  
> 是分析指令长度的 Length Disassembler Engine(LDE)技术。这项技术可以使程序正确地执行校验。  
>   
> A genuine instruction can contain the byte 0xCC and yet not beeing a breakpoint. Eg: Mov eax, 0x4010CC. The detection would trigger a   
> false positive on this instruction, because of the 0xCC inside of it. On the other hand, a LDE would tell us the size of this instruction (5 Bytes).  
>  An int 3 (breakpoint) is either one or two bytes (0xCC or 0xCD 0x03). We would therefore skip the current instruction and check the following one.  
> 正确的指令不是断点，但仍然包含 0xCC。例如：Mov eax, 0x4010CC。此处会造成错误的校验，因为含有 0xCC。而 LDE 却不同，它会校验这些指令的长度  
> （5 个字节）。而 INT3 断点只有一个或 2 个（0xCC or 0xCD 0x03）。因此我们可以跳过这种指令而校验其它。  
>   
> Also, the BPX check is only done once per API at a given location in the binary.Once we have stepped over those checks , we can put a breakpoint   
> on any API function without triggering any error. This weakness wasn't fixed on purpose because this is a common error in Protection Systems.  
> There is another kind of BPX detection that will be described in the next section   
> 另外 BPX 校验只是确定地方 API 的一次校验。因此我们可以在校验后的区域设置断点。由于这是一个普遍现象，因此这个漏洞无法修复。  
> 下一节将介绍另一种 BPX 侦测方式。 

> • The Crazy Layers   
> Here is a little more challenging protection. In order to protect the binary from beeing disassembled, i have written an Encryption   
> Layer generator, that will generate the number of layers i want. For this binary, i used 175 layers. The Layer Generator has many options.   
> Here are the options from the config file: (0 means disabled)   
> SEH=1   
> RANDOM\_LAYER\_SIZE=0   
> RANDOM\_REGISTERS=1   
> RANDOM\_ENCRYPTION=0   
> ENCRYPTED\_RETURN\_ADDRESS=1   
> TIMING\_DETECTION=1   
> RANDOM\_CONSTANT=0   
> JUNKS=0   
> PUSHAD\_POPAD=1   
> RANDOM\_ORDER=0   
> USE\_DIFFERENT\_LOOP\_CODE=0   
> RANDOM\_FIRST\_BLOCK=0   
> NUMBER=175   
> I will comment each options below:   
> SEH:   
> This tells to my layer generator to use (or not) SEH inside the layers.  
> 繁多的 Layers:    
> 这是一个具有挑战性的保护。为了防止反汇编代码，我写了一段可以产生任意数 Layer 的加密器。在代码中，我使用了 175 个这样的 Layer.加密器  
> 包含许多选项。下面是从配置文件中得到的：（0 代表禁止）  
> SEH=1   
> RANDOM\_LAYER\_SIZE=0   
> RANDOM\_REGISTERS=1   
> RANDOM\_ENCRYPTION=0   
> ENCRYPTED\_RETURN\_ADDRESS=1   
> TIMING\_DETECTION=1   
> RANDOM\_CONSTANT=0   
> JUNKS=0   
> PUSHAD\_POPAD=1   
> RANDOM\_ORDER=0   
> USE\_DIFFERENT\_LOOP\_CODE=0   
> RANDOM\_FIRST\_BLOCK=0   
> NUMBER=175   
> 下面一一解释：  
> SEH：  
> 加密器在 Layers 中使用 SEH。  
>   
> RANDOM LAYER SIZE:   
> This tells to my layer generator to use a different size for each layer. This option wasn't enabled to simplify the analysis.   
> RANDOM REGISTERS:   
> If this option is enabled, all the layers are using different registers. Some kind of "polymorphism". This option was enabled.   
> RANDOM ENCRYPTION:   
> When this option is enabled, Each layer will have a different encryption algorithm. I didn't enable this option. Therefore all  
>  the layers have a static encryption code. (Default layer)   
> ENCRYPTED RETURN ADDRESS   
> This option will encrypt the return address inside the layer. It avoids a simple patch to skip the SEH.This option was enabled   
> RANDOM LAYER SIZE:  
> 加密器对每个 Layer 使用不同的大小。此选项不能使分析简单化。  
> RANDOM REGISTERS:  
> 如果此选项打开，所有的 Layers 将会使用不同的寄存器。即呈现多态性。此选项是打开的。  
> RANDOM ENCRYPTION:  
> 当此选项打开时，每个 Layer 都具有独立的加密运算法则。我禁止了它。因此，所有的 Layers 都有同一个加密代码。  
> ENCRYPTED RETURN ADDRESS：  
> 此选项将对 Layer 中返回的地址进行加密，避免了仅仅简单的修改就绕过 SEH 情况的发生。此选项为打开。  
>   
> TIMING\_DETECTION:   
> Tells whether the layers must use Timing Detection or not. I enabled this option.   
> RANDOM\_CONSTANT:   
> The Random constant is to tell whether we want to use a static value for the timing detection or not. This option wasn't enable.  
>  All layers were using the defaut value: E0000h. Enabling this option will also modify the code that checks for the Difference  
>  between both TSC.   
> JUNKS:   
> Enable of Disable Junks in the Layers. I disabled this option because the layers are WAY biggers when it is enabled. The resulting binary  
>  is too huge and slow if you use a big number of layers.   
> PUSHAD\_POPAD:   
> This option tells the Layer Generator to use (or not) Pushad/Popad around the Junk Code. The layer generator directly use the Thrash  
>  Generator (external tool) i have programmed. I was using pushad popad in the junk code, that's why it is enabled. This option does nothing   
> if the Junks option is disabled.   
> TIMING\_DETECTION:  
> 是否 Layers 使用时间侦测技术。我打开了这个选项。  
> RANDOM\_CONSTANT:  
> 此选项表明是否在时间侦测中使用静态数值。此选项禁止，所有 Layer 使用默认值 E0000H。打开它就是修改了两个 TSC 的差距值的上限。  
> JUNKS:  
> 禁止 Layer 中的垃圾代码。因为选项打开后 Layer 将变的冗长，代码也随之变的巨大，运行将变的缓慢。  
> PUSHAD\_POPAD:  
> 此选项决定加密器是否在垃圾代码中使用 PUSHAD/POPAD 格式。我的加密器是直接使用 Thrash Generator（外国的工具）。在代码中我使用了  
> 上面的格式，因此选项是打开的。但如果 JUNKS 选项被禁止，此选项即被屏蔽。  
>   
> RANDOM\_ORDER:   
> Each layer use a table to access part of its code. If this option is enabled, Each layer has a random order of execution. I didn't enable this one on purpose.   
>    
>   
>     
> USE\_DIFFERENT\_LOOP\_CODE:   
> Each layer loops a given number of time. With this option, one can use different code to test the end of the loop. It makes it harder for the reverse engineer to find removal pattern. This option wasn't enabled. A defaut checking code was used.   
> RANDOM\_FIRST\_BLOCK:   
> This option allows one to use random value inside the first elements of the layers tables. You will see in some submissions that the static value were used to bypass the layers. I didn't enable this option to see whether someone was going to use it or not.   
> NUMBER:   
> This is the number of the layer, the generator must use. I used 175 layers in this challenge. I can generate 65000 layers in a few seconds because the generator engine is programmed in Assembly Language.   
> RANDOM\_ORDER:  
> 每个 Layer 使用表访问自身的代码。当此选项打开时，每个 Layer 执行的顺序是任意的。我禁止了这个选项。  
> USE\_DIFFERENT\_LOOP\_CODE:  
> 每个 Layer 循环特定的时间。通过此选项，我们可以使用不同的代码测试循环的结束。这使得逆向者更加难以找到切入点。此选项禁止。使用默认代码校验。  
> RANDOM\_FIRST\_BLOCK:  
> 此选项可以把任意值放入 Layer 表的第一个元素中。一些文章利用静态数值绕过 Layer。我禁止了此选项就是看看是否有人可以做到。  
> NUMBER:  
> 这是加密器必须设置的 Layer 数值。我设置了 175 个。甚至我可以在几秒之内就产生 65000 个 Layer，因为我的加密器核心是由汇编语言编制的。  
>   
> Presentation of the encryption layers:   
> Layer Selector   
> xor esi,esi ; ESI = 0   
> mad\_loop175\_1: ; Loop label   
> inc esi ; ESI++   
> mov edi,dword ptr \[ebp+(esi\*4)+EIPtable175\_1\] ; Grab block address   
> mov ebx,dword ptr \[ebp+(esi\*4)+RETable175\_1\] ; Grab "Encrypted"   
> ; Return address   
> Add ebx, \[ebp+\_startloader\] ; Add Base.   
> push ebx ; Save Return Address   
> ; from the stack   
> Call tricky\_call175\_1 ; Fake call   
> db 0EBh,01,0E8h ; Some junk crap   
> fake\_ret175\_1: ; fake return address label.   
> Add edi, \[ebp+\_startloader\] ; Add EDI Base. EDI now   
> ; contains address of a block ; inside the layer.   
> jmp edi ; Execute that block.   
> return\_addy175\_1:   
> cmp esi, 4 ; When we get back from the   
> ;block, we check whether we   
> ;have done every blocks.   
> jnz mad\_loop175\_1 ; if we didn't, loop!   
> bpxcheck175\_1: ; Label used for BPX check.   
> jmp @layer175\_1   
> tricky\_call175\_1:   
> pop ebx ; Ret address is in EBX   
> jmp fake\_ret175\_1 ; Jmp to fake return address.   
> @layer175\_1: ; end of the layer.   
>    
>   
>     
> This is the main part of a layer. This part loops through the layer blocks using some obfuscated ways. It prepares the stack with  
>  return addresses, and fake a call. If you step over with your debugger on this call, the binary won't break and it will run. If you  
>  were debugging a malware, you would get infected. And if you were analysing the binary, you would need to restart from scatch.  
>  (Except if you have dumped your position regulary).  
> 加密 Layer 的陈述：  
> Layer Selector  
> xor esi,esi ; ESI = 0   
> mad\_loop175\_1: ; 循环标签  
> inc esi ; ESI++   
> mov edi,dword ptr \[ebp+(esi\*4)+EIPtable175\_1\] ;获取 block 地址   
> mov ebx,dword ptr \[ebp+(esi\*4)+RETable175\_1\] ; 获取"Encrypted" 返回地址  
> Add ebx, \[ebp+\_startloader\] ; 添加基址  
> push ebx ; 从堆栈中保存返回地址   
> Call tricky\_call175\_1 ;伪 call   
> db 0EBh,01,0E8h ; 垃圾代码  
> fake\_ret175\_1: ;伪返回地址标签   
> Add edi, \[ebp+\_startloader\] ; 添加 EDI 基址.EDI  
> ;包含 Layer 中的 block 的地址  
> jmp edi ;执行 block.   
> return\_addy175\_1:   
> cmp esi, 4 ; 当我们从 block 返回时，校验是否已经完成了全部的 block   
> jnz mad\_loop175\_1 ; 没有则循环  
> bpxcheck175\_1: ;用于 BPX 校验的标签.   
> jmp @layer175\_1   
> tricky\_call175\_1:   
> pop ebx ; EBX 中包含 ret 地址   
> jmp fake\_ret175\_1 ;跳向伪返回地址.   
> @layer175\_1: ;layer 结束.   
> 上述代码是 Layer 的主体部分。通过某种方式在 Layer 中循环执行 block。同时还包括返回地址和伪 CALL。如果调试时你不过这个 CALL，  
> 代码不会被执行而程序将会运行。假使是一个病毒程序，你的系统将会被感染。如果你不幸的在分析代码时终止，你将不得不重新来过。  
>   
> Layers Blocks   
> include obfuscation/junk198.inc ; I have added a few junks macro   
> ; manually in order to add a   
> ; little fun :)   
> dec\_loader175\_1: ; Decrypt label   
> xor byte ptr \[edx\],cl ; Defaut options were used.   
> ; Very simple encryption.   
> inc edx ; Code to decrypt++   
> dec ecx ; Loop index--   
> test ecx, ecx ; is ECX = 0 ?   
> jnz dec\_loader175\_1 ; no :( therefore we continue   
> ; to decrypt.   
> ; This encryption can be different for each layer if you enable the option in the layer Generator.   
> lea edx , \[ebp+bpxcheck175\_1\] ; Grab address of BPX check.   
> cmp byte ptr \[edx\],0CCh ; Any break point ?   
> jnz return175\_1 ; no. Good boy.   
> rdtsc ; Ah.. he did put a bpx..   
> ; EAX = random value   
> push eax ; push eax on stack   
> ret ; Return to it :) Crash the   
> ; poor guy.   
> return175\_1: ; on return block   
> include obfuscation/junk199.inc ; a few junk   
> include obfuscation/junk19A.inc ; ditto.   
> SEHBLOCK 66137317 28513829 ; SEH block macro with   
> ; keys in parameters.   
> ret ; return   
> inst175\_2\_1: ; another block of code   
> add dword ptr \[esp\], 41952561 ; fix return address and return.   
> ret   
> inst175\_3\_1: ; Another block.   
> mov ecx, (offset \_end174\_1- @layer175\_1) ; Get Size of layer   
> add dword ptr \[esp\], 13007360 ; Fix return address and return   
> ret   
> inst175\_1\_1: ; Another block   
> lea edx, \[ebp+@layer175\_1\] ; Get Layer address   
> add dword ptr \[esp\], 30560857 ; Fix the return address   
> ; and return.   
> ret   
> EIPtable175\_1 dd 000DEADh, (offset inst175\_1\_1 - offset startloader), (offset inst175\_2\_1 - offset startloader), (offset inst175\_3\_1 - offset startloader), (offset \_end174\_1 - offset startloader)   
>    
>   
>     
> ; This is a table of offset used to redirect the code.   
> RETable175\_1 dd 0031000h, (offset return\_addy175\_1 - offset startloader - 30560857) , (offset return\_addy175\_1 - offset startloader - 41952561),(offset return\_addy175\_1 - offset startloader - 13007360),(offset return\_addy175\_1 - offset startloader - 37623488)   
> ; This is a table of return address with a little "encryption".   
> ; You can notice the first member of the tables : DEADh and 31000h. Those values are constants and can be random using the RANDOM\_FIRST\_BLOCK   
> ; option in the layer generator.   
> The layer presented above has been generated by the little Layer generator Engine i have programmed. I have added comments for the readers.   
>   
> Layers Blocks  
> 除了 obfuscation/junk198.inc，我还添加了一些 junks 宏。  
> dec\_loader175\_1: ; 解密标签  
> xor byte ptr \[edx\],cl ; 使用默认选项   
> ; 简单的加密   
> inc edx ;代码解密++   
> dec ecx ;循环索引  
> test ecx, ecx ; 校验 ECX 是否等于 0。  
> jnz dec\_loader175\_1 ;非也:( 继续解密).   
> 如果打开加密器中的选项，每个 Layer 的加密将会不同。  
> lea edx , \[ebp+bpxcheck175\_1\] ; 获取 BPX 校验地址.   
> cmp byte ptr \[edx\],0CCh ;有断点吗?   
> jnz return175\_1 ; 不错。好孩子。  
> rdtsc ;它放了一个 bpx..   
> ; EAX =任意值  
> push eax ;eax 入栈  
> ret ;返回 :)崩了你：-）   
> return175\_1: ; on return block   
> include obfuscation/junk199.inc ;垃圾代码   
> include obfuscation/junk19A.inc ;同上.   
> SEHBLOCK 66137317 28513829 ; 以 key 作为参数的 SEH block 宏   
> ret ;返回  
> inst175\_2\_1: ;又一个代码 block   
> add dword ptr \[esp\], 41952561 ;修正返回地址并返回  
> inst175\_3\_1: ;.   
> mov ecx, (offset \_end174\_1- @layer175\_1) ;大小  
> add dword ptr \[esp\], 13007360 ;   
> inst175\_1\_1: ; 另一个 block   
> lea edx, \[ebp+@layer175\_1\] ; 得到 layer 地址   
> add dword ptr \[esp\], 30560857 ; 修正返回地址并返回   
> ret   
> EIPtable175\_1 dd 000DEADh, (offset inst175\_1\_1 - offset startloader), (offset inst175\_2\_1 - offset startloader),   
> (offset inst175\_3\_1 - offset startloader), (offset \_end174\_1 - offset startloader)   
> 这是一张用来重定向代码的偏移表。  
> RETable175\_1 dd 0031000h, (offset return\_addy175\_1 - offset startloader - 30560857) , (offset return\_addy175\_1 - offset startloader - 41952561),  
> (offset return\_addy175\_1 - offset startloader - 13007360),(offset return\_addy175\_1 - offset startloader - 37623488)   
> 这是一张经过一定“加密”的返回地址表。  
> 注意表中的第一个成员：DDH 和 31000H。它们是常数并可以在加密器的 RANDOM\_FIRST\_BLOCK 选项中任意取值。  
>   
> Protection Weakness:   
> Those layers have a few weaknesses. You can use BPM (Hardware Break Point) on the next layer once you have passed the SEH that is going   
> to clear the debug registers. Another weakness is the static size of the layer. Using this information, one can pass the layers rather  
>  quickly with a few Soft ICE macros for instance. I didn't turn the random size option on, on purpose to allow such attacks.  
> 保护漏洞：  
> 此 Layer 包含了一些漏洞。其中一个是：只要通过了清空寄存器的 SEH，你就可以在下个 Layer 上设置硬件断点。另一个是由于 Layer 的固定大小  
> 。只要运用一些如 SoftICE 宏的插件就可以很快地通过 Layer。我禁止了 RANDOM SIZE 选项，以允许这种攻击。  
>   
> Those layers always use the same encryption algo, which can allow one to write scripts to decrypt the binary. And as you can read in a   
> few submissions, some people did it. I did put this weakness on purpose as well. In a challenge i had done in the past, i had used random  
>  encryption for each layers, this time i choose not to use it. It is possible to bypass the 175 layers in a few seconds easily as well  
>  using a live approach. As we know wich API functions are going to be used, we can set a break point after the BPX checks have occured.  
> Another possibility is to create a little utility that will PATCH the system dll in memory (each application has a copy of the dll)   
> and to redirect them to a place that you contol. This way you can put breakpoints without triggering any Detection code.  
> 另外 Layer 相同加密方式的使用，使得有人可以写出解密所需的脚本文件。一些人就做到了。我也故意放过了这个漏洞。以前，我任意加密  
> 每个 Layer，这次我没有。你可以利用现成的脚本文件很容易的通过这 175 个 Layer。你可以在 BPX 校验后对 API 函数设置断点。另一种可能  
> 就是对内存中的系统 DLL（每个应用程序都用到的）进行修改，使它们重定向到你所控制的范围。这种方法可以使设置的断点不触发任何  
> 的侦测代码而获得成功。  
>   
> Talking of patching the Windows dll files, it is possible to patch ntdll to avoid the Debug Registers access in the context structure,   
> by hooking the Exception Handling Mechanism of Windows. This allows one to put Hardware Breakpoints anywhere without ever having problems,  
>  never seeing his debug breakpoints beeing erased etc. The cool thing is you don't even need a Kernel Mode Driver to do that. I leave this   
> as an exercice for interested people.  
> 说到这里，NTDLL 的修改可以避免寄存器通过挂起系统的异常机制（hooking the exception handling mechanism of windows）访问 CONTEXT STRUCTURE。  
> 因此设置断点将会很顺利，也不用担心被擦除。你甚至连 Kernel Mode Driver 都可弃之一边。留下这个供大家练习。 

> • Virtual Machine   
> The final protection of the binary is a complete Virtual Machine i wrote for the challenge. I have designed a Virtual  
>  CPU that will interpret my own Assembly language. The Virtual Machine is quite simple to understand and isn't very complex.   
> Virtual Machines seem to be a new trend in protection systems, so i thought it could be a good thing to write one for such a  
>  challenge. The instruction encoding is very trivial, and could have been a lot harder to understand. The first Version i  
>  had in mind was a lot more complex. I wanted not only to have a pseudo language, but also to program the instructions handlers   
> 虚拟设备  
> 最后一种代码保护是我为这次挑战而写的虚拟设备。我设计了一个用来解释自身汇编语言的虚拟 CPU。这种保护很容易理解，  
> 并不复杂。  
> 虚拟设备在各种保护方式中是新兴事物，因此我觉得很有必要在我的挑战中写出来。指令编码既微不足道又难于理解。因此我的  
> 首个版本是十分复杂的。不仅是伪语言，我还需要写出指令的操作过程。  
>   
> emulating real x86 instructions. Each handler would be a few hundred instructions long and a lot harder to analyse.   
> A small program has been written with this Virtual Machine Assembly language, and it was used to authenticate the user   
> running the binary.   
> Read next part for further informations   
> 如果仿效 x86 指令，每个操作将包含上百的指令且很难分析。  
> 有一种用虚拟设备汇编语言写成的小程序，可以被用来鉴别作者运行的代码。  
> 阅读下面的部分获取进一步的信息  
>   
> 3. Something uncommon has been used to protect the code from beeing reverse engineered, can you identificate what it  
>  is and how it works?   
>   
> Even though, a few protection systems are using some kind of Virtual Machines, those aren't very common. Especially  
>  in Malwares and other exploits.   
> 3.一些反逆向的特殊保护方式，你能认出它们并知道工作原理吗？  
> 虽然，一些程序使用虚拟设备保护自身，但是这种方式仍然不普遍。特别是在病毒和其它开发中。  
>   
> Virtual CPU description and Inner working:   
> Registers:   
> REGISTERS STRUC   
> R0\_ dd ? ; 000   
> R1\_ dd ? ; 001   
> R2\_ dd ? ; 002   
> COUNTER\_ dd ? ; 003   
> EIP\_ dd ? ; 004 -> reserved   
> STATE\_ dd ? ; 005   
> REGISTERS ENDS   
> This is the original structure from my code source. Every registers is a DWORD. Some registers weren't used because  
>  they are reserved for future version of the Virtual Machine. One can read "EIP\_". I planned to add another information  
>  per instruction, but i didn't do it, because i didn't want it to be too complex. I will add the ability to change the  
>  Instruction pointer for any instruction. The result will be a completely mad code flow. The instruction order in   
> the file will have nothing to do with the real execution flow.   
> The STATE Register is some kind of mini Eflags. This register changes depending of other instructions.   
> The COUNTER Register is used for loop instructions. Similar to the ECX register when we use the LOOP instruction.   
> regs REGISTERS <>   
> R0 equ 000b   
> R1 equ 001b   
> R2 equ 010b   
> COUNTER equ 011b   
> EIP equ 100b   
> STATE equ 101b   
> 虚拟 CPU 描述和内部原理：  
> 寄存器：  
> REGISTERS STRUC   
> R0\_ dd ? ; 000   
> R1\_ dd ? ; 001   
> R2\_ dd ? ; 002   
> COUNTER\_ dd ? ; 003   
> EIP\_ dd ? ; 004 ->保留   
> STATE\_ dd ? ; 005   
> REGISTERS ENDS   
> 这是我源代码的原始结构。每个寄存器都是双字。有些寄存器没有使用，因为它们是为将来新的虚拟设备保留的。就  
> 是“EIP\_”。我打算每个指令中添加其它的信息，但是我没有这样做，因为我不想程序过于复杂。但我会修改所有指  
> 令的指示器。这使得代码流程变的十分疯狂。文件中指令顺序将不会影响真正流程的执行。  
>   
> 状态（STATE）寄存器是最小的 E Flags。此寄存器的修改依赖与其它的指令。  
>   
> 计数（COUNTER）寄存器在循环指令中使用。循环中常用的是 ECX 寄存器。  
>   
> regs REGISTERS <>   
> R0 equ 000b   
> R1 equ 001b   
> R2 equ 010b   
> COUNTER equ 011b   
> EIP equ 100b   
> STATE equ 101b  
>   
> Here are a few other definitions used in my program.I started to represent the registers in binary because i  
>  wanted to do complex opcode decoding. I will do that for another version ;)   
> Registers Initialization:   
> mov dword ptr \[regs.R0\_\],"livE" ; Registers are initialized with a   
> ; Slayer Song Title.   
> ..   
> mov dword ptr \[regs.R1\_\],"saH " ; Evil Has No Boundaries!   
> ..   
> mov dword ptr \[regs.R2\_\]," oN "   
> ..   
> mov dword ptr \[regs.COUNTER\_\],"nuoB"   
> ..   
> mov dword ptr \[regs.EIP\_\],"irad"   
> ..   
> mov dword ptr \[regs.STATE\_\],"! se"   
> At the start of the VM, i first begin to initialise my own registers with the song's title of a thrash metal band.  
>  This title was selected because i planned to do real evil things with the Virtual Machine. It isn't as hard as the   
> initial version, but still evil enough to keep that funny string ;-).   
> There is about 34 Instructions in the Virtual Machine. (I count instructions having different utilization as unique)   
> I will present a few instruction handlers to explain the inner working of the Virtual Machine,  
>  but not every instruction will be presented here.   
> 我赋予了程序一些新的定义。我在二进制代码中描述寄存器，那是因为我想对操作码进行完全的解码。这将在下个版本中实现。  
> 寄存器初始化：  
> mov dword ptr \[regs.R0\_\],"livE" ;寄存器以曲名初始化   
> ..   
> mov dword ptr \[regs.R1\_\],"saH " ; 恶到极点了  
> ..   
> mov dword ptr \[regs.R2\_\]," oN "   
> ..   
> mov dword ptr \[regs.COUNTER\_\],"nuoB"   
> ..   
> mov dword ptr \[regs.EIP\_\],"irad"   
> ..   
> mov dword ptr \[regs.STATE\_\],"! se"   
> 在 VM 的开头，我使用一个金属打击乐队的曲名对我的寄存器初始化。选择它是因为我将会用虚拟设备“作恶”。  
> 尽管没有第一个版本复杂，但是仍然差不了多少。  
> 虚拟设备包含 34 种指令。（根据各个指令独特的作用区分）  
> 我会通过一些指令操作来解释虚拟设备的工作原理，但不是全部（指令）。  
>   
> Pcode Fetcher:   
> The first thing the Virtual Machine does after the Register Init is to Fetch the Pcode entry point and jmp to the first Pcode handler.   
> movzx eax, byte ptr \[esi\] ; ESI is Pcode Entry Point. This code   
> ; gets the first instruction Prefix.   
> mov edi, dword ptr \[eax\*4+poffset\] ; It uses it with the offset table to   
> ; find the Pcode family it has to   
> ; execute.   
> movzx eax, byte ptr \[esi+1\] ; get second byte, use it as an   
> ; index into last table.   
> ; The VM now knows what instruction it   
> ; has to emulate and goes to it.   
> JMPNEXT ; Emulate a jmp dword ptr \[eax\*4+edi\]   
> ; with Exception Handling and   
> ; Context Manipulation.   
> ; Jmp to the next Pcode   
> ; instruction handler   
> 获取 Pcode:  
> 虚拟设备的寄存器初始化后的第一件事就是获取 Pcode 的入口点并来到首个 Pcode 处理中。  
> movzx eax, byte ptr \[esi\] ; ESI 是 Pcode 入口点.此代码得到首个指令名称   
> mov edi, dword ptr \[eax\*4+poffset\] ; 使用偏移表找到 Pcode 执行什么  
> movzx eax, byte ptr \[esi+1\] ;得到第二个字节，作为最后表的索引  
> VM 得到仿效的指令并执行  
> 跳向下个; 使用异常操作和 Context 仿效 jmp dword ptr \[eax\*4+edi\]   
> ;跳向下个 Pcode 指令操作 

> Examples of Instructions implemented inside the Virtual Machine:   
> Before i start with those examples, i would like to say that a few instructions present in the Virtual Machine weren't   
> used and were left as decoy.Three of them are using Self modifying code. People are reporting that they don't work, but  
>  they should. The off by one difference is because the opcode is beeing called from other instruction handlers.    
> 虚拟设备指令实现的实例：  
> 在我开始实例之前，我想讲述一下一些虚拟设备中未被使用，并被留下作为一种陷阱的指令。其中的三个使用了自校验代码。  
> 人们认为它们是不起作用的，但事实正好相反。这是因为操作码是从其它指令操作中被调用的。  
>   
> Two instructions are modifying one instruction on the fly as they need to execute a particular piece of code. They the  
> n restore the instruction state. I am too lazy to check whether those instructions are really bugged or if they didn't  
>  use the good parameters. One of the \_unused\_ instruction HAS a bug, and i am glad some people noticed it. The instruction  
>  isn't used therefore, it is just a decoy instruction.   
> 当需要执行特殊代码块时，两个指令将修改不工作的指令。接着恢复这个指令的状态。我没有检查是否这些指令真的有漏洞  
> 或没有使用正确的参数。不过一个\_unused\_ 指令的漏洞被人们发现了。因为这个指令未被使用，因此它只是一个陷阱指令。  
>   
> The instruction is supposed to be a Virtual BSWAP, but it doesn't save the result of the swaping. Another unused  
>  instruction is the INT 3. This instruction allows one to put breakpoint in his Pcode program and trace with his  
>  debugger from that instruction. I left this instruction in the final Virtual Machine and im glad some people  
>  found it and abused it!.   
> 这个指令被认为是一个虚拟 BSWAP，但是它不保存交换的结果。另一不使用的指令是 INT3。这个指令允许你在 Pcode  
> 程序中设置断点并使用调试器从此指令跟踪。我在最后的虚拟设备中留下这个指令，并且高兴的看到人们发现并废弃了它。  
>   
> STOPVM   
> The first instruction i will present here is a very simple one. It tells to the Virtual Machine to stops and   
> the program will get back to normal x86 assembly program.   
> 第一个陈述的指令是一个很简单的指令。它指示虚拟设备终止并且程序将回到一般的 x86 汇编程序中  
> @STOPVM:   
> pop dword ptr fs:\[0\] ; Im using SEH to jmp from handlers to   
> ;handlers in the VM. 在 VM 中，我使用 SEH 从一个操作跳到另一个操作。  
> add esp,4 ; Therefore i need to remove the handler   
> ; installed before i do anything. 因此我需要在开始之前删除安装操作  
> popad ; i restore the registers.. 恢复寄存器  
> ..   
> push dword ptr \[Pret\] ; Put the Return Address (to get out of   
> ; the VM) on the stack. 把返回地址放入堆栈中（从 VM 中退出）  
> ..   
> xor \[esp\],'HAX0 '; Decrypt it with a funny string: （用有趣的字符串进行解密）  
> ;HAXO(R)   
> ..   
> ret ; Get out of the VM. 退出 VM。  
>   
> This instruction is not using any bytecode fetcher because it doesn't need to jmp to another handler.  
>  I will now present a real instruction. A Virtual PUSH:   
> 此指令没有使用获取任何字节代码，因为它不需要跳到另一个操作中。现在我将讲述真正的指令。一个虚拟的入栈：  
> LOAD   
> @Load:   
> pop dword ptr fs:\[0\]   
> add esp,4   
> popad   
> ; Same as every handler, remove SEH and restore registers. 与每个操作一样，删除 SHE 恢复寄存器。  
> mov eax,dword ptr \[esi+2\] ; Get into EAX the first operand   
> ; of the instruction.  进入 EAX，指令的首个操作数  
> xor eax,37195411h ; Decrypt it. 解码  
> push eax ; Push it onto the stack. 压入堆栈  
> mov eax,0FFFFFF3Fh ; EAX = FFFFFF3Fh   
> not eax ; EAX = not(EAX) = C0h   
> shr eax,5 ; EAX = EAX shr 5 = 6 :   
> ; This is the instruction length 这是指令长度  
> lea esi, \[esi+eax\] ; ESI = Instruction Pointer.指令指针   
> ; Displace the Instruction Pointer 显示指令指针  
> ; 6 bytes further. 6+字节  
> movzx eax, byte ptr \[esi\] ; ESI now points to the new   
> ; instruction to be executed. ESI 指向将执行的新的指令  
> mov edi, dword ptr \[eax\*4+poffset\] ; It uses it with the offset   
> ; table to find the Pcode family 使用偏移表找到需要执行的 Pcode 系  
> ;it has to execute.   
> movzx eax, byte ptr \[esi+1\] ; get second byte, use it as an   
> ; index into last table. 得到第二个字节，作为最后表的索引  
> ; The VM now knows what instruction it    
>   
> ; has to emulate and goes to it. 现在 VM 得到仿效什么指令并执行  
> JMPNEXT ; Emulate a jmp dword ptr \[eax\*4+edi\]; with Exception Handling and   
> ; Context Manipulation. 跳向下个。使用异常操作和 context 仿效 jmp dword ptr \[eax\*4+edi\]  
>   
> ; Jmp to the next Pcode 跳向下个 Pcode 指令操作  
> ; instruction handler   
>   
> As you can see from this little handler, the instruction is 6 bytes long. It takes only one parameter and it  
>  is placed 2 bytes after the start of the instruction. (ESI+2). The parameter is encrypted with 37195411h.  
>  The decrypted parameter is pushed on the stack and then the Virtual Machine calls the next instruction.  
> 从这些操作可以看出，这个指令长度为 6 个字节。使用了一个参数并在起始指令上+2 字节（ESI+2）。此参数用  
> 37195411H 加密。加密参数压入栈，接着虚拟设备调用下个指令。  
>   
> From this, we can say that this instruction is a push. since push is already a x86 instruction,  
>  i named my virtual push : LOAD.   
> One can use it like this: "LOAD number"   
> 因此，我们可以认为这个指令是个 PUSH，因为 PUSH 已经诗歌 x86 指令，因此我命名我的虚拟压栈为 LOAD。 

> VMXOR   
> @VMXORDISPATCHER:   
> pop dword ptr fs:\[0\]   
> add esp,4   
> popad   
> ; Same as every handler, remove SEH and restore registers. 与每个操作一样，删除 SEH 并恢复寄存器  
> movzx eax, byte ptr \[esi+2\] ; Get the Index Register to acces   
> ; the Virtual CPU registers. 得到索引寄存器访问虚拟 CPU 寄存器  
> mov eax, dword ptr \[regs+eax\*4\] ; edi = Register value to know   
> ; which register is going to be EDI=寄存器值以得到与哪个寄存器有关  
> ; concerned (RO, R1 , R2)   
> ; EAX = value used by the XOR. EAX=XOR 使用的值  
> movzx ecx, byte ptr \[esi+3\] ; ECX = type of XOR. ECX=XOR 的类型  
> ; Byte ptr ? Word Ptr ? or   
> ; Dword Ptr..   
> jmp dword ptr \[xortable+ecx\*4\] ; Jmp to the good handler 跳向正确操作  
> ; accordingly.   
> @VMXORBPTR:   
> movzx ecx, byte ptr \[esi+4\] ; Get Index Register for the   
> ; destination. 得到目标索引寄存器  
> mov ecx, dword ptr \[regs+ecx\*4\] ; edi = Register value to know   
> ; which register is going to be EDI=寄存器值以得到哪个寄存器被使用  
> ; used (RO, R1 , R2)   
> xor byte ptr \[ecx\],al ; XOR BYTE PTR   
> add esi,5 ; Instruction Length is 5 指令长度 5  
> movzx eax, byte ptr \[esi\] ; ESI now points to the new   
> ; instruction to be executed. ESI 指向执行的新指令  
> mov edi, dword ptr \[eax\*4+poffset\] ; It uses it with the offset   
> ; table to find the Pcode family 使用偏移表找到执行的 Pcode 系  
> ; it has to execute.    
>   
> movzx eax, byte ptr \[esi+1\] ; get second byte, use it as an   
> ; index into last table. 得到第二个字节，使用它作为最后表的索引  
> ; The VM now knows what   
> ; instruction it has to emulate   
> ; and goes to it. VM 得到仿效哪个指令并执行  
> JMPNEXT ; Emulate a jmp dword ptr   
> ; \[eax\*4+edi\] with Exception   
> ; Handling and Context   
> ; Manipulation. 使用异常造作和 CONTEXT 仿效 jmp dword ptr   
> ; \[eax\*4+edi\]  
> ; Jmp to the next Pcode 跳向下个 Pcode 指令操作  
> ; instruction handler   
> @VMXORWPTR:   
> movzx ecx, byte ptr \[esi+4\] ; Get Index Register for the   
> ; destination 得到目标索引寄存器  
> mov ecx, dword ptr \[regs+ecx\*4\] ; edi = Register value to know   
> ; which register is going to be EDI=寄存器值以得到使用哪个寄存器  
> ; used (RO, R1 , R2)   
> xor word ptr \[ecx\],ax ; XOR WORD PTR   
> add esi,5 ; Instruction Length is 5 指令长度 5  
> movzx eax, byte ptr \[esi\] ; ESI now points to the new instruction to be executed. ESI 指向执行的新指令  
> mov edi, dword ptr \[eax\*4+poffset\] ; It uses it with the offset   
> ; table to find the Pcode family   
> ; it has to execute. 使用偏移表找到执行的 Pcode 系  
> movzx eax, byte ptr \[esi+1\] ; get second byte, use it as an   
> ; index into last table. 得到第二个字节，使用它作为最后表的索引  
> ; The VM now knows what VM 得到仿效什么指令并执行  
> ; instruction it has to emulate   
> ; and goes to it.   
> JMPNEXT ; Emulate a jmp dword ptr   
> ; \[eax\*4+edi\] with Exception   
> ; Handling and Context   
> ; Manipulation. 使用异常操作和 CONTEXT 仿效 jmp dword ptr   
> ; \[eax\*4+edi\]  
> ; Jmp to the next Pcode 跳向 Pcode 指令操作  
> ; instruction handler   
> @VMXORDPTR:   
> movzx ecx, byte ptr \[esi+4\] ; Get Index Register for the   
> ; destination 得到目标索引寄存器  
> mov ecx, dword ptr \[regs+ecx\*4\] ; edi = Register value to know   
> ; wich register is going to be   
> ; used (RO, R1 , R2) EDI=寄存器值以得到哪个寄存器被使用  
> xor dword ptr \[ecx\],eax ; XOR DWORD PTR   
> add esi,5 ; Instruction Length is 5 指令长度 5  
> movzx eax, byte ptr \[esi\] ; ESI now points to the new   
> ; instruction to be executed. ESI 指向执行的新指令  
> mov edi, dword ptr \[eax\*4+poffset\] ; It uses it with the offset   
> ; table to find the Pcode family   
> ; it has to execute. 使用偏移表找到执行的 Pcode 系   
>   
> movzx eax, byte ptr \[esi+1\] ; get second byte, use it as an   
> ; index into last table. 得到第二个字节，使用它作为最后表的索引  
> ; The VM now knows what VM 得到仿效哪个指令并执行  
> ; instruction it has to emulate   
> ; and goes to it.   
> JMPNEXT ; Emulate a jmp dword ptr   
> ; \[eax\*4+edi\] with Exception   
> ; Handling and Context   
> ; Manipulation. 使用异常操作和 CONTEXT 仿效 jmp dword ptr   
> ; \[eax\*4+edi\]  
> ; Jmp to the next Pcode 跳向 Pcode 指令操作  
> ; instruction handler   
> From this piece of code we can learn many things. The XOR instructions are coded with 5 Bytes. It has two parameters.  
> One register has the value used to do the XOR and One Register has a pointer to the location to be xored.It also have   
> a byte saying whether it is a XOR BYTE PTR, a XOR WORD PTR or a XOR DWORD PTR.   
> 上述代码中，我们可以学到许多东西。XOR 指令长度 5 个字节。含有 2 个参数。一个寄存器做 XOR BYTE PTR，另一个指向 XOR  
> 的地址。另外还有一个检验是 XOR BYTE PTR，XOR WORD PTR 还是 XOR DWORD PTR 的字节。  
> This instruction handler is therefore handling Virtual XOR instruction.   
> 这个指令操作是处理虚拟 XOR 指令的。  
> How were the virtual instruction used to create a program ?   
> I will now show how i did to create virtual instruction, because the x86 assembler doesn't know them and will never  
>  compile a LOAD or a VMXOR.To do so, i used a very simple way: MACRO. For each instruction, a corresponding macro has   
> been created, and is used to encode the instruction for me. This way i can write a program with my Assembly mnemonics  
>  without caring of the opcodes representation.I will now show the 3 Macros used for the examples Virtual Instruction   
> descrived above   
> 虚拟指令怎样创建一个程序？  
> 现在我要解释我是怎样创建虚拟指令的，因为 x86 汇编程序不知道它们，也不能编译 LOAD 或 VMXOR，因此我运用简单的宏就  
> 做到了。对于每个指令，创建相应的宏，并使用它来对指令进行加密。通过这种方式，我不必在乎操作码的参与，只要使  
> 用汇编助记忆码就写成了程序。下面我帖出用于下面虚拟指令实例的 3 个宏：  
>   
> STOPVM macro   
> db 02,00   
> endm   
> This is the macro for the STOPVM instruction. Usage: STOPVM 用于 STOPVM 的宏  
> Load macro x   
> db 00,00   
> dd x xor 37195411h   
> endm   
> This is the macro for the LOAD instruction. Usage: LOAD x 用于 LOAD 指令的宏  
> VMXOR macro reg0,kind,reg1   
> db 01,03,reg0,kind,reg1   
> endm   
> This is the macro for the VMXOR instruction. Usage: VMXOR Rx xPTR Rx 用于 VMXOR 指令的宏   
>   
> P-code Program used in this challenge:   
> The P-code is my own assembly language, thus IDA doesn't know anything about it. Here is how it looks under a disassembler:   
> Ok, it doesn't look so good. So now, here is the complete program (copy pasted from my source) i have written with   
> my OWN assembly language. Cool isn't it ? :-)   
> 挑战中使用的 Pcode 程序：  
> Pcode 是使用我自己的汇编语言，因此 IDA 不能获取任何信息。下面是反汇编器中的样子。  
> 哦，它看起来不是很舒服。下面的是完全用我的汇编语言编写的程序。是不是很酷？  
> CODE@:   
> Pcode1:   
> MOVE pcrypt COUNTER   
> LOADPTR startpcodecrypted   
> RestoreREG R0   
> MOVE 'S' R2   
> decryptpcode:   
> VMXOR R2 BPTR R0   
> INCR R0   
> DECR COUNTER   
> BNZ decryptpcode    
>   
> startpcodecrypted:   
> MOVE 05CC80E31h R1   
> APICALL GetCommandLineA   
> SCANB " " 255h   
> BZ youwishdude   
> DLOAD R0 R2   
> ADDREG R2 01D9BDC45h   
> ADDREG R1 74519745h   
> SUBREG R2 0AD45DFE2h   
> ADDREG R1 0DEADBEEFh   
> ADDREG R2 "hell"   
> SUBREG R1 17854165h   
> SUBREG R2 "Awai"   
> ADDREG R1 "show"   
> ADDREG R2 "its "   
> SUBREG R1 " no "   
> ADDREG R2 "driv"   
> ADDREG R1 "merc"   
> SUBREG R2 "nuts"   
> SUBREG R1 "y!!!"   
> SUBREG R2 "eh?!"   
> ANDREG R2 0DFFFFFFFh   
> LOADREG R2   
> LOADREG R1   
> CMPQ firstcheckdone   
> CLEAR COUNTER   
> BZ youwishdude   
> firstcheckdone:   
> INCR R0   
> ADDREG R0 2   
> INCR R0   
> WLOAD R0 R1   
> LOADREG R1   
> RESTOREREG R2   
> LOADREG R0   
> LOADPTR tricky-98547h   
> RESTOREREG R0   
> ADDREG R0 98548h   
> DECR R0   
> VMXOR R2 WPTR R0   
> RESTOREREG R0   
> BLOAD R0 R2   
> ADDREG R0 2   
> BLOAD R0 R1   
> RADD R2 R1   
> VMCALL sub\_check\_routine   
> tricky:   
> ENCRYPTED CLEAR COUNTER ; This one get patched at run time! 加密清除指针在运行时修改   
>   
> cracked:   
> LOADREG COUNTER   
> LOADPTR congrats   
> APICALL printf   
> CleanStack 8   
> BR outout   
> youwishdude:   
> LOAD 0   
> LOADPTR notgood   
> APICALL printf   
> CleanStack 8   
> outout:   
> STOPVM   
> sub\_check\_routine:   
> MOVE 'L' R1   
> INCR R1   
> INCR R1   
> ADDREG R1 5   
> DECR R1   
> SUBREG R1 4   
> SUBREG R2 'Z'   
> CMPREG R1 R2   
> BNZ youwishdude   
> DECR R0   
> BLOAD R0 R2   
> ADDREG R0 2   
> BLOAD R0 R1   
> RADD R2 R1   
> INCR R2   
> SUBREG R2 4Eh   
> LOADPTR retdecrypt-0DEADh ; push ptr to patch 把 PTR 压入栈修改  
> RESTOREREG R0   
> ADDREG R0 0DEACh   
> INCR R0   
> VMXOR R2 BPTR R0   
> MOVE msgcrypt COUNTER   
> LOADPTR goodboy   
> RestoreREG R0   
> INCR R2   
> decryptmsg:   
> VMXOR R2 BPTR R0   
> INCR R0   
> DECR COUNTER   
> BNZ decryptmsg   
> retdecrypt:   
> VMRETCRYPTED    
>   
> DATA@:   
> notgood db "Please Authenticate!",10,13,0   
> goodboy:   
> congrats db "Welcome...",10,13   
> db "Exploit for it doesn't matter 1.x Courtesy of Nicolas Brulez",0   
> goodboyend:   
> This little routine is the password protection used in the binary. For more informations about it, i will let you   
> read the submissions. As you can see, the password protection is VERY SHORT. I could have written a very complex  
>  algo with hundreds of lines to make it harder to analyse. Also this code is clear of junk. I could also have placed P-code Junk instructions inside the program. The password check was very simple and sadly, some people concentrated  
>  on the password rather than the Virtual Machine. Next time i will make it a lot more complex so people has no choice  
>  but to analyse the Virtual Machine and Instruction set.   
> 此小程序在代码中使用密码保护。如你所见，密码保护非常短。我可以使用上百的命令行以使程序难于分析。同时此代码  
> 是明显的垃圾代码。我也可以在程序中放置 Pcode 垃圾指令。此密码校验非常差非常简单，因此关注此密码的程序远远大  
> 于虚拟设备的程度。下次，我会使它更加复杂，以使大家只能分析虚拟设备和指令的设置。  
> You can compare the original P-code program here with the one inside the submissions to realize that they have done a very good job.   
> 你可以比较原始的 Pcode 程序，你会发现教程做的很棒。  
> A Few notes regarding the Password Protection   
> The password is checked using a very simple algorithm, but it is also used to decrypt yet another part of the pcode   
> program. There is a little weakness allowing one to find the correct value without any brute forcing or analysis   
> of the Opcodes:   
> 关于密码保护的一些注意  
> 密码使用非常简单的运算法则校验，但是它也被使用在解密 Pcode 的其它部分。这里有个漏洞可以无强制会操作码分析而找到正确的值。  
> Here is the encrypted String : 下面是加密字符串：  
> 0x14, 0x26, 0x2F, 0x20, 0x2C, 0x2E, 0x26, 0x6D,   
> 0x6D, 0x6D, 0x49, 0x4E, 0x06, 0x3B, 0x33, 0x2F,   
> 0x2C, 0x2A, 0x37, 0x63, 0x25, 0x2C, 0x31, 0x63,   
> 0x2A, 0x37, 0x63, 0x27, 0x2C, 0x26, 0x30, 0x2D,   
> 0x64, 0x37, 0x63, 0x2E, 0x22, 0x37, 0x37, 0x26,   
> 0x31, 0x63, 0x72, 0x6D, 0x3B, 0x63, 0x00, 0x2C,   
> 0x36, 0x31, 0x37, 0x26, 0x30, 0x3A, 0x63, 0x2C,   
> 0x25, 0x63, 0x0D, 0x2A, 0x20, 0x2C, 0x2F, 0x22,   
> 0x30, 0x63, 0x01, 0x31, 0x36, 0x2F, 0x26, 0x39,   
> 0x43   
> Everyone knows that a C String ends with a null byte. Therefore, the value used to encrypt this string is 0x43. The key  
>  is the last byte of the encrypted string. X xor 0 = X. :-)   
> 大家都知道 C 字符串以空结尾。因此，使用于加密此字符串的值是 0x43。关键在于加密字符串的最后字节。X XOR 0=X  
> The other possible ways to find the good value was to look at the code structure.. We were doing a Call routine, therefore  
>  we must have an instruction to do a RET. This instruction is the Virtual RET implemented in the Virtual Machine. From   
> this, we just had to find the opcode of this instruction to compute the key.  
> 另一种可能的方式是找到查看代码结构的正确值。我们做一个调用程序，因此我们需要指令完成返回。此指令在虚拟设备  
> 中以虚拟 RET 实现。由此看来，我们只要找到计算这个密码的指令操作码就可以了。   
>   
>   4. Provide a mean to "quickly" analyse this uncommon feature.   
>   
> With this question, i was expecting a disassembler for my Virtual Machine. A few people sent me fully working disassemblers,  
>  so i didn't write yet another one. I invite you once again to have a look at their submissions.One of the author emailed me   
> after the deadline with a working IDA processor module with source code included. This Processor module wasn't used in my  
>  judgement because it was sent after the deadline, but it is well worth studying it. It will be uploaded on the honeynet   
> web site shortly after the publication of the Results.   
> 4．提供快速分析非一般特点的方式  
> 围绕这个问题，我需要反汇编我的虚拟设备。有人给了我完整工作的反汇编器，因此我没有自己写一个。我再次邀请你看看他  
> 们的教程。一个作者在到期日后 E-MAIL 我一个包括源代码的工作的 IDA 处理模块。此模块我未使用，因为是在到期日后给我的，  
> 但它有值得学习的地方。这个模块将在结果公布后上传到 HONEYNET 上。  
>   5. Which tools are the most suited for analysing such binaries, and why?   
>   
> In my opinion the best tools to analyse such binaries are Interactive Disassemblers or CPU emulators. The disassembler   
> can be used to analyse the code statically, to remove the obfuscations, to decrypt binaries etc. If it offers possibility  
>  to write processor module, you can even write a disassembler for the Virtual Machine and thus, do a full static analysis  
>  of the whole thing. A CPU emulator can be used to quickly decrypt the code , layers etc. If it can be scripted not to show   
> the obfuscations you have a perfect weapon. I don't like Debuggers because they aren't reliable. I could have easily written   
> a driver to hook debug interupts to decrypt the binary for instance. Debuggers would have been useless and would have rebooted   
> the computer if used.   
> 5．哪个工具最适合分析这种代码，为什么？  
> 在我看来是交互式反汇编器或 CPU EMULATORS。反汇编器可以用来分析静态代码，打消困惑，加密代码等。如果它提供写处理模块，  
> 你甚至可以写出虚拟设备的反汇编器并分析整个静态代码。CPU EMULATOR 可以快速的加密代码，LAYERS 等等。如果它可以消除困惑，  
> 你就拥有了一个强劲的工具。我不喜欢调试器，因为他们不可信赖。我可以很容易的写出驱动挂起调试以加密代码。调试器毫无用  
> 处并且可能使系统重起。  
>   6. Identify the purpose (fictitious or not) of the binary.   
>   
> This binary is waiting for an user to authenticate with a password that is passed to the application through the command line.   
> Once the user has been identified, the binary will print a little message. It looks like a fake exploit. In the real world,   
> it could have been a real exploit protected from prying eyes.   
> 6．鉴别代码用途  
> 代码在等待使用者鉴别通过命令行传达到程序中的密码。一旦确认，代码将产生一些消息。看起来象伪使用。在实际中，  
> 它是迷惑眼睛的真实保护。  
>   7. What is the binary waiting from the user? Please detail how you found it.   
>   
> The binary is waiting for a password through the command line. The password is used to access the real program. To find  
>  this password, you have to Reverse Engineer the binary. Decrypt every layers to access the Virtual Machine. This Virtual  
>  Machine has a virtual program used to check the password entered. One has to Reverse Engineer the Virtual Machine (or   
> trace it blindly) in order to understand its instruction set. Then it is just a very simple algo using a few easy operations   
> to reverse. I invite you to read submissions for details about the algo itself.   
> 7．等待使用者的代码是什么？请详述你发现的。  
> 代码等待命令行中的密码。密码用来访问真正的程序。为了找到密码，你需要逆向代码。解密每个访问虚拟设备的 LAYERS。此虚  
> 拟设备包括校验已输密码的虚拟程序。你需要逆向虚拟设备（或盲目的跟踪）以了解指令设置。不过它非常简单，容易逆向。我  
> 推荐你看以下教程了解详细信息。  
>    
>   
>   8. Bonus Question - What techniques or methods can you think of that would make the binary harder to reverse engineer?   
> 8．你认为什么代码可以是逆向难于分析？  
> This binary has a lot of security flaws that were left on purpose and it has a lot of things needing improvements. 此代码留  
> 下许多安全缺陷，需要很多改进。  
>   • Junk Code without pushad/popad 无 PUSH/POPAD 的垃圾代码  
>   • P-code Junk code to really drive people tracing the protection nuts: 80% of useless instructions would definitevely  
>  drive anyone mad. 跟踪 80%的 Pcode 垃圾代码真的使人发疯。  
>   • The encryption has a few weaknesses that were presented in the document. Mainly the static encryption algo and the   
> static size of every layers.文档中提及加密缺陷。主要是静态加密和每个 LAYERS 的固定大小  
>   • Random constants in the first value of the address tables used by the Layers  LAYER 在地址表的第一个值中使用任意常数  
>   • Better BPX detection, it could be greatly improved. 更好的 BPX 侦测将会改进。  
>   • The SEH could be used to initialise/decrypt part of the code in order to make sure they won't be nopped out. SEH 用于  
> 初始化/解密代码防止 NOP。  
>   • Random Constants and code for the timing detection would make it harder to bypass with scripts. 时间侦测的任意常  
> 数将使脚本绕过侦测更加困难。  
>   • The protection has been generated by my own tools and it is a bit repetitive. More variations would make automatic   
> removal harder. 保护又我自己的工具产生。种类的增加会自动的增加找到切入点的难度。  
>   • The Virtual Machine handlers are fairly simples. More code obfuscation (code flow and logic) could be used. 虚拟  
> 设备操作很简单。需要更多的迷惑性的代码  
>   • More Instructions in the Virtual Machine would have made it longer to analyse. 虚拟设备中需要使用更多指令延长你的分析时间  
>   • Complex Opcodes encoding would make it quite challenging to Reverse Engineer. 复杂操作码的解码对逆向是一种挑战。 

  

> • Utilization of Cryptography rather than a simple algo to check for the serial number   
>    利用密码术代替简单方式校验序列号。  
>      • More Layers and different ones. There are a lot of ways to stop ring 3 debuggers that could have been used to stop anyone trying to debug it.   
> 更多不同的 LAYERS。有许多方式都可终止 RING3 级别的调试器进行调试。  
>      • Imports Protection to make sure noone knows the API function used until he meets them in the code.   
> 输入表保护防止得到 API 函数，而只能从代码中得到它们。  
>      • Emulation Macros to emulate simple x86 instructions. With those macros remplacing simple instruction, it would   
> be a lot harder to analyse. one instruction would have a 20 instructions equivalent block of code for instance.   
>    利用宏仿效 x86 指令。通过替换简单指令，使得分析更加困难。一条指令将包含同等的 20  
>    条代码 block。  
>      9. Conclusion   
>   
> Anti Reverse Engineering Techniques can be used to really slow down the analysis of a binary. Malwares could be using   
> such techniques in a near futur and it is time to get used to it. Even though most of the malwares are programmed by  
>  clueless idiots without any programming skill, there is a minority able to write complex code. In the futur we could   
> find exploit binaries on compromised systems that would be protected against Reverse Engineering to hide the vulnerability  
>  exploited. Spywares could also use such techniques to hide their activity. This binary had a lot of vulnerabilities, yet it  
>  was really challenging , even with a trivial password protection algorithm. The protection has been written within a week  
>  (a few hours per day), so with a little more effort, it can be a LOT harder.Finally, I would like to point out that Reverse  
>  Engineering isn't a pirate technique and that it is used by the Security Community on a daily basis. Some people in France  
>  doesn't seem to agree though..   
> 9．结论  
> 反逆向技术的使用可以减缓代码的分析。病毒可以在将来使用此技术，我们需要了解它。虽然大多数病毒是一些毫无技巧可  
> 言的笨蛋编写的，但仍然会有少数编写复杂代码的高手。将来，我们会在隐藏弱点进行反逆向的系统中发现需要的代码。间  
> 谍程序也会使用此技术隐藏它的活动。虽然具有挑战性甚至包含密码保护运算，此代码仍有许多缺陷。此保护一周编写完成  
> ，比较难。最后，我指出逆向工程不是盗版技术而是用语安全社会的日常中。  
> (全文完) 
> 
>  4dd466d0e1e0cc123afbe504da2462ca