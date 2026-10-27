
# [原创]虚拟机检测技术整理

第一次尝试恶意代码分析就遇到了虚拟机检测，于是就想着先学习一下检测的技术然后再尝试绕过。学习后最终发现，似乎最好的方法不应该是去patch所有检测方法，而是直接调试并定位检测函数再绕过。但既然已经研究了两天，索性将收集到的资料整理一下，方便后人查找。

# 1. 基于特征的检测

## 1.1 文件以及目录

相关 API：`GetFileAttributes`、`etModuleHandle`

通常，各种 hypervisor 会在虚拟机上安装驱动程序和其他软件，以使客户机能够利用 hypervisor 提供的功能。恶意软件可以搜索这些文件、目录或进程的存在。VMware 虚拟机中可能会有如下的文件列表：

```
C:\Program Files\VMware\
C:\Windows\System32\vm3dc003.dll
C:\Windows\System32\vm3ddevapi64-debug.dll
C:\Windows\System32\vm3ddevapi64-release.dll
C:\Windows\System32\vm3ddevapi64-stats.dll
C:\Windows\System32\vm3ddevapi64.dll
C:\Windows\System32\vm3dgl64.dll
C:\Windows\System32\vm3dglhelper64.dll
C:\Windows\System32\vm3dservice.exe
C:\Windows\System32\vm3dum64-debug.dll
C:\Windows\System32\vm3dum64-stats.dll
C:\Windows\System32\vm3dum64.dll
C:\Windows\System32\vm3dum64\_10-debug.dll
C:\Windows\System32\vm3dum64\_10-stats.dll
C:\Windows\System32\vm3dum64\_10.dll
C:\Windows\System32\vm3dum64\_loader.dll
C:\Windows\System32\vmGuestLib.dll
C:\Windows\System32\vmGuestLibJava.dll
C:\Windows\System32\vmhgfs.dll
C:\Windows\System32\VMWSU.DLL
C:\Windows\System32\vsocklib.dll
C:\Windows\System32\drivers\vm3dmp.sys
C:\Windows\System32\drivers\vm3dmp\_loader.sys
C:\Windows\System32\drivers\vm3dmp-debug.sys
C:\Windows\System32\drivers\vm3dmp-stats.sys
C:\Windows\System32\drivers\vmnet.sys
C:\Windows\System32\drivers\vmmouse.sys
C:\Windows\System32\drivers\vmusb.sys
C:\Windows\System32\drivers\vmci.sys
C:\Windows\System32\drivers\vmhgfs.sys
C:\Windows\System32\drivers\vmmemctl.sys
C:\Windows\System32\drivers\vmx86.sys
C:\Windows\System32\drivers\vmrawdsk.sys
C:\Windows\System32\drivers\vmusbmouse.sys
C:\Windows\System32\drivers\vmkdb.sys
C:\Windows\System32\drivers\vmnetuserif.sys
C:\Windows\System32\drivers\vmnetadapter.sys
```

## 1.2 Devices

相关API：`CreateFile`

在Windows虚拟机中，虚拟机通常拥有独特命名的 "devices"。

- `\\.\HGFS`，主机和虚拟机之间共享的文件系统
- `\\.\vmci`，虚拟机和主机之间的通信管道

## 1.3 MAC 地址

相关API：`GetAdaptersAddresses`、`GetAdaptersInfo`、`Netbios`

Hypervisors 通常为虚拟机提供虚拟化的网卡(NICs)，具有独占于hypervisor供应商的MAC地址。

已知的 VMware 虚拟机的 MAC地址有：

- `00:50:56:XX:XX:XX`
- `00:1C:14:XX:XX:XX`
- `00:0C:29:XX:XX:XX`
- `00:05:69:XX:XX:XX`

## 1.4 BIOS 字符串

相关API：`RegOpenKeyEx`、`RegQueryValueEx`

虚拟化产品通常会附带一个基本输入/输出系统（BIOS），其中包含虚拟机特有的字符串。可以通过检查Windows注册表来查看。

- `HKEY\_LOCAL\_MACHINE\HARDWARE\DESCRIPTION\System` 中键值 "SystemBiosVersion" 包含 "VMware, Inc."
- `HKEY\_LOCAL\_MACHINE\HARDWARE\DESCRIPTION\System\BIOS` 中包含键值
  - "BIOSVendor" 为 "VMware, Inc."
  - "BIOSVersion" 的起始字符串为 "VM."
  - "SystemManufacturer" 为 "VMware, Inc."
  - "SystemProductName" 包含 "VMware."
- `HKEY\_LOCAL\_MACHINE\SYSTEM\ControlSet001\Control\SystemInformation` 中也有类似的信息

## 1.5 其他 Windows 注册表项

VMware中，以下注册表项会带有 "VMWARE" 或者 "VM" 字符串

```
HKEY\_LOCAL\_MACHINE\HARDWARE\DEVICEMAP\Scsi\Scsi Port 0\Scsi Bus 0\Target Id 0\Logical Unit Id 0
HKEY\_LOCAL\_MACHINE\HARDWARE\DEVICEMAP\Scsi\Scsi Port 1\Scsi Bus 0\Target Id 0\Logical Unit Id 0
HKEY\_LOCAL\_MACHINE\HARDWARE\DEVICEMAP\Scsi\Scsi Port 2\Scsi Bus 0\Target Id 0\Logical Unit Id 0
HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Services\Disk\Enum
HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Enum\IDE
HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Enum\SCSI
```

## 1.6 SMBIOS 字符串

相关API：`GetSystemFirmwareTable`

System Management BIOS 是由BIOS读取的一组数据结构。虚拟机会带有有特殊的字符串。VMware 虚拟机会带有 "VMware"。

## 1.7 ACPI 字符串

相关API：`GetSystemFirmwareTable`

Advanced Configuration and Power Interface (ACPI) 是操作系统通过 BIOS 或 UEFI 与各种硬件组件进行通信的接口。VMware 虚拟机会带有 "VMware"。

## 1.8 进程

相关API：`CreateToolhelp32Snapshot`、`Process32First`、`Process32Next`

一些虚拟机利会有特殊的后台进程。这些进程主要提供一些方便的功能，例如自动配置屏幕分辨率和配置网络设置等。

VMware虚拟机中可能会有以下进程：

- `Vmtoolsd.exe`
- `Vmwaretray.exe`
- `Vmwareuser.exe`
- `VGAuthService.exe`
- `Vmacthlp.exec`

有些恶意软件还会通过进程数来检测虚拟环境，比如 "EvilBunny" 会检测是否存在至少15个进程。

## 1.9 硬盘硬件ID以及名称

相关API：`SetupDiGetClassDevs`、`SetupDiGetDeviceRegistryProperty`

Win32/Winwebsec的一些变种使用了一种通过 Setup API来检查硬盘设备名的方法。类似的也可以检测硬盘硬件ID。

![图片描述](upload/attach/202304/950495\_MGA7GWBGATQCHJ6.png)

## 1.10 通过 WMI 获取硬件信息

Windows Management Instrumentation（WMI），提供了一种标准化的接口，允许程序获取系统信息并执行管理任务。WMI是基于COM（Component Object Model）技术构建的，并使用WQL（WMI查询语言）来查询数据。

- 可以通过 WMI 获取 BIOS 序列号并查看是否带有类似"VMWARE"的字符串。类似的还可以检测 `Win32\_ComputerSystem` 下的 `Model` 以及 `Manufacturer` 字段。

```
VOID CheckBIOSSerialNumberWMI()
{
IWbemServices* pSvc = NULL;
IWbemLocator* pLoc = NULL;
IEnumWbemClassObject* pEnumerator = NULL;
BOOL bFound = FALSE;
HRESULT hRes;
// Init WMI
if (!InitWMI(&pSvc, &pLoc, _T("ROOT\\CIMV2")))
return;
// If success, execute the desired query
if (!ExecWMIQuery(&pSvc, &pLoc, &pEnumerator, _T("SELECT * FROM Win32_BIOS")))
return;
// Get the data from the query
IWbemClassObject* pclsObj = NULL;
ULONG uReturn = 0;
VARIANT vtProp;
while (pEnumerator)
{
hRes = pEnumerator->Next(WBEM_INFINITE, 1, &pclsObj, &uReturn);
if (0 == uReturn)
break;
// Get the value of the Name property
VariantInit(&vtProp);
hRes = pclsObj->Get(_T("SerialNumber"), 0, &vtProp, 0, 0);
if (SUCCEEDED(hRes)) {
if (vtProp.vt == VT_BSTR) {
if (StrStrI(vtProp.bstrVal, _T("VMWare")) != 0)
{
bFound = TRUE;
VariantClear(&vtProp);
pclsObj->Release();
break;
}
}
VariantClear(&vtProp);
}
// release the current result object
pclsObj->Release();
}
// Cleanup
pSvc->Release();
pLoc->Release();
pEnumerator->Release();
CoUninitialize();
PrintResult(bFound, _T("Checking SerialNumber from BIOS using WMI"));
}
```

- 可以通过返回结果的条目数，检查是否存在一些物理机应该有的硬件，比如：

```
SELECT * FROM Win32_Fan // 风扇
SELECT * FROM Win32_CacheMemory //
SELECT * FROM Win32_VoltageProbe // 电压探针
SELECT * FROM Win32_PerfFormattedData_Countentes_ThermalZoneInformation // 热区信息
SELECT * FROM CIM_Memory // 物理内存
SELECT * FROM CIM_Sensor // 传感器设备
SELECT * FROM CIM_NumericSensor // 数字传感器
SELECT * FROM CIM_TemperatureSensor // 温度传感器
SELECT * FROM CIM_VoltageSensor // 电压传感器
```

# 2. 基于指令的检测

某些CPU指令在虚拟机中执行时返回值与在物理机机上执行时返回值不同。恶意软件可以执行这些指令，并尝试根据返回值推断虚拟机的存在。虽然这些检测方法不普遍适用于所有虚拟机，且随着虚拟机的迭代也已经部分失效，但仍然有必要了解。下面列出已知的用于虚拟机检测的指令。

## 2.1 IN

IN指令从指定的I/O端口读取值到指定的内存地址。I/O端口是操作系统和I/O设备（如磁盘控制器和声音设备）之间的通信手段。基于VMware的虚拟机具有端口号为 0x5658 的I/O设备，允许hypervisor 和虚拟机之间通信。通常，在非特权用户模式下运行IN指令将触发特权指令异常。但在 VMware 中，不会引发此类异常。VMware 公司甚至将此作为官方检测 VMware hypervisor 的手段。VMware 也提供了一个接口，允许用户“关闭”此方法。

```
bool isVmwarePresent()
{
__try {
_asm {
mov eax, 0x564d5868 // 'VMXh'
mov ebx, 0
mov cx, 1
mov dx, 0x5658 // 'VX'
in eax, dx
}
return true;
}
__except (GetExceptionCode() == EXCEPTION_PRIV_INSTRUCTION) {
return false;
}
}
```

## 2.2 CPUID

CPUID是一种x86指令，用来获取CPU的各种信息。该指令通过输入 EAX 中的值来确定要在EAX，EBX，ECX和EDX寄存器中返回的信息。下面给出三种基于 CPUID 指令的检测方法。

1. 当 EAX=0 时， CPUID 将按EBX，EDX和ECX寄存器的顺序返回一个12个字符的制造商ID字符串。Intel CPU返回"GenuineIntel"，而AMD CPU在K5型号之后返回"AuthenticAMD"。当在虚拟机中运行时，ID将被专属于某些虚拟机的自定义字符串替换。VMware虚拟机中理论上将返回 "VmwareVmware"。

```
BOOL CheckCPUIDVendor()
{
INT CPUInfo[4] = { -1 };
CHAR szHypervisorVendor[0x40];
__cpuid(CPUInfo, 0x40000000);
SecureZeroMemory(szHypervisorVendor, sizeof(szHypervisorVendor));
memcpy(szHypervisorVendor, CPUInfo + 1, 12);
if (_strcmpi(szHypervisorVendor, "VMwareVMware") == 0)
return TRUE;
return FALSE;
}
```

2. 当 EAX=1 时，CPUID将返回有关处理器功能的信息，例如处理器支持的指令。返回值 ECX 中的第 31 位确定是否存在 hypervisor。在物理机中此值为 0，在虚拟机中为 1。

```
BOOL CheckCPUIDVendor()
{
INT CPUInfo[4] = { -1 };
__cpuid(CPUInfo, 1);
if ((CPUInfo[2] >> 31 ) & 1)
return TRUE;
return FALSE;
}
```

3. 在虚拟机中执行 CPUID 还会触发 VM-Exit Event，使得进程从 guest 切换到 VMM。可以用 RDTSC 指令来计算这个开销。

```
BOOL rdtsc_diff_vmexit()
{
ULONGLONG tsc1 = 0;
ULONGLONG tsc2 = 0;
ULONGLONG avg = 0;
INT cpuInfo[4] = {};
// Try this 10 times in case of small fluctuations
for (INT i = 0; i < 10; i++)
{
tsc1 = __rdtsc();
__cpuid(cpuInfo, 0);
tsc2 = __rdtsc();
// Get the delta of the two RDTSC
avg += (tsc2 - tsc1);
}
// We repeated the process 10 times so we make sure our check is as much reliable as we can
avg = avg / 10;
return (avg < 1000 && avg > 0) ? FALSE : TRUE;
}
```

## 2.3 SIDT（已失效）
