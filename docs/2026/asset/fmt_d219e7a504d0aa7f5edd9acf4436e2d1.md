
# [原创][翻译]规避技术：全局操作系统对象

备注

原文地址：[4e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3k6$3I4G2j5X3q4D9i4K6u0V1L8%4y4Q4x3X3c8G2j5X3A6W2j5%4c8K6i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@8f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3k6$3I4G2j5X3q4D9i4K6u0V1L8%4y4Q4x3X3c8G2j5X3A6W2j5%4c8K6i4K6u0W2K9s2c8E0L8l9`.`.)

原文标题：Evasions: Global OS Objects

更新日期：2021年5月19日

此文后期：根据自身所学进行内容扩充

因自身技术有限，只能尽自身所能翻译国外技术文章，供大家学习，若有不当或可完善的地方，希望可以指出，用于共同完善这篇文章。

![](https://bbs.kanxue.com/upload/attach/202105/767964_NSZ378CQ6PZGBZT.jpg)

## 目录

- 全局目标检测方法
- 1.检查特定的全局互斥体
- 2.检查特定的虚拟设备
- 3.检查特定的全局管道
- 4.检查特定的全局对象
- 5.检查特定对象目录（仅限于沙盒）
- 6.检查系统中是否存在虚拟注册表（仅限于沙盒）
- 反制措施
- 归功于

## 全局目标检测方法

所有全局对象检测方法的基本原理是：在通常的主机上没有这样的对象，但它们存在于特定的虚拟环境和沙盒中。如果存在这样的伪装，则可以检测虚拟环境。

### 1. 检查特定的全局互斥体

此方法检查虚拟环境中存在但通常主机系统中不存在的特定互斥体。

#### 使用的函数

- `CreateMutexA/W`
- `OpenMutexA/W`

#### 代码样本

```cpp
// usage sample:

supMutexExist(L"Sandboxie_SingleInstanceMutex_Control"); // sample value from the table below

BOOL supMutexExist(_In_ LPWSTR lpMutexName)
{
    DWORD dwError;
    HANDLE hObject = NULL;

    if (lpMutexName == NULL) {
        return FALSE;
    }

    SetLastError(0);
    hObject = CreateMutex(NULL, FALSE, lpMutexName); // define around A or W function version
    dwError = GetLastError();

    if (hObject) {
        CloseHandle(hObject);
    }

    return (dwError == ERROR_ALREADY_EXISTS);
}
```

该代码样本归功于：[VMDE project](https://bbs.kanxue.com/elink@59eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6X3W2J5k6h3j5H3P5q4)9J5c8W2k6y4c8p5f1`.)

#### 识别标志

如果以下函数包含表列 `名称` 的第3个参数：

- `CreateMutexA/W(..., ..., registry_path)`
- `OpenMutexA/W(..., ..., registry_path)`

那么这就表明应用程序试图使用规避技术。

#### 检测表

| 检查是否存在以下全局互斥体: |  |
| --- | --- |
| 检测 | 名称 |
| DeepFreeze | `Frz_State` |
| Sandboxie | `Sandboxie_SingleInstanceMutex_Control` |
|  | `SBIE_BOXED_ServiceInitComplete_Mutex1` |
| VirtualPC | `MicrosoftVirtualPC7UserServiceMakeSureWe'reTheOnlyOneMutex` |

注意：DeepFreeze是一个在每次重启时恢复系统的应用程序。

### 2. 检查特定的虚拟设备

此方法检查特定的虚拟设备，这些设备存在于虚拟环境中，但不在常规的主机系统中。

#### 使用的函数

- `NtCreateFile`

#### 代码样本

```cpp
// usage sample:

HANDLE hDummy = NULL;

supOpenDevice(L"\\Device\\Null", GENERIC_READ, &hDummy); // sample values from the table below

BOOL supOpenDevice(
    _In_ LPWSTR lpDeviceName,
    _In_ ACCESS_MASK DesiredAccess,
    _Out_opt_ PHANDLE phDevice)
{
    OBJECT_ATTRIBUTES attr;
    IO_STATUS_BLOCK iost;
    UNICODE_STRING uDevName;
    HANDLE hDevice;
    NTSTATUS Status;

    if (phDevice) {
        *phDevice = NULL;
    }
    if (lpDeviceName == NULL) {
        return FALSE;
    }

    hDevice = NULL;
    RtlSecureZeroMemory(&uDevName, sizeof(uDevName));
    RtlInitUnicodeString(&uDevName, lpDeviceName);
    InitializeObjectAttributes(&attr, &uDevName, OBJ_CASE_INSENSITIVE, 0, NULL);

    Status = NtCreateFile(&hDevice, DesiredAccess, &attr, &iost, NULL, 0,
        0, FILE_OPEN, 0, NULL, 0);
    if (NT_SUCCESS(Status)) {
        if (phDevice != NULL) {
            *phDevice = hDevice;
        }
    }

    return NT_SUCCESS(Status);
}
```

该代码样本归功于：[VMDE project](https://bbs.kanxue.com/elink@69bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6X3W2J5k6h3j5H3P5q4)9J5c8W2k6y4c8p5f1`.)

#### 识别标志

如果下面的函数包含第三个参数，它的字段 `ObjectName->Buffer` 来自表列 `名称`：

- `NtCreateFile(..., ..., attr, ...)`

那么这就表明应用程序试图使用规避技术。

第三个参数的类型如下：

```c
typedef struct _OBJECT_ATTRIBUTES {
    ULONG Length;
    HANDLE RootDirectory;
    PUNICODE_STRING ObjectName;
    ULONG Attributes;
    PVOID SecurityDescriptor;
    PVOID SecurityQualityOfService;
} OBJECT_ATTRIBUTES;
```

#### 检测表

| 检查是否存在以下虚拟设备： |  |  |
| --- | --- | --- |
| 检测 | 路径 |  |
| VirtualBox | `\\.\VBoxMiniRdDN` |  |
|  | `\\.\VBoxMiniRdrDN` |  |
|  | `\\.\VBoxGuest` |  |
|  | `\\.\VBoxTrayIPC` |  |
|  | `\\.\VBoxMouse` |  |
|  | `\\.\VBoxVideo` |  |
| VMware | `\\.\HGFS` |  |
|  | `\\.\vmci` |  |

### 3. 检查特定的全局管道

管道只是虚拟设备的一个特殊情况，请参考上一节的代码样本和识别标志。

#### 检测表

| 检查是否存在以下全局管道: |  |
| --- | --- |
| 检测 | 字符串 |
| VirtualBox | `\\.\pipe\VBoxMiniRdDN` |
|  | `\\.\pipe\VBoxTrayIPC` |

### 4. 检查特定的全局对象

此方法检查特定的全局对象，这些对象存在于虚拟环境中，但不存在于常规的主机系统中。

#### 使用的函数

- `NtOpenDirectoryObject`
- `NtQueryDirectoryObject`

#### 代码样本
