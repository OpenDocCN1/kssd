
# 高级进程注入总结

前几天看玄武的“每日安全”板块，发现了一篇关于`进程注入`的文章，之前了解过一些相关的技术，不过是零散的。看到文章总结的挺详细的，且各种方法的异同细节容易忘，于是就想着总结一下各种`进程注入`的异同和优缺点，因此有了本文。

> - 关于常见的进程注入方法，请参考这个和这个，本文将重点描述值得留意的方法（比如Module Stomping），和一些思路新颖的方法。
> - 以下描述的方法在`进程注入`这个仓库都能找到。
> - 本文在我的博客同时发布。

## 进程注入简介

进程注入就是将代码注入到另一个进程中，shellcode注入和DLL注入都属于进程注入。

进程注入的作用是隐藏自身或者利用另一个进程的权限做一些不好的事情^\_^，比如病毒等恶意程序注入到一个正常的进程，以此隐藏自己。

进程注入的方法非常之多，很多与DLL注入有关，比如注册表（Image File Execution Options）、DLL劫持、输入法、COM、LSP劫持（LayerService Provider，与winsock有关）...

除了DLL注入，还有shellcode注入，因为shellcode更小，所以shellcode的使用也更加多样。

2017年，在黑客大会上Eugene Kogan和Tal Liberman又分享了更加隐蔽和特别的方法，比如Process Doppelganging。那么接下来就开始进程注入方法的介绍。

## Module Stomping

该方法通过在目标进程中加载一个合法DLL，然后将shellcode或恶意DLL覆写到这个合法DLL的地址空间里。

伪代码如下（省略错误检查）：

```c
HANDLE hTargetHandle = OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ | PROCESS_ALL_ACCESS | PROCESS_SUSPEND_RESUME, FALSE, targetPid);
//...
// Use OpenProcess + VirtualAllocEx + CreateRemoteThread(the traditional dll injection method)
injectLoadLibrary(hTargetHandle, legitimateDll);
//...
// Copy every malware dll's section into legitimate dll's corresponding sections within the target process address space.
for(section : malwareDll){
WriteProcessMemory(hTargetHandle, legitimateDllSection, section, section_size...);
if (has executable section)
BypassCFG();
}
RebuildImportTable();
RebuildRelocationTable();
callTLSCallback();
CreateRemoteThread(malwareDllEntryPoint);
```

通过该方法可以将恶意代码隐藏，使得Defender扫描目标进程空间时不会发现恶意代码的存在（因为恶意代码潜伏在合法的模块里）。

该方法有两个值得关注的点：

- 先在目标进程加载一个合法DLL，以此隐藏恶意代码。
- 它用了`SetProcessValidCallTargets`，该方法可以bypass开启CFG的程序。

> 与CFG(control flow guard)相关的知识可参考`MSDN`或者`System Internals 7th part1` security chapter。

不过该方法还是有明显的不足，比如会调用显眼的VirtualAllocEx、WriteProcessMemory、VirtualProtectEx和CreateRemoteThread，通常的EDR都能检测出可能的恶意行为。

> 该方法的详细细节可参考这篇文章。

## Process Hollowing

该方法也比较经典，并被广泛使用，其基本流程如下：

```c
CreateProcess("svchost.exe" , …, CREATE_SUSPENDED, …);
NtUnmapViewOfSection(…) and VirtualAllocEx(…);
For each section:
WriteProcessMemory(..., EVIL_EXE, …);
Relocate Image*;
Set base address in PEB*;
SetThreadContext(…);
ResumeThread(…);
```

该方法和`Module Stomping`很像，都是替换掉了一个模块，不过有以下三个不同：

- `Process Hollowing`替换的模块是目标EXE程序
- 该方法使用NtUnmapViewOfSection将EXE程序解除映射
- 用ResumeThread来启动恶意程序的入口点

> 该方法的详细实现可参考这个。

## Process Doppelganging

接下来描述的几个进程注入方法使用了不同的注入思路，它们之间只有细微的差别，最后会总结它们的异同点。

该方法是2017年在黑客大会介绍的，演讲者首先说明了Process Hollowing涉及的敏感操作和一些不足：

- Unmap目标进程的EXE模块（非常可疑），现代的安全检查一般都有Unmap检查。
- 如果不Unmap，而是直接覆写程序，那么覆写地址的页属性就不是共享的，也很可疑。

> 页属性共享涉及到操作系统的共享内存机制，这里简单描述一下：
>
> 为了节省内存，当一个模块加载进内存时，比如`ntdll.dll`，操作系统会看该模块是否已被其他进程加载过了（即已经在物理内存中了），如果在，那么操作系统只需要简单地将该物理区域映射到需要加载模块的进程中。因此，这些模块对应的内存是共享的，这些模块内存对应的进程pte(page table entry)会指向一个叫prototypePTE的结构，该结构会指向这些共享的物理内存。
>
> 不过，如果我们向模块，比如`ntdll.dll`，的数据段写入数据，那么这时操作系统就会对`ntdll.dll`的数据段分配一段物理内存，然后当前进程对应数据段的pte就不会再指向prototypePTE，而是指向操作系统分配的物理内存。
>
> 关于内存共享机制的详细描述，读者看参考Windows Internals 7th，第五章的Page fault handling部分。

- 直接覆写程序不够好，那就Unmap后再Remap：
- 如果remap时的类型不是IMAGE，通过检查节的类型可判定是否可疑
- 如果remap时的类型是IMAGE，这时可疑的点就不多了。不过因为`Process Hollowing`用SetThreadContext修改了初始线程的执行入口点（ETHREAD.Win32StartAddress），那么我们可以检测其执行入口点是否是ETHREAD.Win32StartAddress。如果不是，那值得怀疑，并且我们可以检测其执行入口点对应的文件名，这样可进一步判定这段内存是否是可疑payload。

> 检测文件名可通过该字段查看：VAD(ETHREAD.Win32StartAddress).Subsection.ControlArea.FilePointer。

另外，如果要使用Remap，那就需要一个section，打开section需要一个文件句柄，也就是说Remap需要一个落地的文件，因此采取`process hollowing`时，攻击者很少会使用Remap的方式。

综上所述，Process Hollowing已经不是那么好了，那还有什么其他更隐蔽的方法吗？

于是演讲者介绍了Process Doppelganging，该方法由`CreateFileTransacted`API开始。由于其内容较多，且不易描述，读者可查看这个PPT和这个源码和这篇文章，这些资料讲得非常详细。

这里归纳了简要的流程：

- 创建一个transaction（事务）
- 打开原程序句柄（通过CreateFileTransacted）
- 向原程序句柄写入恶意代码，根据此时的文件内容，创建一个section
- rollback（回滚）之前的写操作

> 虽然回滚了文件的内容，但已生成的section映射的内容是修改后的，即内容是payload，解释可参考`Process Herpaderping`小节。

- 通过刚刚创建的section，创建进程（通过NtCreateProcessEx）
- 准备参数到目标进程（跨进程）
- 创建初始线程(NtCreateThreadEx)，之后唤醒线程（NtResumeThread）

以上流程可抽象如下：

```c
transact -> write -> map -> rollback -> execute
```

该方法的新颖点在于通过Windows提供的`事务`API，将恶意代码写入打开的文件，并创建一个section，用其创建进程，之后回滚写入操作。这样可以隐藏执行的恶意代码，虽然你查看该进程时（比如procExp），其显示的是原程序的信息，但其真正执行的代码是恶意代码。同时，它比Process Hollowing更隐蔽，因为它不用Unmap，也不需要Remap，它就像正常启动一个进程一样。

最近，我编译了这份代码，发现win10、win8.1和win7都失败了，说明windows已经patch了该方法，以下是win10和win8.1测试的结果：

- 如果覆盖用的是非PE文件，NtCreateSection返回错误，提示无效的PE。
- 如果覆盖用的是PE文件，创建进程成功，不过我们在procExp中看到的是名叫`System Idle Process`，观察该进程信息，其提示“请求的操作是在不再活动的事务的上下文中进行的”。

![image-20220216102919516](upload/attach/202202/744796_E6KXWB65F4V7PDQ.jpg)

虽然该方法已经失效了，不过它的思路很好。之后介绍的`Process Herpaderping`借鉴了该方法，且目前也是有效的。

> 可能我实践时，代码的参数没设置好，所以没有复现demo里的结果。从现在的恶意攻击看，基于`Process Doppelganging`的攻击很少，不知道是不是因为被patch了。不管怎么说，该方法是一个很好的起点。
