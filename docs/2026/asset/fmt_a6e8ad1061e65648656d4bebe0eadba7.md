
# 《Windows 10 Control Flow Guard Internals》 Reading Notes

## 0x00 前言

Wikipedia：

> Control Flow Guard (CFG) was first released for Windows 8.1 Update 3 (KB3000850) in November 2014. Developers can add CFG to their programs by adding the `/guard:cf` linker flag before program linking in Visual Studio 2015 or newer.
>
> As of Windows 10 Creators Update (Windows 10 version 1703), the Windows kernel is compiled with CFG. The Windows kernel uses Hyper-V to prevent malicious kernel code from overwriting the CFG bitmap.
>
> CFG operates by creating a per-process bitmap, where a set bit indicates that the address is a valid destination. Before performing each indirect function call, the application checks if the destination address is in the bitmap. If the destination address is not in the bitmap, the program terminates. This makes it more difficult for an attacker to exploit a use-after-free by replacing an object's contents and then using an indirect function call to execute a payload.

Windows CFG是Control-Flow Integrity(CFI)的具体实现，该机制由Windows 8.1 Update 3 (KB3000850)开始引入，需编译器和操作系统相结合，目的在于防止不可靠间接调用。漏洞利用常常通过修改间接调用地址以劫持执行流，而CFG会于编译链接期间将程序所有间接调用地址记录在PE文件中，并在执行所有间接调用前增加校验，若间接调用地址被修改，则抛出异常。

---

环境：

- 物理机OS：`Windows 10 20H2 x64`
- 物理机WinDbg：`10.0.17134.1`
- 虚拟机OS：`Windows 10 1511(10586.164) x86`
- 虚拟机WinDbg：`10.0.19041.685`
- VMware：`VMware Workstation 15 Pro`
- Visual Studio 2019

## 0x01 How CFG Works:User Mode Part

使用如下代码进行编译及调试：

```
typedef int(*fun_t)(int);
int foo(int a)
{
printf("hellow world %d\n",a);
return a;
}
class CTargetObject
{
public:
fun_t fun;
};
int main()
{
int i = 0;
CTargetObject *o_array = new CTargetObject[5];
for (i = 0; i < 5 ; i++)
o_array[i].fun = foo;
o_array[0].fun(1);
return 0;
}
```

启用CFG：

![](upload/attach/202105/817966_CPTC4UV3GTEMCYU.jpg)

修改"调试信息格式"：

![](upload/attach/202105/817966_UEUTRPX8JQK7JGR.jpg)

编译完成，`dumpbin.exe /headers /loadconfig Project1.exe`：

![](upload/attach/202105/817966_SHD37C62K3Q4D4P.jpg)

可以看到CFG已启用。

![](upload/attach/202105/817966_V96R2YYDCFZ7Z6R.jpg)

- `Guard CF address of check-function pointer`：指向`___guard_check_icall_fptr`，载入PE后，其指向`ntdll!LdrpValidateUserCallTarget`
![](upload/attach/202105/817966_XMT8BQGZ266UKMW.jpg)
- `Guard CF address of dispatch-function pointer`：Reserved
- `Guard CF function table`：指向RVA列表，供NT内核使用
![](upload/attach/202105/817966_PSS4C3WMH43NGY4.jpg)
- `Guard CF function count`：RVA数量
- `Guard Flags`

---

系统若不支持CFG，则不对`___guard_check_icall_fptr`进行更新(如下为Windows 7 SP1 x86)：

![](upload/attach/202105/817966_76S3G8S645EFY4J.jpg)

若系统支持CFG，首先是`nt!PspPrepareSystemDllInitBlock`(由`nt!NtCreateUserProcess`—>`nt!PspAllocateProcess`—>`nt!PspSetupUserProcessAddressSpace`调用)初始化`ntdll!LdrSystemDllInitBlock`：

![](upload/attach/202105/817966_W7MXHR6ZRWN4UYW.jpg)

其偏移`0x60`为Bitmap Address，`0x68`为Bitmap Size。

![](upload/attach/202105/817966_PRCXSH8H2BNX8J5.jpg)

载入PE文件时，`ntdll!LdrpCfgProcessLoadConfig`会校验`OptionalHeader.DllCharacteristics`：

![](upload/attach/202105/817966_2MK6YWCBNQCHW2Z.jpg)

修改`_guard_check_icall_nop`指向`ntdll!LdrpValidateUserCallTarget`：

![](upload/attach/202105/817966_V5T5Y8UPDFAECFN.jpg)

间接调用地址传递给ECX寄存器：

![](upload/attach/202105/817966_JCE7XWHN9GGERUW.jpg)

下面来看`ntdll!LdrpValidateUserCallTarget`校验过程：

![](upload/attach/202105/817966_HUMYKAVBKMY2CNZ.jpg)

1. 获取`ntdll!LdrSystemDllInitBlock+0x60`处Bitmap Address，当前进程空间每8 Bytes对应Bitmap中1 bit[`mov edx, ds:dword_6A30C1A0`]
2. 获取指向Destination Address低8位(即1 Byte)范围内的Bitmap值(低8位范围：0x00—0xFF，256 Bytes；Bitmap值：32 bit—32*8=256 Bytes)[`mov eax, ecx`;`shr eax, 8`;`mov edx, [edx+eax*4]`]
3. Destination Address是否以0x10对齐，是则取最低字节前5位作为索引获取bit；否则取最低字节前5位再与0x01作或运算之后作为索引获取bit
![](upload/attach/202105/817966_X7YDSHQNXXTGF5S.jpg)
4. 该bit值为1，则Destination Address合法

注：

`shr eax, 3` 即Destination Address/8，Bitmap中每1bit对应进程空间8 Bytes。`bt`指令功能如下：

![](upload/attach/202105/817966_YVYADE3AHBRUYZ9.jpg)

由于`SRC`寄存器是32位，故`POSITION`值需模32(0x20—0010 0000)，如此一来，便做到取Destination Address最低字节前5位为索引。若Destination Address合法，则直接`retn`，非法会交由`ntdll!RtlpHandleInvalidUserCallTarget`，最终触发`int 29`中断。

在执行`mov edx,dword ptr [edx+eax*4]`时，若EAX寄存器已被篡改，则可能会触发内存访问异常，而`ntdll!LdrpValidateUserCallTarget`中并未加入异常处理，其异常处理位于`ntdll!RtlDispatchException`中：

![](upload/attach/202105/817966_HS88UMEVAHWGGW9.jpg)

![](upload/attach/202105/817966_X5UZSMM7UWXU83A.jpg)

![](upload/attach/202105/817966_2UJ4DYHTXJTW678.jpg)

最后交由`ntdll!RtlpHandleInvalidUserCallTarget`来处理：

![](upload/attach/202105/817966_CWCCNZDPZ85JUTF.jpg)

## 0x02 How CFG Works:Kernel Mode Part

`nt!MiInitializeCfg`首先调用`nt!PsIsSystemWideMitigationOptionSet`校验注册表`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel\MitigationOptions`键值：

![](upload/attach/202105/817966_C727WJPC3283339.jpg)

`nt!PsIsSystemWideMitigationOptionSet`函数返回零，则调用`MmCreateSection`：

![](upload/attach/202105/817966_ZBEXRNSN8MTJXHM.jpg)

`Section Object Address`保存于`nt!MiState+0x604`处，InputMaximumSize为`0x3000000`。

`nt!MiCfgInitializeProcess`会调用`nt!MiReferenceCfgVad`向`0xC0802174`处写入`_MI_CFG_BITMAP_INFO`：

![](upload/attach/202105/817966_Q2EDGCRZVBB72G8.jpg)

![](upload/attach/202105/817966_SW3XUG44FRHSNTP.jpg)

## 0x03 参阅链接

- <a href="elink@46dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3L8%4N6W2M7X3!0X3j5$3!0E0L8i4g2F1K9i4c8&6i4K6u0W2L8X3g2@1i4K6u0r3M7r3!0U0x3U0l9I4y4q4)9J5c8X3#2B7x3o6l9I4x3g2)9J5k6i4m8V1k6R3`.`.">Windows 10 Control Flow Guard Internals—MJ0011</a>
- <a href="elink@b8aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4B7j5K6q4Q4x3X3c8@1k6g2)9J5k6r3k6@1M7q4)9J5k6i4c8J5k6h3&6V1L8h3W2U0M7X3!0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7%4y4W2N6s2y4Q4x3V1k6%4M7q4)9J5c8X3g2^5M7r3I4G2M7X3W2F1k6#2)9J5k6r3y4G2L8Y4c8J5L8$3I4Q4x3X3c8X3L8r3!0%4i4K6u0V1k6%4g2S2M7X3c8Q4x3X3c8A6L8W2)9J5k6s2N6A6L8X3c8G2N6%4x3I4x3q4)9J5k6i4m8V1k6R3`.`.">Exploring Control Flow Guard in Windows 10—TrendMicro</a>
- <a href="elink@17bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6o6L8$3&6@1M7X3!0D9i4K6u0V1k6X3I4G2N6#2)9#2k6X3W2F1N6r3g2Y4M7X3W2@1P5g2)9J5x3@1#2A6j5%4u0G2M7$3!0X3N6q4)9#2k6V1y4G2L8Y4c8J5L8$3I4Q4y4h3k6r3L8r3!0%4i4K6g2X3c8%4g2S2M7X3b7`.">Control-flow integrity—Wikipedia</a>
- <a href="elink@e8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6n7K9i4c8Q4y4h3k6f1k6i4y4@1">Bit Test—Wikipedia</a>
- <a href="elink@87bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6C8k6i4u0F1k6h3I4K6N6s2u0#2j5%4c8Q4x3X3g2Y4K9i4c8W2k6g2)9J5k6h3W2G2i4K6u0r3K9$3g2J5L8X3g2D9M7#2)9J5c8Y4R3$3y4q4)9J5c8W2N6A6L8X3c8G2N6%4x3I4x3q4)9J5k6i4m8V1k6R3`.`.">_MI_CFG_BITMAP_INFO</a>
- <a href="elink@989K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3c8A6j5h3#2G2L8Y4c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0b7J5y4e0V1#2z5e0l9`.">深入探索Win32结构化异常处理</a>
- <a href="elink@8afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4c8J5L8%4g2T1L8r3g2K6K9r3!0G2N6q4)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8U0L8r3W2W2L8Y4c8Q4x3V1k6K6K9r3g2D9L8q4)9J5k6r3g2^5M7r3g2J5K9h3g2F1j5$3g2Q4x3V1k6X3k6h3q4@1N6i4u0W2i4K6u0V1N6r3!0Q4x3X3c8T1L8r3!0U0K9#2)9J5k6s2g2F1N6s2u0#2M7%4c8W2k6q4)9J5k6r3k6G2L8Y4c8K6i4K6t1K6N6i4y4W2i4K6u0V1M7X3g2Y4K9i4y4@1M7Y4W2Q4x3X3c8W2k6r3W2@1L8%4t1`.">Windows 10 Technical Preview adds a feature that blocks untrusted fonts—Microsoft Docs</a>
- <a href="elink@7d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2%4j5i4u0J5K9h3!0J5i4K6u0W2j5$3&6Q4x3V1k6F1N6r3c8G2j5#2)9J5c8Y4N6J5K9#2)9J5c8X3#2E0i4K6u0r3e0h3#2o6M7X3g2S2N6r3g2e0k6h3y4@1K9h3!0F1i4K6u0W2K9s2c8E0">MmCreateSection</a>
