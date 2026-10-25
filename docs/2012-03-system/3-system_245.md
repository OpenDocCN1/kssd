
# fs 寄存器的菜鸟困惑

> 原文：`www.pediy.com/kssd/pediy12/126305.html`

最近在看 SEH 的东西，说道 `fs:[0]` 总是指向链表顶端，于是打开 OD：

```
fs=003B
.....
fs:[0] = 0012FCAC
fs:[4] = 00130000
```

`fs` 寄存器怎么寻址的？

乘 16+偏移?

是指向内存吗？

so confused ..... ![](img/e06ce445d64470eaf3496ba167dfb4ac.jpg "碌眼")

## TEB 结构偏移说明

`FS` 寄存器指向当前活动线程的 TEB 结构（线程结构）。

| 偏移 | 说明 |
| :--- | :--- |
| `000` | 指向 SEH 链指针 |
| `004` | 线程堆栈顶部 |
| `008` | 线程堆栈底部 |
| `00C` | `SubSystemTib` |
| `010` | `FiberData` |
| `014` | `ArbitraryUserPointer` |
| `018` | `FS` 段寄存器在内存中的镜像地址 |
| `020` | 进程 `PID` |
| `024` | 线程 `ID` |
| `02C` | 指向线程局部存储指针 |
| `030` | `PEB` 结构地址（进程结构） |
| `034` | 上个错误号 |

楼主没发现 `03B` 后面还有东西吗，完整的是：

`FS` `003B` 32 位 `7FFDF000`

其中后面的 `7FFDF000` 才是 `FS:[0]` 的位置，`OD` 的内存窗口中可以直接看到这个地址。

因为 `FS` 是段寄存器，所以只有 16 位，是从以前的实模式保留下来的。

实模式中段寄存器保存的是段地址，但是在保护模式下保存的是段选择子，和实模式是完全不同的。

关于段选择子楼主可以去看看关于保护模式的资料。

## TEB 创建与结构体定义

`Windows` 在创建线程时，操作系统会为每个线程分配 `TEB` 结构，并且将 `FS` 段选择器指向当前线程的 `TEB` 数据结构。

代码:

```c
   Typedef   struct \_NT\_TIB {

   Struct \_EXCEPTION\_REGISTERATION\_RECORD \* ExceptionList;

   PVOID StackBase;

   PVOID StackLimit;

PVOID SubSystemTib;

   Union {

         PVOID FiberData;

         ULONG Version;

     };

     PVOID ArbitaryUserPointer;

   Struct    \_NT\_TIB \* Self;

} NT\_TIB;
```

上面结构体，很明显：

| 偏移 | 说明 |
| :--- | :--- |
| `000` | 指向 SEH 链指针 |
| `004` | 线程堆栈顶部 |
| `008` | 线程堆栈底部 |
| `00C` | `SubSystemTib` |
| `010` | `FiberData` |
| `014` | `ArbitraryUserPointer` |
| `018` | `FS` 段寄存器在内存中的镜像地址 |
| `020` | 进程 `PID` |
| `024` | 线程 `ID` |
| `02C` | 指向线程局部存储指针 |
| `030` | `PEB` 结构地址（进程结构） |
| `034` | 上个错误号
