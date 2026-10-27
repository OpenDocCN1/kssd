
```c++
struct PacketInfinityHookRangeCheck {
// 23 is infinity hook range check
BYTE PacketType;
// address of the function
PVOID Address;
// 64 bytes content of the function
BYTE Content[64];
};
```

## 系统模块检测

遍历内核中加载的所有模块，对其进行检测，但是会跳过以下几个模块。

- `hal.dll`
- `clipsp.sys`
- `CI.dll`
- `tpm.sys`
- `ks.sys`
- `cdd.dll`
- `TSDDD.dll`
- `spsys.sys`
- `atikmpag.sys`

在处理`win32k`模块时，由于`win32k`模块的内存只在`csrss`中进行了映射，因此需要附加到`csrss`后再进行检查。

### 模块异常指令检测（report type 11）

由于该检测模块较为混乱，因此逆向分析的不是很清楚，怀疑是在寻找模块中一些`int 3`、hook的指令，并将指令所在的页面上传到异常链表。

其中对`dxgkrnl.sys`有特殊检测，怀疑是在检测gdi hook，原文链接：<a href="elink@4ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4J5k6i4c8Q4x3X3g2U0L8s2g2T1i4K6u0r3x3U0l9I4z5g2)9J5c8U0p5H3i4K6u0r3x3e0S2Q4x3V1k6C8k6i4u0F1k6h3I4Q4y4h3k6Y4k6r3W2Q4y4h3k6Z5L8$3!0C8i4K6u0W2K9s2c8E0L8l9`.`."><mark class="encrypted">892K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4J5k6i4c8Q4x3X3g2U0L8s2g2T1i4K6u0r3x3U0l9I4z5g2)9J5c8U0p5H3i4K6u0r3x3e0S2Q4x3V1k6C8k6i4u0F1k6h3I4Q4y4h3k6Y4k6r3W2Q4y4h3k6Z5L8$3!0C8i4K6u0W2K9s2c8E0L8l9`.`.</mark></a>

数据包结构：

```c++
struct PacketModuleAbnormalInstructionCheck {
// 11 is module abnormal instruction check
BYTE PacketType;
// length of the module name, up to 64
BYTE ModuleNameLength;
// length depends on ModuleNameLength
CHAR ModuleName[0];
// offset in page
ULONG OffsetInPage;
// content of the page which contains the abnormal instruction, up to 0x1000 bytes
BYTE Content[0];
};
```

### 模块IAT hook检测（report type 19）

通过解析各个模块的内存中的PE结构，检查是否存在某个IAT项的函数地址不在任何一个模块范围内，如果是则会上报异常数据。

数据包结构：

```c++
struct PacketModuleIATHookCheck {
// 19 is module IAT hook check
BYTE PacketType;
// module name
// no length is recorded yet ！
CHAR ModuleName[0];
// function index in the IAT
ULONG FunctionIndex;
// offset of the function IAT entry to the module base
ULONG EntryOffset;
// function in the IAT entry
PVOID Function;
// content of the function
BYTE Content[64];
};
```

## 隐藏驱动检测（report type 15）

通过遍历`\\Device`目录，得到所有`Device`类型的对象，然后遍历`\\Driver`和`\\FileSystem`目录，得到所有`Driver`对象，对每个`Device`对象找到其内部存储的`Driver`指针，然后逐一匹配刚才遍历得到的`Driver`对象，如果没有任何一个`Driver`对象与其匹配，则判定该`Device`对应的驱动被隐藏了，会上报异常数据。

数据包结构：

```c++
struct PacketHiddenDriverCheck {
// 15 is hidden driver check
BYTE PacketType;
// length of the device name
BYTE DeviceNameLength;
// name of the device whose driver is hidden
CHAR DeviceName[0];
// driver name of the hidden driver
// length = PacketLength - OtherFieldsLength
CHAR DriverName[0];
};
```

## PCI设备检测（report type 26）

通过I/O指令遍历PCI设备树，寻找具有指定特征的PCI设备，怀疑是检测DMA作弊工具。如果找到具有指定特征的PCI设备，则会上报异常数据。

PCI设备检测实现参考源码：<a href="elink@486K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8D9j5h3u0Q4x3X3g2X3M7X3g2W2k6r3g2K6K9%4c8G2M7q4)9J5k6h3!0J5k6#2)9J5c8Y4S2G2M7X3N6Q4x3V1k6D9K9h3u0Q4x3V1k6D9K9h3u0H3j5$3W2S2j5$3y4W2M7%4y4Q4x3V1k6Q4x3X3c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3P5o6R3$3i4K6g2X3M7r3y4A6i4K6u0W2j5H3`.`."><mark class="encrypted">fe4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8D9j5h3u0Q4x3X3g2X3M7X3g2W2k6r3g2K6K9%4c8G2M7q4)9J5k6h3!0J5k6#2)9J5c8Y4S2G2M7X3N6Q4x3V1k6D9K9h3u0Q4x3V1k6D9K9h3u0H3j5$3W2S2j5$3y4W2M7%4y4Q4x3V1k6Q4x3X3c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3P5o6R3$3i4K6g2X3M7r3y4A6i4K6u0W2j5H3`.`.</mark></a>

UC上也有人提到过该检测：<a href="elink@1eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0x3H3y4o6f1@1y4g2)9J5k6r3c8W2N6r3g2U0N6r3W2F1k6#2)9J5k6r3c8E0j5g2)9J5k6r3S2S2M7X3c8%4j5i4u0W2i4K6u0V1j5$3S2W2j5i4c8K6i4K6u0V1x3e0u0Q4x3X3g2Z5N6r3#2D9"><mark class="encrypted">006K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0x3H3y4o6f1@1y4g2)9J5k6r3c8W2N6r3g2U0N6r3W2F1k6#2)9J5k6r3c8E0j5g2)9J5k6r3S2S2M7X3c8%4j5i4u0W2i4K6u0V1j5$3S2W2j5i4c8K6i4K6u0V1x3e0u0Q4x3X3g2Z5N6r3#2D9</mark></a>

（注意：在第二种上报类型中，`Info`中的`Dev`貌似被BE的开发者误写成了`Bus`，导致记录了两次`Bus`而没有记录`Dev`，笑）

数据包结构：

```c++
struct PacketHiddenDriverCheckType1 {
// 26 is pci device check
BYTE PacketType;
// PCI enumeration info
struct {
BYTE Bus;
BYTE Dev;
BYTE Func;
} Info;
// 4 bytes read from reg VENDOR\_ID (0x0)
ULONG VendorId;
// 4 bytes read from reg PCI\_CLASS (0x08)
ULONG PciClass;
// 1 byte read from reg HDRTYPE (0x0E)
BYTE HdrType;
// 4 bytes read from reg PCI\_SUB\_VENDOR\_ID (0x2C)
ULONG SubVendorId;
};
```

```c++
struct PacketHiddenDriverCheckType2 {
// 26 is pci device check
BYTE PacketType;
// PCI enumeration info
struct {
BYTE Bus;
BYTE Dev;
BYTE Func;
} Info;
// 256 bytes read from reg VENDOR\_ID (0x0)
BYTE VendorId[256];
};
```

## Win32k函数指针表检测

`gDxgkInterface`和`gDxgkWin32kEngInterface`是存储在`Win32k`中的两张函数表，作用类似于SSDT，<a href="elink@ba5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3#2W2L8h3u0W2M7Y4y4Q4x3V1j5J5y4K6R3@1y4K6N6Q4x3X3g2Z5N6r3#2D9">**IChooseYou**</a>曾将其用于无模块驱动的通信，<a href="elink@62fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0x3K6y4e0f1^5y4g2)9J5k6r3y4G2L8h3#2#2L8X3W2U0j5i4c8A6L8X3N6Q4x3X3c8E0j5i4m8H3k6h3c8Q4x3X3c8V1M7X3W2$3k6i4u0Q4x3X3c8#2M7$3W2F1k6#2)9J5k6h3S2@1L8h3H3`."><mark class="encrypted">bc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0x3K6y4e0f1^5y4g2)9J5k6r3y4G2L8h3#2#2L8X3W2U0j5i4c8A6L8X3N6Q4x3X3c8E0j5i4m8H3k6h3c8Q4x3X3c8V1M7X3W2$3k6i4u0Q4x3X3c8#2M7$3W2F1k6#2)9J5k6h3S2@1L8h3H3`.</mark></a>，故BE对其进行检测。

由于`win32k`仅在`csrss`模块的地址空间中进行了映射，因此在检测时需要附加到`csrss`进程。

### gDxgkInterface 地址范围检测（report type 20）

对`gDxgkInterface`表中的绝大部分函数进行地址范围检测（跳过前两个函数），检测其地址是否在`win32k`模块范围内。

数据包结构：

```c++
struct PacketWin32kRangeCheckType1 {
// 20 is win32k gDxgkInterface range check
BYTE PacketType;
// function index in the gDxgkInterface table
ULONG Index;
// function address
PVOID Function;
// 64 bytes of the function
BYTE Content[64];
};
```

### gDxgkInterface hook检测（report type 21）

对上述函数进行hook检测，检测方式同回调Hook检测（report type 6），仅report type不同。`FunctionType`值为函数在表中的下标。

### gDxgkWin32kEngInterface 地址范围检测（report type 24）

对`gDxgkWin32kEngInterface`表中的所有函数进行地址范围检测，检测其地址是否在`win32k`模块范围内。

数据包结构：

```c++
struct PacketWin32kRangeCheckType2 {
// 20 is win32k gDxgkWin32kEngInterface range check
BYTE PacketType;
// function index in the gDxgkWin32kEngInterface table
ULONG Index;
// function address
PVOID Function;
// 64 bytes of the function
BYTE Content[64];
};
```

### gDxgkWin32kEngInterface hook检测（report type 25）

对上述函数进行hook检测，检测方式同回调Hook检测（report type 6），仅report type不同。`FunctionType`值为函数在表中的下标。

### ext\_ms\_win\_core\_win32k\_full\_export\_l1 地址范围检测（report type 33）

该表未导出，因此BE通过特征码定位的方式获得该表，通过`BRUSHOBJ_hGetColorTransform`函数进行定位，在该函数中搜索如下特征码，`addr1`即为`ext_ms_win_core_win32k_full_export_l1`：

```asm
mov rax, [addr1]
test rax, rax
je addr2
call qword ptr [addr3]
```

对该表中的函数逐个检测地址，查看其是否在`win32k`和`win32kfull`模块的范围内，如果不在则会上报异常数据。

数据包结构：

```c++
struct PacketWin32kRangeCheckType3 {
// 33 is win32k ext\_ms\_win\_core\_win32k\_full\_export\_l1 range check
BYTE PacketType;
// function index in the ext\_ms\_win\_core\_win32k\_full\_export\_l1 table
ULONG Index;
// function address
PVOID Function;
// 64 bytes of the function
BYTE Content[64];
};
```

## Dxgkrnl 函数指针表检测

`DxgCoreInterface`是`Dxgkrnl`模块中的一张函数表。可能曾被用作无模块通信/绘制，或者仅是预防性检查。

### DxgCoreInterface 地址范围检测（report type 12）

对`DxgCoreInterface`表中的所有函数进行地址范围检测，检测其地址是否在`win32k`模块范围内。

数据包结构：

```c++
struct PacketDxgkrnlRangeCheck {
// 12 is Dxgkrnl DxgCoreInterface range check
BYTE PacketType;
// function index in the DxgCoreInterface table
ULONG Index;
// function address
PVOID Function;
// 64 bytes of the function
BYTE Content[64];
};
```

### DxgCoreInterface hook检测（report type 13）
