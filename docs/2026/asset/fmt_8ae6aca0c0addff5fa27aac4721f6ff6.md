
# Windows 内存保护机制

As explained in Chapter 1, “Concepts and Tools,” in Part 1, Windows provides memory protection so that no user process can inadvertently or deliberately corrupt the address space of another process or of the operating system. Windows provides this protection in four primary ways.

## 内存保护

如同在本书上册第一章“概念和工具”中解释的，Windows 提供内存保护机制，以至于没有用户进程可以无意或刻意损坏另一个进程或操作系统的地址空间。Windows 通过4种主要方式提供内存保护。

### 第一

所有内核模式系统组件使用的系统级别数据结构和内存池，只能够在内核模式下被访问——用户模式线程不能够访问这些页面。如果试图这么做，硬件生成一个错误，从而内存管理器反过来向该线程报告一个非法访问。

### Second

每个进程有一个单独，私用的地址空间，防止被属于另一进程的任何线程访问。甚至共享内存也不例外，因为每个进程使用其自身虚拟地址空间的一部分地址，访问该共享区域。唯一的例外是，如果另一个进程有该进程对象的虚拟内存读或写访问需求（或者持有`SeDebugPrivilege`权限），可以使用`ReadProcessMemory` 或 `WriteProcessMemory`函数。每当一个线程引用一个地址，虚拟内存硬件（译注：通常是处理器内置的MMU，即内存管理单元）与操作系统内存管理器配合，介入其中并且将虚拟地址翻译物理地址。Windows通过控制虚拟地址被翻译的方式，可以确保一个进程中运行的线程不会有对属于另一个进程页面的不当访问。

### 第三

除了提供虚拟到物理地址翻译的隐式保护之外，所有Windows支持的处理器提供某种形式的“硬件控制内存保护机制”（例如读/写，只读等等）；这种保护机制的具体细节因处理器而异。

例如，一个进程地址空间中被标记为只读的代码页因而能够防止被用户线程修改。

表10-2列出了Windows API 中定义的内存保护选项（参见 `VirtualProtect`，`VirtualProtectEx`，`VirtualQuery`，以及`VirtualQueryEx`函数的说明文档）

第二，每个进程有一个独立，私用的地址空间，防止被属于另一进程的任何线程访问。甚至共享内存也不例外，因为每个进程使用其自身虚拟地址空间的一部分地址，访问该共享区域。唯一的例外是，如果另一个进程有该进程对象的虚拟内存读或写访问需求（或者持有`SeDebugPrivilege`权限），可以使用`ReadProcessMemory` 或 `WriteProcessMemory`函数。每当一个线程引用一个地址，虚拟内存硬件（译注：通常是处理器内置的MMU，即内存管理单元）与操作系统内存管理器配合，介入其中并且将虚拟地址翻译物理地址。Windows通过控制虚拟地址被翻译的方式，可以确保一个进程中运行的线程不会有对属于另一个进程页面的不当访问。

第三，除了提供虚拟到物理地址翻译的隐式保护之外，所有Windows支持的处理器提供某种形式的“硬件控制内存保护机制”（例如读/写，只读等等）；这种保护机制的具体细节因处理器而异。

例如，一个进程地址空间中被标记为只读的代码页因而能够防止被用户线程修改。

表10-2列出了Windows API 中定义的内存保护选项（参见 `VirtualProtect`，`VirtualProtectEx`，`VirtualQuery`，以及`VirtualQueryEx`函数的说明文档）

![](https://bbs.kanxue.com/upload/attach/201509/533967_4643471705bee503.jpg)

![](https://bbs.kanxue.com/upload/attach/201509/533967_5e0ffa8de08394f9.jpg)

### 第四

And finally, shared memory section objects have standard Windows access control lists (`ACLs`) that are checked when processes attempt to open them, thus limiting access of shared memory to those processes with the proper rights. Access control also comes into play when a thread creates a section to contain a mapped file. To create the section, the thread must have at least read access to the underlying file object or the operation will fail.

Once a thread has successfully opened a handle to a section, its actions are still subject to the memory manager and the hardware-based page protections described earlier. A thread can change the page-level protection on virtual pages in a section if the change doesn’t violate the permissions in the `ACL` for that section object. For example, the memory manager allows a thread to change the pages of a read-only section to have copy-on-write access but not to have read/write access. The copy-on-write access is permitted because it has no effect on other processes sharing the data.

最后，共享内存节对象有标准的Windows访问控制列表（`ACLs`），当进程尝试打开它们时，将检查相应的`ACL`，从而给予那些进程适当的权限来限制对共享内存的访问。当一个线程创建一个节对象来包含一个映射文件时，访问控制也能够发挥作用。该线程必须至少有对底层文件对象的读访问权限，否则操作将失败。

一旦线程成功的打开到一个节的句柄，其行为仍旧受到内存管理器与基于硬件的页面保护限制。一个线程可以更改节对象中虚拟页的页级保护属性，前提是这个更改没有违反该节对象`ACL`中的权限。举例来说，内存管理器允许一个线程将一个只读节中的页面改变为写时复制访问，但是不能改变为可读写访问。允许改变为写时复制访问是因为这不会影响其它进程共享只读的数据。（译注：即前文讲过的，在线程请求写入时，创建一个只有该线程可见的私有副本页，其对进行写操作不会改变原始的只读共享页）

## No Execute Page Protection

No execute page protection (also referred to as data execution prevention, or DEP) causes an attempt to transfer control to an instruction in a page marked as “no execute” to generate an access fault.

This can prevent certain types of malware from exploiting bugs in the system through the execution of code placed in a data page such as the stack. DEP can also catch poorly written programs that don’t correctly set permissions on pages from which they intend to execute code. If an attempt is made in kernel mode to execute code in a page marked as no execute, the system will crash with the `ATTEMPTED_EXECUTE_OF_NOEXECUTE_MEMORY` bugcheck code. (See Chapter 14, “Crash Dump Analysis,” for an explanation of these codes.) If this occurs in user mode, a `STATUS_ACCESS_VIOLATION` (`0xc0000005`) exception is delivered to the thread attempting the illegal reference.

If a process allocates memory that needs to be executable, it must explicitly mark such pages by specifying the `PAGE_EXECUTE`, `PAGE_EXECUTE_READ`, `PAGE_EXECUTE_READWRITE`, or `PAGE_EXECUTE_WRITECOPY` flags on the page granularity memory allocation functions.

## 不可执行页保护

不可执行页保护（也被称为数据执行保护，或DEP）导致试图转移（对CPU的）控制到一个标记为“不可执行”页中的指令时，生成一个访问错误。这可以防止某些类型的恶意软件通过将可执行代码放置在比如栈这种数据页中，从而利用系统中的缺陷或漏洞。DEP也能够捕捉到那些编写不当的程序，这些程序没有为它们打算从中执行代码的页面设置正确的权限。假设在内核模式下尝试在一个标记为不可执行的页中执行代码，系统会崩溃，并且错误检查码为`ATTEMPTED_EXECUTE_OF_NOEXECUTE_MEMORY`。（参见第14章，“崩溃转储分析”，其中有对这些代码的解释）

如果上述情况发生在用户模式，会传递一个`STATUS_ACCESS_VIOLATION(0xc0000005)`异常到尝试非法引用的线程。如果一个进程申请分配的内存需要能够被执行，进程必须通过在页面粒度内存分配函数中指定`PAGE_EXECUTE`，`PAGE_EXECUTE_READ`，`PAGE_EXECUTE_READWRITE`，或者`PAGE_EXECUTE_WRITECOPY`标志，来显式标记此类页面。

On 32-bit x86 systems that support DEP, bit 63 in the page table entry (`PTE`) is used to mark a page as nonexecutable. Therefore, the DEP feature is available only when the processor is running in Physical Address Extension (`PAE`) mode, without which page table entries are only 32 bits wide. (See the section “Physical Address Extension (`PAE`)” later in this chapter.) Thus, support for hardware DEP on 32-bit systems requires loading the PAE kernel (`%SystemRoot%\System32\Ntkrnlpa.exe`), even if that system does not require extended physical addressing (for example, physical addresses greater than 4 GB). The operating system loader automatically loads the PAE kernel on 32-bit systems that support hardware DEP. To force the non-PAE kernel to load on a system that supports hardware DEP, the BCD option `nx` must be set to `AlwaysOff`, and the `pae` option must be set to `ForceDisable`.

在支持DEP的32位x86系统上，页表条目（`PTE`）中的比特位63（以及页目录项`PDE`中的比特位63），用于标记一个页（对于`PDE`而言，则是该`PDE`引用的页表中的所有页）是否为不可执行。故而仅当处理器以物理地址扩展（`PAE`）模式运行时，DEP功能才可用；对于32位宽的页表条目，则不支持DEP。（参见本章后面的“物理地址扩展”部分。）于是，在32位系统上支持硬件DEP，需要加载`PAE`内核（即 `%SystemRoot%\System32\Ntkrnlpa.exe`），即使该系统并不需要扩展物理寻址（例如大于4GB的物理地址）。在支持硬件DEP的32位系统上，操作系统加载器会自动载入`PAE`内核。要在支持硬件DEP的系统上强制加载非`PAE`内核，`BCD` 选项 `nx` 必须设置成 `AlwaysOff`，并且 `pae` 选项必须设置成 `ForceDisable`。( 译注：在CMD命令行提示符下切换到 `C:\Windows\System32`，执行 `bcdedit` 命令，即可显示当前的BCD 选项 `nx` 的值，并对其进行更改；另外，该命令也可以强制启用或禁用OS的PAE功能，如下图所示）

![](https://bbs.kanxue.com/upload/attach/201509/533967_eea4ad4b7e1a3216.jpg)

此外，还需要检查注册表项 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management`，其右侧名称为 `PhysicalAddressExtension` 的 `REG_DWORD` 类型值是否为16进制的1，1表示启用`PAE`，0表示禁用`PAE`。最后，当`IA32_EFER` 寄存器的 `NXE` 标志置1，并且`PDE/PTE` 的比特位63（`XD`，第64位）置1时，就无法从被引用的页中取指令，这些页可能用于栈，数据段，以及堆。
