
# [翻译]使用NtQuerySystemInformation和DuplicateHandle的系统级反调试技术

**翻译**

原文地址：66fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5z5o6k6E0j5i4c8@1K9r3g2%4i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9#2k6Y4m8G2M7%4c8Q4x3@1k6A6k6q4)9K6c8s2y4&6M7%4c8W2L8g2)9#2k6X3q4F1N6r3W2Q4y4h3k6V1k6h3u0#2k6H3`.

功能：使用`NtQuerySystemInformation`和`DuplicateHandle`的系统级反调试技术

---

我开发了一种反调试技术，它不加区别地针对用户模式调试器，而不是检测是否有个别进程被调试。

总的来说，这种方法的工作原理如下：

1. 使用带有`SystemExtendedHandleInformation`的`NtQuerySystemInformation`检索系统中所有打开的句柄列表。
2. 检查是否有进程包含一个活动的调试句柄。
3. 使用带有`DUPLICATE_CLOSE_SOURCE`标志的`DuplicateHandle`终止远程进程中的这个句柄。
4. 循环回到步骤#1。

使上述方法复杂化的一件事是在步骤2中识别调试句柄。每个句柄的类型都是通过`NtQuerySystemInformation`返回的`ObjectTypeIndex`字段来识别的，但这个值在不同版本的Windows中并不一致。

我们显然可以在一个查找表中硬编码各种可能的值，但通用的解决方案总是更好的。为了计算出当前操作系统的调试手柄类型索引，我采取了以下步骤：

1. 使用带有`SystemExtendedHandleInformation`的`NtQuerySystemInformation`检索系统中所有打开的句柄列表。
2. 为当前进程中发现的每个独特的对象类型设置一个标志（例如文件句柄、进程句柄、注册表键等）。
3. 调用PID为0的`DebugActiveProcess`。即使目标PID无效，`DebugActiveProcess`也会在当前进程中创建一个内部调试句柄。即使调用失败，该调试句柄也保持原位。
3. 重复步骤#1。
4. 循环浏览当前进程的最新句柄列表，并寻找在步骤#2中没有标记的对象类型的条目。假设这是一个单线程的应用程序，我们知道这将是之前由`DebugActiveProcess`创建的调试句柄。我们可以从这个条目的`ObjectTypeIndex`字段中提取调试句柄类型。
5. 使用`CloseHandle`手动关闭这个临时调试句柄。

下面是完整的程序代码：
