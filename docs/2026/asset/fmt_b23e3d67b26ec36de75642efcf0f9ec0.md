
这里有个熟悉的程序：任务管理器（`Taskmgr.exe`），以 High 级别运行，仔细回忆一下，启动任务管理器的过程中并没有出现 UAC 弹窗：

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_TXFEE2JZS22R8DD.png)

`manifest`（清单）是伴随并描述并行程序集或独立应用程序的 XML 文件。清单通过程序集的 `assemblyIdentity` 元素唯一标识程序集。它们包含用于绑定和激活的信息，例如 COM 类、接口和类型库，这些信息传统上存储在注册表中。以下为使用命令 `mt -inputresource:Taskmgr.exe;#1 -out:taskmgr.manifest` 从 `Taskmgr.exe` 中提取出的 manifest 文件：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23 | `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>` `<assembly xmlns="urn:schemas-microsoft-com:asm.v1" xmlns:asmv3="urn:schemas-microsoft-com:asm.v3" manifestVersion="1.0">` `<assemblyIdentity processorArchitecture="amd64" version="5.1.0.0" name="Microsoft.Windows.Diagnosis.AdvancedTaskManager" type="win32"></assemblyIdentity>` `<description>Task Manager</description>` `<dependency>` `<dependentAssembly>` `<assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls" version="6.0.0.0" processorArchitecture="amd64" publicKeyToken="6595b64144ccf1df" language="*"></assemblyIdentity>` `</dependentAssembly>` `</dependency>` `<trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">` `<security>` `<requestedPrivileges>` `<requestedExecutionLevel level="highestAvailable"></requestedExecutionLevel>` `</requestedPrivileges>` `</security>` `</trustInfo>` `<asmv3:application>` `<asmv3:windowsSettings xmlns="f4fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6e0e0f1W2Q4x3V1j5J5x3o6l9#2i4K6u0r3g2$3W2F1k6r3!0%4M7#2y4W2N6s2c8A6L8X3N6K6">` `<dpiAware>true</dpiAware>` `<autoElevate>true</autoElevate>` `</asmv3:windowsSettings>` `</asmv3:application>` `</assembly>` |

但如果将 `Taskmgr.exe` 复制到桌面，直接运行该程序，就会发现还是会出现 UAC 弹窗。这又引出自动提升程序需要满足的一个关键条件：从受信任目录执行。无论是 manifest 文件中设置了 `autoElevate true`，还是白名单列表判断，都需要 v13 的第 0x16 个比特位（index 从 0 开始计算）为 1 才行，这是一个前置条件。

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_4Q74Z8FHSYKE3KM.png)

v13 中的值从 a4 中取到，我们可以看一下它（`AiIsEXESafeToAutoApprove`）的上层函数 `RAiLaunchAdminProcess`，在下面的逻辑里会将目标程序字符串和 `g_IncludedSysDir` 进行比较，利用 `RtlPrefixUnicodeString` 和 `wcschr` 函数来判断目标程序是否在 `\??\C:\Windows\System32\` 或 `\??\C:\Windows\SysWow64\` 目录下，这两个目录是受信任的目录，如果满足条件，才会设置 flag 的第 0x16 个比特位（`0x200000`）。

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_YK6D88KSRJBBKUE.png)

不过还有两种特殊情况，是 `g_IncludedXmtExe` 中存储的 `\??\C:\Windows\System32\Sysprep\sysprep.exe` 以及 `\??\C:\Windows\System32\inetsrv\InetMgr.exe`，如果是这两个路径下的程序，且通过 `AipMatchesOriginalFileName` 函数检查（通过比较文件版本信息里的 OriginalFilename 来判断程序名字是否更改过），即使它们不是直接位于 System32 或 SysWow64 目录下，也会跳转到 `LABEL_364` 标签，设置 flag 的第 0x16 个比特位。

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_3EBNM5RRTYXZA9D.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_23RHKT2TQDVYAJ5.png)

**文件版本信息（以 sysprep.exe 为例）：**

处理示例：[711K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4N6A6L8Y4k6W2M7W2)9J5c8X3&6X3i4K6u0V1N6$3W2F1N6X3g2J5i4K6u0V1N6X3g2J5M7i4g2W2M7Y4W2$3j5h3I4#2k6h3p5`.](https://bbs.kanxue.com/elink@48eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4N6A6L8Y4k6W2M7W2)9J5c8X3&6X3i4K6u0V1N6$3W2F1N6X3g2J5i4K6u0V1N6X3g2J5M7i4g2W2M7Y4W2$3j5h3I4#2k6h3p5`.)

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_7YX7BPXW5ZC8UFF.png)

|  |  |
| --- | --- |
