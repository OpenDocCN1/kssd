
# 时间限制

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-3.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-3.htm)

## 1. 定时器

有些程序的试用版每次运行都有时间限制，例如运行 10 分钟或 20 分钟就停止工作，必须重新运行该程序才能正常工作。这些程序里面自然有个定时器来统计程序运行的时间。

### 1.1 使用 SetTimer()

常用的计数器是函数 `SetTimer()`，调用这个函数创建的定时器可以发出消息 `VM_TIMER`，或者在定时期满时调用一个回调函数。使用这个函数会使时间延时，精度不高。

### 1.2 使用 timeSetEvent()

给 Windows 驱动程序最精确的周期性通知是由 Windows 的多媒体服务 `timeSetEvent()` 提供的。它的时间可以精确到 1 毫秒。

### 1.3 使用 VXD

可以使用 VMM 的 `Set_Global_time_Out()` 服务来迫使回调函数的几个毫秒再执行，这就创造了一个“只有一次”的定时器。`VXD` 可以在回调中再次调用 `Set_Global_time_Out()` 来开始下一个定时器，这样提供了一个连续运行的定时器。

### 1.4 其它

- `GetTickCount()`：精度不高；
- `timeGetTime()`：可以以毫秒级返回 windows 开始后的时间。

## 2. 时间限制

一般这类保护的软件都有时间上的限制，如试用 30 天等，当过了共享软件的试用期后，就不予运行，只有向软件作者付费注册之后才能得到一个无时间限制的注册版本。

这种类型程序很多，让你有 10 天、20 天、30 天等，它们在安装时，在你的系统某处做上时间标记，每次运行时用当前系统时间和安装时的时间比较，判断你还否能使用。

如最典型的 30 天限制的一种情况：

```
mov ecx,1E ; 把 1E (30 天 十进制) 放入 ecx
mov eax,[esp+10] ; 把用过天数放到 eax
cmp eax,ecx ; 在此比较
jl ...
```

如碰到这种情况，只需把 `mov eax,[esp+10]` 改成 `mov eax,1`。

要记住当前年份、月份的十六进制的一些表示方法，如：2000 年的十六进制是 `07D0`，然后用 `W32DASM` 反汇编你的程序，用查找字符串的方法找 `D007`（在机器码中位置颠倒了一下）或其它类似时间的数字，有可能会找到有价值的线索。你别小看这种方法，对那些没怎么防范的程序，此招很有效。

如：一程序限定在 2000 年使用，可能有如下一代码：

```
:00037805 817C2404D0070000 cmp dword ptr [esp+04], 000007D0 比较是否在 2000 年。
```

## 3. 与时间相关函数

### 3.1 GetSystemTime 得当前系统时间

在一个 `SYSTEMTIME` 中载入当前系统时间，这个时间采用的是“协同世界时间”（即 UTC，也叫做 GMT）格式。

```
VOID GetSystemTime(
    LPSYSTEMTIME lpSystemTime // SYSTEMTIME，随同当前时间载入的结构
);
```

### 3.2 GetLocalTime 得当前本地时间

```
VOID GetLocalTime(
    LPSYSTEMTIME lpSystemTime // SYSTEMTIME，用于装载本地时间的结构
);
```

### 3.3 SystemTimeToFileTime

根据一个 `FILETIME` 结构的内容，载入一个 `SYSTEMTIME` 结构。

```
BOOL SystemTimeToFileTime(
    CONST SYSTEMTIME * lpst, // SYSTEMTIME，包含了系统时间信息的一个结构
    LPFILETIME lpft // FILETIME，用于装载文件时间的一个结构
);
```

返回值：非零表示成功，零表示失败。

### 3.4 SetTimer 创建一定时器，在指定时间内暂停

```
UINT SetTimer(
    HWND hwnd, // 时间信息句柄
    UINT idtimer, // 定时器 ID 标识符
    UINT uTimeout, // 暂停时间
    TIMERPROC tmprc // 处理定时过程的程序入口地址
);
```

`4dd466d0e1e0cc123afbe504da2462ca`
