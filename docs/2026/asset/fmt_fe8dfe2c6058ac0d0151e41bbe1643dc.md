
# [原创][翻译]规避技术： WMI

备注

原文地址：[292K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3N6$3#2A6i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@5d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3N6$3#2A6i4K6u0W2K9s2c8E0L8l9`.`.)

原文标题：Evasions: WMI

更新日期：2021年6月3日

此文后期：根据自身所学进行内容扩充

因自身技术有限，只能尽自身所能翻译国外技术文章，供大家学习，若有不当或可完善的地方，希望可以指出，用于共同完善这篇文章。

![](https://bbs.kanxue.com/upload/attach/202106/767964_4GUNE7MJ42UKM8Z.jpg)

**目录**

- WMI检测方法
- 背景介绍
- 1. 通用的WMI查询
- 2. 利用WMI逃避追踪
  - 2.1. 使用WMI启动程序
  - 2.2. 通过WMI使用任务计划程序启动进程
- 3. 检查最后的启动时间
- 4. 检查网络适配器的最后重置时间
- 识别标志
- 反制措施
- 归功于

## WMI检测方法

Windows管理接口（WMI）查询是获得操作系统和硬件信息的另一种方式。WMI使用COM接口和它们的方法。

### 背景介绍

标准COM函数被用来处理查询。它们的调用顺序如下所述，可分为6个步骤。

1. COM的初始化。
    - `CoInitialize`/`CoInitializeEx`
2. 创建所需的接口实例。
    - 创建接口实例：`CoCreateInstance`/`CoCreateInstanceEx`
3. 通过接口实例连接到特定的服务，其功能如下。
    - `ConnectServer`
4. 用这些函数获取服务的方法并设置其参数。
    - `Method`（用于获取方法）
    - `Put`（用于设置参数）
5. 从服务中检索信息，并用下面的函数执行服务的方法。左边的函数是右边的函数的代理——这些函数在内部被调用。
    - `ExecQuery` -> `IWbemServices_ExecQuery`（检索信息）
    - `ExecMethod` -> `IWbemServices_ExecMethod`（执行方法）
    - `ExecMethodAsync` -> `IWbemServices_ExecMethodAsync`（执行方法）
6. 用以下函数检查查询的结果。
    - `[enumerator]->Next`
    - `[object]->Get`

要想了解所描述的理论是如何应用于实践的，请查看下面的例子。

### 1. 通用的WMI查询

由于WMI提供了另一种收集系统信息的方式，它可以被用来执行其他文章中描述的规避技术，例如：

- 检查处理器数量是否不足
- 检查硬盘大小是否小
- 检查MAC地址是否特定
- 检查CPU温度信息是否可用

代码样本：

```c
/*
Check number of cores using WMI
*/

BOOL number_cores_wmi()
{
  IWbemServices *pSvc = NULL;
  IWbemLocator *pLoc = NULL;
  IEnumWbemClassObject *pEnumerator = NULL;
  BOOL bStatus = FALSE;
  HRESULT hRes;
  BOOL bFound = FALSE;

  // Init WMI
  bStatus = InitWMI(&pSvc, &pLoc);
  if (bStatus)
  {
    // If success, execute the desired query
    bStatus = ExecWMIQuery(&pSvc, &pLoc, &pEnumerator, _T("SELECT * FROM Win32_Processor"));
    if (bStatus)
    {
      // Get the data from the query
      IWbemClassObject *pclsObj = NULL;
      ULONG uReturn = 0;
      VARIANT vtProp;

      // Iterate over our enumator
      while (pEnumerator)
      {
        hRes = pEnumerator->Next(WBEM_INFINITE, 1, &pclsObj, &uReturn);
        if (0 == uReturn)
          break;

        // Get the value of the Name property
        hRes = pclsObj->Get(_T("NumberOfCores"), 0, &vtProp, 0, 0);
        if (V_VT(&vtProp) != VT_NULL) {

          // Do our comparaison
          if (vtProp.uintVal < 2) {
            bFound = TRUE; break;
          }

          // release the current result object
          VariantClear(&vtProp);
          pclsObj->Release();
        }
      }

      // Cleanup
      pEnumerator->Release();
      pSvc->Release();
      pLoc->Release();
      CoUninitialize();
    }
  }

  return bFound;
}





/*
Check hard disk size using WMI
*/

BOOL disk_size_wmi()
{
  IWbemServices *pSvc = NULL;
  IWbemLocator *pLoc = NULL;
  IEnumWbemClassObject *pEnumerator = NULL;
  BOOL bStatus = FALSE;
  HRESULT hRes;
  BOOL bFound = FALSE;
  INT64 minHardDiskSize = (80LL * (1024LL * (1024LL * (1024LL))));

  // Init WMI
  bStatus = InitWMI(&pSvc, &pLoc);
  if (bStatus)
  {
    // If success, execute the desired query
    bStatus = ExecWMIQuery(&pSvc, &pLoc, &pEnumerator, _T("SELECT * FROM Win32_LogicalDisk"));
    if (bStatus)
    {
      // Get the data from the query
      IWbemClassObject *pclsObj = NULL;
      ULONG uReturn = 0;
      VARIANT vtProp;

      // Iterate over our enumator
      while (pEnumerator)
      {
        hRes = pEnumerator->Next(WBEM_INFINITE, 1, &pclsObj, &uReturn);
        if (0 == uReturn)
          break;

        // Get the value of the Name property
        hRes = pclsObj->Get(_T("Size"), 0, &vtProp, 0, 0);
        if (V_VT(&vtProp) != VT_NULL) {

          // Do our comparaison
          if (vtProp.llVal < minHardDiskSize) { // Less than 80GB
            bFound = TRUE; break;
          }

          // release the current result object
          VariantClear(&vtProp);
          pclsObj->Release();
        }
      }

      // Cleanup
      pEnumerator->Release();
      pSvc->Release();
      pLoc->Release();
      CoUninitialize();
    }
  }

  return bFound;
}
```

该代码样本的作者：[al-khaser](https://bbs.kanxue.com/elink@18bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3L8%4u0V1e0X3!0@1k6i4N6G2M7Y4c8Z5P5g2)9J5c8X3q4D9i4K6u0V1K9$3S2S2M7$3g2J5)项目

代码样本(PowerShell)：

```powershell
(Get-CimInstance -ClassName Win32_BIOS -Property SerialNumber).SerialNumber
```

### 识别标志

如果下面的函数包含来自表列 `Query` 的第3个参数：

- `IWbemServices_ExecQuery(..., query, ...)`

则表明该应用程序试图使用规避技术。

检查表：

以下WMI查询可用于检测虚拟环境:

| Query查询方式 | Field字段 | Value值 | Detect检测 | Comments注释 |
| --- | --- | --- | --- | --- |
| `SELECT * FROM Win32_Processor` | NumberOfCores | < 2 | [general] |  |
| `SELECT * FROM Win32_Processor` | ProcessorId | [empty] |  |  |
| `SELECT * FROM Win32_LogicalDisk` | Size | < 60GB |  |  |
| `SELECT * FROM Win32_BaseBoard` | SerialNumber | None |  |  |
| `SELECT * FROM Win32_BaseBoard` | Version | None |  |  |
| `SELECT * FROM MSAcpi_ThermalZoneTemperature` | CurrentTemperature | "Not supported" |  |  |
| `SELECT * FROM Win32_PnPEntity` | DeviceId | PCI\VEN_80EE&DEV_CAFE | VirtualBox |  |
| `SELECT * FROM Win32_PnPEntity` | DeviceId | IDE\CDROOMVBOX |  |  |
| `SELECT * FROM Win32_PnPEntity` | DeviceId | IDE\DISKVBOX\* |  |  |
| `SELECT * FROM Win32_PnPEntity` | DeviceId | VEN_VMWARE | VMware |  |
| `SELECT * FROM Win32_PnPEntity` | DeviceId | PROD_VMWARE_VIRTUAL |  |  |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 08:00:27 | VirtualBox | See ["Check if MAC address is specific"](https://bbs.kanxue.com/elink@c25K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3L8X3g2@1N6$3!0J5K9#2)9J5k6h3S2@1L8h3I4Q4x3U0y4U0K9r3g2U0K9#2)9J5k6r3W2X3i4K6u0V1L8h3q4U0i4K6u0V1j5h3c8V1M7X3g2K6M7#2)9J5k6r3W2K6i4K6u0V1M7%4m8W2j5$3W2X3K9h3x3`.) section in "Network" chapter |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 00:1C:42 | Parallels |  |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 00:05:69 | VMware |  |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 00:0C:29 |  |  |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 00:1C:14 |  |  |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 00:50:56 |  |  |
| `SELECT * FROM Win32_NetworkAdapterConfiguration` | MACAddress | 00:16:E3 | XEN |  |
| `SELECT * FROM Win32_Bios` | Serial Number | VMware- | VMware |  |
| `SELECT * FROM Win32_Bios` | Serial Number | 0 | VirtualBox |  |
| `SELECT * FROM Win32_Bios` | Version | INTEL - 6040000 | VMware | See "SystemBiosVersion" in ["Check if particular registry keys contain specified strings"](https://bbs.kanxue.com/elink@fa3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3M7X3g2Y4K9i4y4@1M7Y4W2Q4x3X3g2Z5N6r3#2D9i4K6t1K6j5$3S2W2j5$3E0Q4x3X3c8A6k6W2)9J5k6r3E0W2P5i4y4Q4x3X3c8U0L8$3&6@1j5h3W2F1i4K6u0V1M7%4c8J5K9h3&6Y4M7H3`.`.) section in "Registry" chapter |
| `SELECT * FROM Win32_Bios` | Version | BOCHS | BOCHS |  |
| `SELECT * FROM Win32_Bios` | Version | PARALLELS | Parallels |  |
| `SELECT * FROM Win32_Bios` | Version | QEMU | QEMU |  |
| `SELECT * FROM Win32_Bios` | Version | VBOX | VirtualBox |  |
| `SELECT * FROM Win32_ComputerSystem` | Model | VMware | VMware |  |
| `SELECT * FROM Win32_ComputerSystem` | Model | VirtualBox | VirtualBox |  |
| `SELECT * FROM Win32_ComputerSystem` | Manufacturer | VMware | VMware |  |
| `SELECT * FROM Win32_ComputerSystem` | Manufacturer | innotek GmbH | VirtualBox |  |
| `SELECT * FROM Win32_VideoController` | AdapterCompatibility | VMware | VMware |  |
| `SELECT * FROM Win32_VideoController` | AdapterCompatibility | Oracle Corporation | VirtualBox |  |
| `SELECT * FROM Win32_VideoController` | Caption | VMware | VMware |  |
| `SELECT * FROM Win32_VideoController` | Caption | VirtualBox | VirtualBox |  |
| `SELECT * FROM Win32_VideoController` | Description | VMware | VMware |  |
| `SELECT * FROM Win32_VideoController` | Description | VirtualBox | VirtualBox |  |
| `SELECT * FROM Win32_VideoController` | Name | VMware | VMware |  |
| `SELECT * FROM Win32_VideoController` | Name | VirtualBox | VirtualBox |  |
| `SELECT * FROM Win32_PointingDevice` | Description | VMware | VMware |  |

表中所列的查询并不是唯一可能的，提出这些查询是为了让人们了解它们是如何工作的，以及用这些调用可以检索到什么信息

### 反制措施

反制措施取决于通过WMI方法实现的特定检查，它们与相关文章中描述的相应方法相同。此外，你必须重新启动 `winmgmt` 服务。

### 2. 利用WMI逃避追踪

WMI提供了一种创建新进程和安排任务的方法。沙盒通常使用`CreateProcessInternalW`函数挂钩来跟踪子进程。然而，当你使用WMI创建进程时，函数`CreateProcessInternalW`不会在父进程中被调用。因此，使用WMI创建的进程可能不会被沙盒跟踪，其行为也不会被记录。

#### 2.1. 使用WMI启动程序

你可以通过WMI使用 `Win32_Process` 类的 `Create` 方法来创建一个新的进程。

代码样本：
