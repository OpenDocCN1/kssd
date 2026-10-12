
此外，你还应该注意区段名，通过名字你可以大致了解每个区段的作用。我建议你可以先阅读 [Microsoft Portable Executable and the Common Object File Format Specification](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx) 文档的第六部分。我认为，它已经囊括了所有的区段，见表 2。

**Table 2 - Section names**

| 区段名 | 描述 |
| :--- | :--- |
| `.text` | Code Section |
| `CODE` | Code Section of file linked by Borland Delphi or Borland Pascal |
| `.data` | Data Section |
| `DATA` | Data Section of file linked by Borland Delphi or Borland Pascal |
| `.rdata` | Section for Constant Data |
| `.idata` | Import Table |
| `.edata` | Export Table |
| `.tls` | TLS Table |
| `.reloc` | Relocation Information |
| `.rsrc` | Resource Information |

To comprehend the section headers and also the sections, you can run the sample [PE viewer](http://www.codeproject.com/system/inject2exe.asp#PEViewerDownloadLink#PEViewerDownloadLink). By this PE viewer, you only can realize the application of the section headers in a file image, so to observe the main significance in the Virtual Memory, you should try to load a PE file by a debugger, and the next section represents the main idea of using the virtual address and –size in the virtual memory by using a debugger. The last note is about [`IMAGE_NT_HEADERS`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/image_nt_headers_str.asp) -> [`FileHeader`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/image_file_header_str.asp) -> `NumberOfSections`, that provides a number of sections in a PE file, do not forget to adjust it whenever you remove or add some sections to a PE file, I am talking about section injection!

为了充分理解区段头和区段，你可以运行 [PE viewer](http://www.codeproject.com/system/inject2exe.asp#PEViewerDownloadLink#PEViewerDownloadLink)。但是通过 PE viewer，你只能了解区段头在文件映像中的情况，因此，为了观察重要的虚拟内存中的部分，你应该尝试用调试器加载 PE 文件，我们接下来将介绍在调试器中怎样利用虚拟内存中的虚拟地址和–size。最后要注意的是 [`IMAGE_NT_HEADERS`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/image_nt_headers_str.asp) -> [`FileHeader`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/image_file_header_str.asp) -> `NumberOfSections`，它提供 PE 文件的区段数，无论你增加/删除区段，都不要忘了调整它，因为我正在讨论的是区段注入？

## 3 Debugger, Disassembler and some Useful Tools

## 3 调试器，反汇编器和一些有用的工具

In this part, you will become familiar with the necessary and essential equipments to develop your PE tools.

在这部分，将介绍开发 PE 工具所必需的基本装备。

### 3.1 Debuggers

### 3.1 调试器

The first essential prerequisite, to become a PE tools developer, is to have enough experience with bug tracer tools. Furthermore, you should know most of the assembly instructions. To me, the Intel documents are the best references. You can obtain them from the Intel site for IA-32, and on top of that IA-64; the future belongs to IA-64 CPUs, Windows XP 64-bit, and also PE64!

成为 PE 工具开发者的基本条件是要有使用 bug 跟踪工具的丰富经验；此外，还应该熟悉大部分汇编指令。为我来说，Intel 文档是最好的参考书。你可以从 Intel 网站上找到 IA-32 或 IA-64 的文档；未来属于 IA-64 CPU，64 位 Windows XP，以及 PE64！

*   [IA-32 Intel Architecture Software Developer’s Manuals](http://www.intel.com/design/pentium4/manuals/index_new.htm#1)
*   [Intel Itanium Architecture Assembly Language Reference Guide](http://www.intel.com/software/products/compilers/docs/linux/ref/asm_lan_lx.htm#cover.htm)
*   [The Intel Itanium Processor Developer Resource Guide](http://www.intel.com/cd/ids/developer/asmo-na/eng/19415.htm)

To trace a PE file, [SoftICE](http://en.wikipedia.org/wiki/SoftICE) by [Compuware Corporation](http://www.compuware.com/), I knew it also as named [NuMega](http://en.wikipedia.org/wiki/Numega) when I was at high school, is the best [debugger](http://en.wikipedia.org/wiki/Debugger) in the world. It implements process tracing by using [kernel mode](http://en.wikipedia.org/wiki/Kernel_mode) method debugging without applying Windows debugging [application programming interface](http://en.wikipedia.org/wiki/Application_programming_interface) (API) functions. In addition, I am going to introduce one perfect debugger in [user mode](http://en.wikipedia.org/wiki/User_mode) level. It utilizes the [Windows debugging API](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/debugging_reference.asp) to trace a PE file and also attaches itself to an active [process](http://en.wikipedia.org/wiki/Computer_process). These [API](http://en.wikipedia.org/wiki/Application_programming_interface) functions have been provided by Microsoft teams, inside the Windows Kernel32 library, to trace a specific process, by using Microsoft tools, or perhaps, to make your own debugger! Some of those [API](http://en.wikipedia.org/wiki/Application_programming_interface) functions inlude: [`CreateThread()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/createthread.asp), [`CreateProcess()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/createprocess.asp), [`OpenProcess()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/openprocess.asp), [`DebugActiveProcess()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/debugactiveprocess.asp), [`GetThreadContext()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/getthreadcontext.asp), [`SetThreadContext()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/setthreadcontext.asp), [`ContinueDebugEvent()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/continuedebugevent.asp), [`DebugBreak()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/debugbreak.asp), [`ReadProcessMemory()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/readprocessmemory.asp), [`WriteProcessMemory()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/writeprocessmemory.asp), [`SuspendThread()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/suspendthread.asp), and [`ResumeThread()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/resumethread.asp).
