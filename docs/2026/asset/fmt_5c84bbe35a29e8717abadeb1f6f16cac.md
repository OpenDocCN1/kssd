

# [原创]记一次有教益的 vs2022 内存分配失败崩溃分析

## 前言

之前一直以为`64`位进程很难出现内存分配异常，因为`64`位进程的虚拟内存空间非常大（总共`64`位，目前只用了`48`位，也就是`256TB`，用户态可以使用一半，也就是`128TB`）。没想到，前一阵子居然遇到了`vs2022`（`vs`终于有了`64`位的版本）分配内存失败的情况。分析到最后是因为分配`MEM_COMMIT`类型的内存失败导致的异常。一起来看看吧。

> **说明：** 本文很早就写了草稿，一直没时间整理发布，Finally~

## vs2022 卡死了

前一阵子，我在使用`vs2022`编辑代码的时候，不知道做了什么操作导致`vs2022`卡住了。过了一段时间后，`procdump`自动运行起来了（因为我把`procdump`设置为了`JIT`调试器。当有进程崩溃的时候，`procdump`会自动执行转储操作），等了好一会儿才退出。查看`d:\dumps\`目录下的转储文件，真是不看不知道，一看吓一跳，对应的转储文件居然有将近`20GB`。

看来，`vs2022`应该是遇到了内存方面的异常。

![vs2022-dump-file](upload/attach/202310/873494_P3R9WXGDQ83GMQ7.webp)

## 查看调用栈

使用`windbg`打开对应的转储文件，执行`k`查看调用栈。如下图：

![callstack-of-new-memory-throw-exception](upload/attach/202310/873494_WC59RPP3YJWHEMZ.webp)

可以很明显的看到是在调用`new()`分配内存失败后抛出了异常。再多查看几个栈帧，可以发现是由`vector`的`_Emplace_reallocate()`函数触发的内存分配。`vs2022`是`64`位的进程，虚拟内存空间可以说是大的离谱。居然内存分配会失败！有点意思，那到底分配了多大内存呢？

## 分配了多大内存？

查看栈帧`0a`和`0b`的反汇编代码，如下图：

![search-new-size](upload/attach/202310/873494_TZG7XQAJ4XUDSSF.webp)

从上图可知，`new()`的参数`rcx`来自栈帧`0a`中的`rax`，`rax`又来自`rcx+0x27`。`rcx`来自栈帧`0b`中的`rax`，`rax`是`vcpkg!std::_Get_size_of_n<16>()`的返回值，`vcpkg!std::_Get_size_of_n<16>()`的参数`rcx`来自`rax`，而这个`rax`保存到栈上`rsp+0x78`的位置。可以先拿到`rsp+0x78`处的值，然后一步步推导出传递给`new()`的参数。

在开始之前，先了解一下`vcpkg!std::_Get_size_of_n<size_t>()`的逻辑。`vcpkg!std::_Get_size_of_n<size_t>()`是一个模板函数，模板参数是元素类型的大小，这里是`0n16`。该函数的实现很简单，就是返回`_Count * _Ty_size`（当然还有一些界限检查，下面是精简后的代码）。

```c++
template <size_t _Ty_size>
constexpr size_t _Get_size_of_n(const size_t _Count) {
    return _Count * _Ty_size;
}
```

可以通过查看栈帧`0b` `rsp+0x78`位置的值得到要分配的元素个数，然后乘以`0n16`就可以得到最终传递给`new()`的值。

![get-allocate-size-pass-to-new](upload/attach/202310/873494_45VQW5Z9H5EGPMS.webp)

计算后得知，本次分配的空间大约为`923MB`。

> **注意：** 虽然分配的内存空间不是超级大，但是本次尝试分配的元素个数是`0x039bc719`，通过`.formats 0x039bc719`可以查看对应的十进制数是`60540697`，也就是大约`6`千万个对象！

说实话，分析到这里的时候，我是有点儿没底气的。`vs2022`可是`64`位的进程啊！没想到只分配大概`923MB`就失败了！带着这个疑问，继续查看当前进程的地址空间情况。

## 查看空闲空间

可以使用`!address -summary`看一下内存使用情况，如下图：

![address-summary](upload/attach/202310/873494_MW2CYE8GPNSRKQN.webp)

可以发现最大的空闲空间大概有`119.96TB`这么大。

> **说明：** 既可以通过上面的`Largest Region by Usage`查看最大的空闲空间，还可以通过`!address -f:Free -c:".if(%3 > 0x80000000) {.echo %1 %2 %3}"`显示出大于`0x80000000`的空闲段，如下图：
>
> ![get-address-region-larger-than-0x80000000](upload/attach/202310/873494_EE8SN56GJCS5PKU.webp)
>
> 顺便说一句，第一次执行的时候是真的慢！

既然有足够大的空闲空间，为什么分配内存还会失败呢？看到这里我更疑惑了，同时心里有了另外一个疑问——在`x64`进程中，用户态代码到底可以分配多大内存？

## 测试内存分配

于是我写了一段使用`malloc`分配内存的测试代码。如下：

```c++
int main()
{
    size_t size = 1024 * 1024 * 1024;
    size *= 20;
    auto p = malloc(size);
    return 0;
}
```

经过几次调整后发现，大概分配`20GB`的时候就失败了。

当使用`malloc()`分配大块内存时，会调用`ntdll!NtAllocateVirtualMemory()`进行分配。使用`windbg`运行程序，当执行到`auto p = malloc(size);`这一行的时候，执行`bp ntdll!NtAllocateVirtualMemory`设置好断点。然后执行`g`让程序继续运行，很快就中断下来了。执行`gu`跳出当前函数，使用`r`命令查看寄存器，主要关注`rax`，因为它保存了函数的返回值。发现`rax`的值是`00000000c000012d`。由`ntdll!NtAllocateVirtualMemory()`的函数原型可知，返回值是`NTSTATUS`类型的。

```c++
NTSTATUS NtAllocateVirtualMemory(
    [in] HANDLE ProcessHandle,
    [in, out] PVOID *BaseAddress,
    [in] ULONG_PTR ZeroBits,
    [in, out] PSIZE_T RegionSize,
    [in] ULONG AllocationType,
    [in] ULONG Protect
);
```

查看官方文档可知，`0xc000012d`的意义是`STATUS_COMMITMENT_LIMIT`。

![c000012d_status_commitment_limit](upload/attach/202310/873494_7NTURHFN7GSVXSF.webp)

根据`Description`列的描述可知，增大页面文件的大小可能会有帮助。看到这里的时候，我突然想起来，好像`malloc()`在调用`ntdll!NtAllocateVirtualMemory()`的时候，传递的`AllocationType`应该是包含`MEM_COMMIT`标志的（因为可以直接对返回的地址空间进行读写操作了）。而分配这种类型的内存，`windows`会检查是否有足够的内存（物理内存+页文件）支撑，如果剩余的物理内存+页文件（会被系统中的所有进程共同使用）的大小不能满足本次分配，那么会报错。

如果把`MEM_COMMIT`换成`MEM_RESERVE`，能分配多大的内存呢？

## 测试 MEM_RESERVE 最大分配尺寸

于是我又写了一段测试代码，直接调用`VirtualAlloc()`进行内存分配。测试代码如下：

```c++
#include <iostream>
#include "windows.h"

const size_t one_gb = 1LL * 1024LL * 1024LL * 1024LL; // 1 GB

double ToGb(size_t bytes)
{
    return bytes / 1024.0 / 1024.0 / 1024.0;
}

double ToTb(size_t bytes)
{
    return bytes / 1024.0 / 1024.0 / 1024.0 / 1024.0;
}

size_t TestMaxAllocateMemory(size_t init_size, size_t decrease_size, DWORD allocation_type)
{
    LPVOID p = nullptr;
    while (nullptr == (p = VirtualAlloc(nullptr, init_size, allocation_type, PAGE_READWRITE)))
    {
        auto last_error = GetLastError();
        std::cout << "allocate " << ToGb(init_size) << " GB failed. last error:" << last_error
                  << ". try allocate " << ToGb(init_size - decrease_size) << " GB." << std::endl;
        init_size -= decrease_size;
    }
    return init_size;
}

void TestMaxReserveMemory()
{
    size_t size_reserve = 128LL * 1024LL * one_gb; // 128 TB
    size_reserve = TestMaxAllocateMemory(size_reserve, one_gb, MEM_RESERVE);
    std::cout << "allocate " << ToTb(size_reserve) << " TB reserver memory success." << std::endl;
}

int main()
{
    TestMaxReserveMemory();
    std::getchar();
    return 0;
}
```

运行结果如下：

![test-max-reserve-memory](upload/attach/202310/873494_X6EPZ562ZEXKADB.webp)

当分配类型是`MEM_RESERVE`的时候，一次性最多可以分配大概`126TB`的虚拟内存。基本符合之前的认知。

> **注意：** 每次运行结果不完全一致，不过相差不多。

## 测试 MEM_COMMIT 最大分配尺寸

为了更好的展示不同情况下分配`MEM_COMMIT`的结果，我又添加如下测试代码：

```c++
void TestMaxCommitMemory()
{
    size_t size_commit = 128LL * one_gb; // 128 GB
    size_commit = TestMaxAllocateMemory(size_commit, one_gb, MEM_COMMIT);
    std::cout << "allocate " << ToGb(size_commit) << " GB commit memory success." << std::endl;
}

int main()
{
    //TestMaxReserveMemory();
    TestMaxCommitMemory();
    std::getchar();
    return 0;
}
```

我分别在不同系统内存占用的情况下运行了三次，三次运行结果如下：

当系统内存相对充裕的时候，运行结果如下：

![allocate-memory-commit-1](upload/attach/202310/873494_ASVB4R7JKTYSS82.webp)

当系统内存被消耗了一部分的时候，运行结果如下：

![allocate-memory-commit-2](upload/attach/202310/873494_JA77HRZ7ZEWPXRB.webp)

当使用`TestLimit64 -m 2048 -c 10`分配`20GB`的`MEM_COMMIT`内存后，运行结果如下：

![allocate-memory-commit-3](upload/attach/202310/873494_FY8FPN644ZCTK9W.webp)

> **画外音：** 当我尝试模拟系统内存吃紧的时候，突然想起来`Testlimit`就是用来测试各种资源极限的。`-m`模拟的是分配`MEM_COMMIT`类型内存的，`-r`模拟的是分配`MEM_RESERVER`类型内存的。`-c`是分配数量，如果不指定，则无限分配。

现在还剩下两个问题待证实：

1. 明确`malloc()`调用`ntdll!NtAllocateVirtualMemory()`传递的`AllocationType`参数。

2. 确定分配失败时，剩余物理内存+页文件的大小是否足够大。

## 明确 AllocationType

使用`k 3`显示`3`个调用栈帧。栈帧`01`会调用`ntdll!NtAllocateVirtualMemory()`，所以栈帧`01`会传递参数给`ntdll!NtAllocateVirtualMemory()`。使用`ub 00007ffe30492762 L1a`查看相关调用代码，可以发现`AllocationType`保存在`rsp+0x20`的位置，`Protect`保存在`rsp+0x28`的位置，前四个参数分别由`rcx, rdx, r8, r9`进行传递。

![view-AllocationType-param](upload/attach/202310/873494_U4V2ZVNUG9CYTPH.webp)

由此，可以确定之前的理解是正确的。`malloc()`调用`ntdll!NtAllocateVirtualMemory()`时，`AllocationType`的值是`MEM_COMMIT`。

## 剩余的内存是否足够大

因为转储文件只包含当前进程的信息，没有系统级的转储文件，不好确认系统中的其它进程的内存使用情况。但是转储文件的大小已经达到了`18.3GB`，本次尝试分配的大小是`923MB`，加上`18.3GB`，大概是`19GB`。

通过`.time`命令，可以发现系统已经运行了接近`2`天，当前进程已经运行了大概`18.5`个小时。由系统开机时间可以推算，当时应该有不少进程在运行（我的系统上，`chrome`和`firefox`基本是常开状态）。

![show-system-and-process-running-time](upload/attach/202310/873494_89BQ8UCYYGRTRAD.webp)

而且在`vs2022`无响应的时候，整个系统确实有些卡顿。以上种种迹象表明，系统当时的内存吃紧。这时候出现内存分配异常，确实合情合理。

## 彩蛋

前几天，客户的程序也遇到了一个类似的问题。她机器上内存紧张的时候，执行程序中的一个功能需要分配`196MB`的内存，由于物理内存不足，失败了。因为我之前已经调查过类似的问题了，在调查客户的问题的时候，非常快速而且有信心。我想这就是写文章记录的价值之一吧！

## 总结

- `procdump`真是事后调试的好帮手。以管理员权限运行`procdump -i -ma d:\dumps\`即可安装。`-i`表示安装（如果要卸载，可以使用`-u`参数）。`-ma`表示执行完整转储，`d:\dumps\`表示`.dmp`文件保存的位置。

- 相较于`32`位进程的`4GB`（`2`的`32`次方）虚拟内存空间而言，`64`位进程的虚拟内存空间超级大，目前是`256TB`（总共`64`位，目前只用了`48`位），内核态和用户态平均分，用户态可以使用一半，也就是`128TB`。

- 如果使用`malloc()`或者`new()`（内部会调用`malloc()`）分配的内存大小超出堆阈值，那么内部会使用`NtAllocateVirtualMemory()`分配内存，而且`AllocationType`的值是`MEM_COMMIT`。分配`MEM_COMMIT`类型的内存是受**物理内存+分页文件大小**限制的。

## 参考资料

NTSTATUS Values
