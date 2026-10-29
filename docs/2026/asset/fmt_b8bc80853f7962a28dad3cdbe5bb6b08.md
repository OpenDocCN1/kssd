
`egg_loop`是替换过变量值的目的寻找内核态 shellcode，上下两个 EGG 是用户态 shellcode 的寻找标志，用作分隔，最后是被注入进程要执行的 R3 payload。（默认是弹 system 权限的 shell 连接，是 MSF 众多可选 payload 之一。）

而内核态 shellcode 就比较长了，涉及诸多功能，这也解释了为什么 `0x400` 的通道空间放不下的原因，主要是内核 shellcode + R3 payload 比较大。在开始部分，作者硬编码了一些需要用到的函数和结构的 hash 和偏移。由于可选的测试环境都是 Win7/2008，所以这些值都是通用的。如图：

图 27

![](https://bbs.kanxue.com/upload/attach/202002/858669_6JEY6NCR96PHEF4.jpg)

开头的几个 offset 是当 R0 shellcode 执行时，将获取的相关数据保存到相对内核最高段 HAL 地址的偏移，即上图的 `hal_heap_storage`。下面是 shellcode 内容，如图：

图 28

![](https://bbs.kanxue.com/upload/attach/202002/858669_FE8UF9JG4P5SJVY.jpg)

首先调用 `sti` 允许中断发生，然后进入主逻辑。结束后选择直接退出 `IcaChannelInputInternal` 函数，不再执行之后的代码，避免麻烦，因为之后的代码逻辑依赖于正常的数据和调用，而我们劫持调用后的数据是错的。但这也造成一个小问题：此函数之前获得的一个锁因为跳过而没有被释放。不过从多次测试结果来看，并没有什么影响。

这段退出代码和原函数结尾是一样的。另外可以发现，这两段 shellcode 头部并没有大量 `nop` 指令做滑板，因为内核非分页内存布局对齐比较规律，堆喷之后两段 shellcode 首部大概率命中经过计算的整数地址，测试验证成功率很高。

## `r3_to_r0_start` 主逻辑

下面进入 `r3_to_r0_start` 主逻辑，概括起来分为几个步骤：

1. 找到 `ntoskrnl` 内核模块基址。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_VXHXVTJ5ACFTBF9.jpg)

2. 获取当前的 `EPROCESS` 和 `ETHREAD` 结构。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_DF7KHP6TF3XE7TN.jpg)

3. 获取 `EPROCESS` 结构中 `ImageFilename` 的偏移。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_67NJP4QFEJKD62R.jpg)

4. 获取 `EPROCESS` 结构中 `ThreadListHead` 的偏移。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_PGVEGNAR92V4XFN.jpg)

5. 获取 `ETHREAD` 结构中 `ThreadListEntry` 的偏移。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_C75QD3YUHAR58XP.jpg)

6. 获取 `EPROCESS` 结构中 `ActiveProcessLinks`（活动进程链表）的偏移。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_A6PJ38XW4R7BGJY.jpg)

7. 遍历 `ActiveProcessLinks`，找到要注入的 `spoolsv.exe` 进程（`EPROCESS`）。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_ZA4MCQZVKS25VFA.jpg)

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_KBV3CHCHU53RUYE.jpg)

8. 调用 `PsGetProcessPeb` 获取 `spoolsv.exe` 的 `PEB` 结构。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_YGREMDZUDMBSZEQ.jpg)

9. 遍历 `spoolsv.exe` 的 `ThreadList`，执行 `KeInsertQueueApc` 直到成功。当 APC 执行时，处于 `spoolsv.exe` 进程上下文。

   这里插入了两个 APC：一个 R0 APC，一个 R3 APC。R3 APC 函数的地址目前是无效的，待后续步骤中再修改；R0 APC 先被调用。（代码比较长，只截图了关键部分。）

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_PQSRBXRRH3C6ZBS.jpg)

上文中获取的所有数据都保存在内核最高段的 HAL 地址中（非分页，且自系统启动以来就没有用到）。插入的 R0 APC 执行逻辑如下：

1. 通过之前保存的 nt kernel 模块地址找到它的 `ZwAllocateVirtualMemory` 函数。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_M2FMCQNYJWTP9TU.jpg)

2. 调用 `ZwAllocateVirtualMemory` 分配 R3 可执行内存，并将上图第二个参数 R3 APC 函数的地址改为刚分配的 shellcode 地址。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_8VX2ZYW9GHWQ2N8.jpg)

3. 定位 shellcode，并复制到刚分配的内存中。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_V9N7FQ5Y4DBQ9H5.jpg)

4. 通过之前保存的 `spoolsv.exe` 的 `PEB` 遍历加载模块，获得 `kernel32.dll` 地址。

5. 遍历 `kernel32.dll` 的导出表，找到 `CreateThread` 函数地址。

6. 将 `CreateThread` 函数地址保存到参数 `SystemArgument1` 中，此参数会被系统传给 R3 APC。

   ![](https://bbs.kanxue.com/upload/attach/202002/858669_DAPKG7Q239Q9WJY.jpg)

另一个插入的 R3 APC 即为要执行的 payload。开头部分利用参数 `CreateThread` 函数创建子线程，执行若干字节后的真正 payload 功能代码。父线程创建完线程后，通过 `ret` 返回并结束 APC。至此，整个 shellcode 执行流程结束。

# 五、总结

整个过程用逆推法开始分析，逐步找到漏洞点。在分析过程中有一个小插曲：MSF 这个利用模块的作者更新了他的 shellcode 代码，不再使用 hook syscall 的方式，转而通过各种数据结构信息完成利用 APC。猜测原因可能是这种方式比 hook syscall 更加稳定，破坏性的更改较少。

纵观 shellcode，其中也提取了常见的功能且频繁调用，比如计算函数名 hash、根据模块查找函数地址。

经过测试发现，对于目前的 exploit，目标系统内存必须为 2G 或以上才能成功。如果想要达到稳定的效果，可以根据不同内存大小修改 exploit 选项中的 `GROOMBASE`，参考图 25。

写本文时参考了一些网上的文章，发现不论是分析漏洞的过程，还是分析 shellcode 的过程，都比较简略，踩坑无数，真可谓曲折前进。

最后放一张成功弹 shell 的截图：

![](https://bbs.kanxue.com/upload/attach/201912/858669_T3JQSNFRZDYDRZK.jpg)

![](https://bbs.kanxue.com/upload/attach/201912/858669_EH7SM53AMMC9W4Q.png)

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-12-16 13:27 被 `MSA_Li` 编辑，原因：补上不知为何丢失的图片
