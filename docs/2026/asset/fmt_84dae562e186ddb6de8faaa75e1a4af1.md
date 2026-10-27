
# [原创][翻译]规避技术：定时

备注

原文地址：[2ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3N6r3W2E0K9h3&6Y4i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@378K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3N6r3W2E0K9h3&6Y4i4K6u0W2K9s2c8E0L8l9`.`.)

原文标题：Evasions: Timing

更新日期：2021年6月2日

此文后期：根据自身所学进行内容扩充

因自身技术有限，只能尽自身所能翻译国外技术文章，供大家学习，若有不当或可完善的地方，希望可以指出，用于共同完善这篇文章。

![](https://bbs.kanxue.com/upload/attach/202106/767964_9X5RK7K5ZEAYT5B.jpg)

目录

- 基于时间的沙箱规避技术
- 1. 延迟执行
  - 1.1. 简单的延迟操作
  - 1.2. 使用任务计划程序延迟执行
  - 1.3. 在重新启动之前没有可疑的行动
  - 1.4. 只在特定日期运行
- 2. 睡眠跳过检测
  - 2.1. 使用不同的方法进行并行延时
  - 2.2. 使用不同的方法测量时间间隔
  - 2.3. 使用不同的方法获得系统时间
  - 2.4. 检查调用延迟函数后延迟值是否发生变化
  - 2.5. 使用绝对超时
  - 2.6. 从另一个进程获取时间
- 3. 从外部来源（NTP、HTTP）获取当前日期和时间
- 4. 虚拟机和主机中的时间测量差异
  - 4.1. RDTSC（使用CPUID强制退出虚拟机）
  - 4.2. RDTSC（带有GetProcessHeap和CloseHandle的Locky版本）
- 5. 使用不同的方法检查系统的最后启动时间
- 反制措施
- 归功于

## 基于时间的沙箱规避技术

沙盒伪装通常持续时间很短，因为沙盒装载了大量的样本。伪装时间很少超过3-5分钟。因此，恶意软件可以利用这一事实来避免被发现：它可能在开始任何恶意活动之前进行长时间的延迟。

为了抵制这种情况，沙盒可能会实现操纵时间和执行延迟的功能。例如，Cuckoo沙箱有一个跳过睡眠的功能，用一个非常短的值取代延迟。这应迫使恶意软件在分析超时前开始其恶意活动。

![](https://bbs.kanxue.com/upload/attach/202106/767964_M28F863N5PE7Y75.jpg)

然而，这也可以用来检测沙盒。

在一些指令和API函数的执行时间上也有一些差异，可以用来检测虚拟环境。

没有为这一类技术提供签名建议，因为执行本章中描述的函数并不意味着它们被用于规避目的。很难区分旨在执行规避代码的代码和以非规避目的使用相同函数的代码。

## 1. 延迟执行

执行延迟用于避免在伪装期间检测到恶意活动。

### 1.1. 简单的延迟操作

使用的函数：

- `Sleep`, `SleepEx`, `NtDelayExecution`
- `WaitForSingleObject`, `WaitForSingleObjectEx`, `NtWaitForSingleObject`
- `WaitForMultipleObjects`, `WaitForMultipleObjectsEx`, `NtWaitForMultipleObjects`
- `SetTimer`, `SetWaitableTimer`, `CreateTimerQueueTimer`
- `timeSetEvent` (multimedia timers)
- `IcmpSendEcho`
- `select` (Windows sockets)

虽然这些函数的大部分使用是显而易见的，但我们展示了使用多媒体API的`timeSetEvent`函数和Windows套接字API的`select`函数的例子。

代码示例（使用`select`函数进行延迟）：

```c
int iResult;

DWORD timeout = delay; // delay in milliseconds

DWORD OK = TRUE;



SOCKADDR_IN sa = { 0 };

SOCKET sock = INVALID_SOCKET;



// this code snippet should take around Timeout milliseconds

do {

    memset(&sa, 0, sizeof(sa));

    sa.sin_family = AF_INET;

    sa.sin_addr.s_addr = inet_addr("8.8.8.8");    // we should have a route to this IP address

    sa.sin_port = htons(80); // we should not be able to connect to this port



    sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

    if (sock == INVALID_SOCKET) {

        OK = FALSE;

        break;

    }



    // setting socket timeout

    unsigned long iMode = 1;

    iResult = ioctlsocket(sock, FIONBIO, &iMode);



    iResult = connect(sock, (SOCKADDR*)&sa, sizeof(sa));

    if (iResult == false) {

        OK = FALSE;

        break;

    }



    iMode = 0;

    iResult = ioctlsocket(sock, FIONBIO, &iMode);

    if (iResult != NO_ERROR) {

        OK = FALSE;

        break;

    }



    // fd set data

    fd_set Write, Err;

    FD_ZERO(&Write);

    FD_ZERO(&Err);

    FD_SET(sock, &Write);

    FD_SET(sock, &Err);

    timeval tv = { 0 };

    tv.tv_usec = timeout * 1000;



    // check if the socket is ready, this call should take Timeout milliseconds

    select(0, NULL, &Write, &Err, &tv);

    

    if (FD_ISSET(sock, &Err)) {

        OK = FALSE;

        break;

    }



} while (false);



if (sock != INVALID_SOCKET)

    closesocket(sock);
```

代码示例（使用`timeSetEvent`函数进行延迟）：

```c
VOID CALLBACK TimerFunction(UINT uTimerID, UINT uMsg, DWORD_PTR dwUser, DWORD_PTR dw1, DWORD_PTR dw2)

{

    bProcessed = TRUE;

}



VOID timing_timeSetEvent(UINT delayInSeconds)

{

    // Some vars

    UINT uResolution;

    TIMECAPS tc;

    MMRESULT idEvent;



    // We can obtain this minimum value by calling

    timeGetDevCaps(&tc, sizeof(TIMECAPS));

    uResolution = min(max(tc.wPeriodMin, 0), tc.wPeriodMax);



    // Create the timer

    idEvent = timeSetEvent(

        delayInSeconds,

        uResolution,

        TimerFunction,

        0,

        TIME_ONESHOT);



    while (!bProcessed){

        // wait until our function finishes

        Sleep(0);

    }



    // destroy the timer

    timeKillEvent(idEvent);



    // reset the timer

    timeEndPeriod(uResolution);

}
```

该代码样本的作者：[al-khaser](https://bbs.kanxue.com/elink@e13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3L8%4u0V1e0X3!0@1k6i4N6G2M7Y4c8Z5P5g2)9J5c8X3q4D9i4K6u0V1K9$3S2S2M7$3g2J5)项目

### 1.2. 使用任务计划程序延迟执行

这种方法既可用于延迟执行，也可用于躲避沙盒追踪。

代码样本（PowerShell）：

```powershell
$tm = (get-date).AddMinutes(10).ToString("HH:mm")

$action = New-ScheduledTaskAction -Execute "some_malicious_app.exe"

$trigger = New-ScheduledTaskTrigger -Once -At $tm

Register-ScheduledTask TaskName -Action $action -Trigger $trigger
```

### 1.3. 在重新启动之前没有可疑的行动

这种技术背后的想法是，在模拟恶意样本的过程中，沙盒不会重启虚拟机。恶意软件可能只是使用任何可用的方法设置了持久性，并悄悄退出。只有在系统重新启动后，才会进行恶意操作。

### 1.4. 只在特定日期运行

恶意软件样本可能会检查当前日期，只在某些日期执行恶意行动。例如，在Sazoora恶意软件中使用了这种技术，它检查当前日期并验证这一天是否是某个月的第16、17或18号。

示例：

![](https://bbs.kanxue.com/upload/attach/202106/767964_JVBEHN5YZXTDNE3.jpg)

反制措施：

这类规避技术的对策应该是全面的，并且包括所有描述的攻击向量。实现不可能简单，对它的描述应该单独一条。因此，我们在此仅提供一般性建议：

- 实现睡眠跳过。
- 全系统动态时间流速度操纵。
- 在不同日期多次运行仿真。

虽然在Cuckoo沙盒中已经实现了睡眠跳过，但它很容易被欺骗。创建新线程或进程后，将禁用睡眠跳过，以避免检测到睡眠跳过。但是，它仍然可以很容易地检测到，如下所示。

## 2. 睡眠跳过检测

这种类型的技术一般是针对Cuckoo监控器的睡眠跳过功能和其他时间操纵技术，这些技术可以在沙盒中使用，以跳过恶意软件执行的长时间延迟。

### 2.1. 使用不同的方法进行并行延时

这些技术背后的想法是并行地执行不同类型的延迟，并测量运行时间。

代码样本：

```c
DWORD StartingTick, TimeElapsedMs;

LARGE_INTEGER DueTime;

HANDLE hTimer = NULL;

TIMER_BASIC_INFORMATION TimerInformation;

ULONG ReturnLength;



hTimer = CreateWaitableTimer(NULL, TRUE, NULL);

DueTime.QuadPart = Timeout * (-10000LL);



StartingTick = GetTickCount();

SetWaitableTimer(hTimer, &DueTime, 0, NULL, NULL, 0);

do

{

    Sleep(Timeout/10);

    NtQueryTimer(hTimer, TimerBasicInformation, &TimerInformation, sizeof(TIMER_BASIC_INFORMATION), &ReturnLength);

} while (!TimerInformation.TimerState);



CloseHandle(hTimer);



TimeElapsedMs = GetTickCount() - StartingTick;

printf("Requested delay: %d, elapsed time: %d\n", Timeout, TimeElapsedMs);



if (abs((LONG)(TimeElapsedMs - Timeout)) > Timeout / 2)

    printf("Sleep-skipping DETECTED!\n");
```

在上面的代码示例中，延迟超时是用`SetWaitableTimer()`计时器函数设置的。`Sleep()`函数被循环调用，直到定时器超时。在Cuckoo沙盒中，由`Sleep()`函数执行的延迟会被跳过（用一个很短的超时来代替），实际运行时间会比要求的超时高很多：

```text
Requested delay: 60000, elapsed time: 1906975

Sleep-skipping DETECTED!
```

### 2.2. 使用不同的方法测量时间间隔

我们需要执行一个将在沙盒中跳过的延迟，并使用不同的方法来测量运行时间。虽然Cuckoo监控器钩住了`GetTickCount()`, `GetLocalTime()`, `GetSystemTime()`并使它们返回跳过的时间，但我们仍然可以找到Cuckoo监控器没有处理的测量时间的方法。

使用的函数：

- `GetTickCount64`
- `QueryPerformanceFrequency`, `QueryPerformanceCounter`
- `NtQuerySystemInformation`

代码样本（使用`QueryPerformanceCounter`测量运行时间）：

```c
LARGE_INTEGER StartingTime, EndingTime;

LARGE_INTEGER Frequency;

DWORD TimeElapsedMs;



QueryPerformanceFrequency(&Frequency);

QueryPerformanceCounter(&StartingTime);



Sleep(Timeout);



QueryPerformanceCounter(&EndingTime);

TimeElapsedMs = (DWORD)(1000ll * (EndingTime.QuadPart - StartingTime.QuadPart) / Frequency.QuadPart);



printf("Requested delay: %d, elapsed time: %d\n", Timeout, TimeElapsedMs);



if (abs((LONG)(TimeElapsedMs - Timeout)) > Timeout / 2)

    printf("Sleep-skipping DETECTED!\n");
```

代码样本（使用`GetTickCount64`测量运行时间）：

```c
ULONGLONG tick;

DWORD TimeElapsedMs;



tick = GetTickCount64();

Sleep(Timeout);

TimeElapsedMs = GetTickCount64() - tick;



printf("Requested delay: %d, elapsed time: %d\n", Timeout, TimeElapsedMs);



if (abs((LONG)(TimeElapsedMs - Timeout)) > Timeout / 2)

    printf("Sleep-skipping DETECTED!\n");
```

我们也可以使用我们自己的`GetTickCount`实现来检测睡眠跳动。在接下来的代码示例中，我们直接从`KUSER_SHARED_DATA`结构中获取tick计数。这样，即使`GetTickCount()`函数被拦截了，我们也能得到原始的tick计数值。

代码样本（从`KUSER_SHARED_DATA`结构中获取tick计数）：
