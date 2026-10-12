
# 内核读写只读内存方法总结 [Delphi 描述]

> 原文：[`www.pediy.com/kssd/pediy10/63791.html`](https://www.pediy.com/kssd/pediy10/63791.html)

以下代码均以 `Delphi` 描述...至于为什么...

首先我是一个 `Delphi` Coder...虽然我大部分时间使用的是 `ASM` 编译器和 `C` 编译器。

但是我喜欢 `Delphi`...好了不废话了...

已知的三种方法:如果各位有更好的意见欢迎大家提出。

## 1. 使内存可读写

### 1.1 `stl` + `cr0`

这个方法大家想必经常使用...

![](img/ff3c0f73f8ef94ec7919e00dabd13b55.jpg)

(参考 I-32.3A 文档)

由于 `cr0` 是一个 32 位寄存器...假设大家的 CPU 是 32 位的.没有 64 位测试环境。

按照图这个结构我们可以了解到的信息：

- 第 16 位: `WP` (Write Protect), 当设置为 1 时只提供读页权限。
- 第 0 位: `PE` (Paging), 当设置为 1 时提供分页。
- 第 1 位: `MP` (Protection Enable), 当设置为 1 时进入保护模式。

按照这个说明写出代码:

//1 关闭写保护
```
asm
  push eax
  mov eax, CR0
  and eax, 0FFFEFFFFh
  mov CR0, eax
  pop eax
end;
```

//2 打开写保护
```
asm
  push eax
  mov eax, CR0
  or eax, NOT 0FFFEFFFFh
  mov CR0, eax
  pop eax
end;
```

也许大家看得有点模糊.其实这个代码就是修改第 16 位。

`NOT 0FFFEFFFFh` = `00010000h`

因为要设置第 16 位为 1...所以结果也就是代码那种效果...

哎～我也不知道说什么了.也就是位操作你也可以直接使用：

//1 关闭写保护
```
asm
  push  eax
  mov   eax, cr0
  and   eax, not 000010000h
  mov   cr0, eax
  pop   eax
end;
```

//2 打开写保护
```
asm
  push  eax
  mov   eax, cr0
  or    eax, 000010000h
  mov   cr0, eax
  pop   eax
end;
```

一般使用的时候我们都会加上 `cli` 和 `sti`，由于这两个指令只能控制当前 CPU 对于多核系统是无法起到有效的互斥的...所以一般都是配合"自转锁"使用。

关于自转锁的话题我们下面会详细的说的。

### 1.2 通过内存描述表 (MDL)

通过内存描述表 (MDL) 中描述一块内存区域，MDL 包含此内存区域的起始地址, 拥有者进程, 字节数量以及标志.(据说是 Bill 提供的方法.难道和 `VirtualProtect` 一样?)

代码:

```
type
  PMDL = ^_MDL;
  _MDL = packed record
    Next: PMDL;
    Size: USHORT;
    MdlFlags: USHORT;
    Process: Pointer;
    MappedSystemVa: PVOID;
    StartVa: PVOID;
    ByteCount: ULONG;
    ByteOffset: ULONG;
  end;
  MDL = _MDL;

const
  MDL_MAPPED_TO_SYSTEM_VA     = $0001;
  MDL_PAGES_LOCKED            = $0002;
  MDL_SOURCE_IS_NONPAGED_POOL = $0004;
  MDL_ALLOCATED_FIXED_SIZE    = $0008;
  MDL_PARTIAL                 = $0010;
  MDL_PARTIAL_HAS_BEEN_MAPPED = $0020;
  MDL_IO_PAGE_READ            = $0040;
  MDL_WRITE_OPERATION         = $0080;
  MDL_PARENT_MAPPED_SYSTEM_VA = $0100;
  MDL_LOCK_HELD               = $0200;
  MDL_PHYSICAL_VIEW           = $0400;
  MDL_IO_SPACE                = $0800;
  MDL_NETWORK_HEADER          = $1000;
  MDL_MAPPING_CAN_FAIL        = $2000;
  MDL_ALLOCATED_MUST_SUCCEED  = $4000;

//  读写只读内存(源于 Gates 大叔)
function WriteReadOnlyMemoryGates(lpDest, lpSource: Pointer; Length: Integer): NTSTATUS;
var
  tempSpinLock: KSPIN_LOCK;
  oldirql: KIRQL;
  mdl: PMDL;
  writableAddress: Pointer;
begin
  Result := STATUS_UNSUCCESSFUL;
  mdl := MmCreateMdl(nil, lpDest, Length);
  if (mdl <> nil) then
  begin
    MmBuildMdlForNonPagedPool(mdl);
    mdl^.MdlFlags := mdl^.MdlFlags or MDL_MAPPED_TO_SYSTEM_VA;
    writableAddress := MmMapLockedPages(mdl, KernelMode);
    if (writableAddress <> nil) then
    begin
      oldirql := 0;

      KeInitializeSpinLock(@tempSpinLock);
      fast_KfAcquireSpinLock(@tempSpinLock);
      memcpy(writableAddress, lpSource, Length);
      fast_KfReleaseSpinLock(@tempSpinLock, oldirql);

      MmUnmapLockedPages(writableAddress, mdl);
      Result := STATUS_SUCCESS;

    end;
    MmUnlockPages(mdl);
    IoFreeMdl(mdl);
  end;
end;
```

关键一步就是修改 `mdl` 的属性.让他可写....

下面来看看第三种方法

### 1.3 使用 `IoAllocateMdl`

使用 `IoAllocateMdl` 来得到可写内存...

源于 Mark 早期写的代码...我这里直接翻译了...至于原理大家参考一下 DDK 吧...我发现自己表达能力有限...

代码:

```
//  写只读内存(源于 Mark 代码)
function WriteReadOnlyMemoryMark(lpDest, lpSource: Pointer; Length: Integer): NTSTATUS;
var
  tempSpinLock: KSPIN_LOCK;
  oldirql: KIRQL;
  mdl: PMDL;
  writableAddress: Pointer;
begin
  Result := STATUS_UNSUCCESSFUL;
  mdl := IoAllocateMdl(lpDest, Length, False, False, nil);
  if (mdl <> nil) then
  begin
    MmBuildMdlForNonPagedPool(mdl);
    MmProbeAndLockPages(mdl, KernelMode, IoWriteAccess);
    writableAddress := MmMapLockedPages(mdl, KernelMode);
    if (writableAddress <> nil) then
    begin
      oldirql := 0;

      KeInitializeSpinLock(@tempSpinLock);
      fast_KfAcquireSpinLock(@tempSpinLock);
      memcpy(writableAddress, lpSource, Length);
      fast_KfReleaseSpinLock(@tempSpinLock, oldirql);

      MmUnmapLockedPages(writableAddress, mdl);
      Result := STATUS_SUCCESS;
    end;
    MmUnlockPages(mdl);
    IoFreeMdl(mdl);
  end;
end;
```

代码源于 Mark 早期编写的代码。

相信看到这份代码和上面的第二种方法相信大家似乎都明白了什么...

是不是觉得两种方法都差不多区别就是在于

一个是直接修改 MDL 属性: `MDL_MAPPED_TO_SYSTEM_VA`

一个是 `MmProbeAndLockPages(mdl, KernelMode, IoWriteAccess);`

来改写属性...其实这两种方法...嘿嘿~自己看看就明白了...

## 2. 改写内存

平时我们修改内存单核下没有什么顾虑.`stl` | 代码 | `cli`。

开关中断以后就可以自己操作了...

因为现在双核和多核越来越兴起了...

这么说吧.内存在没有完全修改完毕的时候...其他线程去读取的话...

就会造成读取错误的数据或者断码...

如果您是单核的话可以直接修改。

### 2.1 `stl` + `cli`

代码:

```
    asm
      cli                               //disable WP bit
      push  eax
      mov   eax, cr0                    //move CR0 register into EAX
      and   eax, not 000010000h         //disable WP bit
      mov   cr0, eax                    //write register back
      pop   eax
    end;

    //改写的代码

    asm
      push  eax                         //enable WP bit
      mov   eax, cr0                    //move CR0 register into EAX
      or    eax, 000010000h            //enable WP bit
      mov   cr0, eax                    //write register back
      pop   eax
      sti
    end;
```

### 2.2 利用原子互斥操作

比如说我们需要 SSDT, Shadow, etc Hook 修改地址等操作的话...我们有一个简单的方法。

使用... Interlocked 系函数。

修改地址推荐使用 `InterlockedExchange`...

释放方法很简单。

代码:

```
ZwOpenProcessNextHook := TZwOpenProcess(InterlockedExchange(SystemServiceName(GetImportFunAddr(@ZwOpenProcess)), LONG(@ZwOpenProcessNextHook)));
```

提示一下这个函数使用的是 `fastcall` 调用方式,如果你使用的是 Delphi 的话建议你最好注意一下使用方法... `fastcall` 和 Delphi 的 `register` 的是有区别的。

`fastcall`:
- 第一个参数: `edx`
- 第二个参数: `ecx`

`register`:
- 第一个参数: `eax`
- 第二个参数: `edx`
- 第三个参数: `ecx`

提供以下 `InterlockedExchange` 的源代码

代码:

```
FORCEINLINE
LONG
FASTCALL
InterlockedExchange(
    IN OUT LONG volatile *Target,
    IN LONG Value
    )
{
    __asm {
        mov     eax, Value
        mov     ecx, Target
        xchg    [ecx], eax
    }
}
```

### 2.3 自转锁

自转锁是内核中的一种同步机制...

关于自转锁的介绍请大家去看《Programming the Microsoft Windows Driver Model》这本书...

这本书刚开始看许多东西暂时还不太理解...

当线程获取自转锁的时候...会将当前线程提升到 `DISPATCH_LEVEL` 级别。

这个时候内核将不会分配时间片给其他的 CPU.这样就不会有其他线程在你写操作的时候去访问你没有写完的地方了...放心大胆的写了...

怎么写?晕...这个问题这个问题...呵呵我可以不回答吗? `memcpy` 总会用吧...

自转锁使用起来很简单...

(有个疑问.为什么获取和释放自转锁的函数在 `hal.dll` 函数中,初始化自转锁的函数却在 `ntoskrnl.exe` 模块中...)

使用方法:

`KeInitializeSpinLock(@TemSpinLockp);`

`KeAcquireSpinLock(@TemSpinLockp, &oldirql);`

.

.

你的操作代码

.

.

`KeReleaseSpinLock(@TemSpinLockp, oldirql);`

如果有其他内核线程已经获取自转锁的时候你的线程是不会被分配到时间片的。

所以你不用担心你会进去破坏代码(除非~除非~你提升到更高的级别...)

当你获取自转锁成功后..其他线程将进入自转状态..你可以理解为互斥 (`EnterCriticalSection`, `LeaveCriticalSection`)

**[警告]** 请千万不要连续使用两次 `KeAcquireSpinLock`!否则机器会(卡)死的很难看的 etc...

其他方法暂时不清楚.暂时由于自转锁的这种特殊效果.

所以大家能不用的时候尽量不要使用...使用的时候尽量减少自转时间.以达到更高的效能...

小弟初学驱动编程.如果有不对的地方请各位大牛补充说明...这里拜过。

特别感谢:

《如何写 windows 系统已保护的内存区域》的作者.由于被转载了不知道多少次当我看到这篇文章的时候...作者已经不知道是谁了...感谢这位无名英雄...谢谢

鸣谢:

`zhuwg`, `FlowerCode`, 农夫大哥, 冷风(烤鸭), 旋转"AV"群里的各位神牛们

希望感兴趣的 Delphi Fans 可以 PM 我大家一起交流一下开发经验...
