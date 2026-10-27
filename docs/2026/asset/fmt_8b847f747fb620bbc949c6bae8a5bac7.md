
```
#define KI_USER_SHARED_DATA         0x7FFE0000

#define SharedUserData  ((KUSER_SHARED_DATA * const) KI_USER_SHARED_DATA)

#define MyGetTickCount() ((DWORD)((SharedUserData->TickCountMultiplier * (ULONLONG)SharedUserData->TickCount.LowPart) >> 24))



// ...

StartingTick = MyGetTickCount();

Sleep(Timeout);

TimeElapsedMs = MyGetTickCount() - StartingTick;



printf("Requested delay: %d, elapsed time: %d\n", Timeout, TimeElapsedMs);



if (abs((LONG)(TimeElapsedMs - Timeout)) > Timeout / 2)

    printf("Sleep-skipping DETECTED!\n");
```

## 2.3. 使用不同的方法获得系统时间

这种方法与前一种方法类似。我们尝试用不同的方法来获得当前的系统时间，而不是测量间隔时间。

代码样本：

```
SYSTEM_TIME_OF_DAY_INFORMATION  SysTimeInfo;

ULONGLONG time;

LONGLONG diff;



Sleep(60000); // should trigger sleep skipping

GetSystemTimeAsFileTime((LPFILETIME)&time);



NtQuerySystemInformation(SystemTimeOfDayInformation, &SysTimeInfo, sizeof(SysTimeInfo), 0);

diff = time - SysTimeInfo.CurrentTime.QuadPart;

if (abs(diff) > 10000000) // differ in more than 1 second

    printf("Sleep-skipping DETECTED!\n);
```

## 2.4. 检查调用延迟函数后延迟值是否发生变化

睡眠跳过通常被实现为用一个较小的间隔来替换延迟值。让我们看一下`NtDelayExecution`函数。延迟值是用一个指针传递给这个函数的：

```
NTSYSAPI NTSTATUS NTAPI

NtDelayExecution(

    IN BOOLEAN              Alertable,

    IN PLARGE_INTEGER       DelayInterval );
```

因此，我们可以检查`DelayInterval`的值在函数执行后是否改变。如果该值与初始值不同，则跳过了延迟。

代码样本：

```
LONGLONG SavedTimeout = Timeout * (-10000LL);

DelayInterval->QuadPart = SavedTimeout;

status = NtDelayExecution(TRUE, DelayInterval);

if (DelayInterval->QuadPart != SavedTimeout)

    printf("Sleep-skipping DETECTED!\n");
```

## 2.5. 使用绝对超时

对于执行延迟的Nt-函数，我们可以使用相对延迟间隔或绝对超时时间。延迟间隔的负值意味着相对超时，而正值意味着绝对超时。高级别的API函数，如`WaitForSingleObject()`或`Sleep()`，都是用相对的时间间隔来操作。因此，沙盒开发人员可能不关心绝对超时，并错误地处理它们。在Cuckoo沙盒中，这种延迟会被跳过，但跳过的时间和刻度会被错误地计算。这可以用来检测睡眠跳过。

代码样本：

```
void SleepAbs(DWORD ms)

{

    LARGE_INTEGER SleepUntil;



    GetSystemTimeAsFileTime((LPFILETIME)&SleepUntil);

    SleepTo.QuadPart += (ms * 10000);

    NtDelayExecution(TRUE, &SleepTo);

}
```

## 2.6. 从另一个进程获取时间

Cuckoo沙盒中的睡眠跳过不是全系统的。因此，如果有执行延迟，时间在不同的进程中以不同的速度移动。在延迟之后，我们应该同步进程并比较两个进程的当前时间。如果测量到的时间值有很大的差异，说明进行了睡眠跳转。

![](https://bbs.kanxue.com/upload/attach/202106/767964_H22FX9EVPQTX3E8.jpg)

当前版本的Cuckoo监控器在创建新线程或进程后禁用了睡眠跳转。因此，我们应该使用不被Cuckoo监控器跟踪的进程创建方法，例如，使用一个计划任务。

## 3. 从外部来源（NTP、HTTP）获取当前日期和时间

沙盒可以设置不同的日期，以检查分析样本的行为是如何根据日期而改变的。恶意软件可以使用外部日期和时间源来防止虚拟机内的时间操纵企图。这种方法也可用于测量时间间隔，执行延迟，并检测跳过睡眠的尝试。NTP服务器，以及HTTP头 `Date` 可以作为日期和时间的外部来源。例如，恶意软件可以连接到google.com来检查当前日期，并将其作为DGA种子。

反制措施：

实施假的网络基础设施或欺骗NTP数据和由真正的服务器返回的HTTP头。返回/欺骗的日期和时间应与虚拟机中的日期和时间同步。

## 4. 虚拟机和主机中的时间测量差异

一些API函数和指令的执行在虚拟机和通常的主机系统中可能需要不同的时间。这些特殊性可以用来检测虚拟环境。

### 4.1. `RDTSC`（使用`CPUID`强制退出虚拟机）

代码样本：

```
BOOL rdtsc_diff_vmexit()

{

    ULONGLONG tsc1 = 0;

    ULONGLONG tsc2 = 0;

    ULONGLONG avg = 0;

    INT cpuInfo[4] = {};



    // Try this 10 times in case of small fluctuations

    for (INT i = 0; i < 10; i++)

    {

        tsc1 = __rdtsc();

        __cpuid(cpuInfo, 0);

        tsc2 = __rdtsc();



        // Get the delta of the two RDTSC

        avg += (tsc2 - tsc1);

    }



    // We repeated the process 10 times so we make sure our check is as much reliable as we can

    avg = avg / 10;

    return (avg < 1000 && avg > 0) ? FALSE : TRUE;

}
```

该代码样本的作者：[al-khaser](https://bbs.kanxue.com/elink@307K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3L8%4u0V1e0X3!0@1k6i4N6G2M7Y4c8Z5P5g2)9J5c8X3q4D9i4K6u0V1K9$3S2S2M7$3g2J5)项目

### 4.2. `RDTSC`（带有`GetProcessHeap`和`CloseHandle`的Locky版本）

代码样本：

```
#define LODWORD(_qw)    ((DWORD)(_qw))

BOOL rdtsc_diff_locky()

{

    ULONGLONG tsc1;

    ULONGLONG tsc2;

    ULONGLONG tsc3;

    DWORD i = 0;



    // Try this 10 times in case of small fluctuations

    for (i = 0; i < 10; i++)

    {

        tsc1 = __rdtsc();



        // Waste some cycles - should be faster than CloseHandle on bare metal

        GetProcessHeap();



        tsc2 = __rdtsc();



        // Waste some cycles - slightly longer than GetProcessHeap() on bare metal

        CloseHandle(0);



        tsc3 = __rdtsc();



        // Did it take at least 10 times more CPU cycles to perform CloseHandle than it took to perform GetProcessHeap()?

        if ((LODWORD(tsc3) - LODWORD(tsc2)) / (LODWORD(tsc2) - LODWORD(tsc1)) >= 10)

            return FALSE;

    }



    // We consistently saw a small ratio of difference between GetProcessHeap and CloseHandle execution times

    // so we're probably in a VM!

    return TRUE;

}
```

该代码样本的作者：[al-khaser](https://bbs.kanxue.com/elink@5eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3L8%4u0V1e0X3!0@1k6i4N6G2M7Y4c8Z5P5g2)9J5c8X3q4D9i4K6u0V1K9$3S2S2M7$3g2J5)项目

反制措施

实施`RDTSC`指令 “拦截”。有可能使`RDTSC`成为一条特权指令，只能在内核模式下调用。在用户模式下调用 “挂钩” 的`RDTSC`会导致我们的处理程序的执行，它可以返回任何想要的值。

## 5. 使用不同的方法检查系统的最后启动时间

这种技术是常规操作系统查询中描述的技术的组合：[检查系统的正常运行时间是否很小](https://bbs.kanxue.com/elink@99aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3k6$3g2F1k6i4u0A6j5#2)9J5k6r3!0K6i4K6u0V1M7i4g2W2M7X3W2W2M7#2)9J5k6h3S2@1L8h3I4Q4x3U0y4U0K9r3g2U0K9#2)9J5k6r3W2X3i4K6u0V1M7%4W2K6N6r3g2E0i4K6u0V1N6i4m8@1K9h3#2W2)，以及[WMI：检查最后的启动时间部分](https://bbs.kanxue.com/elink@3e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3N6$3#2A6i4K6u0W2K9s2c8E0L8q4)9J5x3$3y4Z5k6h3y4C8i4K6u0V1L8r3q4K6N6q4)9J5k6r3u0G2L8%4c8Q4x3X3c8@1K9h3#2W2)。根据获取系统最后启动时间的方法，测得的沙盒操作系统运行时间可能太小（几分钟），或者相反，太大（几个月甚至几年），因为系统通常在分析开始后从快照中恢复。

我们可以通过比较通过WMI和`NtQuerySystemInformation(SystemTimeOfDayInformation)`获得的最后一次启动时间的两个值来检测沙箱。

代码样本：

```
bool check_last_boot_time()

{

    SYSTEM_TIME_OF_DAY_INFORMATION  SysTimeInfo;

    LARGE_INTEGER LastBootTime;

    

    NtQuerySystemInformation(SystemTimeOfDayInformation, &SysTimeInfo, sizeof(SysTimeInfo), 0);

    LastBootTime = wmi_Get_LastBootTime();

    return (wmi_LastBootTime.QuadPart - SysTimeInfo.BootTime.QuadPart) / 10000000 != 0; // 0 seconds

}
```

反制措施：

- 调整`KeBootTime`值

- 在调整`KeBootTime`后，重置WMI资源库或重新启动 `winmgmt`服务

反制措施

反制措施见于上述适当的分节。

归功于

归功于开放源码项目，代码样本取自这些项目：

- GitHub上的[al-khaser](https://bbs.kanxue.com/elink@baeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3L8%4u0V1e0X3!0@1k6i4N6G2M7Y4c8Z5P5g2)9J5c8X3q4D9i4K6u0V1K9$3S2S2M7$3g2J5)项目

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
