
# [原创]Windows 进程 CPU、内存、I/O 与调度指标采样

开源项目实现：

<mark class="encrypted">836K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4j5h3&6Y4N6$3g2A6i4K6u0V1j5$3#2Q4x3V1k6C8M7%4N6G2M7X3b7`.</mark>

![图片描述](upload/attach/202608/1012968_NV5JPPEG56NXCUT.webp)

## 目标与四步流程

目标是取得指定 Windows 进程的 CPU 使用率、I/O 速率、内存快照和调度状态，并说明这些结果属于哪个进程实例和哪个时间窗口。要完成这件事，分为 4 步。

1. 建立可追溯的样本，并打开目标进程。
2. 采集进程身份、累计时间和实际时钟。
3. 采集内存、I/O 与调度快照。
4. 核验两份样本并计算 CPU 与 I/O 差分。

PID（Process Identifier，进程标识符）是 Windows 分配给运行中进程的数值编号。进程退出后，这个数值未来可以交给另一个进程使用。进程对象由内核维护，包含创建时间、线程集合、访问权限和退出状态。查询方用 `OpenProcess` 得到 `HANDLE`（句柄）。句柄是当前进程持有的受访问控制引用，指向一个具体进程对象，使用完必须关闭。

同一个进程对象连接着多个 Windows 组件。内核调度器把全部线程的用户态和内核态 CPU 时间累积到进程层。内存管理器提供工作集和提交量快照。I/O 管理器积累读、写与其它传输量。对象管理器维护句柄表项。调度器读取优先级类、动态优先级提升设置和处理器亲和范围。各来源的更新时刻并不完全相同，样本需要保存字段状态与采样时钟。

整个过程为：输入 PID -> 建立样本和进程句柄 -> 读取样本 A -> 等待实际时间窗口 -> 读取样本 B -> 核验实例与计时条件 -> 输出快照或差分结果。

一个样本分为三类数据。实例身份由 PID 和创建时间组成。累计计数包括 CPU 时间和 I/O 传输字节。瞬时属性包括工作集、私有提交量、句柄数、优先级类、优先级提升设置和亲和掩码。累计量只有放在两个有效样本之间才有速率含义。瞬时属性的单个数值只描述读取瞬间，报告中应附带采样时间和字段成功状态。

创建时间是识别 PID 复用的关键。监控服务常在每轮按 PID 重新打开进程，也可能在缓存恢复后继续使用旧基线。此时仅比较 PID 会把两个不同生命周期的累计 CPU 或 I/O 相减。创建时间来自进程对象的时间信息，它与 PID 一起形成实例键。键发生变化后，当前样本只可作为新的基线。

## 1. 建立可追溯的样本，并打开目标进程

采样结构需要让读取失败可见。零工作集、零 I/O 增量和零句柄变化都可能是真实结果。把 API 失败也写成零会把“未知”伪装成“没有活动”。下面的结构把值、单位和字段状态分开。`FieldStatus::error` 只在 `available` 为 `false` 时解释为失败代码，成功字段的错误码没有业务含义。

```cpp
struct FieldStatus {
bool available = false; // 本字段的 API 是否成功返回。
DWORD error = ERROR_SUCCESS; // 失败时紧接 API 调用取得的 GetLastError 结果。
};
struct ProcessSample {
DWORD pid = 0; // 数值 PID。单独使用不能区分复用。
ULONGLONG creation100ns = 0; // 创建时间，单位是 100ns，和 pid 构成实例键。
ULONGLONG wall100ns = 0; // 系统墙钟，单位是 100ns，便于记录可读时间线。
ULONGLONG active100ns = 0; // 非休眠运行时间，单位是 100ns，用作速率分母。
ULONGLONG cpu100ns = 0; // 内核态与用户态累计 CPU 时间之和。
ULONGLONG ioBytes = 0; // 读、写、其它 I/O 的累计传输字节之和。
SIZE_T workingSetBytes = 0; // 当前工作集字节数。
SIZE_T privateCommitBytes = 0; // 当前私有提交字节数。
DWORD handleCount = 0; // 当前句柄表项数量。
DWORD priorityClass = 0; // NORMAL_PRIORITY_CLASS 等优先级类。
FieldStatus identity, cpu, io, memory, handles, priority;
};
```

`SIZE_T` 是与当前程序指针宽度相同的无符号整数类型，内存计数以字节为单位时用它保存最合适。`ULONGLONG` 是 64 位无符号整数，适合保存 100ns 累计时间和 I/O 字节。进程长期运行时累计值会变大，使用 32 位 `DWORD` 保存 CPU 时间或 I/O 字节会很快截断高位。

一个成熟的监控记录还应保存每项 API 的错误码、进程退出状态、查询权限、采样线程的计时器类型和程序自身位数。查看结果的人看到 `available=false` 时应显示“未取得”，同时保留错误码供权限与生命周期诊断。将失败字段从速率计算中剔除，能够避免除零、错误尖峰和误判闲置。

### 打开目标进程与管理句柄所有权

`GetLastError` 读取当前线程最近一次失败的 Win32 API 错误。这个值只保留当前线程的最近失败状态。后续 API、格式化库或诊断代码都有机会改写它，因此必须紧贴失败调用保存。它没有参数、缓冲区和资源所有权。

```cpp
// 作用：取得当前线程刚刚发生的 Win32 失败代码。
// 返回：错误编号。只在失败 API 的紧后位置读取。
DWORD GetLastError(void);
// 作用：取得当前进程对象的数值 PID。
// 返回：非零 PID。没有输入、输出缓冲区、权限要求和释放责任。
DWORD GetCurrentProcessId(void);
// 作用：按 PID 打开一个进程对象并建立当前进程拥有的 HANDLE。
// 返回：有效 HANDLE 成功。nullptr 失败，立即用 GetLastError 读取原因。成功句柄必须 CloseHandle。
HANDLE OpenProcess(
DWORD dwDesiredAccess, // 输入：所需权限位。此处使用 PROCESS_QUERY_LIMITED_INFORMATION | PROCESS_VM_READ。
BOOL bInheritHandle, // 输入：FALSE 表示以后创建的子进程不继承这个 HANDLE。
DWORD dwProcessId // 输入：目标 PID。PID 可能在查询前退出或被后续进程复用。
);
// 作用：释放当前进程拥有的内核对象 HANDLE。
// 返回：非零成功。0 失败，立即读取 GetLastError。成功后句柄值失效，不可重复使用或重复关闭。
BOOL CloseHandle(HANDLE hObject); // 输入：OpenProcess 返回的真实 HANDLE，不能为 nullptr。
```

正确流程先把用户输入的 `0` 转成 `GetCurrentProcessId` 返回的 PID，再申请最低限度的权限。`PROCESS_QUERY_LIMITED_INFORMATION` 支持大多数只读属性查询。`PROCESS_VM_READ` 允许内存计数器读取相关进程信息。受保护进程、退出中的进程和安全描述符都可能让 `OpenProcess` 失败，失败后没有句柄可传给后续 API。

```cpp
const DWORD targetPid = suppliedPid == 0 ? GetCurrentProcessId() : suppliedPid;
HANDLE process = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION | PROCESS_VM_READ,
FALSE, targetPid);
if (process == nullptr) {
const DWORD error = GetLastError(); // 先保存，再写日志。
// 记录 targetPid 与 error。当前轮没有任何有效字段。
} else {
// 在这里读取时间、内存、I/O 与调度属性。每项读取都检查自己的返回值。
if (!CloseHandle(process)) {
const DWORD error = GetLastError();
// 当前流程结束，不再把关闭状态不明的句柄交给其它查询。
}
}
```

以下写法把失败句柄带入查询。空句柄没有指向任何进程对象，返回的错误属于错误调用本身，原始的拒绝访问或进程退出原因也会丢失。

```cpp
HANDLE process = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION, FALSE, targetPid);
GetProcessTimes(process, &creation, &exitTime, &kernel, &user); // 错误：process 可能为 nullptr。
CloseHandle(process); // 错误：未拥有有效句柄。
```

完成第 1 步后，已有一条能区分“读取失败”和“真实零值”的样本结构，也已持有指向目标进程实例的有效句柄。接下来要从这个进程对象取得创建时间、累计 CPU 时间和两个时钟，才能让后续速率计算有身份依据和实际分母。

## 2. 采集进程身份、累计时间和实际时钟

`FILETIME` 用两个 32 位字段表达一个 64 位计数，单位为 100ns（1 秒等于 10,000,000 个单位），基准点为 1601-01-01 UTC。`GetProcessTimes` 的创建时间、退出时间、内核 CPU 时间和用户 CPU 时间都采用这一单位。用户态 CPU 时间表示线程在应用代码和用户态库中获得处理器的累计时间。内核态 CPU 时间表示线程在系统服务、驱动和内核工作中获得处理器的累计时间。两者相加得到进程累计 CPU 时间。

```cpp
// 作用：读取一个进程的创建、退出、内核态 CPU 与用户态 CPU 时间。
// 返回：非零成功。0 失败，立即读取 GetLastError。每个输出 FILETIME 均由调用方分配且不可为 nullptr。
BOOL GetProcessTimes(
HANDLE hProcess, // 输入：可查询的进程 HANDLE。需要 PROCESS_QUERY_INFORMATION 或 LIMITED_INFORMATION。
LPFILETIME lpCreationTime,// 输出：创建时间，100ns，调用方存储。
LPFILETIME lpExitTime, // 输出：退出时间。运行中进程通常为零值，调用方存储。
LPFILETIME lpKernelTime, // 输出：累计内核态 CPU 时间，100ns，调用方存储。
LPFILETIME lpUserTime // 输出：累计用户态 CPU 时间，100ns，调用方存储。
);
// 作用：写入当前系统 UTC 墙钟。
// 返回：void。lpSystemTimeAsFileTime 由调用方提供且不可为 nullptr，输出单位为 100ns。
VOID GetSystemTimeAsFileTime(LPFILETIME lpSystemTimeAsFileTime);
// 作用：取得从系统启动以来的非偏置中断时间。
// 返回：非零成功。0 失败，立即读取 GetLastError。UnbiasedTime 不可为 nullptr，输出单位为 100ns，排除睡眠和休眠时间。
BOOL QueryUnbiasedInterruptTime(PULONGLONG UnbiasedTime);
```

CPU 百分比需要真实的采样窗口。单逻辑处理器基准使用 `ΔCPU / ΔActiveTime × 100`，其中 `ΔCPU` 是内核与用户累计时间的差，`ΔActiveTime` 来自 `QueryUnbiasedInterruptTime`。多线程进程可以同时在多个逻辑处理器上运行，因此单逻辑处理器基准能够超过 100%。全机份额需要额外记录可用逻辑处理器数、处理器组、作业对象限制和 CPU 集合范围，再选择相应分母。

墙钟与非休眠时间各自回答不同问题。`GetSystemTimeAsFileTime` 便于记录“何时采到”，也会受人工校时、时间服务校正和睡眠恢复影响。`QueryUnbiasedInterruptTime` 适合描述系统处于运行状态的持续时间，并排除睡眠与休眠时间。连续样本同时保存两者：两者的增量明显不一致时，当前窗口跨越了休眠、恢复或时钟改变，应废弃旧基线。

```cpp
ULONGLONG To100ns(const FILETIME& value) { // FILETIME 的两个部分由调用方传入，不涉及资源释放。
ULARGE_INTEGER combined{};
combined.LowPart = value.dwLowDateTime;
combined.HighPart = value.dwHighDateTime;
return combined.QuadPart;
}
FILETIME creation{}, exitTime{}, kernel{}, user{}, wall{};
if (GetProcessTimes(process, &creation, &exitTime, &kernel, &user)) {
const ULONGLONG creation100ns = To100ns(creation);
const ULONGLONG cpu100ns = To100ns(kernel) + To100ns(user); // 实现中还应检查无符号加法上溢。
GetSystemTimeAsFileTime(&wall); // 记录可追溯的墙钟时间。
ULONGLONG active100ns = 0;
if (!QueryUnbiasedInterruptTime(&active100ns)) {
const DWORD error = GetLastError(); // CPU 累计可保存。本轮不能用于速率分母。
}
} else {
const DWORD error = GetLastError(); // 创建时间和 CPU 字段同时标记为不可用。
}
```

进程退出状态需要独立读取。退出能发生在任何两个 API 调用之间，进程句柄仍可能保持有效并指向已经结束的对象。`GetExitCodeProcess` 返回查询瞬间的退出状态。`STILL_ACTIVE` 表示进程尚未报告退出。该常量值为 259，程序本身也可能以 259 退出，采集器只把它作为状态提示，并结合采样时间处理。

```cpp
// 作用：读取目标进程当前退出码或活动状态。
// 返回：非零成功。0 失败，立即读取 GetLastError。lpExitCode 不可为 nullptr，输出由调用方保存。
BOOL GetExitCodeProcess(
HANDLE hProcess, // 输入：可查询的进程 HANDLE。
LPDWORD lpExitCode // 输出：STILL_ACTIVE 或进程已报告的退出码。
);
```

错误写法把 `Sleep(1000)` 当作精确一秒，并把墙钟直接当作可靠分母。调度延迟会延长等待，睡眠恢复和校时会改变墙钟增量，速率会被放大或压低。两次实际计时结果才是计算依据。

完成第 2 步后，每份样本已有实例键、累计 CPU 时间、墙钟和非休眠时间。接下来读取内存、I/O 和调度字段，使结果同时说明进程此刻占用的资源与可运行范围。

## 3. 采集内存、I/O 与调度快照

工作集（Working Set）是当前驻留在物理内存中的页面集合，其大小由内存管理器回收、预取、共享页活动和进程访问模式共同影响。私有提交量（Private Usage）是只归该进程负责的已提交虚拟内存数量。工作集下降可以来自工作集修剪，私有提交量下降反映提交状态变化。两个字段不能互相替换。

I/O 计数器中的 `ReadTransferCount`、`WriteTransferCount` 和 `OtherTransferCount` 是累计传输字节。其它传输包括控制操作等非普通读写请求。API 成功返回时可把三者相加作为总 I/O 累计量。相加前应检查 64 位无符号溢出。报告速率时也可分别计算读、写、其它速率，以保留活动性质。

```cpp
// 作用：取得进程内存计数器。
// 返回：非零成功。0 失败，立即读取 GetLastError。
BOOL GetProcessMemoryInfo(
HANDLE Process, // 输入：具有查询和虚拟内存读取权限的进程 HANDLE。
PPROCESS_MEMORY_COUNTERS ppsmemCounters,// 输出：调用方分配的结构，不能为 nullptr。结构所有权仍归调用方。
DWORD cb // 输入：结构容量，单位字节。调用前 ppsmemCounters->cb 也要写入相同容量。
);
// 作用：取得进程累计 I/O 操作次数和传输字节数。
// 返回：非零成功。0 失败，立即读取 GetLastError。
BOOL GetProcessIoCounters(
HANDLE hProcess, // 输入：可查询的进程 HANDLE。
PIO_COUNTERS lpIoCounters // 输出：调用方分配的 IO_COUNTERS，不能为 nullptr，字节字段为 ULONGLONG。
);
```
