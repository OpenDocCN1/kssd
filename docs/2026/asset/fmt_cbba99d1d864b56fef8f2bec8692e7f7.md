
`PIRP` `Irp`;      //+0x15C

`..`                //+0x160 貌似是个字串指针

`..`                //+x0164

`PIRP` `Irp:`      //+0x168

`..`                //+0x16C

`..`                //+0x170

`PIRP` `Irp`;      //+0x174

}Important;

还有几个与游戏线程相关的名单, 不过不怎么重要, 就不列出了

```
//***********************************************************************
```

下面说一下HS的过滤, 以进程相关的操作为例 open read write, 过滤步骤如下:

```
if (当前进程在黑名单中)
{
    if (操作目标是要保护进程)
    {
        if (是游戏进程发起的操作)
        {
            从黑名单中移除自己;
            放行;
        }
        else
        {
            拒绝访问;
        }
    }
    else
    {
        放行
    }
}
else
{
    if (操作目标是要保护进程)
    {
        if (是游戏进程发起的操作)
        {
            放行;
        }
        else
        {
            if (当前进程在白名单中)
            {
                放行
            }
            else
            {
                加入黑名单;
                拒绝访问;
            }
        }
    }
    else
    {
        放行
    }
}
```

乍一看过滤方式够严密, 不过具体实施上却暴露出了一个大的漏洞,

因为在这一系列的操作中它都是进程ID来判断过滤的, 呵呵, 这样PID+1 +2 +3 就能简单地在三层绕过它了

当然这只针对`NtOpenProcess`可行, read write函数是传递的是进程句柄,

HS内部通过`ZwQueryInformationProcess`通过句柄得到进程ID来进行过滤

所以我们还是将自身进程添加到白名单中比较稳妥, 有了上面的白名单结构信息,

我们只需要随便跟踪hs的一个代理函数并找到名单的`ListHead`就OK了

### 反调试

另外一个重点是反调试这边了, 涉及到这方面的内核处理有:

`KiAttachProcess`      `inline Hook`

`NtSetContextThread`    `inline hook`

`NtGetContextThread`    `inline hook`

`PsSuspendThread`      `inline hook`  <win7>

dpc例程

RING3层:

`DbgBreakPoint`      指令修改

`DbgUiRemoteBreakin`    指令修改

我所了解到的只有这些, 也可能还有别的地方, 内核inline Hook的几个函数大家都心知肚明, 不必多说

dpc例程主要用来获取CMOS中的时钟信息, CPU的时钟周期数, 然后进行一些运算, 应该是用来反内核调试用

关于dpc, 我的另一片帖子中有介绍: http://bbs.pediy.com/showthread.php?t=148135

另外RING3层的两个函数的修改也是很猥琐的, 我们在用OD附加到游戏上时会创建一个远程线程,

这个线程就是为了执行`DbgBreakPoint`而存在, 原来的`DbgBreakPoint`指令为 `int 3` , `retn`

也就是执行一个中断指令以中断到调试器, 现在已经被修改为 `retn`, `retn` 直接返回了,

这样调试器就接收不到中断信号

另外一个`DbgUiRemoteBreakin`, 是执行`DbgBreakPoint`前的一个调用, 具体的作用我也没仔细研究,

不过看名称也是中断用的, hs会将这个函数头写入一个jmp, 直接跳转到`LdrShutdownProcess`,

`LdrShutdownProcess`将直接给每个已经加载的dll模块发消息, dll将被卸载

这样就导致了OD一加载, dll等模块就被卸载掉了

关于`KiAttachProcess`  `NtSetContextThread`  `NtGetContextThread`  `PsSuspendThread`如何绕过,

这个是仁者见仁智者见智的了, 不过这几个代理函数中都有这样的指令序列:

![](https://bbs.kanxue.com/upload/attach/201205/440388_4x9iyabmtsf2oea.jpg)

最后的jz指令就跳转到原函数了, 呵呵, 这里就不言自明了吧

```
//***********************************************************************
```

hs内部的一套Hook框架很不错, 当初让我学到了很多东西, 受益匪浅, 在这里也跟大家分享下,

<这框架是我在hs框架基础上完善的>

### 用于索引

```c
typedef enum _SSDTNameIndex{
  ssdtZwCreateFile,
  ssdtZwOpenFile,
  MAX_SSDT_HOOK_ITEM_COUNT
}SSDTNameIndex;
```

```c
#define SERVICE_NAME_LEN  64

typedef struct _SSDTHookInfo{
  ULONG          ServiceIndex;          // ssdt索引
  hintSSDTNameIndex    Number;              // 序号
  PVOID          RealAddress;          // ssdt真实地址
  PVOID          PrevAddress;          // hook ssdt之前地址
  PVOID          ProxyAddress;          // 代理地址
  ULONG          ProxyBusyNow;          // 正忙标志
  BOOLEAN          IsHookInstalled;        // 是否安装了hook
  BOOLEAN          IsInUses;            // 是否启用这个hook
  WCHAR          ServiceName[SERVICE_NAME_LEN];  // ssdt名称
}SSDTHookInfo, *PSSDTHookInfo, *PSSDTOOKINFO;
```

```c
NTSTATUS CollectSSDTInfo()
{
  在这里初始化结构数组, 获取函数索引, 解析PE得到其真实地址等等
}
```

### 安装Hook

```c
NTSTATUS SetupAllSSDTHook()
{
  ..
  for ( i = 0; i < MAX_SSDT_HOOK_ITEM_COUNT; i++ )    // 遍历结构数组
  {
    pEntry = (PSSDTHookInfo)&g_SSDTHookItems[i];
    if ( !pEntry ) continue;
    if ( !pEntry->IsInUses ) continue;          // 根据此标志启动这个函数Hook
    if ( pEntry->IsHookInstalled ) continue;      // is reinstall?
    if ( !pEntry->RealAddress ) continue;
    if ( pEntry->ServiceIndex > KeServiceDescriptorTable->ulNumberOfServices ) continue;
    ..进行Hook
    pEntry->PrevAddress = OldAddress;
    pEntru->IsHookInstalled = TRUE;
    ...
  }
}
```

### 代理例程

```c
NTSTATUS Proxy_ZwOpenFile()
{
  ..
  pEntry = (PSSDTHookInfo)&g_SSDTHookItems[ssdtZwCreateFile];    // 取得对应项
  InterlockedIncrement(&pEntry->ProxyBusyNow);
  ..过滤
  InterlockedDecrement(&pEntry->ProxyBusyNow);
  return status;
}
```

### 恢复Hook

```c
NTSTATUS RestoreAllSSDTHook()
{
  ..
  for ( i = 0; i < MAX_SSDT_HOOK_ITEM_COUNT; i++ )
  {
    pEntry = (PSSDTHookInfo)&g_SSDTHookItems[i];
    if ( !pEntry ) continue;
    if ( !pEntry->IsInUses ) continue;
    if ( !pEntry->IsHookInstalled ) continue;
    if ( !pEntry->RealAddress ) continue;
    ..取当前SSDT地址
    if ( CurrentAddress == pEntry->ProxyAddress )
    {
      // 没有被别人恢复时, 恢复Hook
      if ( MmIsAddressValid(pEntry->PrevAddress) )
      {
        ..使用Hook之前地址恢复
      }
      else
      {
        ..使用真实地址恢复
      }
    }
  }
  // 在这里等待所有代理函数都处理完毕 正常退出
  for ( i = 0; i < MAX_SSDT_HOOK_ITEM_COUNT; i++ )
  {
    pEntry = (PSSDTHookInfo)&g_SSDTHookItems[i];
    if ( !pEntry->IsInUses ) continue;
    if ( pEntry->ProxyBusyNow > 0 )
    {
      DueTime.QuadPart = 0;
      KeInitializeTimerEx(&timer, SynchronizationTimer);
      KeSetTimerEx(&timer, DueTime, 100, NULL);
      for ( j = 0; j < 20; j++ )
      {
        KeWaitForSingleObject(&timer, Executive, KernelMode, FALSE, NULL);
        if ( (LONG)pEntry->ProxyBusyNow <= 0 ) break;
      }
      KeCancelTimer(&timer);
    }
    pEntry->PrevAddress = NULL;
    pEntry->IsHookInstalled = FALSE;
    pEntry->ProxyBusyNow = 0;
  }
}
```

`inline hook`的框架总结起来比较麻烦了, 大家就去看一下IDA数据库自己总结下吧~~![](https://bbs.kanxue.com/view/img/face/27.gif)

[hs.rar](https://bbs.kanxue.com/attach-download-67406.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [01.jpg](javascript:void(0);)

  （37.78kb，3327次下载）

- [hs.rar](javascript:void(0);)

  （1.11MB，889次下载）
