
# 打造自己的 HOOK 引擎 之一 --- SSDT HOOK 引擎

> 原文：[`www.pediy.com/kssd/pediy10/79247.html`](https://www.pediy.com/kssd/pediy10/79247.html)

SSDT HOOK 已经是老生常谈的话题了，这里我就不对其进行介绍了，其实 HOOK 技术已经被大牛们分析的淋漓尽致,无以复加了，我也只是个初学者，失误之处，还望大牛们多多指教

读者可以通过后面的参考连接得到更多的关于 SSDT HOOK 的信息

我就相当于在这里对论坛的关于 SSDT 的精华贴做个总结了 呵呵

* [【原创】扫盲贴，HOOK SSDT 短文一篇。](http://bbs.pediy.com/showthread.php?t=78218)
* [【原创】分享比较完整的 ROOTKIT DEMO! 原来 Shadow Hook 和 SSDT Hook 一样容易！](http://bbs.pediy.com/showthread.php?t=70083)
* [【原创】SSDT Hook 的妙用－对抗 ring0 inline hook](http://bbs.pediy.com/showthread.php?t=40832)
* [【原创】寻找原始表，恢复 ssdt 表](http://bbs.pediy.com/showthread.php?t=49848)
* [【原创】RootKit hook 之[二] SSDT hook](http://bbs.pediy.com/showthread.php?t=58199)
* [【原创】用 DDDK 编写驱动,修改 SSDT 表 HOOK NTDebugActiveProcess 函数](http://bbs.pediy.com/showthread.php?t=50052)
* [【转帖】城里城外看 SSDT](http://bbs.pediy.com/showthread.php?t=51105)

关于 HOOK 的文章实在是太多了，但是不同的文章由于作者的不同，导致文章的写作风格不同，代码的风格也各不相同，这样就导致了同样的技术，不同的作者用代码表现出来的形式却大相径庭，这样对我们初学者来讲实在是很不方便，于是便有了打造自己的 HOOK 引擎的想法，将技术细节封装起来，因为这个本来就不用去修改，需要修改的仅仅是用户自定义的 HOOK 例程

这个也就是我的 HOOK 引擎的基本思想，将 HOOK 的具体实现封装在引擎中，用户想 HOOK 某个函数时，不用去想 HOOK 的技术细节了，直接编写自己的 HOOK 函数然后调用引擎提供的某个接口即可

因为是第一次写，就写个最简单的，SSDT 的 HOOK

譬如如果用户想把 `NtOpenProcess` HOOK 成自己的 `MyNtOpenProcess`

用户只需要在 `DriverEntry` 中调用 `HookService((ULONG)ZwOpenProcess, (ULONG)MyNtOpenProcess)`

然后编写自己的钩子函数即可 接口很简单也易于实现

这里是个调用的例子

代码:

```
// Unload 例程 卸载钩子
VOID Unload(IN PDRIVER_OBJECT DriverObject)
{
  KdPrint(("Unload Routine.\n"));
  UnHookService((ULONG)ZwSetInformationFile);
  UnHookService((ULONG)ZwOpenProcess);
}

// DriverEntry 例程 初始化并安装钩子
NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject,
             IN PUNICODE_STRING RegistryPath)
{
  DriverObject->DriverUnload = Unload;
  InitServicesTable();
  HookService((ULONG)ZwSetInformationFile, (ULONG)MyZwSetInformationFile);
  HookService((ULONG)ZwOpenProcess, (ULONG)MyNtOpenProcess);

  return STATUS_SUCCESS;
}
```

然后只需要关注自己的函数了，如下编写了两个示例，第一个例子 HOOK 了 `ZwSetInformationFile` 保护 `test.txt` 文件不被删除，第二个例子 HOOK 了 `NtOpenProcess` 保护 PID 大于 1000 的进程不被结束，代码都是参考前人的，我只是在这里做个总结罢了

代码:

```
// 定义 HOOK 的函数原型
typedef
NTSTATUS
(__stdcall *ZWSETINFORMATIONFILE)(IN HANDLE FileHandle,
          OUT PIO_STATUS_BLOCK IoStatusBlock,
          IN PVOID FileInformation,
          IN ULONG Length,
          IN FILE_INFORMATION_CLASS FileInformationClass);
typedef
NTSTATUS
(__stdcall *NTOPENPROCESS)( OUT PHANDLE ProcessHandle,
          IN ACCESS_MASK AccessMask,
          IN POBJECT_ATTRIBUTES ObjectAttributes,
          IN PCLIENT_ID ClientId);

// 对于 ntddk.h 中未定义的函数
// 可以根据<<Undocument>>一书在这里给出定义
NTSYSAPI
NTSTATUS
NTAPI 
ZwOpenProcess( OUT PHANDLE ProcessHandle,
        IN ACCESS_MASK AccessMask,
        IN POBJECT_ATTRIBUTES ObjectAttributes,
               IN PCLIENT_ID ClientId);

// ==================================================
// 用户自定义 HOOK 例程
NTSTATUS MyZwSetInformationFile(IN HANDLE FileHandle,
      OUT PIO_STATUS_BLOCK IoStatusBlock,
      IN PVOID FileInformation,
      IN ULONG Length,
      IN FILE_INFORMATION_CLASS FileInformationClass)
{
  PFILE_OBJECT pFileObject;

  // 在 OldServiceAddressTable 中取出原服务函数地址
  ZWSETINFORMATIONFILE OldZwSetInformationFile = 
    (ZWSETINFORMATIONFILE)OldServiceAddressTable[SERVICE_ID(ZwSetInformationFile)];
  
  NTSTATUS ret = ObReferenceObjectByHandle(FileHandle, 
             GENERIC_READ,
             *IoFileObjectType, 
             KernelMode, 
             (PVOID*)&pFileObject, 
             0); 
  if(NT_SUCCESS(ret))
  {
    KdPrint(("%S opened.\n", pFileObject->FileName.Buffer));
    if (wcsstr(pFileObject->FileName.Buffer, L"test.txt"))
    {
      KdPrint(("test.txt opened. Deny it.\n"));
      return STATUS_ACCESS_DENIED;
    }
  }
      ObDereferenceObject(pFileObject);   
  // 调用原服务函数
  return OldZwSetInformationFile( FileHandle, IoStatusBlock, FileInformation, 
     Length, FileInformationClass);
}

NTSTATUS MyNtOpenProcess(OUT PHANDLE ProcessHandle,
      IN ACCESS_MASK DesiredAccess,
      IN POBJECT_ATTRIBUTES ObjectAttributes,
      IN PCLIENT_ID ClientId )
{
  NTSTATUS rc;
  ULONG PID;
  KPROCESSOR_MODE PreMode;
  
  NTOPENPROCESS OldNtOpenProcess = 
    (NTOPENPROCESS)OldServiceAddressTable[SERVICE_ID(ZwOpenProcess)];
  PreMode = ExGetPreviousMode();
  if(PreMode != KernelMode)
  {
    __try
    {
      ProbeForRead(ClientId, sizeof(CLIENT_ID), sizeof(ULONG));
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
      return GetExceptionCode();
    }
  }
  if(ClientId != NULL)
  {
    PID = (ULONG)ClientId->UniqueProcess;
        if(PID > 1000)
        {
      return STATUS_ACCESS_DENIED;
        }
  }
  return OldNtOpenProcess(ProcessHandle, DesiredAccess, ObjectAttributes, ClientId);
}
```

上面的代码都是对前人的代码进行了一些整合而得到的，仅仅是对引擎做个测试而已

然后说明一点，就是 `HookService()` 的第一个参数都应该是以 Zw 开头的函数，因为代码是根据这个函数来计算服务 ID 的

还有就是在调用 `HookService()` 之前应该先调用 `InitServicesTalbe()` 来对 SSDT 进行一次性的保存，避免后面多次 HOOK 就要保存多次

附件中是全部代码 写的很烂 还望大大们批评指正。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `ssdt_hook.rar` |

## 打造自己的 HOOK 引擎 之二 --- HOOK CHAIN 处理

本来第二次想做个 IDT 的 HOOK 引擎的，但是由于上次那个 SSDT 的引擎太搓，很多东西没处理好，最严重的问题就是对 HOOK 链的处理存在问题，安全性太差。

经过几位热心网友的指点，偶对 HOOK 引擎的理解更深一步，在这里表示感谢。

这次就对 HOOK CHAIN 的处理提出一些我自己的想法和实现，有一些也仅仅停留在想法上，依我现在的能力还不能做到十全十美，还希望各位提出更好的解决办法。没什么技术含量，高手见笑了。

在论坛搜了下，发现这方面的资料相当贫乏，基本上没有对这个话题进行讨论的主题。这里我相当与做个引子，提出 HOOK CHAIN 的概念，希望能抛砖引玉。我自己的理解也很有限，希望大家批评指正。

其实 HOOK 的本质大多相同，我理解的 HOOK 无非两种：一种我称之为地址替换，像 SSDT, IDT, IAT, EAT 等等等等的钩子都属于这一种，另外像 `jmp/call xxx` 型的 HOOK 也可以归为这一种，只是这里的地址不是以某种表的形式存在内存中，而仅仅是一个地址，这个地址可以简单的称作 HOOK 点，选取一个好的 HOOK 点是另一个重要话题。另外一种 HOOK 就是内联(inline)，这种 HOOK 灵活性大，不是对地址做手脚，而是直接修改代码，将代码变成地址，尽管理论上可以在任意地方下钩，但是处理起来会有些不同，管理也会比较麻烦。

尽管存在不同形式的 HOOK，但是有一点却是相同的，那就是所有的 HOOK 都会面临多重 HOOK 的情形，多重 HOOK 形成 HOOK 链，于是对 HOOK 链的处理便成了每种 HOOK 都必须处理的问题。

首先还是从上一篇的 SSDT 的 HOOK 说起吧 在这里我对 SSDT HOOK 做了几幅图，讨论起来更方便些吧

我们看第一张图 作为最原始的 SSDT 服务函数的地址我依次用数字代替

![](img/984839a4e8e4e32130c57d653d541637.jpg)

下面我们对服务函数 3 下钩,下钩的函数名为 `hook_1`，`hook_1` 会将服务函数 3 的原始地址保存起来，即有 `original = 3`，需要的时候会调用原始的函数，即在新的 HOOK 例程中会有这样的语句 `call original` 即 `call 3`。另外不难理解，`hook_1` 的卸载例程里需要做的事情就是将服务函数 3 的地址替换回去，用符号★代表 SSDT 中服务函数 3 的地址，即有★ = `original` = 3。

![](img/e5930453191b0ee558e69c83d71b6851.jpg)

当然如果只有这么一个钩子，系统的所有安全责任由它负责就行了，那就不存在我们所说的安全隐患了。但是往往情况不是这样，对于同一个地方，你 HOOK，我 HOOK，他也 HOOK，但是安全性呢，你管你的，我管我的，他管他的，到最后整体的安全性没有人来负责了，BSOD 了应该找谁？

所以 HOOK 是存在风险的，下面就依次把我们的 HOOK 链建立起来，并提出几个存在的问题。

譬如这时另一个叫做 `hook_2` 的函数也想对服务函数 3 下钩，当然他可以检测出这个已经存在的 HOOK 的，但是检测出来了又能怎么样呢？有三种方法：第一，就此罢手，不钩了。这是最简单的方法，也最安全，但是很显然，这个多不公平啊，你想啊，同样在 `RING0`，为什么别人就能钩我却不能钩，就因为别人先我一步？想想就不爽。而且如果那是一个很好的 HOOK 点，就这样放弃未免太可惜了。第二种方法，把他卸了，安装我的钩子。这种想法很简单也很暴力，相当不和谐，而且仍然存在问题，如果每个人都这样，挂钩前就把别人的卸了，别人挂钩还不是可以把我们的钩子给卸了！一个解决方法是加上定时检测，这个应该是个解决方法，后面再做讨论。第三种，不管他，我也钩上，卸载时把他还原，调用原来的函数呢就调用他。这样就形成了链，于是就有了今天的话题。

我们假设 `hook_2` 是按第三种方法对服务函数 3 下钩的，得到下面的示意图

![](img/0b1e3532296ffe7e5059161687ea6fc9.jpg)

然后 `hook_3` 同样要对这里下钩 得到下图

![](img/5ad25a8484edbd9b7068d26429bacf9d.jpg)

到这里钩子链已经显现出来了，但是很显然这是个单链，甚至单链都称不上。这里的链只是形象上的说法，并不是传统意义的链表，链表中通过 `NextNode` 指向下一结点，HOOK 链而是通过 `call next_hook` 将所以的 `Hook_Routine` 形成一个链状结构。

我们来分析一下这个 HOOK 链的特点，首先一个显著的特点是：谁最后下钩，谁就最先被调用。究其原因是因为我们的 HOOK 都是从 SSDT 中的某个欲下钩的地方开始的，我们不能从 HOOK 链中的某点进行下钩，也就是说 HOOK 链不存在插入结点算法。HOOK 链的另一个特点是链结构不稳定，随时都有可能出现某个结点从链中断掉，而且前面的结点并不知道，导致前面的结点仍然指向那个断掉的结点，从而导致安全问题。也就是说 HOOK 链也不存在删除结点算法。

这样分析下来，HOOK 链的根本问题就在于不能实现或很难实现链表的两种基本算法，插入和删除。回忆一下数据结构课程中学习的链表，我们就会发现原因可以归结为一点，无论是插入和删除都需要遍历链表，所以如果实现 HOOK 链的遍历，那么 HOOK 链的管理就会大大简单化。

下面再从另外三个角度并对照上面的 `hook_3` 那幅图对 HOOK 链进一步讨论。

### 第一：卸载例程.



# 1. HOOK 链的卸载与维护

这个最简单的处理方法就是写回 `SSDT` 中被 `HOOK` 的那项，然后卸载掉。存在的问题就是上面讲的，上一个 `HOOK` 例程还是调用这个函数，造成 `BSOD`。譬如 `hook_2` 卸载了，但是 `hook_3` 的 `Hook Routine` 里仍然还是 `call hook_2`。另外一种方法，依赖上面的遍历算法，遍历到上一 `HOOK` 例程，修改指针指向下一 `HOOK` 例程，就像从单链表中删除一个结点一样。譬如 `hook_1` 要卸载，可以从 `SSDT` 中找到 `hook_3` 的地址，再从 `hook_3` 的内存空间找到 `hook_2` 的地址，再从 `hook_2` 的内存空间找到 `call hook_1` 修改为 `call 3`，但是，这种方法很难实现，如果能实现，对检测钩子的存在意义应该是重大的。

## 2. 调用原函数

当然这是针对需要调用原函数进行讨论的。如果不需要对原函数进行调用,显然不存在这个问题。一个保险的方法是自己编写原服务函数,但是这样也就失去了链这个概念，不用去调用下一个 `HOOK` 例程了。还有一种方法简单的方法，调用原函数前对 `SSDT` 做检查，如果 `SSDT` 中保存的是我的地址，显然没有新的 `HOOK` 加载也没有旧的 `HOOK` 卸载，即调用下一 `HOOK` 例程。如果发现 `SSDT` 中的表项已经被修改了，这个时候无法判断是有新的 `HOOK` 加载了，还是有旧的 `HOOK` 卸载了，保险的方法是调用最原始的服务函数，可以通过 `ntoskrnl.exe` 获取最原始的服务函数地址。这样下来，我们分析下这个 `HOOK` 链，`hook_1` 检测到 `hook_2` 加载，便开始 `call 3`，(3 是最原始的服务地址)。而当 `hook_2` 检测到 `hook_3` 加载，也开始 `call 3`，然后 `hook_1` 又卸载了，`hook_3` 检测到以后，也开始 `call 3`，如此而来，整个 `HOOK` 链已经七零八落，谈不上链了。

## 3. 保存原函数地址

我们可以看到，对一个钩子例程来讲，最简单的方法是将原函数地址保存起来，如果需要调用下个钩子例程，即 `call` 这个地址。对这个钩子，他不知道这个地址是合法还是非法的。现在想，如果每个钩子例程都将原函数地址保存在统一的区域，在这个区域对各个地址进行管理不是就简单了吗？但是怎么让这个区域起到作用，这个是我想到的另外一个解决方案，使用 `HookMgr`。

最后对上面提到的两种解决方案画两幅草图，实现起来还是有很多问题，我在这里只是提出一些思路罢了，如果您有更好的方法，欢迎联系我，也可以跟帖讨论。能力有限，错误之处还请见谅。

### 第一种方案：定时检测

![](img/9d77eda0c34fc60c305888f6829820e9.jpg)

这种方案已经跟 `HOOK` 链无关了。而是和 `CPU` 的分时复用原理有些类似，将被 `HOOK` 的地方理解为 `CPU`，每隔一段时间即被 `HOOK` 修改一次。和 `CPU` 不同的是，`CPU` 是主动分配时间片，这里是各个 `HOOK` 去抢 `CPU`，譬如 `hook_1` 每隔 `xms` 改一次，`hook_2` 每隔 `yms` 改一次，`hook_3` 每隔 `zms` 改一次，有可能会出现同时修改的情况，所以修改前要上锁。

不足之处是每个 `HOOK` 的时间都不一样，不便于管理。而且系统的开销可能很大。或许可以设计一个针对每个 `HOOK` 的时间片管理器。

### 第二种方案：钩子管理器

![](img/cde65ba2ab32e4a778ee7f06fe159389.jpg)

如上图所示，在每个 `HOOK` 点上建立一个钩子管理器（`HookMgr`），钩子管理器负责钩子例程的创建和退出，集中管理各种钩子例程。原理感觉和 `SetWindowsHook` 差不多，我没有深入研究过，偏颇之处还请见谅。当一个钩子例程创建时，可以向 `HookMgr` 发出请求，或者是某个钩子修改 `HookMgr` 所在的 `HOOK` 点时，会被 `HookMgr` 检测到，然后在管理器中自动创建 `HOOK` 链进行管理。当钩子例程卸载时，同样可以发出卸载请求，或者是某个钩子卸载时，被 `HookMgr` 检测到，`HookMgr` 便删除相应的钩子结点。而对于调用下一个 `HOOK` 例程，可以直接调用 `Mgr` 提供的接口 `CallNextHook`。

管理器的思路很明晰，但处理起来会面临很多很多问题，偶的能力只能分析到这里了，要实现这个钩子管理器，不是我这个初学者的能力所及的。。。

好了，分析就到此为止了。

现在才意识到钩子的管理也有这么多的问题，要编写安全的钩子，仅仅学习钩子的原理技术还是远远不够的。

本人能力有限，失误之处还望批评指正。

## 4. 附件

![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)

`hook chain.rar`
