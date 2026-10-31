
# 目标进程用户模式句柄转换为内核句柄

网上很多方法都是Ring0内核进程和句柄互相转换的方法，查了一下没怎么看到说怎么将用户模式句柄转换为内核句柄的。如果内核获取到的是**目标进程的句柄**，拷贝到内核模式是不能直接使用的，会引发异常。

目前想到的实现思路是：

1. 如果是伪句柄，直接返回。
2. 调用`ObReferenceObjectByHandle`指定用户模式句柄获取任意对象的体指针（`EProcess`、`EThread`）
3. 调用`ObOpenObjectByPointer`打开指针所引用的对象，并返回对象的句柄，此时设置为内核句柄。就获取到啦。

本人是菜鸟，各位师傅如果知道其他的方法跪求指点！！！

实现代码：

```c
NTSTATUS ConvertKernelHandle(
IN HANDLE UserHandle,
OUT PHANDLE KernelHandle,
POBJECT_TYPE ObjectType
)
{
NTSTATUS Status = STATUS_SUCCESS;
PVOID Object = NULL; //任意对象的体指针 EProcess EThread
LONG v1 = (LONG)UserHandle;
if (KernelHandle == NULL)
{
Status = STATUS_INVALID_PARAMETER;
goto Exit;
}
if (v1 <= 0)
{
*KernelHandle = UserHandle;
Status = STATUS_SUCCESS;
goto Exit;
}
Status = ObReferenceObjectByHandle(
UserHandle,
0,
ObjectType,
UserMode,
&Object,
NULL
);
if (Status != STATUS_SUCCESS)
{
goto Exit;
}
Status = ObOpenObjectByPointer(
Object,
OBJ_KERNEL_HANDLE,
0,
0,
NULL,
KernelMode,
KernelHandle
);
if (*KernelHandle == UserHandle) {
Status = STATUS_UNSUCCESSFUL;
}
Exit:
if (Object != NULL) {
ObDereferenceObject(Object);
}
return Status;
}
```

伪句柄转换为真实句柄，可以调用复制对象句柄的`DuplicateHandle`。

```c++
BOOL DuplicateHandle(
[in] HANDLE hSourceProcessHandle,
[in] HANDLE hSourceHandle,
[in] HANDLE hTargetProcessHandle,
[out] LPHANDLE lpTargetHandle,
[in] DWORD dwDesiredAccess,
[in] BOOL bInheritHandle,
[in] DWORD dwOptions
);
```

微软官方在对此函数的解释中，说明了如果`hSourceHandle`是伪句柄，会将其转换为进程或线程的实际句柄。

# 参考

[ObOpenObjectByPointer 函数 (ntifs.h) - Windows drivers | Microsoft Learn](elink@81bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9r3q4J5k6s2N6S2M7X3g2Q4x3V1k6V1M7X3W2$3k6i4u0K6i4K6u0r3k6r3c8A6i4K6u0r3L8Y4c8A6k6Y4y4Q4x3V1k6F1k6W2)9J5k6r3&6@1K9h3k6K6i4K6u0V1L8$3u0G2M7r3g2F1L8$3u0B7k6h3y4@1j5Y4W2H3L8$3W2F1N6r3g2J5)

[ObReferenceObjectByHandle 函数 (wdm.h) - Windows drivers | Microsoft Learn](elink@776K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9r3q4J5k6s2N6S2M7X3g2Q4x3V1k6V1M7X3W2$3k6i4u0K6i4K6u0r3k6r3c8A6i4K6u0r3N6$3c8E0i4K6u0r3L8X3k6Q4x3X3c8%4k6r3#2Q4x3X3c8G2j5Y4u0W2k6X3g2J5k6h3&6U0k6h3!0T1K9X3g2U0N6r3u0&6K9r3q4F1k6r3I4W2)

[DuplicateHandle 函数 (handleapi.h) - Win32 apps | Microsoft Learn](elink@b70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6S2M7r3W2Q4x3V1k6Z5j5h3&6V1L8r3g2S2M7r3W2Q4x3V1k6F1k6W2)9J5k6r3S2S2L8X3c8D9k6h3q4H3K9g2)9J5k6r3c8#2M7r3I4A6j5$3q4@1k6h3S2S2L8X3c8D9k6b7`.`.)

[伪句柄转换为真正的句柄 - 沉疴 - 博客园 (cnblogs.com)](elink@8ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2y4Z5x3e0t1K6i4K6u0r3M7q4)9J5c8U0M7K6y4e0l9@1z5e0k6Q4x3X3g2Z5N6r3#2D9)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)
